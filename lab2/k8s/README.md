# Часть 1. Локальный кластер

Чтобы начать работу, я установила `minikube` и `kuberctl`.
[image](/lab2/k8s/images/install.png)

Дальше для уверенности проверила, что все работает.
Использовала команду
``` bash
minikube status
```
[image](/lab2/k8s/images/status.png)

Потом - переходим к моему тестовому приложению. Сначала я хотела запихнуть все в один файл и похвастаться, что знаю как разделять манифесты внутри одного текстового файла. Но в последствии путем ресерча выяснила, что в будущем лучше манифесты разносить по разным файлам для лучшей масштабируемости и для работы с helm в будущем. 

Так у меня получились configmap.yaml
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  index.html: |
    <html>
      <head><title>Lab2 - k8s lab</title></head>
      <body>
        <h1>lab2</h1>
      </body>
    </html>
```
service.yml
```yml
apiVersion: v1
kind: Service
metadata:
  name: hello-service
spec:
  type: NodePort
  selector:
    app: hello
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
```
И deployment.yml
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: hello-container
          image: nginx:latest
          ports:
            - containerPort: 80
          volumeMounts:
            - name: html-volume
              mountPath: /usr/share/nginx/html/index.html
              subPath: index.html
      volumes:
        - name: html-volume
          configMap:
            name: nginx-config
```

``deployment`` - используется для управления приложением в кластере

``service`` - используется для организации доступа к приложению

``configmap`` - используется для хранения и передачи конфигурационных данных

Далее я задеплоила с помощью команды: `kuberctl apply -f k8s/`
И проверила, что они вообще запускаются 
[image](/lab2/k8s/images/pods.png)

Прописав команду `minikube service hello-service --url` окаазалась в браузере. nginx работает
[image](/lab2/k8s/images/final.png)
