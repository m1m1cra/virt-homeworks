 
# Домашнее задание к занятию "5.2. Применение принципов IaaC в работе с виртуальными машинами"

## Задача 1

- Опишите своими словами основные преимущества применения на практике IaaC паттернов.
- Какой из принципов IaaC является основополагающим?

#### Ответ
* Основные преимущества IaaC паттернов - процесс создания ПО, его сборки, тестирования и доставки до продуктовой среды является полностью прозрачным, предсказуемам и автоматизированным. 

* Основополагающим является принцип идемпотентности, т.е получения предсказуемого и одного и того же результата как раньше, так и в будующем.

## Задача 2
- Чем Ansible выгодно отличается от других систем управление конфигурациями?
- Какой, на ваш взгляд, метод работы систем конфигурации более надёжный push или pull?
#### Ответ
* Ansible выгодно отличается тем, что использует в работе tcp/22 ssh, что является нативным для linux-систем и является agentless.
* На мой взгляд более надежной моделью распространения является pull, так как хост может быть офлайн при пуше конфига, а в случае pull-модели - хост максимально оперативно при доступности сети заберет конфиг.

## Задача 3

Установить на личный компьютер:

- VirtualBox
- Vagrant
- Ansible

#### Ответ
* 
```
C:\Program Files\Oracle\VirtualBox>VBoxManage -version
6.1.34r150636
```

* 
```
C:\vagrantcfg> vagrant --version
Vagrant 2.2.19
PS C:\vagrantcfg>
```

* 
```
vagrant@vagrant:~$ ansible --version
[WARNING]: You are running the development version of Ansible. You should only run Ansible from "devel" if you are
modifying the Ansible engine, or trying out features under development. This is a rapidly changing source of code and
can become unstable at any point.
ansible [core 2.14.0.dev0]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/vagrant/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/vagrant/.local/lib/python3.10/site-packages/ansible
  ansible collection location = /home/vagrant/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.10.6 (main, Aug  2 2022, 15:11:28) [GCC 9.4.0] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = False
```

*Приложить вывод команд установленных версий каждой из программ, оформленный в markdown.*

## Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

- Создать виртуальную машину.
- Зайти внутрь ВМ, убедиться, что Docker установлен с помощью команды
```
docker ps
```
