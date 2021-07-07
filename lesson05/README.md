Проверяем работоспособность нашего кластера

    $ kubectl cluster-info
    Kubernetes control plane is running at https://37.139.42.65:6443
    CoreDNS is running at https://37.139.42.65:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
    
    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

Создаем namespace

    $ kubectl create ns redmine
    namespace/redmine created

Переключаем контекст на нужный нам namespace
 
    $ kubectl config set-context --current --namespace=redmine
    Context "default/kubernetes-cluster-2485" modified.

Подготавливаем сетевой диск для базы

    $ kubectl apply -f pvc.yaml
    persistentvolumeclaim/pg-storage created

Проверяем его статус

    $ kubectl get pvc
    NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
    pg-storage   Bound    pvc-e0888440-6ec2-4ef2-92b7-1dd9ee0e9aa5   10Gi       RWX            csi-ceph-ssd-ms1   7s

Подготавливаем `secret` для хранения пароля от базы

    $ kubectl create secret generic pg-secret --from-literal=PASS=rmdbpassword
    secret/pg-secret created

Разворачиваем базу

    $ kubectl apply -f deployment-pg.yaml
    deployment.apps/pg-db created

Разворачиваем сервис для доступа к базе из других контейнеров

    $ kubectl apply -f service-pg.yaml
    service/redmine-service created
    
Подготавливаем `secret` для хранения ключа от Redmine

    $ kubectl create secret generic redmine-secret --from-literal=KEY=supersecretkey
    secret/redmine-secret created

Разворачиваем Redmine

    $ kubectl apply -f deployment-redmine.yaml
    deployment.apps/redmine-app created

Разворачиваем сервис для доступа к Redmine из других контейнеров

    $ kubectl apply -f service-redmine.yaml
    service/redmine-service created

Создаем `ingress`

    $ kubectl apply -f ingress.yaml
    Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
    ingress.extensions/redmine-ingress created

Смотрим все ли поднялось

    $ kubectl get po
    NAME                           READY   STATUS    RESTARTS   AGE
    pg-db-775bd8c95c-dhk48         1/1     Running   0          2m31s
    redmine-app-58fc94cf66-64pbq   1/1     Running   0          50s
    
Ищем внешний IP для доступа к Redmine   
    
    $ kubectl get svc -A
    NAMESPACE       NAME                               TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                      AGE
    default         kubernetes                         ClusterIP      10.254.0.1       <none>           443/TCP                      69m
    ingress-nginx   nginx-ingress-controller           LoadBalancer   10.254.57.43     89.208.211.102   80:30080/TCP,443:30443/TCP   66m
    ingress-nginx   nginx-ingress-controller-metrics   ClusterIP      10.254.172.174   <none>           9913/TCP                     66m
    ingress-nginx   nginx-ingress-default-backend      ClusterIP      10.254.2.138     <none>           80/TCP                       66m
    kube-system     calico-node                        ClusterIP      None             <none>           9091/TCP                     69m
    kube-system     csi-cinder-controller-service      ClusterIP      10.254.15.65     <none>           12345/TCP                    68m
    kube-system     dashboard-metrics-scraper          ClusterIP      10.254.124.142   <none>           8000/TCP                     67m
    kube-system     kube-dns                           ClusterIP      10.254.0.10      <none>           53/UDP,53/TCP,9153/TCP       68m
    kube-system     kubernetes-dashboard               ClusterIP      10.254.104.1     <none>           443/TCP                      67m
    kube-system     metrics-server                     ClusterIP      10.254.73.34     <none>           443/TCP                      67m
    magnum-tiller   tiller-deploy                      ClusterIP      10.254.209.243   <none>           44134/TCP                    68m
    redmine         pg-service                         ClusterIP      10.254.135.82    <none>           5432/TCP                     104s
    redmine         redmine-service                    ClusterIP      10.254.3.166     <none>           80/TCP                       61s

Внешний IP находится в `EXTERNAL-IP` у сервиса `nginx-ingress-controller` с типом `LoadBalancer`. 

Открываем этот адрес в  браузере 

![скриншот](https://raw.githubusercontent.com/bostspb/conteinerization/main/lesson05/redmine-screenshot.jpg)
