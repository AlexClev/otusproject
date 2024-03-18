# Курсовой проект OTUS
## Преамбула

Для проекта для выбраны решения, которые планируется в дальнейшем применять в работе, при этом для развертывания использовались различные методы с целью развития навыков обращения с Kubernetes, Yandex Cloud и GitHub Action.

## Описание проекта

Проект включает в cебя демонстрационное приложение, в качестве которого выбран Google microservices-demo (<https://github.com/GoogleCloudPlatform/microservices-demo>), подсистему мониторинга на основе Grafana (<https://grafana.com/>), Loki (<https://grafana.com/oss/loki/>), Prometheus (<https://prometheus.io/>) и AlertManager (<https://prometheus.io/docs/alerting/latest/alertmanager/>) , и подсистему развертывания (непрерывной доставки, CD) в качестве которой выступает ArgoCD (<https://argo-cd.readthedocs.io>).

Для хранения кода используется публичный репозиторий GitHub (<https://github.com/AlexClev/otusproject>).

Для обеспечения автоматизации получения и обновления SSL сертификатов развернут сервис cert-manager <https://cert-manager.io/>

Инфраструктуры проекта базируется на 2 нодах сервиса Managed Service for Kubernetes Yandex Cloud

## Описание взаимодействия компонентов проекта

Развертывание демонстрационного приложения осуществляется через GitHub Action посредством workflow **install_apps.yml** (<https://github.com/AlexClev/otusproject/blob/main/.github/workflows/install_apps.yml>). Инструкцией для развертывания служит helm chart “demo “ (<https://github.com/AlexClev/otusproject/tree/main/helm-charts/demo>).

Любые последующие изменения данного helm chart автоматически применятся через политику ArgoCD.

Аналогичным образом происходит и развертывание подсистемы мониторинга:

workflow: **install_monitoring.yml** (<https://github.com/AlexClev/otusproject/blob/main/.github/workflows/install_monitoring.yml>)

helm chart: kube-prometheus-stack ( <https://github.com/AlexClev/otusproject/tree/main/helm-charts/kube-prometheus-stack>)

Для обращения извне к развернутым выше ресурсам используется ingress контроллер на базе nginx

Контроллер развертывается через GitHub Action workflow **install_infrastructure.yml** (<https://github.com/AlexClev/otusproject/blob/main/.github/workflows/install_infrastructure.yml>)

Инструкцией для развертывания выступает манифест ingress-deploy.yaml (<https://github.com/AlexClev/otusproject/blob/main/ingress/ingress-deploy.yaml>)

Для обращения к ресурсам зарегистрирован домен **projectotus.publicvm.com.** Защита соединений обеспечивается с использованием сертификатов Let's Encrypt через сервис cert-manager.

Развертывание cert-manager производится через GitHub Action workflow **install_infrastructure.yml,** в качестве инструкции используется helm chart **cert-manager** ( <https://github.com/AlexClev/otusproject/tree/main/helm-charts/cert-manager> ) изменения отслеживаются Argocd.

Сам ArgoCD развёртывается также через GitHub Action workflow **install_infrastructure.yml,** в качестве инструкции используется манифест **argocd-deploy.yaml** (<https://github.com/AlexClev/otusproject/blob/main/argocd/argocd-deploy.yaml>)

Для развертывания инфраструктуры в облаке Yandex используется GitHub Action workflow **install_cluster.yml** (<https://github.com/AlexClev/otusproject/blob/main/.github/workflows/install_cluster.yml>)

## Описание компонентов

### Демонстрационное приложение Demo

Исходный helm-chart получен из <https://github.com/GoogleCloudPlatform/microservices-demo>

Внесены изменения для работы через ingress

Извне доступен по имени <https://shop.projectotus.publicvm.com/>

Используется сертификат Let's Encrypt, полученный через сервис cert-manager.

### Централизованное логирование Loki

Исходный helm-chart получен из <https://github.com/bitnami/charts/tree/main/bitnami/grafana-loki/>

### Исходный helm-chart получен из <https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack>

Внесены изменения для работы через ingress.

Логи доступны в интерфейсе Grafana в меню Explore.

### Сбор метрик, их визуализация и оповещения Grafana/Prometheus/AlertManager

### Исходный helm-chart получен из <https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack>

Примечание: изначально были использованы отдельные Help charts для каждого из компонентов, однако настройки из взаимодействия и работы в этом случае занимаю значительно больше времени, в связи с чем был выполнен переход на комплексное решение)

Внесены изменения для работы через ingress, доработаны CDR для работы с ArgoCD

Комплект Dashboard в Grafana доступен сразу после установки, дополнительные шаблоны dashboard размещаются в <https://github.com/AlexClev/otusproject/tree/main/helm-charts/kube-prometheus-stack/templates/grafana/dashboards-1.14>

Интерфейс Grafana доступен по адресу <https://grafana.projectotus.publicvm.com>

Пароль для входа задаётся на этапе установки через GitHub secrets.

Подсистема непрерывной доставки ArgoCD

### Использован исходный манифест разработчика <https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml>

Внесены изменения для работы через loadBalancer.

Используется отдельный файл сертификата и закрытого ключа для обеспечения TLS (сохранены как секреты GitHub ARGO_CRT и ARGO_KEY)

Интерфейс ArgoCD доступен по адресу <https://argocd.projectotus.publicvm.com/>

Пароль для входа задаётся на этапе установки через GitHub secrets (AGRO_PASS).

В процессе развертывания добавляется репозиторий с содержимым helm chart для развертываемых приложений. Ссылка на репозиторий сохранена как переменная GitHub GIT_ADDRESS. Логин и токен для доступа к репозиторию сохранены в GitHub secrets GIT_LOGIN и GIT_TOKEN.

### Инфраструктура Yandex Cloud

Создание кластера осуществляется через GitHub Action отдельным workflow. Используются yc-actions <https://github.com/yc-actions> для установки Yandex CLI.

Предварительно в Yandex Cloud должен быть заведен платёжный аккаунт, создано облако, каталог и сервисный аккаунт с необходимыми ролями, заведена базовая подсеть в одной из зон Яндекса.

Подробное описание доступно тут : <https://cloud.yandex.ru/ru/docs/managed-kubernetes/quickstart>

Для сервисного аккаунта требуется создать ключ в формате json и сохранить его содержимое в GitHub secrets **YC_SA**. Имя или ID сервисного аккаунта должны быть сохранены в GitHub secrets **YA_ACCOUNT.**

SSH – ключи для входа на виртуальные машины должны быть сохранены в GitHub secrets **YC_SSH_KEY** (при необходимости).

Также должны быть заполнены переменные **YA_CLUSTER_NAME** (имя создаваемого кластера Kubernetes), **YA_CLUSTER_ID** (ID или имя облака Яндекс, в котором создаётся кластер Kubernetes), **YA_FOLDER_ID** (ID или имя каталога Яндекс в облаке Яндекс, в котором создаётся кластер Kubernetes)

## Описание процесса установки

Установка разделена на 4 модуля с целью упрощения дальнейшего применения в других проектах:  

1. Развертывание кластера.
2. Развертывание инфраструктуры
3. Развертывание подсистемы мониторинга
4. Развертывание демонстрационного приложения

### Развертывание кластера

#### Условия

### Требуется подготовить Yandex Cloud (см. раздел Инфраструктура Yandex Cloud), заполнить значения переменных GitHub YA_ACCOUNT, YC_SA, YC_SSH_KEY YA_CLUSTER_ID, YA_CLUSTER_NAME, YA_FOLDER_ID и GitHub secrets YA_ACCOUNT, YC_SA, YC_SSH_KEY

#### Действия

Запустить « create yandex cluster» через GitHub Action. Будет запущен workflow [install_cluster.yml](https://github.com/AlexClev/otusproject/blob/main/.github/workflows/install_cluster.yml)

#### Результат

Будет создан кластер Kubernetes с указанным именем , состоящий из 2 нод.

### Развертывание инфраструктуры

#### Условия

### Заполнить значения переменной GitHub GIT_ADDRESS и GitHub secrets GIT_LOGIN, GIT_TOKEN, AGRO_PASS, ARGO_CRT, ARGO_KEY

#### Действия

Запустить «create infrastructure» через GitHub Action. Будет запущен workflow [install_infrastructure.yml](https://github.com/AlexClev/otusproject/blob/main/.github/workflows/install_infrastructure.yml)

#### Результат

Будет развернуто приложение ArgoCD, ingress controller, а также cert-manager

Приложение будет доступно по адресу <https://EXTERNAL_IP.nip.io> , где EXTERNAL_IP -внешний адрес, выданный Argocd в процессе установки.

Вход с использованием пароля из AGRO_PASS

### Развертывание подсистемы мониторинга

#### Условия

Присвоить доменные имя для внешних IP-адресов AgroCD и ingress, указать их значения в переменных GitHub ARGO_URI и GRAFANA_URI, заполнить GitHub secret GRAFANA_PASS

#### Действия

Запустить «create monitoring» через GitHub Action. Будет запущен workflow [install_monitoring.yml](https://github.com/AlexClev/otusproject/blob/main/.github/workflows/install_monitoring.yml)

#### Результат

Будут развернуты приложения Grafana, Loki, Alertmanager, а также агенты для сбора данных в кластере.

Grafana будет доступна по адресу GRAFANA_URI (<https://grafana.projectotus.publicvm.com>),

Вход с использованием пароля из GRAFANA_PASS

### Развертывание демонстрационного приложения

#### Действия

Запустить «install web application» через GitHub Action. Будет запущен workflow [install_apps.yml](https://github.com/AlexClev/otusproject/blob/main/.github/workflows/install_apps.yml)

#### Результат

Будут развернуты микросервисы приложения Google microservices-demo

Демонстрационное приложение будет доступно по адресу (<https://demo.projectotus.publicvm.com>)
