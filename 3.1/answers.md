# Домашнее задание к занятию «Компоненты Kubernetes»


### Задание. Необходимо определить требуемые ресурсы
Известно, что проекту нужны база данных, система кеширования, а само приложение состоит из бекенда и фронтенда. Опишите, какие ресурсы нужны, если известно:

1. Необходимо упаковать приложение в чарт для деплоя в разные окружения. 
2. База данных должна быть отказоустойчивой. Потребляет 4 ГБ ОЗУ в работе, 1 ядро. 3 копии. 
3. Кеш должен быть отказоустойчивый. Потребляет 4 ГБ ОЗУ в работе, 1 ядро. 3 копии. 
4. Фронтенд обрабатывает внешние запросы быстро, отдавая статику. Потребляет не более 50 МБ ОЗУ на каждый экземпляр, 0.2 ядра. 5 копий. 
5. Бекенд потребляет 600 МБ ОЗУ и по 1 ядру на копию. 10 копий.

----

Минимальные ресурсы для нод кластера:

|            |RAM  |HDD   |CPU   |
|------------|-----|------|------|
|Control Node|2 Гб |50 Гб |2 ядра|
|Worker Node |1 Гб |100 Гб|1 ядро|

Требуемые ресурсы для проекта

|            |RAM    |CPU     |Replicas|Total RAM|Total CPU|
|------------|-------|--------|--------|---------|---------|
|БД          |4 Гб   |1 ядро  |3       |12 Гб    |3 ядра   |
|Кэш         |4 Гб   |1 ядро  |3       |12 Гб    |3 ядра   |
|Фронтенд    |0,05 Гб|0,2 ядра|5       |0,25 Гб  |1 ядро   |
|Бэкенд      |0,6 Гб |1 ядро  |10      |6 Гб     |10 ядер  |


Расчёт:  
Суммарно для организации кластера требуется:

 - 17 ядер CPU
 - 30,25 Гб RAM
 
Для обеспечения отказоустойчивости необходимо использовать не менее 3 Worker Node и 3 Control Node (для миграции вышедшей из строя ноды на оставшиеся 2 рабочие ноды)

Количество ресурсов для Worker Node:

 - 17 ядер CPU (для проекта) + 3 ядра CPU (для Worker Node) = 20 ядер CPU 
 - 30,25 Гб RAM (для проекта) + 3 Гб RAM (для Worker Node)  = 33,25 Гб RAM 
 
Расчёт одной Worker Node ноды (всего потребуется три ноды):

 - CPU: 20 / 3 = 7 CPU
 - RAM: 34 / 3 = 12 RAM  

Резерв ресурсов в случае выхода из строя одной Worker Node:

 - CPU: 7 / 2 = 4 CPU 
 - RAM: 12 / 2 = 6 RAM 

Для обеспечения отказоустойчивости необходимы ресурсы 

для одной Worker Node:

 - CPU: 7 CPU + 4 CPU = 11 CPU
 - RAM: 12 RAM + 6 RAM = 18 RAM

для одной Control Node
 
 - CPU: 2 CPU + 1 CPU (для резерва под миграцию) = 3 CPU
 - RAM: 2 RAM + 1 RAM (для резерва под миграцию) = 3 RAM


Итого:
 - Потребуется 3 рабоче ноды(Worker Node), каждая по 11 CPU, 18 RAM
 - Потребуется 3 управляющие ноды(Control Node), каждая по 3 CPU, 3 RAM