# Задание 1. Подготовить Helm-чарт для приложения
- Необходимо упаковать приложение в чарт для деплоя в разные окружения.
- Каждый компонент приложения деплоится отдельным deployment’ом или statefulset’ом.
- В переменных чарта измените образ приложения для изменения версии.
# Задание 2. Запустить две версии в разных неймспейсах
- Подготовив чарт, необходимо его проверить. Запуститe несколько копий приложения.
- Одну версию в namespace=app1, вторую версию в том же неймспейсе, третью версию в namespace=app2.
- Продемонстрируйте результат.
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
vagrant@vagrant:~/helm$ kubectl scale deployment demo --replicas=3
deployment.apps/demo scaled
vagrant@vagrant:~/helm$ kubectl get pods
NAME                               READY   STATUS              RESTARTS     AGE
demo-b4bfd4457-94fsn               1/1     Running             0            7m8s
demo-b4bfd4457-dztdf               1/1     Running             0            7m8s
demo-b4bfd4457-rtmb5               1/1     Running             0            38m
vagrant@vagrant:~/helm/charts$ cat Chart.yaml
# Chart.yaml
apiVersion: v2
name: chart
description: A Helm chart for deploying your application
version: 0.1.2
```
Удалим helm chart, затем создадим новый в namespace app1 и app2
```
vagrant@vagrant:~/helm$ helm uninstall chart
release "chart" uninstalled
vagrant@vagrant:~/helm$ helm install chart2 --namespace app1 --create-namespace --wait --set replicaCount=2 chart
Error: INSTALLATION FAILED: non-absolute URLs should be in form of repo_name/path_to_chart, got: chart
vagrant@vagrant:~/helm$ helm install chart2 --namespace app1 --create-namespace --wait --set replicaCount=2 ./charts
NAME: chart2
LAST DEPLOYED: Sun Apr 28 10:26:46 2024
NAMESPACE: app1
STATUS: deployed
REVISION: 1
TEST SUITE: None
vagrant@vagrant:~/helm$ helm install chart3 --namespace app2 --create-namespace --wait --set replicaCount=1 ./charts
NAME: chart3
LAST DEPLOYED: Sun Apr 28 10:29:59 2024
NAMESPACE: app2
STATUS: deployed
REVISION: 1
TEST SUITE: None
vagrant@vagrant:~/helm$ helm list -n app1
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
chart2  app1            1               2024-04-28 10:26:46.44858354 +0000 UTC  deployed        chart-0.1.0
vagrant@vagrant:~/helm$ helm list -n app2
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
chart3  app2            1               2024-04-28 10:29:59.095093461 +0000 UTC deployed        chart-0.1.0
```
