## Задача 1

Архитектор ПО решил проконсультироваться у вас, какой тип БД 
лучше выбрать для хранения определенных данных.

Он вам предоставил следующие типы сущностей, которые нужно будет хранить в БД:

- Электронные чеки в json виде
- Склады и автомобильные дороги для логистической компании
- Генеалогические деревья
- Кэш идентификаторов клиентов с ограниченным временем жизни для движка аутенфикации
- Отношения клиент-покупка для интернет-магазина

Выберите подходящие типы СУБД для каждой сущности и объясните свой выбор.

#### Ответ
- Подойдет БД ключ-значение  или документоориентированная БД
- Подойдет или сетевая БД или документоориентированная
- Подойдет иерархическая БД
- Подойдет БД ключ-значение 


## Задача 2

Вы создали распределенное высоконагруженное приложение и хотите классифицировать его согласно 
CAP-теореме. Какой классификации по CAP-теореме соответствует ваша система, если 
(каждый пункт - это отдельная реализация вашей системы и для каждого пункта надо привести классификацию):

- Данные записываются на все узлы с задержкой до часа (асинхронная запись)
- При сетевых сбоях, система может разделиться на 2 раздельных кластера
- Система может не прислать корректный ответ или сбросить соединение

А согласно PACELC-теореме, как бы вы классифицировали данные реализации?

#### Ответ:
Согласно CAP-теореме:
- AP
- CA
- CP

Согласно PACELC-теореме:
- PA/EL
- PC/EC
- PC/EC

## Задача 3

Могут ли в одной системе сочетаться принципы BASE и ACID? Почему?

#### Ответ:
Не могут, п ACID- это про high availability, BASE - про hi-load

## Задача 4

Вам дали задачу написать системное решение, основой которого бы послужили:

- фиксация некоторых значений с временем жизни
- реакция на истечение таймаута

Вы слышали о key-value хранилище, которое имеет механизм [Pub/Sub](https://habr.com/ru/post/278237/). 
Что это за система? Какие минусы выбора данной системы?

#### Ответ
Взял бы БД "ключ-значение", например - Redis, в ней удобно хранить TTL а также делать триггер на таймаут хранения значения.
PUB-SUB - удобная система, реализующая процип брокера сообщений, что у всех сообщений есть издатель и подписчик.
Минусы - возможно, в сложном обслуживании и настройке, требовании к ресурсам (RAM), отсутствие SQL

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
