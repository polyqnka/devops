# Часть 2. Docker compose (docker-compose, compose etc)

Честно, я уже делала эту работу у вас на курсе, но сейчас я разрабатываю свой пет-проект, для которого нужен компоуз как никогда, поэтому решила сделать эту часть максимально прикладной и практической :)))). (а еще мне показалось слишком сложным для моего уровня писать свой докер :))

Мой проект - свой aternos для Майнкрафта, со всем бэкендом, нужными сервисами и тому подобным на собственном сервере с полной свободой и безопасностью. 

### Вот как выглядел мой изначальный compose, который я старалась писать по доке докера 

``` yml
version: "3.8"

services:
  minecraft:
    image: itzg/minecraft-server:java17
    container_name: mc-server
    ports:
      - 25565:25565 
    environment:
      EULA: "TRUE"
      MEMORY: "2G"
      TYPE: "PAPER"
      ENABLE_RCON: "true"
      RCON_PASSWORD: "Jotnom95"
    volumes: 
      - ./server:/data
      - ./logs:/logs
      - ./backups:/backups
    deploy: 
      resources:
        limits:
          cpus: "1.5"
          memory: "4G"
    networks:
      - mcnet

backend: 
  build: ./backend
  container_name: mc-backend
  restart: unless-stopped
  ports: 
    - "8000:8000"
  environment:
    MC_HOST: "mc-server"
    MC_PORT: 25565
  networks:
    - mcnet

networks: 
  mcnet:
    driver: bridge
```

В целом неплохо как мне показалось, я старалась соблюдать базовые best practices, такие как:
- указывать четкую версию. никаких latest (у нас в лабе была страшная история с ними связанная)
- использование volumes, они же тома 
- разделение на отдельные сервисы (у меня это minecraft и backend)
- использование сети (у меня mcnet)
- ограничение ресурсов и ввод переменных окружения вместо хардкода

Да не самый идеальный, но я же старалась). А теперь к минусам, которые мне, моему парню и чату гпт удалось найти:

- как оказалось deploy в обычном компоузе не заработает. Пришлось переписать эту часть
- кошмар, который почему-то я не заметила -- пароль в открытом доступе. Я знала, что это очень плохо и пароли, ключи и тд надо прятать в секреты, но видимо компоуз писался в ночи и я просто забила). спрятала пароль в env файлик (я поменяла его на фейк пароль, чтобы использовать в учебных целях, все безопасно)
- конкретно по моему компоузу можно убрать папки для логов и бэкапов, тк образ minecraftserver использует папку data
- гпт посоветовал мне добавить restart, это важно для серверов, тк без этого компоуз просто не будет перезапускаться из-за падения
- добавить зависимости в сервисы, так как например backend зависит от minecraft
- добавить healthcheck. вообще это странная тема, работающие люди мне сказали, что это старомодно и редко встречается, советует гпт и на форумах (последний вопрос о хелсчек почти 5 лет назад [image](./images/Снимок%20экрана%202026-03-24%20в%2020.33.04.png))
- можно убрать рудимент в виде `version: "3.8"`
- еще я добавила файлик с requirements и для этого написала микро докерфайл

# Итоговый compose, который я постаралась улучшить

```yml
services:
  minecraft:
    image: itzg/minecraft-server:java17
    container_name: mc-server
    restart: unless-stopped

    ports:
      - "25565:25565"

    environment:
      EULA: "TRUE"
      MEMORY: "2G"
      TYPE: "PAPER"
      ENABLE_RCON: "true"
      RCON_PASSWORD: ${RCON_PASSWORD}

    volumes:
      - ./server:/data

    mem_limit: 4g
    cpus: "1.5"

    networks:
      - mcnet

    healthcheck:
      test: ["CMD", "mc-health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  backend:
    build: ./backend
    container_name: mc-backend
    restart: unless-stopped

    ports:
      - "8000:8000"

    environment:
      MC_HOST: mc-server   
      MC_PORT: 25565

    networks:
      - mcnet

    depends_on:
      minecraft:
        condition: service_healthy

    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/"]
      interval: 30s
      timeout: 10s
      retries: 3

networks:
  mcnet:
    driver: bridge
```

В задании отдельно написано сделать так, чтобы сервисы (если их нет - создать) не видели друг друга в сети, но запускались.

### Почему это важно?

В основном, как я поняла, это нужно для безопасности и тестирования. Майн и бэк, в моем случае, слушают разные порты, оба работают, но друг другу не мешают. До этого они находились оба в mcnet - одной сети, видели друг друга по хостнеймам. В будущем такое разделение позволит мне масштабировать сервисы, или если бэк упадет, я смогу запустить свой сервер, ведь контейнер с майном еще жив :)

Как этого добились? На самом деле очень просто! Добавили новые сети
```yml
networks:
  mcnet:
    driver: bridge
  backendnet:
    driver: bridge
```
И подключили контейнеры к своим сетям

```yml
services:
  minecraft:
    networks:
      - mcnet

  backend:
    networks:
      - backendnet
```
Вот и все. Еще пара лаб, и я добью этот долгострой :)
[meme](./images/_.jpeg)


upd: скрины запуска хорошего компоуза
