# Заметки

## Задание

Само приложение: https://github.com/adan1lov/muffin-wallet

1) Собрать и запаблишить Docker image
2) Создать docker compose, в котором запуститься Docker image
3) Подготовить необходимые манифесты: Pod, ReplicaSet, Service, ConfigMap, Secret, Deployment, Ingress(tls)

Формат защиты:

1. Демонстрация пересборки Docker-образа с внесёнными изменениями.
2. Запуск и управление развёртыванием в Kubernetes:
    - Роллаут обновления (kubectl set image или изменение манифеста).
    - Проверка статуса развёртывания (kubectl rollout status).
    - Масштабирование (kubectl scale) и откат (kubectl rollout undo).
3. Выполнение дополнительных требований преподавателя (например, изменение конфигурации "на лету" или добавление новых
   сервисов).

Максимальная оценка 8, сдача на 10 будет возможна при успешной сдачи на 8 и будет выполняться во время сдачи ДЗ

## Шаги выполнения задания:

### 1. Собрать и запаблишить Docker image

- Сделал Dockerfile
  ```dockerfile
    FROM amazoncorretto:23-alpine3.19

    WORKDIR /app
    COPY muffin-wallet-server/build/libs/muffin-wallet-server-1.0.0.jar app.jar

    CMD ["java", "-jar", "app.jar"]
  ```
- Собрал проект локально
  ```bash
  docker build -t whomustnotbenamed/muffin-wallet-server:latest .
  ```
- Запушил в DockerHub
  ```bash
  docker push whomustnotbenamed/muffin-wallet-server:latest
  ```

### 2. Создать docker compose, в котором запуститься Docker image
- Дополнил имеющийся docker-compose.yml
  ```yaml 
    muffin-wallet-server:
        image: whomustnotbenamed/muffin-wallet-server:latest
        ports:
            - 8080:8080
        environment:
            - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/muffin_wallet
            - SPRING_DATASOURCE_USERNAME=muffin-wallet
            - SPRING_DATASOURCE_PASSWORD=muffin-wallet
        depends_on:
            - postgres
  ```
- Запустил docker-compose перейдя в директорию с файлом и выполнив команду:
  ```bash
    docker compose up -d
  ```
### 3. Подготовить необходимые манифесты: Pod, ReplicaSet, Service, ConfigMap, Secret, Deployment, Ingress(tls)
  - Подготовил манифест для Pod
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: muffin-wallet-pod
      labels:
        app: muffin-wallet
    spec:
      containers:
      - image: whomustnotbenamed/muffin-wallet-server:latest
        name: muffin-wallet
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_DATASOURCE_URL
          value: jdbc:postgresql://host.minikube.internal:5432/muffin_wallet
        - name: SPRING_DATASOURCE_USERNAME
          value: muffin-wallet
        - name: SPRING_DATASOURCE_PASSWORD
          value: muffin-wallet
      ```
    Чтобы был достук к базе данных, которая запущена на хостовой машине, использовал `host.minikube.internal`
    (https://minikube.sigs.k8s.io/docs/handbook/host-access/)

    Запустил Pod командой:
    ```bash
    kubectl apply -f pod.yml
    ```
    
- Подготовил манифест для ReplicaSet
    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: muffin-wallet
      labels:
        app: muffin-wallet
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: muffin-wallet
      template:
        metadata:
          labels:
            app: muffin-wallet
        spec:
          containers:
          - image: whomustnotbenamed/muffin-wallet-server:latest
            name: muffin-wallet
            ports:
            - containerPort: 8080
            env:
              - name: SPRING_DATASOURCE_URL
                value: jdbc:postgresql://host.minikube.internal:5432/muffin_wallet
              - name: SPRING_DATASOURCE_USERNAME
                value: muffin-wallet
              - name: SPRING_DATASOURCE_PASSWORD
                value: muffin-wallet
  ```
- Подготовил манифест для Service
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: muffin-wallet
      labels:
        app: muffin-wallet
    spec:
      selector:
        app: muffin-wallet
      ports:
      - port: 80
        targetPort: 8080
    ```
    Чтобы получить доступ из браузера, надо пробросить порты использую команду:
    ```bash
    kubectl port-forward service/muffin-wallet 8080:80
    ```
    или можно пользоваться `minikube service` (https://minikube.sigs.k8s.io/docs/handbook/accessing/):
    ```bash
    minikube service muffin-wallet
    ```
- Подготовил манифест для ConfigMap
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: muffin-wallet-config
      labels:
        app: muffin-wallet
    data:
      SPRING_DATASOURCE_URL: jdbc:postgresql://host.minikube.internal:5432/muffin_wallet
      SPRING_DATASOURCE_USERNAME: muffin-wallet
      SPRING_DATASOURCE_PASSWORD: muffin-wallet
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: muffin-wallet-pod
      labels:
        app: muffin-wallet
    spec:
      containers:
      - image: whomustnotbenamed/muffin-wallet-server:latest
        name: muffin-wallet
        ports:
          - containerPort: 8080
        envFrom:
          - configMapRef:
            name: muffin-wallet-config
    ```
  Можно посмотреть на логи пода и проверить конфиг командой
    ```bash
    kubectl get configmap
    ```
- Подготовил манифест для Secret
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: muffin-wallet-secret
    stringData:
      SPRING_DATASOURCE_URL: jdbc:postgresql://host.minikube.internal:5432/muffin_wallet
      SPRING_DATASOURCE_USERNAME: muffin-wallet
      SPRING_DATASOURCE_PASSWORD: muffin-wallet
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: muffin-wallet-pod-with-secret
      labels:
        app: muffin-wallet
    spec:
      containers:
        - image: whomustnotbenamed/muffin-wallet-server:latest
          name: muffin-wallet
          ports:
            - containerPort: 8080
          envFrom:
            - secretRef:
              name: muffin-wallet-secret
  ```
    Можно посмотреть на логи пода и проверить секрет командами
    ```bash
    kubectl get secret
    ```
    ```bash
    kubectl get secret muffin-wallet-secret -o yaml
    ```
- Подготовил манифест для Deployment
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: muffin-wallet
      labels:
        app: muffin-wallet
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: muffin-wallet
      template:
        metadata:
          name: muffin-wallet
          labels:
            app: muffin-wallet
        spec:
          containers:
            - image: whomustnotbenamed/muffin-wallet-server:latest
              name: muffin-wallet
              ports:
                - containerPort: 8080
              envFrom:
                - configMapRef:
                  name: muffin-wallet-config
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: muffin-wallet
      labels:
        app: muffin-wallet
    spec:
      selector:
        app: muffin-wallet
      ports:
        - port: 80
          targetPort: 8080
  ```
  Можем посмотреть логи пода и статус деплоймента командой:
    ```bash
    kubectl rollout status deployment/muffin-wallet
    ```
  Также можем изменить количество реплик командой:
    ```bash
    kubectl scale deployment/muffin-wallet --replicas=2
    ```
- Подготовил манифест для Ingress(tls)
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: muffin-wallet-ingress
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      tls:
      - hosts:
        - muffin-wallet.local
        secretName: muffin-wallet-tls
      rules:
      - host: muffin-wallet.local
        http:
          paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: muffin-wallet
                  port:
                    number: 80
    ```
  Для работы необходимо включить аддон ingress в minikube (https://minikube.sigs.k8s.io/docs/handbook/addons/ingress-dns/#Windows):
    ```bash
    minikube addons enable ingress
    minikube addons enable ingress-dns
    ```
  Также необходимо добавить в `C:\Windows\System32\drivers\etc\hosts` строку:
    ```
    127.0.0.1 muffin-wallet-ingress.ru
    ```
  И чтобы был доступ к minikube из браузера, надо пробросить туннель командой:
    ```bash
    minikube tunnel
    ```