 
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

#### Ответ

```
root@bhdevops:/home/avdeevan/vagrant# vagrant up
Bringing machine 'server1.netology' up with 'virtualbox' provider...
==> server1.netology: Checking if box 'bento/ubuntu-20.04' version '202206.03.0'                                                                                                                                                                                                                                              is up to date...
==> server1.netology: Clearing any previously set forwarded ports...
==> server1.netology: Clearing any previously set network interfaces...
==> server1.netology: Preparing network interfaces based on configuration...
    server1.netology: Adapter 1: nat
    server1.netology: Adapter 2: hostonly
==> server1.netology: Forwarding ports...
    server1.netology: 22 (guest) => 20011 (host) (adapter 1)
    server1.netology: 22 (guest) => 2222 (host) (adapter 1)
==> server1.netology: Running 'pre-boot' VM customizations...
==> server1.netology: Booting VM...
==> server1.netology: Waiting for machine to boot. This may take a few minutes..                                                                                                                                                                                                                                             .
    server1.netology: SSH address: 127.0.0.1:2222
    server1.netology: SSH username: vagrant
    server1.netology: SSH auth method: private key
    server1.netology: Warning: Connection reset. Retrying...
    server1.netology: Warning: Remote connection disconnect. Retrying...
    server1.netology: Warning: Connection reset. Retrying...
    server1.netology:
    server1.netology: Vagrant insecure key detected. Vagrant will automatically                                                                                                                                                                                                                                              replace
    server1.netology: this with a newly generated keypair for better security.
    server1.netology:
    server1.netology: Inserting generated public key within guest...
    server1.netology: Removing insecure key from the guest if it's present...
    server1.netology: Key inserted! Disconnecting and reconnecting using new SSH                                                                                                                                                                                                                                              key...
==> server1.netology: Machine booted and ready!
==> server1.netology: Checking for guest additions in VM...
==> server1.netology: Setting hostname...
==> server1.netology: Configuring and enabling network interfaces...
==> server1.netology: Mounting shared folders...
    server1.netology: /vagrant => /home/avdeevan/vagrant
==> server1.netology: Running provisioner: ansible...
    server1.netology: Running ansible-playbook...

PLAY [nodes] *******************************************************************

TASK [Gathering Facts] *********************************************************
ok: [server1.netology]

TASK [Create directory for ssh-keys] *******************************************
ok: [server1.netology]

TASK [Adding rsa-key in /root/.ssh/authorized_keys] ****************************
An exception occurred during task execution. To see the full traceback, use -vvv                                                                                                                                                                                                                                             . The error was: If you are using a module and expect the file to exist on the r                                                                                                                                                                                                                                             emote, see the remote_src option
fatal: [server1.netology]: FAILED! => {"changed": false, "msg": "Could not find                                                                                                                                                                                                                                              or access '~/.ssh/id_rsa.pub' on the Ansible Controller.\nIf you are using a mod                                                                                                                                                                                                                                             ule and expect the file to exist on the remote, see the remote_src option"}
...ignoring

TASK [Checking DNS] ************************************************************
changed: [server1.netology]

TASK [Installing tools] ********************************************************
ok: [server1.netology] => (item=['git', 'curl'])

TASK [Installing docker] *******************************************************
changed: [server1.netology]

TASK [Add the current user to docker group] ************************************
changed: [server1.netology]

PLAY RECAP *********************************************************************
server1.netology           : ok=7    changed=3    unreachable=0    failed=0    s                                                                                                                                                                                                                                             kipped=0    rescued=0    ignored=1

root@bhdevops:/home/avdeevan/vagrant# vagrant ssh
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-110-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 System information disabled due to load higher than 1.0


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento
Last login: Sun Aug 28 17:16:21 2022 from 10.0.2.2
vagrant@server1:~$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
vagrant@server1:~$

```
