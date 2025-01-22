# 03-kuber
# 03-kuber
# Задача 1
При запуске деплоя возникает ошибка заняты портов для исправления ошибки добавлена часть:
![Screenshot 2025-01-22 132834](https://github.com/user-attachments/assets/0d0a55b9-b76b-4cf9-aaa3-6642e6ecbb9d)

![Screenshot 2025-01-22 134448](https://github.com/user-attachments/assets/c3827ad5-2a7e-45e1-8a75-80ee06329264)

```
        env:
        - name: HTTP_PORT
          value: "8080"
```

Манифесты:

deployment:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: multi-container-app
  labels:
    app: multi-container-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: multi-container-app
  template:
    metadata:
      labels:
        app: multi-container-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
      - name: multitool
        image: praqma/network-multitool
        ports:
        - containerPort: 8080
        env:
        - name: HTTP_PORT
          value: "8080"
```

service:
![Screenshot 2025-01-22 135059](https://github.com/user-attachments/assets/e0296dd9-f608-45a0-9b07-cb4531efb303)

```
apiVersion: v1
kind: Service
metadata:
  name: multi-container-service
spec:
  selector:
    app: multi-container-app
  ports:
    - name: nginx-port
      protocol: TCP
      port: 80
      targetPort: 80
    - name: multitool-port
      protocol: TCP
      port: 8080
      targetPort: 8080
```


pod:
```
apiVersion: v1
kind: Pod
metadata:
  name: multitool-pod
spec:
  containers:
  - name: multitool
    image: wbitt/network-multitool
    command: ["tail", "-f", "/dev/null"]
```

Для увелечения количества реплик используется команда:

![Screenshot 2025-01-22 134559](https://github.com/user-attachments/assets/97e2ba40-0f11-4cd1-8531-f16f8683a949)

```
kubectl scale deployment multi-container-app --replicas=2
deployment.apps/multi-container-app scaled
```

![Screenshot 2025-01-22 134719](https://github.com/user-attachments/assets/acae4daa-39db-4e1b-afbc-a353deedbce1)

Для проверки доступности создан pod и в pod's через shell выполнен запрос curl:

![Screenshot 2025-01-22 135301](https://github.com/user-attachments/assets/7bd72cb4-175a-4b4b-9587-4ce0c28695b9)


# Часть 2

Созданы deployment и service манифесты:

deployment:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        command: ['sh', '-c', 'echo The app is running! && sleep 3600']
        ports:
          - containerPort: 80
      initContainers:
      - name: init-myservice
        image: busybox:1.28
        command: ['sh', '-c', "until nslookup nginx-service.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for nginx-service; sleep 2; done"]
```

service:
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Из скриншотов видно что при запуске deployment контейнер ожидает запуска:
![Screenshot 2025-01-22 141608](https://github.com/user-attachments/assets/39b6a90c-b401-4d86-8e1e-b0b02ccbb3d8)

После запуска сервиса контейнер запускается:
![Screenshot 2025-01-22 141729](https://github.com/user-attachments/assets/077f78ae-ba5f-4123-8791-ed2a0a0772bd)

