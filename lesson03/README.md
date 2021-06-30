## Урок 3. Введение в Kubernetes

Устанавливаем утилиту `kubectl`<br>
https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/
Так как у меня уже установлен Docker Desktop, то эта утилита уже имеется.
Конфигурируем ее - скачиваем конфигурацию нашего кластера на MCS и помещаем ее в файл `~/.kube/config`.

Чтобы проверить корректность конфигурации - выполняем команду `kubectl cluster-info`

    $ kubectl cluster-info
    Kubernetes control plane is running at https://89.208.211.102:6443
    CoreDNS is running at https://89.208.211.102:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

Проверяем ноды на кластере

    $ kubectl get node
    NAME                                      STATUS   ROLES    AGE   VERSION
    kubernetes-cluster-9828-default-group-0   Ready    <none>   20h   v1.20.4
    kubernetes-cluster-9828-default-group-1   Ready    <none>   20h   v1.20.4
    kubernetes-cluster-9828-default-group-2   Ready    <none>   41m   v1.20.4
    kubernetes-cluster-9828-master-0          Ready    master   20h   v1.20.4

Cоздаем namespace **kubedoom** 

    $ kubectl create ns kubedoom

Переключаем контекст на нужный нам namespace
 
    $ kubectl config set-context --current --namespace=kubedoom`

Описываем нужные абстракции в манифесте `kubedoom.yaml`

Разворачиваем абстракции

    $ kubectl apply -f kubedoom.yaml
    serviceaccount/kubedoom created
    clusterrolebinding.rbac.authorization.k8s.io/kubedoom created
    deployment.apps/kubedoom-deployment created

Проверяем

    $ kubectl get po
    NAME                                   READY   STATUS    RESTARTS   AGE
    kubedoom-deployment-6d69946486-9r5v5   1/1     Running   0          38m

Запускаем проксирование портов - `kubectl port-forward pod/kubedoom-deployment-6d69946486-9r5v5 5901:5900`

Скачиваем **VNC Viewer** - https://www.realvnc.com/en/connect/download/viewer/, подключаемся  по адресу `127.0.0.1:5901`, вводим пароль `idbehold` и режемся в DOOM ))