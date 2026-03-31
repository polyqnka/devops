# Часть 1. Dockerfile 

Эту часть я писала последней, докерфайлы я уже писала, поэтому решила просто сделать утрированно плохой докерфайл, который потом лучше бы сжечь. 

### Вот он 
```yml
FROM ubuntu:latest

RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get install -y python3 python3-pip nodejs npm mysql-client redis nginx curl wget vim git gcc make

RUN apt-get update

COPY . /app

WORKDIR /

RUN pip install -r /app/requirements.txt

RUN pip install flask fastapi django

USER root

EXPOSE 80 443 3000 5000 8000 8080 27017

RUN wget http://example.com/some-script.sh && chmod +x some-script.sh

CMD service nginx start && service redis-server start && python3 /app/app.py
```

Чем он плох?
- latest. На всю жизнь запомню, насколько это плохо после той истории из LiSA (лаба ФПИн)
- мало того, что latest, так еще и огромный убунту, который будет билдиться полгода (посмотрев далее на строчки файла, можно сказать что он вообще не сбилдится)))
- очень много команды `RUN`, лучше объеденить все и почистить.
- просто много рандомных установок. зачем? (чтобы сделать карикатурно плохой файл по приколу конечно :))
- нарушен принцип "один контейнер - один процесс"
- сбрасывается кэш
- копируется весь проект с мусором, секретами и тд и тд
- работа в руте это вообще что 
- дублирование зависимостей 
- слушаем вообще все порты
- снова нарушаем принцип одного процесса на контейнер в `CMD`, а потом в довесок скачиваем рандомный скрипт, чтобы убить безопасность до конца

Большой Dockerfile не значит хороший, и вот наш исправленный хорошенький Dockerfile

```yml
FROM python:3.12-slim AS base

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

RUN useradd -m appuser

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN chown -R appuser:appuser /app

USER appuser

EXPOSE 8000

CMD ["python", "app.py"]
```

