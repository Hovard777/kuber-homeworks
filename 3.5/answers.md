# Домашнее задание к занятию Troubleshooting


### Задание. При деплое приложение web-consumer не может подключиться к auth-db. Необходимо это исправить

1. Установить приложение по команде:
```shell
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
```
При установке возниувют проблемы - отсутствуют namespace. 
```commandline
ifebres@ifebres-nb:~/.kube$ kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
Error from server (NotFound): error when creating "https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml": namespaces "web" not found
Error from server (NotFound): error when creating "https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml": namespaces "data" not found
Error from server (NotFound): error when creating "https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml": namespaces "data" not found
```
Для решения проблемы создадим namespaces

```commandline
ifebres@ifebres-nb:~/.kube$ kubectl create namespace web
namespace/web created
ifebres@ifebres-nb:~/.kube$ kubectl create namespace data
namespace/data created
ifebres@ifebres-nb:~/.kube$ kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
deployment.apps/web-consumer created
deployment.apps/auth-db created
service/auth-db created
```
Проверяем работоспособность
```commandline
ifebres@ifebres-nb:~/.kube$ kubectl get all -n data
NAME                           READY   STATUS    RESTARTS   AGE
pod/auth-db-7b5cdbdc77-7vs7x   1/1     Running   0          9m49s

NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/auth-db   ClusterIP   10.152.183.235   <none>        80/TCP    9m49s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/auth-db   1/1     1            1           9m49s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/auth-db-7b5cdbdc77   1         1         1       9m49s
ifebres@ifebres-nb:~/.kube$ kubectl get all -n web
NAME                                READY   STATUS    RESTARTS   AGE
pod/web-consumer-5f87765478-2jj99   1/1     Running   0          10m
pod/web-consumer-5f87765478-8kdw5   1/1     Running   0          10m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web-consumer   2/2     2            2           10m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/web-consumer-5f87765478   2         2         2       10m
```
Проверим логи
```commandline
ifebres@ifebres-nb:~/.kube$ kubectl logs pod/web-consumer-5f87765478-2jj99 -n web
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'

ifebres@ifebres-nb:~/.kube$ kubectl logs -f pod/auth-db-7b5cdbdc77-7vs7x -n data 
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
^C
```
Pod web-cosumer не может подключиться к узлу auth-db. Проверим доступ по ip изнутри пода web-cosumer
```commandline
ifebres@ifebres-nb:~/.kube$ kubectl exec -it pod/web-consumer-5f87765478-2jj99 -n web /bin/sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/bin/sh: shopt: not found
[ root@web-consumer-5f87765478-2jj99:/ ]$ curl 10.152.183.235
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
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
```
Проверим резолв имени
```commandline
[ root@web-consumer-5f87765478-2jj99:/ ]$ nslookup auth-db
Server:    10.152.183.10
Address 1: 10.152.183.10 kube-dns.kube-system.svc.cluster.local

nslookup: can't resolve 'auth-db'
```
Исправим, прописав в /etc/hosts
```commandline
[ root@web-consumer-5f87765478-2jj99:/ ]$ echo "10.152.183.235   auth-db" >> etc/hosts
[ root@web-consumer-5f87765478-2jj99:/ ]$ curl auth-db
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
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
```
Добавим изменения в deployment
```yaml
hostAliases:
      - ip: "10.152.183.235"
        hostnames:
        - "auth-db"
```

Проверяем
```commandline
ifebres@ifebres-nb:~/.kube$ kubectl exec -it web-consumer-d977bbc88-bd4g5 -n web cat /etc/hosts 
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.1.221.153	web-consumer-d977bbc88-bd4g5

# Entries added by HostAliases.
10.152.183.235	auth-db
```
Проверяем логи
```commandline
ifebres@ifebres-nb:~/.kube$ kubectl logs -f web-consumer-d977bbc88-bd4g5 -n web|more
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
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
```

```commandline
ifebres@ifebres-nb:~/.kube$ kubectl logs -f auth-db-7b5cdbdc77-7vs7x -n data| more
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
10.1.221.151 - - [14/May/2024:18:49:03 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.35.0" "-"
10.1.221.151 - - [14/May/2024:18:51:52 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.35.0" "-"
10.1.221.151 - - [14/May/2024:18:52:44 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.35.0" "-"
10.1.221.151 - - [14/May/2024:18:52:49 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.35.0" "-"
10.1.221.151 - - [14/May/2024:18:52:54 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.35.0" "-"
```