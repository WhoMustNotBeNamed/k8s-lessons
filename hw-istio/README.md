# Евсюков Александр БПИ224

## Задание
muffin-wallet и muffin-currency должны разворачиваться через helmfile
1. Установка Istio
   * Установить Istio в кластер.
   * Включить автоматическую инжекцию sidecar-прокси для пространства имён, где развёрнуты микросервисы.

2. Конфигурация Istio

   `Ingress Gateway:`
    * Настроить Istio Ingress Gateway для внешнего доступа к muffin-wallet.
    * Обеспечить маршрутизацию трафика по хосту (например, wallet.example.com).
   
   `Observability:`
   * Установить Kiali, Prometheus для визуализации трафика и метрик.
   * Настроить сбор метрик и трассировку запросов между микросервисами.
   
   `Security:`
   * Включить mTLS для шифрования трафика между микросервисами.
   * Настроить AuthorizationPolicy для ограничения доступа к muffin-currency только с muffin-wallet.
   
   `Resilience:`
   * Настроить Circuit Breaker и Retry для устойчивости к сбоям.
   * Реализовать таймауты и лимиты на количество запросов.
   
   `VirtualService, ServiceEntry, Gateway:`
   * Создать VirtualService для маршрутизации трафика между muffin-wallet и muffin-currency.
   * Взаимодействие с внешней БД через ServiceEntry

[Ссылка на задание](https://docs.google.com/document/d/15WfrGchGK4a0Q34Hgpzu6UeBMm_idR9_NkYWNsOTlwc/edit?tab=t.0#heading=h.y22hi4vhg18b)

## Шаги выполнения задания:
1. Установил Istio скачав [архив](https://github.com/istio/istio/releases/tag/1.28.1) и добавив его в PATH. Проверил установку командой:
    ```bash
    istioctl version
    ```
2. Установил Istio в кластер командой:
    ```bash
    istioctl install --set profile=demo -y
    ```
3. Проверил установку командой:
    ```bash
     kubectl get pods -n istio-system
    ```
4. Создал namespace для приложения и включил автоматическую инжекцию sidecar-прокси командой:
    ```bash
    kubectl create namespace muffin-app
    kubectl label namespace muffin-app istio-injection=enabled
    ```
5. Задаем namespace по умолчанию:
    ```bash
    kubectl config set-context --current --namespace=muffin-app
    ```
6. Установил helmfile скачав [архив](https://github.com/helmfile/helmfile/releases) и добавив его в PATH. Проверил установку командой:
    ```bash
    helmfile version
    ```
7. Создал файл `helmfile.yaml` с конфигурацией для установки muffin-wallet и muffin-currency. Запустил командой:
    ```bash
    helmfile sync
    ```
8. Проверяем запуск чартов:
    ```bash
    helm list
    ```
   ```bash
    kubectl get all
   ```
9. Настроил Istio Ingress Gateway для внешнего доступа к muffin-wallet, создав манифесты Gateway и VirtualService.
10. Проверяем доступ к приложению через Ingress Gateway.
    * http://muffin-wallet.ru
11. Установил Kiali, Prometheus, а также Jaeger и Grafana для визуализации трафика и метрик, используя команду:
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.28/samples/addons/prometheus.yaml

    kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.28/samples/addons/kiali.yaml

    kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.28/samples/addons/jaeger.yaml

    kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.28/samples/addons/grafana.yaml
    ```
12. Проверил установку аддонов командой:
    ```bash
    kubectl get pods -n istio-system
    ```
    Также можно проверить командами:
    ```bash
    istioctl dashboard kiali
    
    istioctl dashboard prometheus
    
    istioctl dashboard grafana
    
    istioctl dashboard jaeger
    ```
13. Согласно [документации](https://istio.io/latest/docs/tasks/observability/distributed-tracing/jaeger/) настраиваем трассировку в Jaeger. 
    Изменяем configmap kiali добавив туда:
    ```yaml
    tracing:
      enabled: true
      in_cluster_url: http://jaeger-collector.istio-system:14268/api/traces
    ```
    Команды:
    ```bash
    kubectl edit configmap kiali -n istio-system
    kubectl rollout restart deployment/kiali -n istio-system
    ```
    Затем создаем манифест telemetry для включения сбора трассировки.
14. Включил mTLS для шифрования трафика между микросервисами, создав peer authentication. Для проверки выполнил команду:
    ```bash
    kubectl get peerauthentication 
    ```
    Также можно посмотреть в Kiali.
15. Настроил AuthorizationPolicy для ограничения доступа к muffin-currency только с muffin-wallet, создав манифест authorization-policy, peer-authentication, а также пришлось сделать ServiceAccount и добавить его в хельм манифест. 
    Для проверки можно сделать запрос к muffin-currency через Swagger. 
16. Настроил ServiceEntry для взаимодействия с внешней БД, создав манифест service-entry и создал VirtualService для маршрутизации трафика между muffin-wallet и muffin-currency.
17. Настроил Circuit Breaker, создав манифест destination-rule для muffin-currency и обновив для БД. Также обновил имеющиеся VirtualService, добавив туда retry.