# Дипломный практикум в Yandex.Cloud Баранов Сергей
  * [Цели:](#цели)
  * [Этапы выполнения:](#этапы-выполнения)
     * [Создание облачной инфраструктуры](#создание-облачной-инфраструктуры)
     * [Создание Kubernetes кластера](#создание-kubernetes-кластера)
     * [Создание тестового приложения](#создание-тестового-приложения)
     * [Подготовка cистемы мониторинга и деплой приложения](#подготовка-cистемы-мониторинга-и-деплой-приложения)
     * [Установка и настройка CI/CD](#установка-и-настройка-cicd)
  * [Что необходимо для сдачи задания?](#что-необходимо-для-сдачи-задания)
  * [Как правильно задавать вопросы дипломному руководителю?](#как-правильно-задавать-вопросы-дипломному-руководителю)

**Перед началом работы над дипломным заданием изучите [Инструкция по экономии облачных ресурсов](https://github.com/netology-code/devops-materials/blob/master/cloudwork.MD).**

---
## Цели:

1. Подготовить облачную инфраструктуру на базе облачного провайдера Яндекс.Облако.
2. Запустить и сконфигурировать Kubernetes кластер.
3. Установить и настроить систему мониторинга.
4. Настроить и автоматизировать сборку тестового приложения с использованием Docker-контейнеров.
5. Настроить CI для автоматической сборки и тестирования.
6. Настроить CD для автоматического развёртывания приложения.

---
## Этапы выполнения:


### Создание облачной инфраструктуры

Для начала необходимо подготовить облачную инфраструктуру в ЯО при помощи [Terraform](https://www.terraform.io/).

Особенности выполнения:

- Бюджет купона ограничен, что следует иметь в виду при проектировании инфраструктуры и использовании ресурсов;
Для облачного k8s используйте региональный мастер(неотказоустойчивый). Для self-hosted k8s минимизируйте ресурсы ВМ и долю ЦПУ. В обоих вариантах используйте прерываемые ВМ для worker nodes.
- Следует использовать версию [Terraform](https://www.terraform.io/) не старше 1.5.x .

Предварительная подготовка к установке и запуску Kubernetes кластера.

1. Создайте сервисный аккаунт, который будет в дальнейшем использоваться Terraform для работы с инфраструктурой с необходимыми и достаточными правами. Не стоит использовать права суперпользователя

![monitoring](https://github.com/12sergey12/Diplom/blob/main/images/%D0%BE%D0%B1%D1%89%D0%B8%D0%B9%20%D0%B2%D0%B8%D0%B4.yc.png)

2. Подготовьте [backend](https://www.terraform.io/docs/language/settings/backends/index.html) для Terraform:  
   а. Рекомендуемый вариант: S3 bucket в созданном ЯО аккаунте(создание бакета через TF)
   б. Альтернативный вариант:  [Terraform Cloud](https://app.terraform.io/)  

![monitoring](https://github.com/12sergey12/Diplom/blob/main/images/bucket.png)

3. Создайте VPC с подсетями в разных зонах доступности.

![monitoring](https://github.com/12sergey12/Diplom/blob/main/images/subnet.png)

4. Убедитесь, что теперь вы можете выполнить команды `terraform destroy` и `terraform apply` без дополнительных ручных действий.

```

```

5. В случае использования [Terraform Cloud](https://app.terraform.io/) в качестве [backend](https://www.terraform.io/docs/language/settings/backends/index.html) убедитесь, что применение изменений успешно проходит, используя web-интерфейс Terraform cloud.

Ожидаемые результаты:

1. Terraform сконфигурирован и создание инфраструктуры посредством Terraform возможно без дополнительных ручных действий.
2. Полученная конфигурация инфраструктуры является предварительной, поэтому в ходе дальнейшего выполнения задания возможны изменения.

---
### Создание Kubernetes кластера

На этом этапе необходимо создать [Kubernetes](https://kubernetes.io/ru/docs/concepts/overview/what-is-kubernetes/) кластер на базе предварительно созданной инфраструктуры.   Требуется обеспечить доступ к ресурсам из Интернета.

Это можно сделать двумя способами:

1. Рекомендуемый вариант: самостоятельная установка Kubernetes кластера.  
   а. При помощи Terraform подготовить как минимум 3 виртуальных машины Compute Cloud для создания Kubernetes-кластера. Тип виртуальной машины следует выбрать самостоятельно с учётом требовании к производительности и стоимости. Если в дальнейшем поймете, что необходимо сменить тип инстанса, используйте Terraform для внесения изменений.  
   б. Подготовить [ansible](https://www.ansible.com/) конфигурации, можно воспользоваться, например [Kubespray](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/)  
   в. Задеплоить Kubernetes на подготовленные ранее инстансы, в случае нехватки каких-либо ресурсов вы всегда можете создать их при помощи Terraform.
2. Альтернативный вариант: воспользуйтесь сервисом [Yandex Managed Service for Kubernetes](https://cloud.yandex.ru/services/managed-kubernetes)  
  а. С помощью terraform resource для [kubernetes](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kubernetes_cluster) создать **региональный** мастер kubernetes с размещением нод в разных 3 подсетях      
  б. С помощью terraform resource для [kubernetes node group](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kubernetes_node_group)
  
Ожидаемый результат:

1. Работоспособный Kubernetes кластер.
2. В файле `~/.kube/config` находятся данные для доступа к кластеру.
3. Команда `kubectl get pods --all-namespaces` отрабатывает без ошибок.

Создал Kubernetes кластер на базе предварительно созданной инфраструктуры. Обеспечил доступ к ресурсам из Интернета.

Для выполнения данного задания использовал Kubespray

```
root@baranovsa:/home/baranovsa/kubespray# ansible-playbook -i inventory/mycluster/hosts.yaml cluster.yml -b

PLAY [Check Ansible version] **********************************************************************************************

TASK [Check 2.15.4 <= Ansible version < 2.17.0] ***************************************************************************
ok: [node-0] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [Check that python netaddr is installed] *****************************************************************************
ok: [node-0] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [Check that jinja is not too old (install via pip)] ******************************************************************
ok: [node-0] => {
    "changed": false,
    "msg": "All assertions passed"
}
[WARNING]: Could not match supplied host pattern, ignoring: kube-master

PLAY [Add kube-master nodes to kube_control_plane] ************************************************************************
skipping: no hosts matched
[WARNING]: Could not match supplied host pattern, ignoring: kube-node

PLAY [Add kube-node nodes to kube_node] ***********************************************************************************
skipping: no hosts matched
[WARNING]: Could not match supplied host pattern, ignoring: k8s-cluster

PLAY [Add k8s-cluster nodes to k8s_cluster] *******************************************************************************
skipping: no hosts matched
[WARNING]: Could not match supplied host pattern, ignoring: calico-rr

PLAY [Add calico-rr nodes to calico_rr] ***********************************************************************************
skipping: no hosts matched
[WARNING]: Could not match supplied host pattern, ignoring: no-floating

PLAY [Add no-floating nodes to no_floating] *******************************************************************************
skipping: no hosts matched
[WARNING]: Could not match supplied host pattern, ignoring: bastion

PLAY [Install bastion ssh config] *****************************************************************************************
skipping: no hosts matched

PLAY [Bootstrap hosts for Ansible] ****************************************************************************************
[WARNING]: raw module does not support the environment keyword
[WARNING]: raw module does not support the environment keyword
[WARNING]: raw module does not support the environment keyword

TASK [bootstrap-os : Fetch /etc/os-release] *******************************************************************************
ok: [node-0]
ok: [node-2]
ok: [node-1]

TASK [bootstrap-os : Create remote_tmp for it is used by another module] **************************************************
ok: [node-0]
ok: [node-2]
ok: [node-1]

TASK [bootstrap-os : Gather facts] ****************************************************************************************
ok: [node-2]
ok: [node-1]
ok: [node-0]

TASK [bootstrap-os : Assign inventory name to unconfigured hostnames (non-CoreOS, non-Flatcar, Suse and ClearLinux, non-Fedora)] ***
ok: [node-2]
ok: [node-1]
ok: [node-0]

TASK [bootstrap-os : Ensure bash_completion.d folder exists] **************************************************************
ok: [node-0]
ok: [node-2]
ok: [node-1]

PLAY [Gather facts] ***********************************************************************************************
TASK [network_plugin/calico : Check vars defined correctly] ***************************************************************
ok: [node-0] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [network_plugin/calico : Check calico network backend defined correctly] *********************************************
ok: [node-0] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [network_plugin/calico : Check ipip and vxlan mode defined correctly] ************************************************
ok: [node-0] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [network_plugin/calico : Check ipip and vxlan mode if simultaneously enabled] ****************************************
ok: [node-0] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [network_plugin/calico : Get Calico default-pool configuration] ******************************************************
ok: [node-0]

TASK [network_plugin/calico : Set calico_pool_conf] ***********************************************************************
ok: [node-0]

TASK [network_plugin/calico : Check if inventory match current cluster configuration] *************************************
ok: [node-0] => {
    "changed": false,
    "msg": "All assertions passed"
}

PLAY RECAP ****************************************************************************************************************
node-0                     : ok=655  changed=24   unreachable=0    failed=0    skipped=1142 rescued=0    ignored=1   
node-1                     : ok=449  changed=10   unreachable=0    failed=0    skipped=705  rescued=0    ignored=1   
node-2                     : ok=449  changed=10   unreachable=0    failed=0    skipped=701  rescued=0    ignored=1   

root@baranovsa:/home/baranovsa/kubespray# 
```

Файл inventory для ansible playbook [hosts.yaml](https://github.com/12sergey12/Diplom/blob/main/Kuberspray/inventory/mycluster/hosts.yaml)

```
root@node-0:/home/ubuntu# kubectl get nodes
NAME     STATUS   ROLES           AGE   VERSION
node-0   Ready    control-plane   21m   v1.29.5
node-1   Ready    <none>          20m   v1.29.5
node-2   Ready    <none>          20m   v1.29.5
root@node-0:/home/ubuntu# kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-68485cbf9c-vxx9l   1/1     Running   0          19m
kube-system   calico-node-8nzqm                          1/1     Running   0          20m
kube-system   calico-node-kzdq5                          1/1     Running   0          20m
kube-system   calico-node-wbxw2                          1/1     Running   0          20m
kube-system   coredns-69db55dd76-clths                   1/1     Running   0          18m
kube-system   coredns-69db55dd76-tvkh2                   1/1     Running   0          18m
kube-system   dns-autoscaler-6f4b597d8c-jj2vz            1/1     Running   0          18m
kube-system   kube-apiserver-node-0                      1/1     Running   2          21m
kube-system   kube-controller-manager-node-0             1/1     Running   2          21m
kube-system   kube-proxy-m28z6                           1/1     Running   0          5m3s
kube-system   kube-proxy-rslls                           1/1     Running   0          5m3s
kube-system   kube-proxy-xpck8                           1/1     Running   0          5m3s
kube-system   kube-scheduler-node-0                      1/1     Running   1          21m
kube-system   nginx-proxy-node-1                         1/1     Running   0          21m
kube-system   nginx-proxy-node-2                         1/1     Running   0          21m
kube-system   nodelocaldns-7g58k                         1/1     Running   0          18m
kube-system   nodelocaldns-cszrn                         1/1     Running   0          18m
kube-system   nodelocaldns-kqkgb                         1/1     Running   0          18m
root@node-0:/home/ubuntu#
```

Генерация сертификата:
```
root@node-0:/home/ubuntu/myapp# rm /etc/kubernetes/pki/apiserver.* -f
root@node-0:/home/ubuntu/myapp# kubeadm init phase certs apiserver --apiserver-cert-extra-sans 10.233.0.1,10.10.1.5 --apiserver-cert-extra-sans 158.160.111.242 --apiserver-cert-extra-sans localhost
I0612 19:56:28.277268 1731388 version.go:256] remote version is much newer: v1.30.2; falling back to: stable-1.29
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local localhost node-0] and IPs [10.96.0.1 10.10.1.5 10.233.0.1 158.160.111.242]
root@node-0:/home/ubuntu/myapp#
```

файл `~/.kube/config` выглядит так:

```
root@node-0:/home/ubuntu/myapp# kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://158.160.111.242:6443
  name: cluster.local
contexts:
- context:
    cluster: cluster.local
    user: kubernetes-admin
  name: kubernetes-admin@cluster.local
current-context: kubernetes-admin@cluster.local
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
root@node-0:/home/ubuntu/myapp# 

```

---
### Создание тестового приложения

Для перехода к следующему этапу необходимо подготовить тестовое приложение, эмулирующее основное приложение разрабатываемое вашей компанией.

Способ подготовки:

1. Рекомендуемый вариант:  
   а. Создайте отдельный git репозиторий с простым nginx конфигом, который будет отдавать статические данные.  
   б. Подготовьте Dockerfile для создания образа приложения.  
2. Альтернативный вариант:  
   а. Используйте любой другой код, главное, чтобы был самостоятельно создан Dockerfile.

Ожидаемый результат:

1. [Git репозиторий](https://github.com/12sergey12/myapp) с тестовым приложением и [Dockerfile](https://github.com/12sergey12/myapp/blob/main/Dockerfile).

2. Регистри с собранным docker image. В качестве регистри может быть [DockerHub](https://hub.docker.com/layers/12sergey12/myapp/0.0.1/images/sha256-1b56d3e635fdee5beb3e8d16edac61d0ef3cb009f722357618b7c755048d53b8?context=repo) или [Yandex Container Registry](https://cloud.yandex.ru/services/container-registry), созданный также с помощью terraform.

В целях проверки работоспособности соберем образ:

```
root@baranovsa:/home/baranovsa/myapp# docker build --build-arg VEESION=v0.0.3 -t 12sergey12/myapp:0.0.3 .
[+] Building 3.9s (9/9) FINISHED                                                      docker:default
 => [internal] load build definition from Dockerfile                                            0.0s
 => => transferring dockerfile: 281B                                                            0.0s
 => [internal] load metadata for docker.io/library/nginx:1.23.3                                 0.7s
 => [internal] load .dockerignore                                                               0.0s
 => => transferring context: 2B                                                                 0.0s
 => [1/4] FROM docker.io/library/nginx:1.23.3@sha256:f4e3b6489888647ce1834b601c6c06b9f8c03dee6  0.0s
 => [internal] load build context                                                               0.0s
 => => transferring context: 126B                                                               0.0s
 => CACHED [2/4] ADD conf /etc/nginx                                                            0.0s
 => CACHED [3/4] ADD content /usr/share/nginx/html                                              0.0s
 => [4/4] RUN sed -i 's/{{VERSION}}/'"0.0.3"'/g' /usr/share/nginx/html/index.html               2.2s
 => exporting to image                                                                          0.5s
 => => exporting layers                                                                         0.4s
 => => writing image sha256:d6ad24ec1b148f79a3836d9370d37b2c0a7062b8294190be79b18cb407c47792    0.1s
 => => naming to docker.io/12sergey12/myapp:0.0.3                                               0.0s
root@baranovsa:/home/baranovsa/myapp#
```
Образ собран и присутствует в репозитории под тегом 12sergey12/myapp:0.0.3:

```
root@baranovsa:/home/baranovsa/myapp# docker images
REPOSITORY         TAG       IMAGE ID       CREATED       SIZE
12sergey12/myapp   0.0.3     d6ad24ec1b14   4 hours ago   142MB
root@baranovsa:/home/baranovsa/myapp# 
```

![monitoring]()

---
### Подготовка cистемы мониторинга и деплой приложения

Уже должны быть готовы конфигурации для автоматического создания облачной инфраструктуры и поднятия Kubernetes кластера.  
Теперь необходимо подготовить конфигурационные файлы для настройки нашего Kubernetes кластера.

Цель:
1. Задеплоить в кластер [prometheus](https://prometheus.io/), [grafana](https://grafana.com/), [alertmanager](https://github.com/prometheus/alertmanager), [экспортер](https://github.com/prometheus/node_exporter) основных метрик Kubernetes.
2. Задеплоить тестовое приложение, например, [nginx](https://www.nginx.com/) сервер отдающий статическую страницу.

Способ выполнения:
1. Воспользовать пакетом [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus), который уже включает в себя [Kubernetes оператор](https://operatorhub.io/) для [grafana](https://grafana.com/), [prometheus](https://prometheus.io/), [alertmanager](https://github.com/prometheus/alertmanager) и [node_exporter](https://github.com/prometheus/node_exporter). При желании можете собрать все эти приложения отдельно.
2. Для организации конфигурации использовать [qbec](https://qbec.io/), основанный на [jsonnet](https://jsonnet.org/). Обратите внимание на имеющиеся функции для интеграции helm конфигов и [helm charts](https://helm.sh/)
3. Если на первом этапе вы не воспользовались [Terraform Cloud](https://app.terraform.io/), то задеплойте и настройте в кластере [atlantis](https://www.runatlantis.io/) для отслеживания изменений инфраструктуры. Альтернативный вариант 3 задания: вместо Terraform Cloud или atlantis настройте на автоматический запуск и применение конфигурации terraform из вашего git-репозитория в выбранной вами CI-CD системе при любом комите в main ветку. Предоставьте скриншоты работы пайплайна из CI/CD системы.

Ожидаемый результат:
1. Git репозиторий с конфигурационными файлами для настройки Kubernetes.

[Kubespray](https://github.com/kubernetes-sigs/kubespray)

```
root@node-0:/home/ubuntu# curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
root@node-0:/home/ubuntu# chmod 700 get_helm.sh
root@node-0:/home/ubuntu# ./get_helm.sh
[WARNING] Could not find git. It is required for plugin installation.
Downloading https://get.helm.sh/helm-v3.15.2-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm
root@node-0:/home/ubuntu# 
root@node-0:/home/ubuntu# kubectl create namespace monitoring
namespace/monitoring created
root@node-0:/home/ubuntu# sudo helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
"prometheus-community" has been added to your repositories
root@node-0:/home/ubuntu# sudo helm install stable prometheus-community/kube-prometheus-stack --namespace=monitoring
NAME: stable
LAST DEPLOYED: Wed Jun 12 18:09:41 2024
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace monitoring get pods -l "release=stable"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
root@node-0:/home/ubuntu#
```
```
root@node-0:/home/ubuntu# kubectl get all -n monitoring
NAME                                                         READY   STATUS    RESTARTS   AGE
pod/alertmanager-stable-kube-prometheus-sta-alertmanager-0   2/2     Running   0          37s
pod/prometheus-stable-kube-prometheus-sta-prometheus-0       2/2     Running   0          37s
pod/stable-grafana-785b7999d-spl28                           3/3     Running   0          52s
pod/stable-kube-prometheus-sta-operator-f844d969f-gqkmh      1/1     Running   0          52s
pod/stable-kube-state-metrics-5477f4cb54-mnq5j               1/1     Running   0          52s
pod/stable-prometheus-node-exporter-65kbk                    1/1     Running   0          52s
pod/stable-prometheus-node-exporter-fqrzs                    1/1     Running   0          52s
pod/stable-prometheus-node-exporter-jvbwv                    1/1     Running   0          52s

NAME                                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/alertmanager-operated                     ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   37s
service/prometheus-operated                       ClusterIP   None            <none>        9090/TCP                     37s
service/stable-grafana                            ClusterIP   10.233.16.168   <none>        80/TCP                       52s
service/stable-kube-prometheus-sta-alertmanager   ClusterIP   10.233.13.148   <none>        9093/TCP,8080/TCP            52s
service/stable-kube-prometheus-sta-operator       ClusterIP   10.233.49.191   <none>        443/TCP                      52s
service/stable-kube-prometheus-sta-prometheus     ClusterIP   10.233.37.107   <none>        9090/TCP,8080/TCP            52s
service/stable-kube-state-metrics                 ClusterIP   10.233.55.210   <none>        8080/TCP                     52s
service/stable-prometheus-node-exporter           ClusterIP   10.233.13.252   <none>        9100/TCP                     52s

NAME                                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/stable-prometheus-node-exporter   3         3         3       3            3           kubernetes.io/os=linux   52s

NAME                                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/stable-grafana                        1/1     1            1           52s
deployment.apps/stable-kube-prometheus-sta-operator   1/1     1            1           52s
deployment.apps/stable-kube-state-metrics             1/1     1            1           52s

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/stable-grafana-785b7999d                        1         1         1       52s
replicaset.apps/stable-kube-prometheus-sta-operator-f844d969f   1         1         1       52s
replicaset.apps/stable-kube-state-metrics-5477f4cb54            1         1         1       52s

NAME                                                                    READY   AGE
statefulset.apps/alertmanager-stable-kube-prometheus-sta-alertmanager   1/1     37s
statefulset.apps/prometheus-stable-kube-prometheus-sta-prometheus       1/1     37s
root@node-0:/home/ubuntu#
```
2. Http доступ к web интерфейсу grafana.

```
root@node-0:/home/ubuntu# kubectl edit svc stable-kube-prometheus-sta-prometheus -n monitoring
service/stable-kube-prometheus-sta-prometheus edited
root@node-0:/home/ubuntu# kubectl edit svc stable-grafana -n monitoring
service/stable-grafana edited

root@node-0:/home/ubuntu# kubectl get svc -n monitoring
NAME                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
alertmanager-operated                     ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP      42m
prometheus-operated                       ClusterIP   None            <none>        9090/TCP                        42m
stable-grafana                            NodePort    10.233.16.168   <none>        80:32680/TCP                    42m
stable-kube-prometheus-sta-alertmanager   ClusterIP   10.233.13.148   <none>        9093/TCP,8080/TCP               42m
stable-kube-prometheus-sta-operator       ClusterIP   10.233.49.191   <none>        443/TCP                         42m
stable-kube-prometheus-sta-prometheus     NodePort    10.233.37.107   <none>        9090:30643/TCP,8080:30217/TCP   42m
stable-kube-state-metrics                 ClusterIP   10.233.55.210   <none>        8080/TCP                        42m
stable-prometheus-node-exporter           ClusterIP   10.233.13.252   <none>        9100/TCP                        42m
root@node-0:/home/ubuntu#
```

3. Дашборды в grafana отображающие состояние Kubernetes кластера.



4. Http доступ к тестовому приложению.



---
### Установка и настройка CI/CD

Осталось настроить ci/cd систему для автоматической сборки docker image и деплоя приложения при изменении кода.

Цель:

1. Автоматическая сборка docker образа при коммите в репозиторий с тестовым приложением.
2. Автоматический деплой нового docker образа.

Можно использовать [teamcity](https://www.jetbrains.com/ru-ru/teamcity/), [jenkins](https://www.jenkins.io/), [GitLab CI](https://about.gitlab.com/stages-devops-lifecycle/continuous-integration/) или GitHub Actions.

Ожидаемый результат:

1. Интерфейс ci/cd сервиса доступен по http.
2. При любом коммите в репозиторие с тестовым приложением происходит сборка и отправка в регистр Docker образа.
3. При создании тега (например, v1.0.0) происходит сборка и отправка с соответствующим label в регистри, а также деплой соответствующего Docker образа в кластер Kubernetes.

---
## Что необходимо для сдачи задания?

1. Репозиторий с конфигурационными файлами Terraform и готовность продемонстрировать создание всех ресурсов с нуля.
2. Пример pull request с комментариями созданными atlantis'ом или снимки экрана из Terraform Cloud или вашего CI-CD-terraform pipeline.
3. Репозиторий с конфигурацией ansible, если был выбран способ создания Kubernetes кластера при помощи ansible.
4. Репозиторий с Dockerfile тестового приложения и ссылка на собранный docker image.
5. Репозиторий с конфигурацией Kubernetes кластера.
6. Ссылка на тестовое приложение и веб интерфейс Grafana с данными доступа.
7. Все репозитории рекомендуется хранить на одном ресурсе (github, gitlab)
