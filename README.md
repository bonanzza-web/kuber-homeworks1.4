# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к приложению, установленному в предыдущем ДЗ и состоящему из двух контейнеров, по разным портам в разные контейнеры как внутри кластера, так и снаружи.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.
2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.
3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.
4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.


### Ответы:

```
ubuntu@server-1:~/kuber$ micro apply -f one-final-deploy.yaml
deployment.apps/nginx-final created
ubuntu@server-1:~/kuber$ micro get deploy
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
nginx-final   3/3     3            3           8s
ubuntu@server-1:~/kuber$ micro get po -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP             NODE       NOMINATED NODE   READINESS GATES
nginx-final-b66867677-knklv   2/2     Running   0          33s   10.1.125.202   server-1   <none>           <none>
nginx-final-b66867677-tq77n   2/2     Running   0          33s   10.1.125.203   server-1   <none>           <none>
nginx-final-b66867677-htp87   2/2     Running   0          33s   10.1.125.201   server-1   <none>           <none>

```

```
ubuntu@server-1:~/kuber$ micro apply -f nginx-svc.yaml
service/nginx-svc created
ubuntu@server-1:~/kuber$ micro get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP             74m
nginx-svc    ClusterIP   10.152.183.241   <none>        9001/TCP,9002/TCP   6s

```

```
ubuntu@server-1:~/kuber$ micro apply -f mult.yaml
pod/mult created
ubuntu@server-1:~/kuber$ micro get po -o wide
NAME                          READY   STATUS    RESTARTS   AGE     IP             NODE       NOMINATED NODE   READINESS GATES
nginx-final-b66867677-knklv   2/2     Running   0          2m48s   10.1.125.202   server-1   <none>           <none>
nginx-final-b66867677-tq77n   2/2     Running   0          2m48s   10.1.125.203   server-1   <none>           <none>
nginx-final-b66867677-htp87   2/2     Running   0          2m48s   10.1.125.201   server-1   <none>           <none>
mult                          1/1     Running   0          5s      10.1.125.204   server-1   <none>           <none>
ubuntu@server-1:~/kuber$ micro exec mult -- curl 10.152.183.241:9001
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   615  100   615    0     0   2870      0 --:--:-- --:--:-- --:--:--  2860
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
ubuntu@server-1:~/kuber$ micro exec mult -- curl 10.152.183.241:9002
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0WBITT Network MultiTool (with NGINX) - nginx-final-b66867677-tq77n - 10.1.125.203 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
100   146  100   146    0     0   6982      0 --:--:-- --:--:-- --:--:--  7300

```

```
ubuntu@server-1:~/kuber$ micro exec mult -- curl nginx-svc:9001
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
100   615  100   615    0     0   1441      0 --:--:-- --:--:-- --:--:--  1443
ubuntu@server-1:~/kuber$ micro exec mult -- curl nginx-svc:9002
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   146  100   146    0     0  82907      0 --:--:-- --:--:-- --:--:--  142k
WBITT Network MultiTool (with NGINX) - nginx-final-b66867677-tq77n - 10.1.125.203 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)

```
Deployment https://github.com/bonanzza-web/kuber-homeworks1.4/blob/main/docs/one-final-deploy.yaml    
Service https://github.com/bonanzza-web/kuber-homeworks1.4/blob/main/docs/nginx-svc.yaml    
Pod https://github.com/bonanzza-web/kuber-homeworks1.4/blob/main/docs/mult.yaml    

------

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.
2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.

### Ответы:

```
ubuntu@server-1:~/kuber$ micro apply -f nginx-nodeport.yaml
service/nginx-nodeport created
ubuntu@server-1:~/kuber$ micro get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
kubernetes       ClusterIP   10.152.183.1     <none>        443/TCP                         82m
nginx-svc        ClusterIP   10.152.183.241   <none>        9001/TCP,9002/TCP               7m38s
nginx-nodeport   NodePort    10.152.183.41    <none>        9001:30080/TCP,9002:30088/TCP   24s
ubuntu@server-1:~/kuber$ curl localhost:30080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
ubuntu@server-1:~/kuber$ curl localhost:30088
WBITT Network MultiTool (with NGINX) - nginx-final-b66867677-tq77n - 10.1.125.203 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)

```

Screenshots     
    
![alt text](https://github.com/bonanzza-web/kuber-homeworks1.4/blob/main/img/nginx.png)    

![alt text](https://github.com/bonanzza-web/kuber-homeworks1.4/blob/main/img/multitools.png)    


Service NodePort https://github.com/bonanzza-web/kuber-homeworks1.4/blob/main/docs/nginx-nodeport.yaml

------

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
