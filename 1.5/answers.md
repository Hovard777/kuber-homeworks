# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 2»

### Задание 1. Создать Deployment приложений backend и frontend

1. Создать Deployment приложения _frontend_ из образа nginx с количеством реплик 3 шт. [deployment_front.yaml](src%2Fdeployment_front.yaml)
2. Создать Deployment приложения _backend_ из образа multitool. [deployment_back.yaml](src%2Fdeployment_back.yaml)
3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера. [service_front.yaml](src%2Fservice_front.yaml) , [service_back.yaml](src%2Fservice_back.yaml)
4. Продемонстрировать, что приложения видят друг друга с помощью Service.
```commandline
ifebres@ifebres-nb:~/github/kuber-homeworks/1.5/src$ kubectl get service
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
back-svc     ClusterIP   10.152.183.137   <none>        9002/TCP   29s
front-svc    ClusterIP   10.152.183.146   <none>        9001/TCP   3m38s
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP    2d5h

ifebres@ifebres-nb:~/github/kuber-homeworks/1.5/src$ kubectl get pod
NAME                                 READY   STATUS    RESTARTS   AGE
multitool-backend-6fd75dffb5-9fjj2   1/1     Running   0          76s
nginx-frontend-dff5f78dc-8v27d       1/1     Running   0          84s
nginx-frontend-dff5f78dc-kp5cl       1/1     Running   0          84s
nginx-frontend-dff5f78dc-r4dlq       1/1     Running   0          84s

ifebres@ifebres-nb:~/github/kuber-homeworks/1.5/src$ kubectl exec -it multitool-backend-6fd75dffb5-9fjj2 /bin/sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/ # curl front-svc:9001
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
/ # 
```

------

### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

1. Включить Ingress-controller в MicroK8S.
```commandline
ifebres@ifebres-nb:~/github/kuber-homeworks/1.5/src$ microk8s enable ingress
Infer repository core for addon ingress
Enabling Ingress
ingressclass.networking.k8s.io/public created
ingressclass.networking.k8s.io/nginx created
namespace/ingress created
serviceaccount/nginx-ingress-microk8s-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-microk8s-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-microk8s-role created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
configmap/nginx-load-balancer-microk8s-conf created
configmap/nginx-ingress-tcp-microk8s-conf created
configmap/nginx-ingress-udp-microk8s-conf created
daemonset.apps/nginx-ingress-microk8s-controller created
Ingress is enabled
```
2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_. [ingress.yaml](src%2Fingress.yaml)
3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
```commandline
ifebres@ifebres-nb:~/github/kuber-homeworks/1.5/src$ curl simple.ingress.com
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


ifebres@ifebres-nb:~/github/kuber-homeworks/1.5/src$ curl simple.ingress.com/api
WBITT Network MultiTool (with NGINX) - multitool-backend-6fd75dffb5-9fjj2 - 10.1.221.172 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)

```


