# Задание 1. Подготовить Helm-чарт для приложения
- Необходимо упаковать приложение в чарт для деплоя в разные окружения.
- Каждый компонент приложения деплоится отдельным deployment’ом или statefulset’ом.
- В переменных чарта измените образ приложения для изменения версии.
```
vagrant@vagrant:~/helm$ tree
.
├── charts
│   └── chart.yaml
├── templates
│   ├── deployment.yaml
│   └── service.yaml
└── values
    └── values.yaml
3 directories, 4 files
vagrant@vagrant:~/helm/charts$ sudo helm package .
Successfully packaged chart and saved it to: /home/vagrant/helm/charts/chart-0.1.0.tgz
vagrant@vagrant:~/helm/charts$ ls
chart-0.1.0.tgz  Chart.yaml
vagrant@vagrant:~/helm$ helm install chart ./charts/chart-0.1.0.tgz
NAME: chart
LAST DEPLOYED: Sun Apr 28 09:07:34 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
vagrant@vagrant:~/helm$ kubectl create -f ./templates/deployment.yaml
deployment.apps/demo created
vagrant@vagrant:~/helm$ kubectl create -f ./templates/service.yaml
service/demo created
vagrant@vagrant:~/helm$ kubectl get pods
NAME                               READY   STATUS              RESTARTS     AGE
demo-b4bfd4457-rtmb5               1/1     Running             0            3m3s
vagrant@vagrant:~/helm$ helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
chart   default         1               2024-04-28 09:07:34.573794249 +0000 UTC deployed        chart-0.1.0
```
Запустим несколько версий приложения
```
vagrant@vagrant:~/helm$ helm upgrade chart --set replicaCount=3 ./charts
Release "chart" has been upgraded. Happy Helming!
NAME: chart
LAST DEPLOYED: Sun Apr 28 09:36:47 2024
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
vagrant@vagrant:~/helm$ kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
demo               1/1     1            1           30m

```
