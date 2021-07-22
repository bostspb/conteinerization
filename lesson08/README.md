
## Подготовка
Подключаемся к кластеру

    $ export KUBECONFIG="c:\\work\\_software\\cfg\\kubernetes-cluster-8025_kubeconfig.yaml"
    
    $ kubectl cluster-info
    Kubernetes control plane is running at https://89.208.210.253:6443
    CoreDNS is running at https://89.208.210.253:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
    
    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

Кластер в строю, теперь настраиваем GitLab.

Генерируем SSH ключ командой `ssh-keygen` и добавляем его в настройки аккаунта **Preferences -> SSH keys**

Создаем новый проект с именем **geekbrains** и заливаем туда файлы из 
учебного репозитория `https://github.com/adterskov/geekbrains-conteinerization/tree/master/practice/8.ci-cd`.

## Настраиваем интеграцию GitLab и Kubernetes
Отключаем **Shared Runners** (Settings -> CI/CD -> Runners)

Создаем Namespace для Runner'а

    $ kubectl create ns gitlab
    namespace/gitlab created

Добавляем в манифест Runner'а (репозиторий **geekbrains**) свой регистрационный токен 
из `Settings -> CI/CD -> Runners -> Specific runners -> Set up a specific runner manually -> Registration token` 

Применяем манифесты для Runner'а

    $ kubectl apply --namespace gitlab -f gitlab-runner/gitlab-runner.yaml
    serviceaccount/gitlab-runner created
    role.rbac.authorization.k8s.io/gitlab-runner created
    rolebinding.rbac.authorization.k8s.io/gitlab-runner created
    secret/gitlab-runner created
    configmap/gitlab-runner created
    deployment.apps/gitlab-runner created

Runner появился в списке **Available specific runners**

Создаем Namespace'ы для приложения

    $ kubectl create ns stage
    namespace/stage created
    
    $ kubectl create ns prod
    namespace/prod created

Создаем авторизационные объекты, чтобы Runner мог деплоить в наши Namespace'ы

    $ kubectl create sa deploy --namespace stage
    serviceaccount/deploy created
    
    $ kubectl create rolebinding deploy --serviceaccount stage:deploy --clusterrole edit --namespace stage
    rolebinding.rbac.authorization.k8s.io/deploy created
    
    $ kubectl create sa deploy --namespace prod
    serviceaccount/deploy created
    
    $ kubectl create rolebinding deploy --serviceaccount prod:deploy --clusterrole edit --namespace prod
    rolebinding.rbac.authorization.k8s.io/deploy created

Получаем токены для деплоя в Namespace'ы

    $ export NAMESPACE=stage; kubectl get secret $(kubectl get sa deploy --namespace $NAMESPACE -o jsonpath='{.secrets[0].name}') --namespace $NAMESPACE -o jsonpath='{.data.token}'
    ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNkluYzFWWFpLTFdSSGVtZHlPVFZ3U0VGMFIyRnVkelpWZVZKU0xUUllXUzF1T1hBM2JVdHBTbEJaVW1NaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWR
    DOXVZVzFsYzNCaFkyVWlPaUp6ZEdGblpTSXNJbXQxWW1WeWJtVjBaWE11YVc4dmMyVnlkbWxqWldGalkyOTFiblF2YzJWamNtVjBMbTVoYldVaU9pSmtaWEJzYjNrdGRHOXJaVzR0Tm1oeVkyd2lMQ0pyZFdKbGNtNWxkR1Z6TG1sdkwzTmxjblpwWTJWaFkyTnZkVzUwTDNObGNuWnBZMlV0WVdOamIzVnVkQzV1WV
    cxbElqb2laR1Z3Ykc5NUlpd2lhM1ZpWlhKdVpYUmxjeTVwYnk5elpYSjJhV05sWVdOamIzVnVkQzl6WlhKMmFXTmxMV0ZqWTI5MWJuUXVkV2xrSWpvaVpHVTBaVGM1WmprdE9Ea3dNeTAwTmpVeUxXRTJZalV0WXpGbE5qYzVaVFpqTW1Jd0lpd2ljM1ZpSWpvaWMzbHpkR1Z0T25ObGNuWnBZMlZoWTJOdmRXNTBPb
    k4wWVdkbE9tUmxjR3h2ZVNKOS5Mcmd6ZC1WdUNfWll4QTcyRGZXWlI1c2NyTVcxbTF0enVZLXZrT0F3YnBHWU9FWmRoc0lrQkRQQzhubWx2S25nNDJZVTZ5V0NUQnBVMGduX1E5ZER5OGdGak1BMDFJX1VESWFXb3lveGpSV0kwbElOaHF5QlJtc3hQNWpPdDBmMm9kM0JCUFBPQ0hfTFdsOUhMa3E5QWFoQzIzUkZh
    TlJFNmFhbWRzTUxWTjJXa2hqOTBNVklrYjdnbXRwZjJIanl6OXpZOC0teUs2bmJzQlp5NmQ3dzBMSXE2aXhDVEptTGNIZVU2bGF4NmlBMDBYaTdhb1RKcGpaR2lnNS11MV9kTUk3RUU4dTgtNmJKUlI4RVdmRGRUT1o0ZEhjdW9fZkUwelFrUHBJeF9zcUltVlVZWjR0OGtvZEVsTU9wbkxMbnFjUFlpdE0tVGl4bnh
    DeW05dGdTY1E=
    
    $ export NAMESPACE=prod; kubectl get secret $(kubectl get sa deploy --namespace $NAMESPACE -o jsonpath='{.secrets[0].name}') --namespace $NAMESPACE -o jsonpath='{.data.token}'
    ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNkluYzFWWFpLTFdSSGVtZHlPVFZ3U0VGMFIyRnVkelpWZVZKU0xUUllXUzF1T1hBM2JVdHBTbEJaVW1NaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWR
    DOXVZVzFsYzNCaFkyVWlPaUp3Y205a0lpd2lhM1ZpWlhKdVpYUmxjeTVwYnk5elpYSjJhV05sWVdOamIzVnVkQzl6WldOeVpYUXVibUZ0WlNJNkltUmxjR3h2ZVMxMGIydGxiaTAwT1hCM2VpSXNJbXQxWW1WeWJtVjBaWE11YVc4dmMyVnlkbWxqWldGalkyOTFiblF2YzJWeWRtbGpaUzFoWTJOdmRXNTBMbTVoYl
    dVaU9pSmtaWEJzYjNraUxDSnJkV0psY201bGRHVnpMbWx2TDNObGNuWnBZMlZoWTJOdmRXNTBMM05sY25acFkyVXRZV05qYjNWdWRDNTFhV1FpT2lKaVlUZGhNMlppWVMxa1pUYzBMVFF5TW1JdE9EQmhZaTB4WVRGaU9HTXhaakZoTm1ZaUxDSnpkV0lpT2lKemVYTjBaVzA2YzJWeWRtbGpaV0ZqWTI5MWJuUTZjS
    Ep2WkRwa1pYQnNiM2tpZlEudENYZ3NvazR6UEZ6NGwwYmVVTEJveE9QY1N0V0ExdnFWNjRMMVpYSzk4SHdDTklueUI0dXk0UFZaLXZMOExRVzVvTGpSc2E2RnFZSHBjaC1VWDNsWmpYT2RVNWJualN6V010WmJHZzUwck5zQ2gyQVVwNjljV3haYVhVYXd1SUFYcDFPcjRkU05kU01IQ2tHcHBfUEN6UGVkLTUyYTRJ
    NWFycjhqeEJQbDl5SW1EMWVKam5POXZJY28wbF9NUER5MkNlYV9MNEJFZmQtWnRzVkJlTm9sZE5SYmlHQjMyczFXMS1TRkN4aGc5ZzduN3B2cFdBazhXNUJPNFl0S2pwOV9zTFZ3bS00bHphV3puYUlDNWt0WjlFNTY5MDFXa2M0T1VwRkxCZUZuWWpaWEphbnJEWXh4VlRMQmpGUHA2OTVLaHZuejNQRWUzeUdwM1h
    mSmc1SnRn

Помещаем токены в новые переменные проекта в Gitlab (Settings -> CI/CD -> Variables) с именами **K8S_STAGE_CI_TOKEN** и **K8S_PROD_CI_TOKEN**.

Создаем Token в **Settings -> Repository -> Deploy Tokens**. Из него нам понадобятся Username и Password.

Создаем секреты для авторизации Kubernetes в Gitlab registry. Используем для этого Username и Password из предыдущего шага.

    $ kubectl create secret docker-registry gitlab-registry --docker-server=registry.gitlab.com --docker-username=gitlab+deploy-token-519505 --docker-password=rYrYaa__TTFirYoQ9_LB --docker-email=admin@admin.admin --namespace stage
    secret/gitlab-registry created
    
    $ kubectl create secret docker-registry gitlab-registry --docker-server=registry.gitlab.com --docker-username=gitlab+deploy-token-519505 --docker-password=rYrYaa__TTFirYoQ9_LB --docker-email=admin@admin.admin --namespace prod
    secret/gitlab-registry created

Патчим дефолтный сервис аккаунт для автоматического использования `pull secret`

    $ kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "gitlab-registry"}]}' -n stage
    serviceaccount/default patched
    
    $ kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "gitlab-registry"}]}' -n prod
    serviceaccount/default patched


## Запуск приложения
Применяем манифесты для БД в `stage` и `prod`

    $ kubectl apply --namespace stage -f app/kube/postgres/
    secret/app created
    service/database created
    statefulset.apps/database created

    $ kubectl apply --namespace prod -f app/kube/postgres/
    secret/app created
    service/database created
    statefulset.apps/database created

Редактируем `app/kube/ingress.yaml` - для `host` прописываем значение `stage` и применяем манифесты для Namespace `stage`

    $ kubectl apply --namespace stage -f app/kube
    deployment.apps/geekbrains created
    Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
    ingress.extensions/geekbrains created
    service/geekbrains created

Снова редактируем `app/kube/ingress.yaml` - для `host` прописываем значение `prod` и применяем манифесты для Namespace `prod`

    $ kubectl apply --namespace prod -f app/kube
    deployment.apps/geekbrains created
    Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
    ingress.extensions/geekbrains created
    service/geekbrains created


## Проверяем работу приложения

Подсмотрим внешний IP нашего `ingres-controller (LoadBalancer)`.

    $ kubectl get svc -A
    NAMESPACE       NAME                               TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                      AGE
    default         kubernetes                         ClusterIP      10.254.0.1       <none>           443/TCP                      40h
    ingress-nginx   nginx-ingress-controller           LoadBalancer   10.254.107.96    87.239.108.221   80:30080/TCP,443:30443/TCP   40h
    ingress-nginx   nginx-ingress-controller-metrics   ClusterIP      10.254.135.201   <none>           9913/TCP                     40h
    ingress-nginx   nginx-ingress-default-backend      ClusterIP      10.254.48.128    <none>           80/TCP                       40h
    kube-system     calico-node                        ClusterIP      None             <none>           9091/TCP                     40h
    kube-system     csi-cinder-controller-service      ClusterIP      10.254.245.37    <none>           12345/TCP                    40h
    kube-system     dashboard-metrics-scraper          ClusterIP      10.254.54.67     <none>           8000/TCP                     40h
    kube-system     kube-dns                           ClusterIP      10.254.0.10      <none>           53/UDP,53/TCP,9153/TCP       40h
    kube-system     kubernetes-dashboard               ClusterIP      10.254.22.238    <none>           443/TCP                      40h
    kube-system     metrics-server                     ClusterIP      10.254.66.126    <none>           443/TCP                      40h
    magnum-tiller   tiller-deploy                      ClusterIP      10.254.223.18    <none>           44134/TCP                    40h
    prod            database                           ClusterIP      10.254.28.157    <none>           5432/TCP                     20m
    prod            geekbrains                         ClusterIP      10.254.4.213     <none>           8000/TCP                     3m18s
    stage           database                           ClusterIP      10.254.228.2     <none>           5432/TCP                     20m
    stage           geekbrains                         ClusterIP      10.254.197.89    <none>           8000/TCP                     5m7s

EXTERNAL-IP 87.239.108.221, пробуем сделать к нему запрос на host stage

    $ curl 87.239.108.221/users -H "Host: stage" -X POST -d '{"name": "Vasiya", "age": 34, "city": "Vladivostok"}'
    {"ID":1,"CreatedAt":"2021-07-22T08:24:08.878442232Z","UpdatedAt":"2021-07-22T08:24:08.878442232Z","DeletedAt":null,"name":"Vasiya","city":"Vladivostok","age":34,"status":false}

    $ curl 87.239.108.221/users -H "Host: stage"
    [{"ID":1,"CreatedAt":"2021-07-22T08:24:08.878442Z","UpdatedAt":"2021-07-22T08:24:08.878442Z","DeletedAt":null,"name":"Vasiya","city":"Vladivostok","age":34,"status":false}]


## Домашнее задание

> Переделайте шаг деплоя в **CI/CD**, который демонстрировался на лекции таким образом, 
> чтобы при каждом прогоне шага **deploy** в кластер применялись манифесты приложения. 
> При этом версия докер образа в деплойменте при апплае должна подменяться на ту, что была собрана в шаге **build**.
 
Вносим правки в `kube/deployment.yaml` - меняем `image: nginx:1.12` на `image: __IMAGE__`

Вносим правки в `.gitlab-ci.yml`, для деплоя добавляем строки

    - sed -i "s,__IMAGE__,$CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG.$CI_PIPELINE_ID,g" kube/deployment.yaml
    - kubectl apply -f kube/ --namespace $CI_ENVIRONMENT_NAME

и удаляем строку

    - kubectl set image deployment/$CI_PROJECT_NAME *=$CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG.$CI_PIPELINE_ID --namespace $CI_ENVIRONMENT_NAME

Пушим изменения в репозиторий и смотрим пайплайн - пайплайн выполняется без ошибок

![пайплайн](https://raw.githubusercontent.com/bostspb/conteinerization/main/lesson08/home-task-01.jpg)