# Решение домашнего задания к занятию «Конфигурация приложений»
https://github.com/netology-code/kuber-homeworks/blob/main/2.3/2.3.md

## Задание 1. Создать Deployment приложения и решить возникшую проблему с помощью ConfigMap. Добавить веб-страницу
### Манифест деплаймента
```
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


## Задание 2. Создать приложение с вашей веб-страницей, доступной по HTTPS

### Выпуск сертификата
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=microk8s.mylocalserver.com"
```
### Кодироварие сертифката и ключа в base64
```
cat tls.crt | base64
cat tls.key | base64
```
### Манифест деплоймента
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuber23d2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-ssl
  template:
    metadata:
      labels:
        app: nginx-ssl
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
      volumes:
        - name:  nginx-page
          configMap:
            name: config-nginx-ssl
```
### ConfigMap для веб страницы nginx
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-nginx-ssl
  labels:
    app: nginx-page
data:
  index.html: |
    <html>
        <body>
            <p>Hellow SSL!</p>
        </body>
    </html>
```
### Манифест с сертификатом и ключем
```
apiVersion: v1
kind: Secret
metadata:
  name: secret-tls
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURLekNDQWhPZ0F3SUJBZ0lVR0p3dDdnYjk4QXg4S1U3czJOT25zeEZrQ3JZd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0pURWpNQ0VHQTFVRUF3d2FiV2xqY205ck9ITXViWGxzYjJOaGJITmxjblpsY2k1amIyMHdIaGNOTWp
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2Z0lCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktnd2dnU2tBZ0VBQW9JQkFRREUrUWMvR3FkOFk2OEoKblpNdFZlZGRmQ1prUHlvVWk1VjhLMDlZcGljRHFncklyd1ROT1FiUHJZOEwzVE9uOEVoNjlhTEpyeXg4eEV
```
### Манифест сервиса
```
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx
spec:
  selector:
    app: nginx-ssl
  type: ClusterIP
  ports:
  - name: nginx
    port: 80
    targetPort: nginx-web
```
### Манифест ingress (включение nginx ingress было выполнено в одном из предыдущих  ДЗ)
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-ssl
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: "nginx"
  rules:
    - host: microk8s.mylocalserver.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: svc-nginx
                port:
                  name: nginx
  tls:
    - hosts:
        - microk8s.mylocalserver.com
      secretName: secret-tls
```
### Скриншот барузера
![image](https://github.com/user-attachments/assets/75d319bc-8857-497d-bc09-bbbf7ba61508)
### Скриншот сертификата
![image](https://github.com/user-attachments/assets/48e403bd-6bc2-4d84-873c-cc5466e36e45)


