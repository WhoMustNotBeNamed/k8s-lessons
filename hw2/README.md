# Евсюков Алексанедр БПИ224

## Задание
Презентация к заданию:
https://docs.google.com/presentation/d/1H-xHU6KlTCqD2LTRdIJnsMqbkZBRqXa0Z29i7V5L20E/edit?usp=sharing

https://raw.githubusercontent.com/adan1lov/muffin-currency/refs/heads/main/k8s.yaml

Ссылка на манифест:
Задание переписать данный манифест с использованием helm

Сдача до конца ПН 23:59, на семинаре вы продемонстрируете знания развертывания вашего решения

## Шаги выполнения задания:
1. Создаем чарты helm
    ```bash
    helm create muffin-chart
   ```
2. Отредактируем получившиеся файлы в соответствии с манифестом
   </br>P.S. По сути отредактировал файл values.yaml под нужный манифест
3. Проверяем шаблоны
    ```bash
    helm template muffin-chart
    ```
4. Устанавливаем чарт в кластер
    ```bash
    helm install muffin-release muffin-chart
    ```
5. Проверяем установку
    ```bash
    helm list
    kubectl get all
    helm status muffin-release
    ```
6. Обновляем чарт (меняем количество реплик)
    ```bash
    helm upgrade muffin-release muffin-chart
    ```
7. Проверяем обновление
    ```bash
    helm list
    kubectl get pods
   ```
8. Делаем откат
    ```bash
    helm rollback muffin-release 1
    ```
9. Проверяем откат
    ```bash
    helm list
    kubectl get pods
    ```
10. Проверяем историю релизов
    ```bash
    helm history muffin-release
    ```
11. Удаляем релиз
    ```bash
    helm uninstall muffin-release
    ```
12. Проверяем удаление
    ```bash
    helm list
    kubectl get all
    ```
