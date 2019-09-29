# boygruv_platform
## Homework-1
### Знакомство с Kubernetes
- Установил Minikube
- Запуск кластера: `minikube start`
- Информация о кластере: `kubectl cluster-info`
- Подключение к ВМ: `minikube ssh`
### Восстановление подов после удаления
- Поды из control-plain восстанавливаются сервисом kubelet
- Поды созданные при помощи deployment восстанавливает API-SERVER

### Запуск пода
- Запустили под с использованием INIT-контейнера подготавливающего общий VOLUME
- Для отслеживания состояния развертывания пода используем команду: `kubectl describe po web`
- Для мониторинг статуса развертывания используем ключ -w: `kubectl get pods -w`
- Для подключения к запущенному поду используем команду: `kubectl port-forward --address 0.0.0.0 pod/web 8000:8000`. Для упрощения доступа можно использовать https://kube-forwarder.pixelpoint.io/

## Homework-2
### ServiceAccount
- Создал service account bob и присвоил ему кластерную роль admin
- Создал service account dave без присвоения роли
- Создал нэймспейс
- Создал сервисаккаунт в нэймспейс
- Создал кластерную роль для просмотра подов и присвоил группе system:serviceaccounts в нэймспейсе созданную роль
- Создал в нэймспейсе два сервисаккаунта и присоил одному роль admin, а второму view в рамках нэймспейса

## Homework-3
- Изучили механизм readinessProbe и livenessProbe для пода
- Изучили сущность Deployment 
- Изучили сущность Service
- Переключили Minikube на режим IPVS
- Изучили механизм LoadBalancer на примере MetalLB
- Изучили сущность Ingress

## Homework-4 (Volumes, Storages,StatefulSetStatefulS)
- Изучили сущность StatefulSet
- Изучили сущность PV
- Изучили сущность PVC

## Homework-5 (CSI-драйвера)
- Создал кластер с указанием параметров для включения Snapshot (VolumeSnapshotDataSource=true)
- Добавил драйвер CSI - csi-driver-host-path (https://github.com/kubernetes-csi/csi-driver-host-path)
- Добавил в кластер StorageClass с указанием провизионера hostpath.csi.k8s.io
- Создал PersistentVolumeClaim с указанным storageClassName созданным на предидущем этапе
- Создал Pod с указанием созданного PersistentVolumeClaim
- VolumeShapshotClass для нашего провизионера создался автоматически при установке драйвера
- Создал снапшот с созданного PersistentVolumeClaim (kubectl get volumesnapshot)
- Удалил под, удалил PVC
- Восстановил PVC из снапшота, восстановил под  

## Homework-6 (Debug)
### kubectl debug
Для запуска `sctarce -c -p1` необходимы capabilites "SYS_PTRACE". Для устанвоки необходимых capabilites в манифест описания пода надо добавить
```
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_PTRACE", "SYS_ADMIN"]
```
### iptables-tailer
- https://github.com/box/kube-iptables-tailer
- Выводит информацию об отброшенных iptables пакетах в события пода (describe po), а также в журнал событий Kubernetes (kubectl get events)
- Для корректной работы необходима настройка ServiceAccount
- **netperf-operator** (https://github.com/piontec/netperf-operator). Kubernetes-оператор,который позволяет запускать тесты пропускной способности сети между нодами кластера
- Проверка заблокированных пакетов на ноде?
```
      $ iptables --list -nv | grep DROP
      $ iptables --list -nv | grep LOG
      $ journalctl -k | grep calico-pack
```

## Homework-7 (Операторы)
### CustomResourceDefinition
- CustomResourceDefinition- ресурс для определения CustomResource
- Cоздание CustomResourceDefinition создает RESTful путь для каждой описанной версии CustomResource
- Определяет структуру и доступные версии для конкретного CustomResource
- CustomResourceDefinition: это ресурс для определения других ресурсов (далее CRD)
- Посмотреть CRD и CR
```
    $ kubectl get crd
    $ kubectl get mysqls.otus.homework
    $ escribe mysqls.otus.homework mysql-instance
```
- Validation
- Operator SDK (ansible) https://github.com/operator-framework/operator-sdk/blob/master/doc/proposals/ansible-operator.md
- Kubernetes Operator Pythonic Framework (Kopf) https://github.com/zalando-incubator/kopf
- Выполнение комманды `kubectl get jobs`
```
NAME                         COMPLETIONS   DURATION   AGE
backup-mysql-instance-job    1/1           2s         4m37s
restore-mysql-instance-job   1/1           6m21s      7m6s
```
- Вывод при запущенном MySQL
```
+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+
```
