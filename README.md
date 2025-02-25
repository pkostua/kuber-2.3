# Решение домашнего задания к занятию «Конфигурация приложений»
https://github.com/netology-code/kuber-homeworks/blob/main/2.3/2.3.md

## Задание 1. Создать Deployment приложения и решить возникшую проблему с помощью ConfigMap. Добавить веб-страницу
### Манифест деплаймента
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuber23d1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-multitool
  template:
    metadata:
      labels:
        app: nginx-multitool
    spec:
      containers:
        - name: nginx
          image: nginx:1.24.0
          ports:
            - name: nginx-web
              containerPort: 80
          volumeMounts:
            - name: nginx-page
              mountPath: "/usr/share/nginx/html"
        - name: multitool
          image: praqma/network-multitool:alpine-extra
          env:
            - name: HTTP_PORT
              valueFrom:
                configMapKeyRef:
                  name: config-mt
                  key: http
            - name: HTTPS_PORT
              valueFrom:
                configMapKeyRef:
                  name: config-mt
                  key: https
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
            - name: https
              containerPort: 8443
              protocol: TCP
      volumes:
        - name:  nginx-page
          configMap:
            name: config-nginx
```
### Манифест ConfigMap для решения проблемы запуска
```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-mt
  labels:
    app: multitool-ports
data:
  http: "8080"
  https: "8443"
```
### Манифест ConfigMap для веб страницы nginx
```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-nginx
  labels:
    app: nginx-page
data:
  index.html: |
    Hellow World!!
```
### Скриншот, демонстрирующий состояние подов и curl
![image](https://github.com/user-attachments/assets/380c9dfd-0bb2-463d-b24f-2187a2c3ad8b)


