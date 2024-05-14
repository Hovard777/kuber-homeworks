# Домашнее задание к занятию «Хранение в K8s. Часть 2»

### Задание 1

**Что нужно сделать**

Создать Deployment приложения, использующего локальный PV, созданный вручную.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool. [deployment_pv.yaml](src%2Fdeployment_pv.yaml)
2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде. [pv.yaml](src%2Fpv.yaml), [pvc.yaml](src%2Fpvc.yaml)
3. Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории. 
```commandline
ifebres@ifebres-nb:~/github/kuber-homeworks/2.2/src$ kubectl get pod
NAME                           READY   STATUS    RESTARTS   AGE
data-pv-pod-6cc4f9f9bf-tmzzm   2/2     Running   0          44s
ifebres@ifebres-nb:~/github/kuber-homeworks/2.2/src$ kubectl exec -it data-pv-pod-6cc4f9f9bf-tmzzm -c multitool -- cat /shared-multitool/data.txt
Mon May 13 18:16:19 UTC 2024
Mon May 13 18:16:24 UTC 2024
Mon May 13 18:16:29 UTC 2024
Mon May 13 18:16:34 UTC 2024
Mon May 13 18:16:39 UTC 2024
Mon May 13 18:16:44 UTC 2024
Mon May 13 18:16:49 UTC 2024
Mon May 13 18:16:54 UTC 2024
Mon May 13 18:16:59 UTC 2024
Mon May 13 18:17:04 UTC 2024
Mon May 13 18:17:09 UTC 2024
Mon May 13 18:17:14 UTC 2024
Mon May 13 18:17:19 UTC 2024
Mon May 13 18:17:24 UTC 2024
Mon May 13 18:17:29 UTC 2024
```
4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.
```commandline
ifebres@ifebres-nb:~/github/kuber-homeworks/2.2/src$ kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM               STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
local-pv   1Gi        RWO            Retain           Released   default/local-pvc                  <unset>                          8m11s
```
  Cостояние PV изменилось на "Released", то есть PV больше не привязан к какому-либо PVC и доступен для повторного использования.  
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
```commandline
ifebres@ifebres-nb:/var/local-pv$ ll
итого 12
drwxr-xr-x  2 root root 4096 мая 13 21:16 ./
drwxr-xr-x 16 root root 4096 мая 13 21:14 ../
-rw-r--r--  1 root root  870 мая 13 21:18 data.txt
ifebres@ifebres-nb:~/github/kuber-homeworks/2.2/src$ kubectl delete pv local-pv 
persistentvolume "local-pv" deleted
ifebres@ifebres-nb:/var/local-pv$ ll
итого 12
drwxr-xr-x  2 root root 4096 мая 13 21:16 ./
drwxr-xr-x 16 root root 4096 мая 13 21:14 ../
-rw-r--r--  1 root root  870 мая 13 21:18 data.txt
```
  После удаления PV с помощью команды kubectl delete pv local-pv, файл success.txt остается на локальном диске ноды. Это происходит потому, что PV в Kubernetes управляет только ресурсами хранилища в кластере, но не управляет данными, которые могут храниться внутри него. Удаление PV просто освобождает ресурсы, занимаемые PV в кластере, но не затрагивает данные, которые были сохранены на узле хоста.  

------

### Задание 2

**Что нужно сделать**

Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

1. Включить и настроить NFS-сервер на MicroK8S.
```commandline
ifebres@ifebres-nb:~/.kube$ kubectl get sc
NAME   PROVISIONER                            RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs    cluster.local/nfs-server-provisioner   Delete          Immediate           true                   24h
```
2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS. [deployment_nfs.yaml](src%2Fdeployment_nfs.yaml), [nfs-pvc.yaml](src%2Fnfs-pvc.yaml)
3. Продемонстрировать возможность чтения и записи файла изнутри пода. 
```commandline
ifebres@ifebres-nb:~/github/kuber-homeworks/2.2/src$ kubectl get pod
NAME                       READY   STATUS    RESTARTS   AGE
nfs-app-657d7fbfc5-9jkkb   1/1     Running   0          10s
ifebres@ifebres-nb:~/github/kuber-homeworks/2.2/src$ kubectl exec -it nfs-app-657d7fbfc5-9jkkb -c multitool -- /bin/sh
/ # cd shared-data/
/shared-data # ls
/shared-data # date > simple.txt
/shared-data # cat simple.txt 
Tue May 14 19:53:46 UTC 2024
```


