## Данный проект демонстрирует развертывание микросервисного [приложения](https://github.com/GoogleCloudPlatform/microservices-demo) в kubernetes
Автоматизация развертывания kubernetes cluster выполнена при помощи terraform для yandex cloud  
В качестве менеджера установки приложения в kubernetes используются helm 3  
Деплой приложения в кластер осуществляется c помощью gitlab ci  
Мониторинг приложения производится с использованием Istio, Prometheus, визуализация в Kiali, Grafana  
## Описание структуры проекта
/shop
Исходные коды микросервисов
 /src
  adservice  
  cartservice  
  checkoutservice  
  currencyservice  
  Dockerfile  
  emailservice  
  frontend  
  loadgenerator  
  paymentservice  
  productcatalogservice  
  recommendationservice  
  shippingservice  
хелм чары кажждого микросервиса  
  /helm  
Деплой всех микросервсов   
   /all  
Хелм чарты каждого микросервиса в отдельности  
   /deploy 
    /templates  
    Chart.yaml  
    values.yaml  

 .gitlab-ci.yml  
Имеет четыре задачи.  
  - build_tst пустой тестовый контейнер выдающий в лог при запуске слово hello. Используется для отладки и настройки подключения к репозиторию  
  - deploy_tst  Деплой контейнера собранного в build_tst или деплой microservices-demo предворительно скачанного [здесь](https://github.com/GoogleCloudPlatform/microservices-demo)  
  - build_frontend Тестовый билт микросервиса и размещение его в предвоительно созданном Container Registry yandex (иструкция для подключения ниже)  
  - deploy деплой всех микросевисов одним релизом с возможностью определять имя образа через values.yaml (сделано только для frontend как демонстрация)  

Деплой для отладки работы Gitlab  
Dockerfile  
k8s.yaml  

# Иструкция развертыавания ифроструктуры
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
Инструкция 
~~~
адрес кластера  
kubectl cluster-info | grep -E 'Kubernetes master|Kubernetes control plane' | awk '/http/ {print $NF}'  

Сертификат  
kubectl get secrets  
List the secrets with kubectl get secrets, and one should be named similar to  
default-token-xxxxx. Copy that token name for use below.  
Get the certificate by running this command:  
kubectl get secret <secret name> -o jsonpath="{['data']['ca\.crt']}" | base64 --decode  

Применение привязки учетной записи службы и роли кластера к вашему кластеру:  
kubectl apply -f ./install_gitlab/gitlab-admin-service-account.yaml  

Получите токен для gitlab учетной записи службы  
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep gitlab | awk '{print $1}')  

~~~
Там же можно добавить интеграцию Prometheus. Описание подключения тут же на странице  

### Получить адрес поднявшегося сайта  
~~~
kubectl get service frontend-external | awk '{print $4}'  
~~~
[Screenshot of store homepage](./docs/img/shop.PNG)
### Установка и развертывание istio

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
