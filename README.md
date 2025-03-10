# Решение домашнего задания к занятию «Как работает сеть в K8s»
https://github.com/netology-code/kuber-homeworks/blob/main/3.3/3.3.md
## Задание 1. Создать сетевую политику или несколько политик для обеспечения доступа frontend -> backend -> cache.
### Демлойменты
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: wbitt/network-multitool
        ports:
          - name: front-http
            containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: wbitt/network-multitool
        ports:
          - name: back-http
            containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cache
  namespace: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cache
  template:
    metadata:
      labels:
        app: cache
    spec:
      containers:
      - name: cache
        image: wbitt/network-multitool
        ports:
          - name: cache-http
            containerPort: 80
```
### Сервисы
```
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: app
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: front-http
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: app
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: back-http
---
apiVersion: v1
kind: Service
metadata:
  name: cache
  namespace: app
spec:
  selector:
    app: cache
  ports:
    - protocol: TCP
      port: 80
      targetPort: cache-http
```
### Сетевая политика frontend->backend
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-backend
  namespace: app
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
```

### Сетевая политика backend -> cache
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-cache
  namespace: app
spec:
  podSelector:
    matchLabels:
      app: cache
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: backend
```

### Запрет всех остальных маршрутов
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: app
spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

### Проверка соединений внутри кластера
![image](https://github.com/user-attachments/assets/4112e323-0138-4bb3-8d01-3a3b4325ac8e)

### Состояние кластера
![image](https://github.com/user-attachments/assets/2271e91d-8552-4d79-8601-3ec625913600)


      
