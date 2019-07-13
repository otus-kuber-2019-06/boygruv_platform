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