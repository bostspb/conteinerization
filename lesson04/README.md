##Урок 4. Хранение данных и ресурсы

Экспортируем настройки кластера для подключения

    $ export KUBECONFIG="E:\\kubernetes-cluster-9166_kubeconfig.yaml"

Проверяем, что доступ к кластеру появился
    
    $ kubectl cluster-info
    Kubernetes control plane is running at https://89.208.210.253:6443
    CoreDNS is running at https://89.208.210.253:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
    
    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

На всякий случай смотрим все ли в порядке с нодами на кластере    
    
    $ kubectl get node
    NAME                                      STATUS   ROLES    AGE   VERSION
    kubernetes-cluster-9166-default-group-0   Ready    <none>   14h   v1.20.4
    kubernetes-cluster-9166-default-group-1   Ready    <none>   14h   v1.20.4
    kubernetes-cluster-9166-default-group-2   Ready    <none>   14h   v1.20.4
    kubernetes-cluster-9166-master-0          Ready    master   14h   v1.20.4

Создаем нужный нам namespace
    
    $ kubectl create ns pg
    namespace/pg created

Переключаемся на него

    $ kubectl config set-context --current --namespace=pg
    Context "default/kubernetes-cluster-9828" modified.

Для запроса PVC нам понадобится класс хранилища, посмотрим из чего можно выбрать
    
    $ kubectl get storageclasses.storage.k8s.io
    NAME                       PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
    csi-ceph-hdd-dp1           cinder.csi.openstack.org   Delete          Immediate           true                   16h
    csi-ceph-hdd-dp1-retain    cinder.csi.openstack.org   Retain          Immediate           true                   16h
    csi-ceph-hdd-ms1           cinder.csi.openstack.org   Delete          Immediate           true                   16h
    csi-ceph-hdd-ms1-retain    cinder.csi.openstack.org   Retain          Immediate           true                   16h
    csi-ceph-ssd-dp1           cinder.csi.openstack.org   Delete          Immediate           true                   16h
    csi-ceph-ssd-dp1-retain    cinder.csi.openstack.org   Retain          Immediate           true                   16h
    csi-ceph-ssd-ms1           cinder.csi.openstack.org   Delete          Immediate           true                   16h
    csi-ceph-ssd-ms1-retain    cinder.csi.openstack.org   Retain          Immediate           true                   16h
    csi-high-iops-dp1          cinder.csi.openstack.org   Delete          Immediate           true                   16h
    csi-high-iops-dp1-retain   cinder.csi.openstack.org   Retain          Immediate           true                   16h
    csi-high-iops-ms1          cinder.csi.openstack.org   Delete          Immediate           true                   16h
    csi-high-iops-ms1-retain   cinder.csi.openstack.org   Retain          Immediate           true                   16h

Выбираем `csi-ceph-ssd-ms1` и создаем манифест для Persistent Volume, накатываем его

    $ kubectl apply -f pvc.yaml
    persistentvolumeclaim/pg-storage created

Проверяем, что сетевой диск создался

    $ kubectl get pvc
    NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
    pg-storage   Bound    pvc-55eea3b9-14ac-48fa-b681-bb105fe5b3cc   10Gi       RWX            csi-ceph-ssd-ms1   100s

Переходим деплойменту, создадим предварительно secret для хранения пароля от БД.

    $ kubectl create secret generic pg-secret --from-literal=PASS=testpassword
    secret/pg-secret created

Проверим, что secret создался

    $ kubectl get secret pg-secret
    NAME        TYPE     DATA   AGE
    pg-secret   Opaque   1      22s
  
Проверим содержимое манифеста secret'а    

    $ kubectl get secret pg-secret -oyaml
    apiVersion: v1
    data:
      PASS: dGVzdHBhc3N3b3Jk
    kind: Secret
    metadata:
      creationTimestamp: "2021-07-04T06:23:48Z"
      name: pg-secret
      namespace: pg
      resourceVersion: "139211"
      selfLink: /api/v1/namespaces/pg/secrets/pg-secret
      uid: 594a73ca-af8e-4531-9219-9ecedb848998
    type: Opaque

Развернем деплоймент с базой

    $ kubectl apply -f deployment.yaml
    deployment.apps/pg-db created

Проверим, что POD с базой поднялся

    $ kubectl get po
    NAME                    READY   STATUS    RESTARTS   AGE
    pg-db-cfff94bc5-hpfz6   0/1     Pending   0          16s

Не поднялся, смотрим в чем причина

    $ kubectl describe po pg-db-cfff94bc5-hpfz6
    ...
    Events:
    Type     Reason             Age   From                Message
    ----     ------             ----  ----                -------
    Warning  FailedScheduling   50s   default-scheduler   0/4 nodes are available: 1 node(s) had taint {CriticalAddonsOnly: True}, that the pod didn't tolerate, 3 Insufficient cpu.
    Warning  FailedScheduling   50s   default-scheduler   0/4 nodes are available: 1 node(s) had taint {CriticalAddonsOnly: True}, that the pod didn't tolerate, 3 Insufficient cpu.
    Normal   NotTriggerScaleUp  40s   cluster-autoscaler  pod didn't trigger scale-up:

Как видно проблема в нехватке ресурсов на CPU, что очень странно, т.к. выделено было не больше, чем есть на нодах.
Несколько попыток подобрать нужный объем ресурсов путем уменьшения параметров в разделе `resources:requests:cpu` до `100Mi` в манифесте `deployment.yaml` не увенчались успехом.
Переустановка всего кластера тоже не помогла.
Поэтому пришлось удалить весь раздел `resources` из манифеста `deployment.yaml` - это помогло, используем это как временное решение, пока не разберемся в исходных причинах.

    $ kubectl get po
    NAME                     READY   STATUS    RESTARTS   AGE
    pg-db-6b7db8f7f5-z9vvf   1/1     Running   0          8m10s


##Проверка

Узнаем IP-адрес POD-а 

    $ kubectl get pod -o wide
    NAME                     READY   STATUS    RESTARTS   AGE   IP               NODE                                      NOMINATED NODE   READINESS GATES
    pg-db-6b7db8f7f5-z9vvf   1/1     Running   0          14m   10.100.154.192   kubernetes-cluster-9166-default-group-2   <none>           <none>

Проксируем порт БД

    $ kubectl port-forward pod/pg-db-6b7db8f7f5-z9vvf 5433:5432
    Forwarding from 127.0.0.1:5433 -> 5432
    Forwarding from [::1]:5433 -> 5432

Пробуем подключиться к БД через DBeaver

![Скриншот](https://github.com/bostspb/conteinerization/blob/main/lesson04/connection_test.jpg)

... это по привычке, извините

Теперь согласно заданию - через консоль отдельного POD'a в кластере
Запускаем POD, с которого будем тестировать POD с базой

    $ kubectl run -t -i --rm --image postgres:10.13 test bash
    If you don't see a command prompt, try pressing enter.
    root@test:/#

Подключаемся к БД

    root@test:/# psql -h 10.100.154.192 -U testuser testdatabase
    Password for user testuser:
    psql (10.13 (Debian 10.13-1.pgdg90+1))
    Type "help" for help.
    
    testdatabase=#

Создаем тестовую таблицу в базе

    testdatabase=# CREATE TABLE testtable (testcolumn VARCHAR (50) );
    CREATE TABLE

Проверяем что таблица создалась

    testdatabase=# \dt
               List of relations
     Schema |   Name    | Type  |  Owner
    --------+-----------+-------+----------
     public | testtable | table | testuser
    (1 row)

Выходим

    testdatabase=# \q
    root@test:/# exit

Удаляем POD с базой

    $ kubectl delete po pg-db-6b7db8f7f5-z9vvf
    pod "pg-db-6b7db8f7f5-z9vvf" deleted

Ждем пока POD пересоздастся, проверяем

    $ kubectl get po
    NAME                     READY   STATUS    RESTARTS   AGE
    pg-db-6b7db8f7f5-zcl8f   1/1     Running   0          94s

Снова смотрим IP-адрес нового POD-а 

    $ kubectl get pod -o wide
    NAME                     READY   STATUS    RESTARTS   AGE     IP              NODE                                      NOMINATED NODE   READINESS GATES
    pg-db-6b7db8f7f5-zcl8f   1/1     Running   0          3m25s   10.100.160.65   kubernetes-cluster-9166-default-group-0   <none>           <none>

Заходим в отдельный тестовый POD

    $ kubectl run -t -i --rm --image postgres:10.13 test bash
    If you don't see a command prompt, try pressing enter.
    root@test:/#

Подключаемся к БД

    root@test:/# psql -h 10.100.160.65 -U testuser testdatabase
    Password for user testuser:
    psql (10.13 (Debian 10.13-1.pgdg90+1))
    Type "help" for help.

Проверяем что таблица, созданная ранее осталась на месте

    testdatabase=# \dt
               List of relations
     Schema |   Name    | Type  |  Owner
    --------+-----------+-------+----------
     public | testtable | table | testuser
    (1 row)

Выходим

    testdatabase=# \q
    root@test:/# exit
