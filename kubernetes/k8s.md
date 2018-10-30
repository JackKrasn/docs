# Upgrade

Пример обновления кластера с помощью плейбука kubespray.

Обновление с версии v1.10.5 до v1.11.2
Сначала обновиться до версии 1.11.0
~~~
ansible-playbook upgrade-cluster.yml -b -i inventory/mycluster/hosts.ini -e @extra.yml -e kube_version=v1.11.0
~~~
До версии 1.11.2
~~~
ansible-playbook upgrade-cluster.yml -b -i inventory/mycluster/hosts.ini -e @extra.yml -e kube_version=v1.11.2
~~~

Обновление прошло с ошибкой.
В результате состояние нод такое
~~~
kubectl get nodes                                                                                                                                                                                                     
NAME      STATUS                     ROLES     AGE       VERSION                                                                                                                                                                             
bs-01     Ready,SchedulingDisabled   node      22d       v1.11.2                                                                                                                                                                             
bs-02     Ready,SchedulingDisabled   node      22d       v1.10.5                                                                                      
bs-03     Ready                      node      22d       v1.10.5
~~~

Запустил
```
#kubectl uncordon bs-01
node/bs-01 uncordoned
#kubectl get nodes                                                                                                                                                                                                    
NAME      STATUS    ROLES     AGE       VERSION
bs-01     Ready     node      22d       v1.11.2
bs-02     Ready     node      22d       v1.10.5
bs-03     Ready     node      22d       v1.10.5
bs-m-01   Ready     master    22d       v1.11.2
bs-m-02   Ready     master    22d       v1.11.2
bs-m-03   Ready     master    22d       v1.11.2
```

bs-02, bs-03 не обновились. И еще кучу подов в статусе evicted.

Удалил  ноды bs-02,bs-03
~~~
ansible-playbook -i inventory/mycluster/hosts.ini remove-node.yml -b -v --extra-vars "node=bs-02,bs-03"
~~~

И добавил узлы вновь.


# Commands

Информация о кластере

```
kubectl cluster-info
```
Информация о сервисе kubernetes-dashboard в кластере
```
kubectl -n kube-system get service kubernetes-dashboard
```
Информация о нодах
```
kubectl describe nodes
```
Информация о pod-е c указанием ip и имени ноды
~~~
kubectl get pods -o wide
~~~
Получить информацию о pode в yaml формате
~~~
kubectl get pod apprepo-sync-stable-1533628800-2k78n -n kubeapps -o yaml
~~~

Описание pod-а
~~~
kubectl -n kubeapps describe pod apprepository-controller-558c68c455-lgzs9
~~~

Запустить контейнер в кластере c командой /bin/sh и сразу удалить
~~~
kubectl run bash --rm -it --image kubeapps/chart-repo:v1.0.0-alpha.4 /bin/sh
~~~

Просмотреть все ConfigMap-ы в namespace
~~~
kubectl -n kubeapps get cm -o yaml
~~~

Описание роли apprepository-controller
~~~
kubectl -n kubeapps get role apprepository-controller -o yaml
~~~

Внутренний днс, как именуется сервис
~~~
<svc name>.<namespace>.svc.<cluster name>
~~~

Cписок всего, что есть в кластере
~~~
kubectl get all
~~~


Создать сервисный аккаунт для доступа к dashboard (https://blog.heptio.com/on-securing-the-kubernetes-dashboard-16b09b1b7aca)
~~~
# Create the service account in the current namespace
# (we assume default)
kubectl create serviceaccount my-dashboard-sa

# Give that service account root on the cluster
kubectl create clusterrolebinding my-dashboard-sa \
  --clusterrole=cluster-admin \
  --serviceaccount=default:my-dashboard-sa

# Find the secret that was created to hold the token for the SA
kubectl get secrets

# Show the contents of the secret to extract the token
kubectl describe secret my-dashboard-sa-token-xxxxx
~~~

# Настройки

> Может быть не актуально для будущих версий. По сотоянию на 1.11.2, который деплоится с помощью kubespray это актуально

На всех хостах выполнить:
~~~
echo KUBE_FEATURE_GATES=\"PersistentLocalVolumes=true,VolumeScheduling=true,MountPropagation=true\" >> /etc/kubernetes/kubelet.env
sed -i 's/PersistentLocalVolumes=False,VolumeScheduling=False,MountPropagation=False/PersistentLocalVolumes=True,VolumeScheduling=True,MountPropagation=True/' /etc/kubernetes/kubelet.env; systemctl restart kubelet
~~~

а это только на мастер:
~~~
sed -i 's/PersistentLocalVolumes=False,VolumeScheduling=False,MountPropagation=False/PersistentLocalVolumes=True,VolumeScheduling=True,MountPropagation=True/' /etc/kubernetes/manifests/kube-controller-manager.manifest
sed -i 's/PersistentLocalVolumes=False,VolumeScheduling=False,MountPropagation=False/PersistentLocalVolumes=True,VolumeScheduling=True,MountPropagation=True/' /etc/kubernetes/manifests/kube-apiserver.manifest
sed -i 's/PersistentLocalVolumes=False,VolumeScheduling=False,MountPropagation=False/PersistentLocalVolumes=True,VolumeScheduling=True,MountPropagation=True/' /etc/kubernetes/manifests/kube-scheduler.manifest
~~~

# debug
Как проверить readinessProbe.
Определить на каком хосте запустился контейнер
~~~
kubectl describe pod sentry-sentry-web-5db496d776-pthwq
~~~

Запустить bash в контейнере
~~~
kubectl exec -it  sentry-sentry-web-5db496d776-pthwq bash
~~~
Залогиниться на хост, на котором запущен sentry
~~~
ps -ef | grep master
polkitd  25836 25817  0 15:49 ?        00:00:00 tini -- sentry run worker
~~~

Подключиться к сетевому namespace и проверить что порт прослушивается с помощью nsenter [Attach to your Docker containers with ease using nsenter](https://coderwall.com/p/xwbraq/attach-to-your-docker-containers-with-ease-using-nsenter)
~~~
[root@bs-02 ~]# nsenter -t 27415 -n netstat -tunlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:9000            0.0.0.0:*               LISTEN      27415/[Sentry] uWSG
~~~

С помощью curl сделать пробу
~~~
nsenter -t 27415 -n curl -vL http://localhost:9000/_health                                                                                                                                                                   
* About to connect() to localhost port 9000 (#0)
< HTTP/1.1 200 OK
< Content-Length: 2
< X-XSS-Protection: 1; mode=block
< Content-Language: en
< X-Content-Type-Options: nosniff
< Vary: Accept-Language
< X-Frame-Options: deny
< Content-Type: text/plain
<
* Connection #1 to host localhost left intact
ok
~~~

# Helm

## Commands
Для того, чтобы протестировать рендеринг шаблона, но в действительности ничего не выполнять, можно выполнить:
```
helm install --debug --dry-run ./mychart
```
