# OTUS.Lesson38.LDAP
Тестовый стенд: 
3 VM (2COU, 2Gb Ram, 8Gb HDD, OS Centos8)

Описание домашнего задания
1) Установить FreeIPA
2) Написать Ansible-playbook для конфигурации клиента

Выполнение домашнего задания:
1.  Установить FreeIPA
Фиксим репозиторий yum:
```
sed -i s/mirror.centos.org/vault.centos.org/g /etc/yum.repos.d/*.repo 
sed -i s/^#.*baseurl=http/baseurl=http/g /etc/yum.repos.d/*.repo 
sed -i s/^mirrorlist=http/#mirrorlist=http/g /etc/yum.repos.d/*.repo 
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-* 
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-* 
yum clean all
```
Установим часовой пояс: timedatectl set-timezone Europe/Moscow
Установим утилиту chrony: yum install -y chrony
Запустим chrony и добавим его в автозагрузку: systemctl enable chronyd
Поменяем имя нашего сервера: hostnamectl set-hostname ipa.otus.lan
Выключим Firewall: systemctl stop firewalld
Отключаем автозапуск Firewalld: systemctl disable firewalld
Остановим Selinux: setenforce 0
Поменяем в файле /etc/selinux/config, параметр Selinux на disabled
Добавим запись '10.200.3.221 ipa.otus.lan ipa' в файл /etc/hosts
Установим модуль DL1: yum install -y @idm:DL1
Установим FreeIPA-сервер: yum install -y ipa-server
Запустим скрипт установки: ipa-server-install
Далее, нам потребуется указать параметры нашего LDAP-сервера, после ввода каждого параметра нажимаем Enter, если нас устраивает параметр, указанный в квадратных скобках, то можно сразу нажимать Enter:
Do you want to configure integrated DNS (BIND)? [no]: no
Server host name [ipa.otus.lan]: <Нажимаем Enter>
Please confirm the domain name [otus.lan]: <Нажимем Enter>
Please provide a realm name [OTUS.LAN]: <Нажимаем Enter>
Directory Manager password: otuspassword
Password (confirm): otuspassword
IPA admin password: otuspassword
Password (confirm): otuspassword
NetBIOS domain name [OTUS]: <Нажимаем Enter>
Do you want to configure chrony with NTP server or pool address? [no]: no
The IPA Master Server will be configured with:
Hostname:       ipa.otus.lan
IP address(es): 10.200.3.221
Domain name:    otus.lan
Realm name:     OTUS.LAN

The CA will be configured with:
Subject DN:   CN=Certificate Authority,O=OTUS.LAN
Subject base: O=OTUS.LAN
Chaining:     self-signed
Continue to configure the system with these values? [no]: yes
После успешной установки FreeIPA, проверим, что сервер Kerberos может выдать нам билет: 
[root@IPA ~]# kinit admin
Password for admin@OTUS.LAN: 
[root@IPA ~]# klist 
Ticket cache: KCM:0
Default principal: admin@OTUS.LAN

Valid starting       Expires              Service principal
04/24/2025 14:42:27  04/25/2025 13:46:43  krbtgt/OTUS.LAN@OTUS.LAN
[root@IPA ~]# klist 
Ticket cache: KCM:0
Default principal: admin@OTUS.LAN

Valid starting       Expires              Service principal
04/24/2025 14:42:27  04/25/2025 13:46:43  krbtgt/OTUS.LAN@OTUS.LAN

Зайдем на Web-интерфейс FreeIPA-сервера:
тут скрин

2. Запустим плейбук provision.yml
Дожидаемся выполнения плейбука.

Создадим пользователя otus-user
на IPA сервере: 
```
[root@IPA ~]# kinit admin
Password for admin@OTUS.LAN: 
[root@IPA ~]# ipa user-add otus-user --first=Otus --last=User --password

Password: otus2025
Enter Password again to verify: otus2025
----------------------
Added user "otus-user"
----------------------
  User login: otus-user
  First name: Otus
  Last name: User
  Full name: Otus User
  Display name: Otus User
  Initials: OU
  Home directory: /home/otus-user
  GECOS: Otus User
  Login shell: /bin/sh
  Principal name: otus-user@OTUS.LAN
  Principal alias: otus-user@OTUS.LAN
  User password expiration: 20250424123202Z
  Email address: otus-user@otus.lan
  UID: 1434000003
  GID: 1434000003
  Password: True
  Member of groups: ipausers
  Kerberos keys available: True
```
На client1 выполним: kinit otus-user
```
[root@client1 ~]# kinit otus-user
Password for otus-user@OTUS.LAN: otus2025
Password expired.  You must change it now.
Enter new password: otus2025
Enter it again: otus2025

попробуем подключиться к client1 по ssh 
```
ssh otus-user@10.200.3.222


Connecting to 10.200.3.222:22...
Connection established.
To escape to local shell, press Ctrl+Alt+[.

WARNING! The remote SSH server rejected X11 forwarding request.
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Thu Apr 24 15:40:35 2025 from 10.1.1.21
[otus-user@client1 ~]$ 
```