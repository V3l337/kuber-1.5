# Задание 1. Создать Deployment приложений backend и frontend

## Создал deployment и service nginx 

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-nginx-deployment
  labels:
    web: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      web: frontend
  template:
    metadata:
      labels:
        web: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    web: frontend
  ports:
  - name: nginx-port
    protocol: TCP
    port: 9001          # Порт, на котором сервис принимает запросы
    targetPort: 80    # Порт, на который пересылает внутри пода (nginx)
```

## Создал deployment и service multitool

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-multitool-deployment
  labels:
    app: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        command: ["/bin/sh", "-c", "httpd -p 8080 -h / & sleep infinity"]

---
apiVersion: v1
kind: Service
metadata:
  name: multitool-service
spec:
  selector:
    app: backend 
  ports:
  - name: multitool-port
    protocol: TCP
    port: 9002       # Порт сервиса для multitool
    targetPort: 8080  # Внутренний порт multitool
  type: ClusterIP
```

## Проверка, что приложения видят друг друга с помощью Service.

![Продемонстрировать, что приложения видят друг друга с помощью Service](https://github.com/user-attachments/assets/bcd881b9-2b41-4a4f-9962-12db9b1bf91f)


# Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

## Включил Ingress-controller в MicroK8S.

```bash
 microk8s enable ingress
```

## Создал Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался frontend а при добавлении /api - backend.

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  # ingressClassName: test-example
  rules:
  - host: v3l.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 9001
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: multitool-service
            port:
              number: 9002
```

## Продемонстрировать доступ с помощью браузера или curl с локального компьютера.


![Ingress и обеспечить доступ к приложениям снаружи кластера](https://github.com/user-attachments/assets/7b3121af-7ef4-4b28-9643-0d87425a8cb1)
