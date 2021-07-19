Экспортируем настройки кластера для подключения

    $ export KUBECONFIG="c:\work\_software\cfg\kubernetes-cluster-2485_kubeconfig.yaml"

Проверяем подключение

    $ kubectl cluster-info
    Kubernetes control plane is running at https://37.139.42.65:6443
    CoreDNS is running at https://37.139.42.65:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
    
    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl apply -f configmap.yaml
configmap/prometheus-config created

$ kubectl apply -f serviceaccount.yaml
serviceaccount/prometheus created

$ kubectl apply -f clusterrole.yaml
Warning: rbac.authorization.k8s.io/v1beta1 ClusterRole is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRole
clusterrole.rbac.authorization.k8s.io/prometheus created

$ kubectl apply -f clusterrolebinding.yaml
Warning: rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
clusterrolebinding.rbac.authorization.k8s.io/prometheus created

$ kubectl apply -f statefulset.yaml
statefulset.apps/prometheus created

$ kubectl apply -f service.yaml
service/prometheus created

$ kubectl apply -f ingress.yaml
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
ingress.extensions/prometheus created

$ kubectl get po
NAME           READY   STATUS    RESTARTS   AGE
prometheus-0   1/1     Running   0          104s

$ kubectl get svc -A
NAMESPACE       NAME                               TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                      AGE
default         kubernetes                         ClusterIP      10.254.0.1       <none>           443/TCP                      12d
default         prometheus                         ClusterIP      None             <none>           80/TCP                       30m
ingress-nginx   nginx-ingress-controller           LoadBalancer   10.254.57.43     89.208.211.102   80:30080/TCP,443:30443/TCP   12d
ingress-nginx   nginx-ingress-controller-metrics   ClusterIP      10.254.172.174   <none>           9913/TCP                     12d
ingress-nginx   nginx-ingress-default-backend      ClusterIP      10.254.2.138     <none>           80/TCP                       12d
kube-system     calico-node                        ClusterIP      None             <none>           9091/TCP                     12d
kube-system     csi-cinder-controller-service      ClusterIP      10.254.15.65     <none>           12345/TCP                    12d
kube-system     dashboard-metrics-scraper          ClusterIP      10.254.124.142   <none>           8000/TCP                     12d
kube-system     kube-dns                           ClusterIP      10.254.0.10      <none>           53/UDP,53/TCP,9153/TCP       12d
kube-system     kubernetes-dashboard               ClusterIP      10.254.104.1     <none>           443/TCP                      12d
kube-system     metrics-server                     ClusterIP      10.254.73.34     <none>           443/TCP                      12d
magnum-tiller   tiller-deploy                      ClusterIP      10.254.209.243   <none>           44134/TCP                    12d



