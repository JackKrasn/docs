# Установка HELM

Author: Evgeniy Krasnukhin(e.krasnukhin@cft.ru)

# Для чего нужен HELM

[Статья от Microsoft](https://docs.microsoft.com/ru-ru/azure/aks/kubernetes-helm)
[Kubernetes 1.7 – Использование Helm](https://dev-ops-notes.ru/kubernetes/kubernetes-%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-helm)

>[Helm](https://helm.sh) - это средство упаковки с открытым кодом, которое помогает установить приложения Kubernetes и управлять их жизненным циклом.
Аналогично диспетчерам пакетов Linux, таких как APT и Yum, Helm используется для управления чартами Kubernetes, представляющими собой пакеты предварительно настроенных ресурсов Kubernetes.

>Использование Helm – это самый простой способ запуска и управления приложениями в кластере Kubernetes. Helm позволяет выполнять ключевые операции по управлению приложениями, т.к. установка, обновление или их удаление. Если совсем просто, то Helm можно легко рассматривать yum и apt в CentOS или Ubuntu.

>Helm состоит из двух частей: Helm (клиент) и Tiller (сервер). 

## Установка HELM(client)

[Официальная дока по установке](https://docs.helm.sh/using_helm/#quickstart)

Для установки Helm необходимо выполнить следующие команды на хосте, с которого происходит администрирование кластера  Kubernetes (там, где установлен kubectl).
В моем случае на хосте **bs-m-01.ftc.ru**. Я выбрал установку с помощью скрипта. 
~~~
# curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
# chmod 700 get_helm.sh
# ./get_helm.sh
~~~

В результате этого скрипта поставится клиент *helm*. По сути это бинарник, который поставится в /usr/local/bin/helm

## Создание учетной записи службы
>Перед развертыванием Helm в кластере с поддержкой RBAC необходимо создать учетную запись службы и привязку роли для службы Tiller. 

Я создал файл helm-rbac.yaml
~~~ yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
~~~

И создал учетную запись службы и привязку роли
~~~
#kubectl create -f helm-rbac.yaml
~~~

## Настройка HELM

Установка Tiller
~~~ 
#helm init --service-account tiller
~~~

Для  проверки, что Tiller установлен корректно, необходимо выполнить команду:
~~~
# kubectl --namespace kube-system get pods | grep tiller
tiller-deploy-5c688d5f9b-tgtxd          1/1       Running   0          1d
~~~

Обязательно проверить вывод команды
~~~
#helm list
~~~

**Вернет пустоту, т.к. пока что я не устанавливал никакого чарта**

Я не стал пока что заморачиваться с защитой tiller и helm. Сделаю это позже.
 
> Клиент Helm и служба Tiller проверяют подлинность и взаимодействуют друг с другом с помощью протокола TLS/SSL. Этот метод проверки подлинности позволяет обеспечить 
безопасность кластера Kubernetes и развертываемых служб. Для повышения безопасности можно создавать собственные подписанные сертификаты. Каждый пользователь Helm получит 
собственный сертификат клиента, и инициализация Tiller в кластере Kubernetes будет выполняться с помощью сертификатов. 

## Поиск чартов HELM
Чарты Helm используются для развертывания приложений в кластере Kubernetes. Чтобы найти предварительно созданные чарты Helm, используйте команду helm search.
~~~
#healm search
NAME                                    CHART VERSION   APP VERSION                     DESCRIPTION                                                                                                                                          
stable/acs-engine-autoscaler            2.2.0           2.1.1                           Scales worker nodes within agent pools                                                                                                               
stable/aerospike                        0.1.7           v3.14.1.2                       A Helm chart for Aerospike in Kubernetes                                                                                                             
stable/anchore-engine                   0.2.0           0.2.3                           Anchore container analysis and policy evaluatio...                                                                                                   
stable/apm-server                       0.1.0           6.2.4                           The server receives data from the Elastic APM a...                                                                                                   
stable/ark                              1.1.0           0.9.0                           A Helm chart for ark                                                                                                                                 
stable/artifactory                      7.2.2           6.0.0                           Universal Repository Manager supporting all maj...                                                                                                   
stable/artifactory-ha                   0.2.2           6.0.0                           Universal Repository Manager supporting all maj...                                                                                                   
stable/auditbeat                        0.1.0           6.2.4                           A lightweight shipper to audit the activities o...                                                                                                   
stable/aws-cluster-autoscaler           0.3.3                                           Scales worker nodes within autoscaling groups.                                                                                                       
stable/bitcoind                         0.1.3           0.15.1                          Bitcoin is an innovative payment network and a ...                                                                                                   
stable/buildkite                        0.2.3           3                               Agent for Buildkite                                                                                                                                  
stable/burrow                           0.4.4           0.17.1                          Burrow is a permissionable smart contract machine                                                                                                    
stable/centrifugo                       2.0.1           1.7.3                           Centrifugo is a real-time messaging server.                                                                                                          
stable/cerebro                          0.3.0           0.8.1                           A Helm chart for Cerebro - a web admin tool tha...                                                                                                   
stable/cert-manager                     v0.4.0          v0.4.0                          A Helm chart for cert-manager                                                                                                                        
stable/chaoskube                        0.8.0           0.8.0                           Chaoskube periodically kills random pods in you...                                                                                                   
stable/chartmuseum                      1.6.0           0.7.1                           Helm Chart Repository with support for Amazon S...                                                                                                   
stable/chronograf                       0.4.5           1.3                             Open-source web application written in Go and R...                                                                                                   
stable/cluster-autoscaler               0.7.0           1.2.2                           Scales worker nodes within autoscaling groups.                                                                                                       
stable/cockroachdb                      1.1.1           2.0.0                           CockroachDB is a scalable, survivable, strongly...                                                                                                   
stable/concourse                        1.15.0          3.14.1                          Concourse is a simple and scalable CI system.                                                                                                        
stable/consul                           3.3.0           1.0.0                           Highly available and distributed service discov...                                                                                                   
stable/coredns                          0.9.0           1.0.6                           CoreDNS is a DNS server that chains plugins and...                                                                                                   
stable/coscale                          0.2.1           3.9.1                           CoScale Agent                                                                                                                                        
stable/dask                             1.0.4           0.17.4                          Distributed computation in Python with task sch...
stable/dask-distributed                 2.0.2                                           DEPRECATED: Distributed computation in Python
stable/datadog                          1.0.0           6.3.2                           DataDog Agent
stable/dex                              0.2.0           2.10.0                          CoreOS Dex
stable/distributed-tensorflow           0.1.1           1.6.0                           A Helm chart for running distributed TensorFlow...
...
~~~

## Установка чартов Helm
Для установки чартов используется команда [helm install](https://docs.helm.sh/helm/#helm-install)

Пример установки **Ingress**:

~~~
helm install stable/nginx-ingress --name my-release -f values.yaml                                                                                                                                                         
NAME:   my-release                                                                                                                                                                                                                           
LAST DEPLOYED: Thu Aug  2 15:19:58 2018                                                                                                                                                                                                      
NAMESPACE: default                                                                                                                                                                                                                           
STATUS: DEPLOYED                                                                                                                                                                                                                             
                                                                                                                                                                                                                                             
RESOURCES:                                                                                                                                                                                                                                   
==> v1beta1/RoleBinding                                                                                                                                                                                                                      
NAME                      AGE                                                                                                                                                                                                                
my-release-nginx-ingress  0s                                                                                                                                                                                                                 
                                                                                                                                                                                                                                             
==> v1/Service                                                                                                                                                                                                                               
NAME                                      TYPE          CLUSTER-IP    EXTERNAL-IP   PORT(S)                     AGE                                                                                                                          
my-release-nginx-ingress-controller       LoadBalancer  10.233.8.63   172.16.40.33  80:30389/TCP,443:32209/TCP  0s                                                                                                                           
my-release-nginx-ingress-default-backend  ClusterIP     10.233.3.212  <none>        80/TCP                      0s                                                                                                                           
                                                                                                                                                                                                                                             
==> v1/Pod(related)                                                                                                                                                                                                                          
NAME                                                       READY  STATUS             RESTARTS  AGE                                        
my-release-nginx-ingress-controller-566bd6ff59-7bq4q       0/1    ContainerCreating  0         0s                                         
my-release-nginx-ingress-default-backend-7fc47f9c79-hdwxx  0/1    ContainerCreating  0         0s                                  
                                                                                                                                          
==> v1/ConfigMap
NAME                                 DATA  AGE
my-release-nginx-ingress-controller  1     0s

==> v1beta1/ClusterRole
NAME                      AGE
my-release-nginx-ingress  0s

==> v1beta1/ClusterRoleBinding
NAME                      AGE
my-release-nginx-ingress  0s

==> v1beta1/PodDisruptionBudget
NAME                                      MIN AVAILABLE  MAX UNAVAILABLE  ALLOWED DISRUPTIONS  AGE
my-release-nginx-ingress-controller       1              N/A              0                    0s
my-release-nginx-ingress-default-backend  1              N/A              0                    0s

==> v1/ServiceAccount
NAME                      SECRETS  AGE
my-release-nginx-ingress  1        0s

==> v1beta1/Role
NAME                      AGE
my-release-nginx-ingress  0s

==> v1beta1/Deployment
NAME                                      DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
my-release-nginx-ingress-controller       1        1        1           0          0s
my-release-nginx-ingress-default-backend  1        1        1           0          0s


NOTES:
The nginx-ingress controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace default get services -o wide -w my-release-nginx-ingress-controller'

An example Ingress that makes use of the controller:

  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls

~~~

Более подробно об установке и настройке **Ingress** в отдельном документе.