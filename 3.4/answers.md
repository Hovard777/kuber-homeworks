# Домашнее задание к занятию «Обновление приложений»


### Задание 1. Выбрать стратегию обновления приложения и описать ваш выбор

1. Имеется приложение, состоящее из нескольких реплик, которое требуется обновить.
2. Ресурсы, выделенные для приложения, ограничены, и нет возможности их увеличить.
3. Запас по ресурсам в менее загруженный момент времени составляет 20%.
4. Обновление мажорное, новые версии приложения не умеют работать со старыми.
5. Вам нужно объяснить свой выбор стратегии обновления приложения.

### Задание 2. Обновить приложение

1. Создать deployment приложения с контейнерами nginx и multitool. Версию nginx взять 1.19. Количество реплик — 5.
2. Обновить версию nginx в приложении до версии 1.20, сократив время обновления до минимума. Приложение должно быть доступно.
3. Попытаться обновить nginx до версии 1.28, приложение должно оставаться доступным.
4. Откатиться после неудачного обновления.

### Решение 1.

 - Если приложение уже протестировано ранее, правильным вариантом будет использовать стратегию обновления Rolling Update, с указанием параметров maxSurge maxUnavailable для избежания ситуации с нехваткой ресурсов. Проводить обновление следует в менее загруженный момент времени сервиса. k8s постепенно заменит все поды без ущерба производительности. В случае непредвиденных ошибок, можно будет быстро откатится к предыдущему состоянию.
 - Также можно использовать Canary Strategy, указав параметры maxSurge maxUnavailable чтобы избежать нехватки ресурсов. Это позволит протестировать новую версию программы на реальной пользовательской базе(группа может выделяться по определенному признаку) без полного развертывания. После тестирования и собора метрик пользователей можно постепенно переводить поды к новой версии приложения.

### Решение 2.

1. deployment приложения с контейнерами nginx и multitool. Версия nginx 1.19. Количество реплик — 5. [deployment.yaml](src%2Fdeployment.yaml)
2. Обновляем версию nginx в приложении до версии 1.20, сократив время обновления до минимума. Приложение должно быть доступно.
```commandline
ifebres@ifebres-nb:~/github/kuber-homeworks/3.4/src$ kubectl apply -f service.yaml 
service/update-svc created
ifebres@ifebres-nb:~/github/kuber-homeworks/3.4/src$ kubectl get pods
NAME                                 READY   STATUS              RESTARTS   AGE
update-deployment-68785c58d9-4p5jc   0/2     ContainerCreating   0          14s
update-deployment-68785c58d9-7msrf   0/2     ContainerCreating   0          14s
update-deployment-68785c58d9-bjn8c   2/2     Running             0          14s
update-deployment-68785c58d9-dcnxw   0/2     ContainerCreating   0          14s
update-deployment-68785c58d9-sjkdh   0/2     ContainerCreating   0          14s
ifebres@ifebres-nb:~/github/kuber-homeworks/3.4/src$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
update-deployment-68785c58d9-4p5jc   2/2     Running   0          21s
update-deployment-68785c58d9-7msrf   2/2     Running   0          21s
update-deployment-68785c58d9-bjn8c   2/2     Running   0          21s
update-deployment-68785c58d9-dcnxw   2/2     Running   0          21s
update-deployment-68785c58d9-sjkdh   2/2     Running   0          21s
```

Обновляем: меняем версию и добавляем RollingUpdate
```yaml
strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

```commandline
ifebres@ifebres-nb:~/github/kuber-homeworks/3.4/src$ kubectl apply -f deployment.yaml 
deployment.apps/update-deployment configured
ifebres@ifebres-nb:~/github/kuber-homeworks/3.4/src$ kubectl get pods
NAME                                 READY   STATUS              RESTARTS   AGE
update-deployment-68785c58d9-7msrf   2/2     Running             0          3m21s
update-deployment-68785c58d9-bjn8c   2/2     Running             0          3m21s
update-deployment-68785c58d9-dcnxw   2/2     Running             0          3m21s
update-deployment-68785c58d9-sjkdh   2/2     Running             0          3m21s
update-deployment-757656d46f-5xl4z   0/2     ContainerCreating   0          11s
update-deployment-757656d46f-mjk5c   0/2     ContainerCreating   0          11s
ifebres@ifebres-nb:~/github/kuber-homeworks/3.4/src$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
update-deployment-757656d46f-45st5   2/2     Running   0          10s
update-deployment-757656d46f-5xl4z   2/2     Running   0          25s
update-deployment-757656d46f-m8869   2/2     Running   0          5s
update-deployment-757656d46f-mjk5c   2/2     Running   0          25s
update-deployment-757656d46f-tlh8w   2/2     Running   0          10s
```

Пробуем обновить до 1.28:

```commandline
ifebres@ifebres-nb:~/github/kuber-homeworks/3.4/src$ kubectl apply -f deployment.yaml 
deployment.apps/update-deployment configured
ifebres@ifebres-nb:~/github/kuber-homeworks/3.4/src$ kubectl get pods
NAME                                 READY   STATUS              RESTARTS   AGE
update-deployment-6d445f45b9-4mzdc   0/2     ContainerCreating   0          4s
update-deployment-6d445f45b9-zqpq9   0/2     ContainerCreating   0          4s
update-deployment-757656d46f-45st5   2/2     Running             0          109s
update-deployment-757656d46f-5xl4z   2/2     Running             0          2m4s
update-deployment-757656d46f-mjk5c   2/2     Running             0          2m4s
update-deployment-757656d46f-tlh8w   2/2     Running             0          109s
ifebres@ifebres-nb:~/github/kuber-homeworks/3.4/src$ 
```

Получаем ошибки запуска некоторых pod'ов, Kubernetes не может получить образ для некоторых контейнеров pod'а
```commandline
ifebres@ifebres-nb:~/github/kuber-homeworks/3.4/src$ kubectl get pods
NAME                                 READY   STATUS             RESTARTS   AGE
update-deployment-6d445f45b9-4mzdc   1/2     ImagePullBackOff   0          17s
update-deployment-6d445f45b9-zqpq9   1/2     ImagePullBackOff   0          17s
update-deployment-757656d46f-45st5   2/2     Running            0          2m2s
update-deployment-757656d46f-5xl4z   2/2     Running            0          2m17s
update-deployment-757656d46f-mjk5c   2/2     Running            0          2m17s
update-deployment-757656d46f-tlh8w   2/2     Running            0          2m2s
```

Откатываемся

```commandline
ifebres@ifebres-nb:~/github/kuber-homeworks/3.4/src$ kubectl apply -f deployment.yaml 
deployment.apps/update-deployment configured
ifebres@ifebres-nb:~/github/kuber-homeworks/3.4/src$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
update-deployment-757656d46f-45st5   2/2     Running   0          7m18s
update-deployment-757656d46f-5xl4z   2/2     Running   0          7m33s
update-deployment-757656d46f-mjk5c   2/2     Running   0          7m33s
update-deployment-757656d46f-ppjb4   2/2     Running   0          4s
update-deployment-757656d46f-tlh8w   2/2     Running   0          7m18s
```


