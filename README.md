# Евсюков Александр БПИ224
# Задания с курса "Промышленное развертывание серверных приложений"

Во всех заданиях разворачиваются два учебных микросервиса в Kubernetes (minikube):

- **[muffin-wallet](https://github.com/adan1lov/muffin-wallet)** — основное приложение (Java / Spring Boot + PostgreSQL);
- **[muffin-currency](https://github.com/adan1lov/muffin-currency)** — сервис курсов валют, к которому обращается muffin-wallet.

## Задания

| # | Папка | Тема | Что сделано |
|---|-------|------|-------------|
| 1 | [`hw-k8s-manifests`](./hw-k8s-manifests) | Docker и базовые объекты Kubernetes | Dockerfile и публикация образа в Docker Hub, docker compose, манифесты Pod, ReplicaSet, Deployment, Service, ConfigMap, Secret, Ingress (TLS); rollout, scale, rollback |
| 2 | [`hw-helm`](./hw-helm) | Helm | Манифест muffin-currency переписан в Helm-чарт; install / upgrade / rollback / history / uninstall |
| 3 | [`hw-helm-charts`](./hw-helm-charts) | Helm-чарты для двух сервисов | Отдельные чарты для muffin-wallet и muffin-currency (ConfigMap, Secret, Ingress), доступ через `minikube tunnel` |
| 4 | [`hw-istio`](./hw-istio) | Service mesh (Istio) | Развёртывание через helmfile, sidecar-инжекция, Ingress Gateway, mTLS, AuthorizationPolicy, Circuit Breaker / Retry / таймауты, VirtualService, ServiceEntry для внешней БД, Kiali и Prometheus |
| 5 | [`hw-prometheus`](./hw-prometheus) | Метрики | Prometheus и Grafana в docker compose, сбор метрик с приложения в кластере, PromQL-запросы (RPS, ошибки, p99 латентности, соединения с PostgreSQL), дашборд, нагрузочный тест JMeter |
| 6 | [`hw-logs-and-traces`](./hw-logs-and-traces) | Логи и трейсы | Loki + Promtail (sidecar) для логов, Zipkin для трейсов, дашборд Grafana с поиском по traceId и уровню логирования |
| 7 | [`hw-open-telemetry`](./hw-open-telemetry) | OpenTelemetry | OpenTelemetry Operator, OpenTelemetryCollector в кластере, автоинструментирование Java, сбор метрик, логов и трейсов, дашборды |

Подробное описание задания, шаги выполнения, команды и скриншоты лежат в `README.md` внутри каждой папки.

## Стек

Docker, Docker Compose, Kubernetes (minikube), kubectl, Helm, helmfile, Istio, Kiali, Prometheus, Grafana, Loki, Promtail, Zipkin, OpenTelemetry Operator, PostgreSQL.
