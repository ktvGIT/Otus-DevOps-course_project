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
## Данный проект демонстрирует развертывание микросервисного [приложения](https://github.com/GoogleCloudPlatform/microservices-demo) в kubernetes
Автоматизация развертывания kubernetes cluster выполнена при помощи terraform для yandex cloud  
В качестве менеджера установки приложения в kubernetes используются helm 3  
Деплой приложения в кластер осуществляется c помощью gitlab ci  
Мониторинг приложения производится с использованием Istio, Prometheus, визуализация в Kiali, Grafana  
## Неоходимое локальное окружение
[CLI](https://cloud.yandex.ru/docs/cli/operations/install-cli)  
[terraform](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-quickstart)  
[kubectl](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/)  
[docker](https://docs.docker.com/engine/install/ubuntu/)  

## Порядок развертывания.

#### Развертывание kubernetes  
./kubernetes  
Переименовать  terraform.tfvars.example  в terraform.tfvars  
Задать пременные   
terraform init  
terraform plan  
terraform apply -auto-approve  

Подключиться локально:  
rm ~/.kube/config 
yc managed-kubernetes cluster get-credentials [имя кластера] --external

#### Равернуть GitLab, запусить CI\CD
./install_gitlab  
Переименовать  terraform.tfvars.example  в terraform.tfvars  
Задать пременные   
terraform init  
terraform plan  
terraform apply -auto-approve  
Сброс пароля  
ssh -i ~/.ssh/ubuntu ubuntu@[адрес хоста]  
~~~
sudo gitlab-rake "gitlab:password:reset[root]"  
~~~

В GitLab
Создать проект  
Создать группу  


Локально
git remote remove origin
git remote add origin http://[адрес gitlab]/[название группы]/[название проекта].git

При использовани vs code, установить плагин  GitLab VS Code Extension  
Подключиться к GitLab  F1-GitLab:(You have registered an access token for that GitLab instance.)
Получить токен доступа (В GitLab settings-Access Tokens)

Сделать commit.. каталога ./shop  

Создать shell ранер 
[Установить kubectl](https://galaxy.ansible.com/codecap/kubectl)  
Исправить hosts  
./ansible/install_kubectl  
ansible-playbook -i hosts pb.yml  

Установить docker helm mc 
/ansible/install_dependencies  
ansible-playbook -i hosts dependencies.yml  
добавить на GitLab машине пользвателю прав на docker  
sudo usermod -aG docker gitlab-runner  

[CLI](https://cloud.yandex.ru/docs/cli/operations/install-cli)   
```
ssh -i ~/.ssh/ubuntu ubuntu@<ip Gitlab>
curl https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
После завершения установки перезапустите командную оболочку
yc init
```

[Enable Docker commands in your CI/CD jobs](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html)
```
sudo gitlab-runner register -n \
  --url https://gitlab.com/ \
  --registration-token REGISTRATION_TOKEN \
  --executor shell \
  --description "My Runner"
sudo usermod -aG docker gitlab-runner  
```
Инструкция в разделе settings CI/CD  
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
адрес кластера  
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
Там же можно добавить интеграцию Prometheus. Описание подключения тут же на странице  


### Получить адрес поднявшегося сайта  
~~~
kubectl get service frontend-external | awk '{print $4}'  
~~~

### Устанокака и равертывание istio

[istio](https://istio.io/latest/docs/setup/install/istioctl/)  
~~~
kubectl create ns istio-system
istioctl install
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/addons/kiali.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/addons/prometheus.yaml

export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_DOMAIN=${INGRESS_HOST}.nip.io

https://istio.io/latest/docs/tasks/observability/gateways/
~~~

Чтобы убедиться, что служба запущена в вашем кластере, выполните следующую команду:
~~~
kubectl -n istio-system get deploy
~~~

Получить адрес
~~~
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_DOMAIN=${INGRESS_HOST}.nip.io
echo $INGRESS_HOST
echo $INGRESS_DOMAIN
~~~

[Visualizing Your Mesh](https://istio.io/latest/docs/tasks/observability/kiali/)  
[Accessing Kiali](https://kiali.io/docs/installation/installation-guide/accessing-kiali/)  
[Remotely Accessing Telemetry Addons](https://istio.io/latest/docs/tasks/observability/gateways/)  



Пример исходнго кода сервиса frontend```./shop/frontend ```  
Хелм чарты для установки сервисов ```./helm ```  
Пример запуска:  
~~~
helm dependency build ./helm/all
helm install shop helm/all -n prod
~~~




