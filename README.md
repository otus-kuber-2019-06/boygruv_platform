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
## Homework-10 (Helm)
### Helm
- Запуск пода с Tiller в kube-system `helm init --service-account=tiller`
- Для запуска Tiller в определенном неймспейс: `helm init --service-account tiller --tiller-namespace cert-manager`
- Создать релиз Helm `helm install <chart_name> --name=<release_name>`
- Обновить релиз Helm `helm upgrade <release_name> <chart_name>`
- Создать или обновить релиз Helm `helm upgrade --install <release_name> <chart_name>`

### Helm Hooks
- Действия, выполняемые в различные моменты жизненного цикла поставки. Hook, как правило, запускает Job (ноэто не обязательно)
- Виды hooks:
  - pre/post-install
  - pre/post-delete
  - pre/post-upgrade
  - pre/post-rollback
  - crd-install
- Документация `https://github.com/helm/helm/blob/master/docs/charts_hooks.md`
- Например, pre-install hook:
```
apiVersion: ...
kind: ....
metadata:  
  annotations:
    "helm.sh/hook": "pre-install"
```
- Hooks-Weight Позволяют определить порядок выполнения одинаковых типов hooks
- Чем меньше вес - тем раньше будет выполнен hook
```
apiVersion: ...
kind: ....
metadata:  
  annotations:
    "helm.sh/hook": "pre-install"
    "helm.sh/hook-weight": "10"
```

### Helm-tiller (плагин)
- Позволяет обойтись  без  tiller  внутри  кластера
- Установка плагина `helm plugin install https://github.com/rimusz/helm-tiller`
- Пример утановки чарта через плагин
```
helm tiller run \
  helm upgrade --install chartmuseum stable/chartmuseum --wait \
  --namespace=chartmuseum \
  --version=2.3.2 \
  -f kubernetes-templating/chartmuseum/values.yaml
```
- Список чартов `helm tiller run helm list`

### Helmfile
- Установка https://github.com/roboll/helmfile/releases
- Для работы необходим helm plugin diff `helm plugin install https://github.com/databus23/helm-diff --version master`

### Helm secret (плагин)
- Устанока `helm plugin install https://github.com/futuresimple/helm-secrets --version 2.0.2`
- Инсталляция чарта
```
helm secrets upgrade --install frontend kubernetes-templating/frontend --namespace socks-shop \
  -f kubernetes-templating/frontend/values.yaml \
  -f kubernetes-templating/frontend/secrets.yaml
```
- В процессе установки helm-secrets расшифрует наш секретный файл в другой временный файл secrets.yaml.dec, а после выполнения установки - удалит этот временный файл

### Kubecfg
- Установка `https://github.com/bitnami/kubecfg`
- Проверка корректности файлов *.jsonnet `kubecfg show services.jsonnet`
- Установка чарта `kubecfg update services.jsonnet --namespace socks-shop`

### Kustomize
- Установка `https://github.com/kubernetes-sigs/kustomize`
- Установить манифесты с кастомизацией `kustomize build . | kubectl apply -f -`
- Базовая структура
```
├── base
│   ├── deployment.yaml
│   ├── kustomization.yaml
│   └── service.yaml
└── overrides
    ├── socks-shop
    │   └── kustomization.yaml
    └── socks-shop-prod
        └── kustomization.yaml
```

## Homework-11 (Vault)
### Vault install
- `git clone https://github.com/hashicorp/consul-helm.git`
- `helm install --name=consul consul-helm`
- `git clone https://github.com/hashicorp/vault-helm.git`
- `helm install --name=vault .`
```
$ helm status vault
-----------------------
LAST DEPLOYED: Tue Oct 22 16:43:26 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME          DATA  AGE
vault-config  1     76s

==> v1/Pod(related)
NAME     READY  STATUS   RESTARTS  AGE
vault-0  0/1    Running  0         76s
vault-1  0/1    Running  0         76s
vault-2  0/1    Running  0         76s

==> v1/Service
NAME      TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)            AGE
vault     ClusterIP  10.113.5.123   <none>       8200/TCP,8201/TCP  76s
vault-ui  ClusterIP  10.113.10.219  <none>       8200/TCP           76s

==> v1/ServiceAccount
NAME   SECRETS  AGE
vault  1        76s

==> v1/StatefulSet
NAME   READY  AGE
vault  0/3    76s

==> v1beta1/PodDisruptionBudget
NAME   MIN AVAILABLE  MAX UNAVAILABLE  ALLOWED DISRUPTIONS  AGE
vault  N/A            1                0                    76s


NOTES:

Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get vault
```
### Vault login
```
kubectl exec -it vault-0 --  vault login
---------------
Token (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.6PRNZDjElVOcorJc6yWYssD7
token_accessor       xHZMspmJR71h0b8SRSgsn0rx
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```
### Vault get secret 
```
====== Data ======
Key         Value
---         -----
password    asajkjkahs
username    otus
```
### Авторизация через K8S
```
---------------
Path           Type          Accessor                    Description
----           ----          --------                    -----------
kubernetes/    kubernetes    auth_kubernetes_994d7b95    n/a
token/         token         auth_token_51bf42f6 
```
### ClusterRoleBinding
```
---------------
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
    name: role-tokenreview-binding
    namespace: default
roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: system:auth-delegator
subjects:
  - kind: ServiceAccount
    name: vault-auth
    namespace: default
```
- Вопрос: Почему мы смогли записать otus-rw/config1 но не смогли otus-rw/config. Ответ: Потому что в политике нет прав на изменение "update"
### Create certificate
```
---------------
Key                 Value
---                 -----
ca_chain            [-----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUQUzZOdwnfylp0Uijf6SBcbEIx2wwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0xOTEwMjIxODU0NDJaFw0yNDEw
MjAxODU1MTJaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMPtf4vefFaq
BQiu6/RyldxTKM7Q1dCBk1qtdEipn1agzHb6rlrogdPxR2/MKDd0cpNi4XWBvWEY
EhE4cq4VKshxqLL1RkH0GlFIoN1PMvheipYZX1/k0yvk7ImRCVhE3w7WDskKqjfZ
BNna0VEAcRMWla4aqcCiv3CcyJQ9wv88Gwle1/M9lFwO6Rvi+fc5AZKXVjgtil8d
emziXXbdeM84l6/lnebFKces4/wPi3ppkmVn+hwGyHEHBJcltHQLhT39pQvW9nUR
3rOKrY23slWKG1JQCYi4F5IcjKmbMzrg8JLzgV5IxHTlfFv4iUSB9eC6b5IWPfWL
DKMmKD6bGs0CAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUX2ERfBa/VuDhGnzu3JdoBK61+nswHwYDVR0jBBgwFoAU
XLWFVoyevyINdxxJS4KP9lazoQIwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
qY7KJ6VGm7NV8dpQ4ProupurE91259FkF1GcU6Q2/O1Xi6a62nH6wkDiA7bx/WmK
ARDXmUtJz6jTVqyqDUsZ+lEWQfzR8WDgoaP82B55kLq2z0yg8LZuKJrywGKI49En
p9Qvm3nDL+YFLagNcmUXf8oXWCJf7ceLEqrF9XvNZb+TFwIN1L5mvV+Xj8Fh99SV
le2G6QLxXeX4AGZSgW0fg68qot1EujwJvCfomWSgmttBrzpjQ0cLU1KOEhiRFHMy
U6J4fyczisDXnPXErx9sIyN6OFS3YaFg6PjsRDPc9nWwT1EEqnHdoEPbx3LNqL2t
mDbvscLYZClzJ997WdTWGQ==
-----END CERTIFICATE-----]
certificate         -----BEGIN CERTIFICATE-----
MIIDZzCCAk+gAwIBAgIUbLqiuHEnj6UBqkV51QDAiIt2+k0wDQYJKoZIhvcNAQEL
BQAwLDEqMCgGA1UEAxMhZXhhbXBsZS5ydSBJbnRlcm1lZGlhdGUgQXV0aG9yaXR5
MB4XDTE5MTAyMjE5MDAwMVoXDTE5MTAyMzE5MDAzMVowHDEaMBgGA1UEAxMRZ2l0
bGFiLmV4YW1wbGUucnUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC6
mVsmvZevGTuS+5S3Ifiu5v4rXGa17sl3IEXMuLA+GDNpvRBCR9EzVbf6WpAumqF3
DBsNQJVosmYVoMAsRMLSVgc8x08cqvFwgBH9O1gn5mVSeAsmj8WedPQgPw+wjuW2
qUaDETMdrRCEJeolI4/ILO3ouHD/KGu7/FKxwstdUoj3xhWv2R9Qeyg7m/Dbgre+
yPnPGcRKldNbMKCb+gjwCSNMQSYFDg8BuLG0gjQJSykDHVFN27VKzVAf62P2cZBr
9LQp8UH7qXuHUxu4mQolD7ENpPCPzNScfpRbWY9qsqDaK/Fffpe0btcaQSgJaIrx
+NuKgpV3xysjk+KvEJelAgMBAAGjgZAwgY0wDgYDVR0PAQH/BAQDAgOoMB0GA1Ud
JQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAdBgNVHQ4EFgQUgsnQ4mCYPF8gV77A
I9vPKBEbs6IwHwYDVR0jBBgwFoAUX2ERfBa/VuDhGnzu3JdoBK61+nswHAYDVR0R
BBUwE4IRZ2l0bGFiLmV4YW1wbGUucnUwDQYJKoZIhvcNAQELBQADggEBACFHLXRW
9AB8yH3awr1Km676ih7zNbMAdUyduV4j0t+Zui6Hf011xR6MMBWOK9FBFv8fIaEU
ggZ14K1gVecgwCMnTG22RBiEFxKenDslJpYxxxKPLC16uGZJzpY9Na3vbKimCRM0
z8l/6h/9IlMl7chZP0DKTVWSQB082LFmnfR/si9JJYmVOxdgot2vzVNdQ5AODxQW
3VlQpPCgZm/kP+JFoNxt5LjHefN7zT6AS7dJFuiZXDQHJs0WWnlIkzTngiGxhMaL
mj7058asX3KKAk4KRV8G29eG89L5OClUoFiNKJTHfv/oHygDIGjo65+dKyC/CtbB
txdFfNRfOA3pVV0=
-----END CERTIFICATE-----
expiration          1571857231
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUQUzZOdwnfylp0Uijf6SBcbEIx2wwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0xOTEwMjIxODU0NDJaFw0yNDEw
MjAxODU1MTJaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMPtf4vefFaq
BQiu6/RyldxTKM7Q1dCBk1qtdEipn1agzHb6rlrogdPxR2/MKDd0cpNi4XWBvWEY
EhE4cq4VKshxqLL1RkH0GlFIoN1PMvheipYZX1/k0yvk7ImRCVhE3w7WDskKqjfZ
BNna0VEAcRMWla4aqcCiv3CcyJQ9wv88Gwle1/M9lFwO6Rvi+fc5AZKXVjgtil8d
emziXXbdeM84l6/lnebFKces4/wPi3ppkmVn+hwGyHEHBJcltHQLhT39pQvW9nUR
3rOKrY23slWKG1JQCYi4F5IcjKmbMzrg8JLzgV5IxHTlfFv4iUSB9eC6b5IWPfWL
DKMmKD6bGs0CAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUX2ERfBa/VuDhGnzu3JdoBK61+nswHwYDVR0jBBgwFoAU
XLWFVoyevyINdxxJS4KP9lazoQIwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
qY7KJ6VGm7NV8dpQ4ProupurE91259FkF1GcU6Q2/O1Xi6a62nH6wkDiA7bx/WmK
ARDXmUtJz6jTVqyqDUsZ+lEWQfzR8WDgoaP82B55kLq2z0yg8LZuKJrywGKI49En
p9Qvm3nDL+YFLagNcmUXf8oXWCJf7ceLEqrF9XvNZb+TFwIN1L5mvV+Xj8Fh99SV
le2G6QLxXeX4AGZSgW0fg68qot1EujwJvCfomWSgmttBrzpjQ0cLU1KOEhiRFHMy
U6J4fyczisDXnPXErx9sIyN6OFS3YaFg6PjsRDPc9nWwT1EEqnHdoEPbx3LNqL2t
mDbvscLYZClzJ997WdTWGQ==
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAuplbJr2Xrxk7kvuUtyH4rub+K1xmte7JdyBFzLiwPhgzab0Q
QkfRM1W3+lqQLpqhdwwbDUCVaLJmFaDALETC0lYHPMdPHKrxcIAR/TtYJ+ZlUngL
Jo/FnnT0ID8PsI7ltqlGgxEzHa0QhCXqJSOPyCzt6Lhw/yhru/xSscLLXVKI98YV
r9kfUHsoO5vw24K3vsj5zxnESpXTWzCgm/oI8AkjTEEmBQ4PAbixtII0CUspAx1R
Tdu1Ss1QH+tj9nGQa/S0KfFB+6l7h1MbuJkKJQ+xDaTwj8zUnH6UW1mParKg2ivx
X36XtG7XGkEoCWiK8fjbioKVd8crI5PirxCXpQIDAQABAoIBAQCYz5oMIdukc3+M
ISdqyhFD+rtPez5J46FtQyqmOuFqGJuSDljQTswNxDcEeUH2SH/OZEWLFsbElMRx
FdIK7sy1P+pxZa9uBLfwr5EL1pudIVr8rH5FOBxYZeK+vaX1qzCH5BxwnZdEyEPs
VLhpbbQD9HdozBMNgman7j0yghnU86gBceSEEDQyf4+I0OB/wR8C96GNFwUW7vQe
4QbRk6edSDZDD6zwJaOTilnmpQFoCTGgjC8ICVRqOYL65de8MA8kcDczLcjZRAV2
kYEVwtmp7qZZRc6UZEY1kEmlIe56Hp2Inx7wuH3srrDwjpaL1oiaD/CDqW9cjMTi
2EYE2w3JAoGBAPVoFBAwKCYBssB6ddiDo2uLbrALisKJfOnp1TMGUKSoqkb5O3es
xpujADma0wFqBOLOWYXAwujUImPWcTgdo7hXcMyJvD1ZtTDYFE6dJ2cFD4FbjXpV
wpXrhcxDRf+egZofL0zYVSVfeJJsyeF5Y49RC7MyweUgepNKjj7h3b0vAoGBAMKn
aWoWIVUHhZao6AEBhR1NeDPRp9tsdc4kCCDOOvaMOsE3OTvoD1bIf32efiwth2KF
e1euEItsZzkUjl+ef3fqsifGMJds/qrj8q9Yuk3wyW+pBzJ72dPgCejHtK5QrL1k
aQa2T2lvabEer7BKOKX1Y5dZfTppCuU3mnmydotrAoGAb+rHO5q6tJqRqrYuiE+A
d8te72pUHeQV05eQY3S90loZTcTcGffxm4j60UqKcFqpX8Y7jYQbX5NfG6jweWcL
A0bdampmLpR6zYu9txx0m8bzC0t1ehUiaLzAiCcmBS2EbYVLTQBb1G91zVFPwERb
40BS1aaQRq5JOGMH/CWFuoECgYBFBGDfCYu1/13BZpAkUyqkkiRNk0fGCDXY6nOr
VhQX+O6YNYFomUZfCeYSX1DzTw1SxGtQUlpxZPVQitZUVvlxRlj2u1HdTvsZEouo
2nfsTLTPj4oKv1kjw6sfyzdoGxi0albG13tese8yquO2SQq+5TvznPpG7Jm9XjK6
damMkwKBgDcY3jMXfH2yfYxyrHi+R3joDgkMZQtfCOVVto8bWU769MDAHv/Y5Sdm
RmDrslaFHACsGSp7qx+eip/EucaSDZiivM6olYqwjNZtQAq40IFkQXZOv3BeEnCL
fMpebyN+cgAEkiU50dXdeVJZFB0ODkx0Zwx9FjQnghatpbl+u8zw
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       6c:ba:a2:b8:71:27:8f:a5:01:aa:45:79:d5:00:c0:88:8b:76:fa:4d
```
### Revoke certificate
```
---------------
Key                        Value
---                        -----
revocation_time            1571770958
revocation_time_rfc3339    2019-10-22T19:02:38.395602164Z
```
