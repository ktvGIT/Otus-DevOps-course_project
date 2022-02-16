# Проектная работа в рамках обучающего курса Otus -</p> "Devops практики и инструменты".
# Тема проектной работы Создание процесса непрерывной поставки для приложения с применением Практик CI/CD и быстрой обратной связью

# Требования
## Автоматизированные процессы создания и управления платформой
- Ресурсы Ya.cloud
- Инфраструктура для CI/CD
- Инфраструктура для сбора обратной связи
## Использование практики IaC (Infrastructure as Code) для управления конфигурацией и инфраструктурой
- Настроен процесс CI/CD
- Все, что имеет отношение к проекту хранится в Git

## Настроен процесс сбора обратной связи
- Мониторинг (сбор метрик, алертинг, визуализация)
- Логирование (опционально)
- Трейсинг (опционально)
- ChatOps (опционально)

## Документация
- README по работе с репозиторием
- Описание приложения и его архитектуры
- How to start?
- ScreenCast
- CHANGELOG с описанием выполненной работы
- Если работа в группе, то пункт включает автора изменений

## Сроки досдачи проверки ДЗ
- [x] Дедлайн по созданию PR к ДЗ 25.02.2022
- [] Дата окончания курса 01.03.2022

## Сроки сдачи проектов
- [x] 15.02.2022 - день сдачи MVP проекта (готовность не менее 50%). Все кто показал MVP с репо, становятся претендентами на приемку проекта
- [] 25.02.2022 - все кто показывал MVP вносят последние правки и приводят в порядок документацию, changelog и т.п.
- [] 01.03.2022 - финальное занятие
---
---
---
## Данный проект демонстрирует развертывание микросервисного [приложения][https://github.com/GoogleCloudPlatform/microservices-demo] в kubernetes
Автоматизация развертывания kubernetes cluster выполнена при помощи terraform для yandex cloud  
В качестве менеджера установки приложения в kubernetes используются helm 3  
Деплой приложения в кластер осуществляется c помощью gitlab ci  
Мониторинг приложения производится с использованием Istio, Prometheus, визуализация в Kiali, Grafana  
## Неоходимое локальное окружение
[CLI][https://cloud.yandex.ru/docs/cli/operations/install-cli]  
```
curl https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
После завершения установки перезапустите командную оболочку
yc init
```

[terraform](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-quickstart)  
[kubectl]:https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/  
[docker][https://docs.docker.com/engine/install/ubuntu/]  

## Порядок развертывания.
---
### Развертывание kubernetes  
./kubernetes  
Переименовать  terraform.tfvars.example  в terraform.tfvars  
Задать пременные   
terraform init  
terraform plan  
terraform apply -auto-approve  
---
#### Равернуть GitLab, запусить CI\CD
./install_gitlab  
Переименовать  terraform.tfvars.example  в terraform.tfvars  
Задать пременные   
terraform init  
terraform plan  
terraform apply -auto-approve  
ssh -i ~/.ssh/ubuntu ubuntu@[адрес хоста]  
  
Создать поект  
Создать группу  
Создать ранер 
*** Тут надо, либо опиать процесс устанвки shell ранера со всем, что необходимо для его работы, например ансиблом, либо докер в докере ***  
[Enable Docker commands in your CI/CD jobs][https://docs.gitlab.com/ee/ci/docker/using_docker_build.html]
```
sudo gitlab-runner register -n \
  --url https://gitlab.com/ \
  --registration-token REGISTRATION_TOKEN \
  --executor shell \
  --description "My Runner"
sudo usermod -aG docker gitlab-runner  
```
Инструкция в ращделе settings CI/CD
Там же задать необходимые преременные  
KUBE_TOKEN  
KUBE_URL  
Если испольщуется репозиторий  
OAUTH  
REGISTRYID  
  
Подключение kubernetes
Раздел Infrestructure  
Connect with Sertificate  
Инсрукция 
~~~
адрес сластера  
kubectl cluster-info | grep -E 'Kubernetes master|Kubernetes control plane' | awk '/http/ {print $NF}'  

Сектификат  
kubectl get secrets  
List the secrets with kubectl get secrets, and one should be named similar to  
default-token-xxxxx. Copy that token name for use below.  
Get the certificate by running this command:  
kubectl get secret <secret name> -o jsonpath="{['data']['ca\.crt']}" | base64 --decode  

Примение привязки учетной записи службы и роли кластера к вашему кластеру:  
kubectl apply -f ./install_gitlab/gitlab-admin-service-account.yaml  

Получите токен для gitlab учетной записи службы  
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep gitlab | awk '{print $1}')  

~~~
Там же можно добавить интеграцию Prometheus. Описание подключения тут же  
При использовани vs code, установить плагин  GitLab VS Code Extension  
Подключиться к GitLab  
***Тут нужны подробности***
Сделать commit.. каталога ./shop  

### Получить адрес  
~~~
kubectl get service frontend-external | awk '{print $4}'  
~~~

### Устанокака и равертывание istio
[istio][https://istio.io/latest/docs/setup/getting-started/]  
[Visualizing Your Mesh][https://istio.io/latest/docs/tasks/observability/kiali/]  
[Accessing Kiali][https://kiali.io/docs/installation/installation-guide/accessing-kiali/]
[Remotely Accessing Telemetry Addons][https://istio.io/latest/docs/tasks/observability/gateways/]
***Добавлю позже, пока только ссылки***
