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