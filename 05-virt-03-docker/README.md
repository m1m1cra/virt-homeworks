
# Домашнее задание к занятию "5.3. Введение. Экосистема. Архитектура. Жизненный цикл Docker контейнера"


## Задача 1

Сценарий выполения задачи:

- создайте свой репозиторий на https://hub.docker.com;
- выберете любой образ, который содержит веб-сервер Nginx;
- создайте свой fork образа;
- реализуйте функциональность:
запуск веб-сервера в фоне с индекс-страницей, содержащей HTML-код ниже:
```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
```
Опубликуйте созданный форк в своем репозитории и предоставьте ответ в виде ссылки на https://hub.docker.com/username_repo.

#### Ответ
https://hub.docker.com/repository/docker/m1cra/netology#


## Задача 2

Посмотрите на сценарий ниже и ответьте на вопрос:
"Подходит ли в этом сценарии использование Docker контейнеров или лучше подойдет виртуальная машина, физическая машина? Может быть возможны разные варианты?"

Детально опишите и обоснуйте свой выбор.

--

Сценарий:

- Высоконагруженное монолитное java веб-приложение;
- Nodejs веб-приложение;
- Мобильное приложение c версиями для Android и iOS;
- Шина данных на базе Apache Kafka;
- Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;
- Мониторинг-стек на базе Prometheus и Grafana;
- MongoDB, как основное хранилище данных для java-приложения;
- Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry.

#### Ответ:
- Так как приложение hi-load и монолитное, я бы использовал или виртуалку bare-metal (например, ESXI или KVM, или XEN), естественно, с файловер-нодой в режиме load-balancing, но также, вполне подошло бы и решение в докере, например - в кубере, в целях быстрого горизонтального и вертикального  масштабирования (при необходимости) , более эффективно бы использовались мощности
- Nodejs веб - аналогично, запустил бы в докере, даже не обязательно строить оркестрацию при этом
- Мобильное приложение разве лежит не в облаке (в маркете)? Не очень понял) 
- Для Apache Kafka, насколько я ознакомился, используют вполне успешно докер, но можно и поставить на виртуалку bare-metal
- всю данную связку ELK вполне можно собрать на docker-compose
- Такой мониторинг-стек стоит у нас на проде, крутится в докере
- MongoDB - аналогично. Вполне можно вертеть в докере с примапленным стором.
- А вот GItlab сервак и свой Docker registry я бы хранил на 2 машинах bare-metal. 1 Машина = 1 сервис. Эти сервера не критика на мой взгляд, поэтому кластер тут избыточен

## Задача 3

- Запустите первый контейнер из образа ***centos*** c любым тэгом в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
- Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
- Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```;
- Добавьте еще один файл в папку ```/data``` на хостовой машине;
- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.


#### Ответ

```bash
root@bhdevops:/home/avdeevan/data# docker run  -ti -d -v /home/avdeevan/data:/data centos
4c27a6ec02fd078b5cd0850f5205e40609f0b7cfa5f281b774e08161fc2fcb8e
root@bhdevops:/home/avdeevan/data# docker run  -ti -d -v /home/avdeevan/data:/data debian
d5bd86aee5c723a128c7cef3f0a07f3206322e9c745d0a5f7cba166e89bb52d7
root@bhdevops:/home/avdeevan/data# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS         PORTS     NAMES
d5bd86aee5c7   debian    "bash"        10 seconds ago   Up 9 seconds             distracted_montalcini
4c27a6ec02fd   centos    "/bin/bash"   6 minutes ago    Up 6 minutes             inspiring_herschel
root@bhdevops:/home/avdeevan/data# docker exec -ti inspiring_herschel bash
[root@4c27a6ec02fd /]# cd /data/
[root@4c27a6ec02fd data]# touch centos_test
[root@4c27a6ec02fd data]# exit
root@bhdevops:/home/avdeevan/data# ls
centos_test  test
root@bhdevops:/home/avdeevan/data# touch host_test
root@bhdevops:/home/avdeevan/data# docker exec -ti distracted_montalcini bash
root@d5bd86aee5c7:/# cd /data/
root@d5bd86aee5c7:/data# ls
centos_test  host_test  test
````

## Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

Соберите Docker образ с Ansible, загрузите на Docker Hub и пришлите ссылку вместе с остальными ответами к задачам.

#### Ответ
```bash
...............................................
(36/37) Purging libressl3.3-libcrypto (3.3.6-r0)
(37/37) Purging libmagic (5.40-r1)
Executing busybox-1.33.1-r8.trigger
OK: 98 MiB in 69 packages
Removing intermediate container bc04fdcb2b1b
 ---> 0cfb5979f90e
Step 3/5 : RUN  mkdir /ansible &&      mkdir -p /etc/ansible &&      echo 'localhost' > /etc/ansible/hosts
 ---> Running in a1638d16fb2b
Removing intermediate container a1638d16fb2b
 ---> 791b64b62a22
Step 4/5 : WORKDIR /ansible
 ---> Running in d4325e2bfd3a
Removing intermediate container d4325e2bfd3a
 ---> 9fcadfda649a
Step 5/5 : CMD  [ "ansible-playbook", "--version" ]
 ---> Running in 2db1cbb6c249
Removing intermediate container 2db1cbb6c249
 ---> af72680f2c25
Successfully built af72680f2c25
Successfully tagged m1cra/ansible:2.9.24
root@bhdevops:/home/avdeevan/m1cra:ansible/ansible#
root@bhdevops:/home/avdeevan/m1cra:ansible/ansible#
root@bhdevops:/home/avdeevan/m1cra:ansible/ansible# docker push m1cra/ansible:2.9.24
The push refers to repository [docker.io/m1cra/ansible]
2afdbead30d2: Pushed
75452bb44f99: Pushed
63493a9ab2d4: Mounted from library/alpine
2.9.24: digest: sha256:573cdcfb3ea94aca248ce2db982ec23e516f9e6d476e644e24696b0587b57877 size: 947
root@bhdevops:/home/avdeevan/m1cra:ansible/ansible#
root@bhdevops:/home/avdeevan/m1cra:ansible/ansible# docker push m1cra/ansible:2.9.24
The push refers to repository [docker.io/m1cra/ansible]
2afdbead30d2: Layer already exists
75452bb44f99: Layer already exists
63493a9ab2d4: Layer already exists
2.9.24: digest: sha256:573cdcfb3ea94aca248ce2db982ec23e516f9e6d476e644e24696b0587b57877 size: 947
root@bhdevops:/home/avdeevan/m1cra:ansible/ansible#
```

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
