# Домашнее задание к занятию «Как работает сеть в K8s»

### Цель задания

Настроить сетевую политику доступа к подам.

### Чеклист готовности к домашнему заданию

1. Кластер K8s с установленным сетевым плагином Calico.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Документация Calico](https://www.tigera.io/project-calico/).
2. [Network Policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/).
3. [About Network Policy](https://docs.projectcalico.org/about/about-network-policy).

-----

### Задание 1. Создать сетевую политику или несколько политик для обеспечения доступа

1. Создать deployment'ы приложений frontend, backend и cache и соответсвующие сервисы.

Пример deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-deployment01
  labels:
    app: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: network-multitool
        image: wbitt/network-multitool
        env:
          - name: HTTP_PORT
            value: "8081"

```

Пример service
```
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
  namespace: app
spec:
  ports:
  - name: frontend-svc
    protocol: TCP
    port: 8081
    targetPort: 8081
  selector:
    app: frontend
  type: ClusterIP # Можно не писать, по умолчанию
```




2. В качестве образа использовать network-multitool.
3. Разместить поды в namespace App.
4. Создать политики, чтобы обеспечить доступ frontend -> backend -> cache. Другие виды подключений должны быть запрещены.
Политики:
deny.yaml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-network-policy
  namespace: app
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```
allow01.yaml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-network-policy
  namespace: app
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: cache 
```

allow02.yaml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-network-policy
  namespace: app
spec:
  podSelector:
    matchLabels:
      app: backend 
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```


allow03.yaml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-network-policy
  namespace: app
spec:
  podSelector:
    matchLabels:
      app: cache 
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend 
```

6. Продемонстрировать, что трафик разрешён и запрещён.

Применяем все yaml файлы
kubectl apply -f *.yaml

смотрим статусы:

```
ubuntu@ubuntu:~/src/kuber/3.3/kuber-homeworks_3.3$ kubectl get networkpolicy --namespace app
NAME                            POD-SELECTOR   AGE
allow-network-policy-backend    app=backend    52s
allow-network-policy-cache      app=cache      55s
allow-network-policy-frontend   app=frontend   97s
deny-network-policy             <none>         10m
```
```
ubuntu@ubuntu:~/src/kuber/3.3/kuber-homeworks_3.3$ kubectl get pod --namespace app
NAME                                     READY   STATUS    RESTARTS   AGE
netology-deployment01-784c5b8b79-hxcsv   1/1     Running   0          12m
netology-deployment02-6f57b6c644-lccrm   1/1     Running   0          12m
netology-deployment03-85f57df67-7mklg    1/1     Running   0          12m
```
```
ubuntu@ubuntu:~/src/kuber/3.3/kuber-homeworks_3.3$ kubectl get svc --namespace app
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
backend-svc    ClusterIP   10.152.183.69    <none>        8082/TCP   13m
cache-svc      ClusterIP   10.152.183.241   <none>        8083/TCP   12m
frontend-svc   ClusterIP   10.152.183.35    <none>        8081/TCP   13m
```

смотрим ip
```
ubuntu@ubuntu:~/src/kuber/3.3/kuber-homeworks_3.3$ kubectl get po -A -o wide --namespace app
NAMESPACE     NAME                                         READY   STATUS    RESTARTS   AGE    IP               NODE     NOMINATED NODE   READINESS GATES
app           netology-deployment01-784c5b8b79-hxcsv       1/1     Running   0          17m    10.1.243.203     ubuntu   <none>           <none>
app           netology-deployment02-6f57b6c644-lccrm       1/1     Running   0          17m    10.1.243.254     ubuntu   <none>           <none>
app           netology-deployment03-85f57df67-7mklg        1/1     Running   0          17m    10.1.243.208     ubuntu   <none>           <none>
app1          one-74999b58f5-5gt2z                         2/2     Running   22         23d    10.1.243.234     ubuntu   <none>           <none>
app1          one-74999b58f5-787jp                         2/2     Running   22         23d    10.1.243.240     ubuntu   <none>           <none>
app1          one-74999b58f5-dvlfh                         2/2     Running   22         23d    10.1.243.242     ubuntu   <none>           <none>
app1          one1-74999b58f5-6t8lr                        2/2     Running   22         23d    10.1.243.247     ubuntu   <none>           <none>
app1          one1-74999b58f5-7v862                        2/2     Running   22         23d    10.1.243.219     ubuntu   <none>           <none>
app1          one1-74999b58f5-m88zf                        2/2     Running   22         23d    10.1.243.216     ubuntu   <none>           <none>
app2          one1-74999b58f5-9kc4w                        2/2     Running   22         23d    10.1.243.238     ubuntu   <none>           <none>
app2          one1-74999b58f5-h6dkf                        2/2     Running   22         23d    10.1.243.243     ubuntu   <none>           <none>
app2          one1-74999b58f5-qf2zn                        2/2     Running   22         23d    10.1.243.231     ubuntu   <none>           <none>
default       netology-deployment01-56497b8fdf-9psth       1/1     Running   0          43m    10.1.243.252     ubuntu   <none>           <none>
default       netology-deployment02-84568c8c4f-5pt89       1/1     Running   0          43m    10.1.243.207     ubuntu   <none>           <none>
default       netology-deployment03-657c6c8c49-qgjgr       1/1     Running   0          43m    10.1.243.200     ubuntu   <none>           <none>
kube-system   calico-kube-controllers-5947598c79-frvkk     1/1     Running   21         40d    10.1.243.221     ubuntu   <none>           <none>
kube-system   calico-node-wrx75                            1/1     Running   6          3d3h   192.168.80.129   ubuntu   <none>           <none>
kube-system   coredns-79b94494c7-9vfzm                     1/1     Running   21         40d    10.1.243.212     ubuntu   <none>           <none>
kube-system   dashboard-metrics-scraper-5bd45c9dd6-xmzsm   1/1     Running   20         40d    10.1.243.241     ubuntu   <none>           <none>
kube-system   hostpath-provisioner-c778b7559-jv6x8         1/1     Running   18         26d    10.1.243.244     ubuntu   <none>           <none>
kube-system   kubernetes-dashboard-57bc5f89fb-xxdw9        1/1     Running   22         40d    10.1.243.249     ubuntu   <none>           <none>
kube-system   metrics-server-7dbd8b5cc9-sfdqt              1/1     Running   20         40d    10.1.243.230     ubuntu   <none>           <none>
```
Подключаемся к POD
```
kubectl exec --namespace app -it netology-deployment01-784c5b8b79-hxcsv -- /bin/bash
```




### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
