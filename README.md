# GB: Микросервисная архитектура и контейнеризация
> **Geek University Data Engineering**

`Docker` `Kubernetes` `CI/CD`

https://github.com/adterskov/geekbrains-conteinerization

### Урок 1. Микросервисы и контейнеры
1. История появления микросервисной архитектуры
2. Область применения микросервисов
3. Паттерны разработки микросервисных приложений ([The Twelwe-Factor App](https://12factor.net/ru/), [GRASP](https://ru.wikipedia.org/wiki/GRASP))
4. Containerization vs Virtualization
5. Docker и аналоги


### Урок 2. Docker
1. Запуск контейнеров
2. Создание образов
3. Dockerfile
4. Запуск мультисервисных окружений через `docker-compose`
5. Оркестратор контейнеров


**Задание** <br>
Напишите Dockerfile к любому приложению из директорий `golang` или `python` на ваш выбор (можно к обоим).<br>
Образ должен собираться из официального базового образа для выбранного языка. <br>
На этапе сборки должны устанавливаться все необходимые зависимости, а так же присутствовать команда для запуска приложения.<br>
Старайтесь следовать рекомендациям (Best Practices) из лекции при написании Dockerfile.<br>
При запуске контейнера из образа с указанием проксирования порта (флаг `-p` или `-P` если указан `EXPOSE`) при обращении 
на `localhost:port` должно быть доступно приложение в контейнере (оно отвечает `Hello, World!`).<br>
Сохраните получившийся Dockerfile в любом публичном Git репозитории, например GitHub, и пришлите ссылку на репозиторий.<br>

**Решение** <br>
- [Dockerfile для приложения на Python](https://github.com/bostspb/conteinerization/blob/main/lesson02/python/Dockerfile)
- [Dockerfile для приложения на Golang](https://github.com/bostspb/conteinerization/blob/main/lesson02/golang/Dockerfile)

_Примечание_: 
1. Осознанное отступление от Best Practices - использование `ADD` вместо `COPY` и неиспользование Volumes
2. Сборка образа: `docker build . -t app:1.1`
3. Запуск контейнера: `docker run -d -P app:1.1`


### Урок 3. Введение в Kubernetes
1. Описание технологии
2. Основные абстракции приложений
    - Pod
    - ReplicaSet
    - Deployment
3. Работа с kubectl

**Задание** <br>
Напишите deployment для запуска игры Kube DOOM.
[Подробные условия](https://github.com/bostspb/conteinerization/blob/main/lesson03/task.md)

**Решение** <br>
- [Подробное описание хода выполнения задания](https://github.com/bostspb/conteinerization/blob/main/lesson03/README.md)
- [Манифест с описанием абстракций](https://github.com/bostspb/conteinerization/blob/main/lesson03/kubedoom.yaml)
- [Скриншот VNC Viewer с запущенной игрой](https://github.com/bostspb/conteinerization/blob/main/lesson03/screenshot.jpg)


### Урок 4. Хранение данных и ресурсы
1. Ресурсы
2. Хранение конфигураций
    - Secret
    - Configmap
3. Персистентность
    - Persistent Volume
    - Persistent Volume Claim

**Задание** <br>
Напишите deployment для запуска сервера базы данных Postgresql.
[Подробные условия](https://github.com/bostspb/conteinerization/blob/main/lesson04/task.md)

**Решение** <br>
- [Подробное описание хода выполнения задания и его проверки](https://github.com/bostspb/conteinerization/blob/main/lesson04/README.md)
- [Манифест с описанием абстракции PersistentVolumeClaim](https://github.com/bostspb/conteinerization/blob/main/lesson04/pvc.yaml)
- [Манифест с описанием абстракции Deployment](https://github.com/bostspb/conteinerization/blob/main/lesson04/deployment.yaml)


### Урок 5. Сетевые абстракции Kubernetes
1. Service
2. Проверки доступности
3. Ingress
4. Сетевые плагины

**Задание** <br>
Развернуть приложение Redmine с доступом из браузера по белому IP.
[Подробные условия](https://github.com/bostspb/conteinerization/blob/main/lesson05/task.md)

**Решение** <br>
[Подробное описание хода выполнения задания и набор манифестов](https://github.com/bostspb/conteinerization/blob/main/lesson05)


### Урок 6. Устройство кластера
1. Etcd
2. Kube-API-server
3. Kube-controller-manager
4. Kube-scheduler
5. Kubelet
6. Kube-proxy


### Урок 7. Продвинутые абстракции
1. DaemonSet
2. StatefulSet
3. Job
4. CronJob
5. Horizontal Pod Autoscaler

**Задание** <br>
Развернуть в кластере сервер системы мониторинга Prometheus.
[Подробные условия](https://github.com/bostspb/conteinerization/blob/main/lesson07/task.md)

**Решение** <br>
[Подробное описание хода выполнения задания и набор манифестов](https://github.com/bostspb/conteinerization/blob/main/lesson07)
