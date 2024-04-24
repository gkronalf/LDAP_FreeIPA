# LDAP на основе FreeIPA

### Цель задания
Настраивать LDAP-сервер и организовать подключение к нему LDAP-клиентов

### Описание задания
  
1) Установить FreeIPA  
2) Написать Ansible-playbook для конфигурации клиента  
  
Дополнительное задание  
3) Настроить аутентификацию по SSH-ключам  
4) Firewall должен быть включен на сервере и на клиенте  

### Введение
LDAP (Lightweight Directory Access Protocol — легковесный протокол доступа к каталогам) —  это протокол для хранения и получения данных из каталога с иерархической структурой.
LDAP не является протоколом аутентификации или авторизации 

С увеличением числа серверов затрудняется управление пользователями на этих сервере. LDAP решает задачу централизованного управления доступом. 
С помощью LDAP можно синхронизировать:
- UID пользователей
- Группы (GID)
- Домашние каталоги
- Общие настройки для хостов 
- И т. д. 

LDAP работает на следующих портах: 
- 389/TCP — без TLS/SSL
- 636/TCP — с TLS/SSL

Основные компоненты LDAP  
<img src="images/LDAP_shema.jpeg" width=450 alt="LDAP_shema">

Атрибуты — пара «ключ-значение». Пример атрибута: mail: admin@example.com
Записи (entry) — набор атрибутов под именем, используемый для описания чего-либо

Пример записи:
dn: sn=Ivanov, ou=people, dc=digitalocean,dc=com
objectclass: person
sn: Ivanov
cn: Ivan Ivanov

Data Information Tree (DIT) — организационная структура, где каждая запись имеет ровно одну родительскую запись и под ней может находиться любое количество дочерних записей. Запись верхнего уровня — исключение
На основе LDAP построенно много решений, например: Microsoft Active Directory, OpenLDAP, FreeIPA и т. д.

В данной лабораторной работе будет рассмотрена установка и настройка FreeIPA. FreeIPA — это готовое решение, включающее в себе:
Сервер LDAP на базе Novell 389 DS c предустановленными схемами
Сервер Kerberos
Предустановленный BIND с хранилищем зон в LDAP
Web-консоль управления
  
### Установка и первоначальна настройка FreeIPA на сервере
  
После развертывания стенда, необходимо запустить playbook, который подготовить виртуальные машины (ВМ) сервера IPA и клиентов. Сделать это можно командой ```ansible-playbook ./ansible/playbook.yaml -i ./ansible/hosts.ini```  
  
При выполении playbook осуществляется:  
- выставление необходимой временной зоны,
- перезапускает служба синхронизации времени,
- копирование преднастроенного файла hosts,
- остановка службы firewalld,
- отключение SELinux
Эти подготовительные настройки производятся на всех ВМ стенда.  
  
Дополнительно playbook выполняет установку FreeIPA на сервер и на клиентские ВМ.  
  
По завершению работы playbook необходимо зайти на сервер, с помощью команды ```vagrant ssh ipa.otus.lan``` и выполить запуск скрипта, который завершит настройку сервера. Для этого, в консоле сервера, необходимо ввести команду ```ipa-server-install```  
  
Далее, нам потребуется указать параметры нашего LDAP-сервера, после ввода каждого параметра нажимаем Enter, если нас устраивает параметр, указанный в квадратных скобках, то можно сразу нажимать Enter:  
  
```
Do you want to configure integrated DNS (BIND)? [no]: no  
Server host name [ipa.otus.lan]: <Нажимем Enter>  
Please confirm the domain name [otus.lan]: <Нажимем Enter>  
Please provide a realm name [OTUS.LAN]: <Нажимем Enter>  
Directory Manager password: <Указываем пароль минимум 8 символов>  
Password (confirm): <Дублируем указанный пароль>  
IPA admin password: <Указываем пароль минимум 8 символов>  
Password (confirm): <Дублируем указанный пароль>  
NetBIOS domain name [OTUS]: <Нажимем Enter>  
Do you want to configure chrony with NTP server or pool address? [no]: no  
The IPA Master Server will be configured with:  
Hostname: ipa.otus.lan  
IP address(es): 192.168.57.10  
Domain name: otus.lan  
Realm name: OTUS.LAN  
  
The CA will be configured with:  
Subject DN:   CN=Certificate Authority,O=OTUS.LAN  
Subject base: O=OTUS.LAN  
Chaining:     self-signed  
Проверяем параметры, если всё устраивает, то нажимаем yes  
Continue to configure the system with these values? [no]: yes  
```
  
После чего начнётся процесс установки. Процесс установки занимает примерно 10-15 минут (иногда время может быть другим). Если мастер успешно выполнит настройку FreeIPA то в конце мы получим сообщение:  
```The ipa-server-install command was successful```  
  
При вводе параметров установки мы вводили 2 пароля:  
- Directory Manager password — это пароль администратора сервера каталогов, У этого пользователя есть полный доступ к каталогу.  
- IPA admin password — пароль от пользователя FreeIPA admin  
  
После успешной установки FreeIPA, проверим, что сервер Kerberos может выдать нам билет: 
```kinit admin```  
```Password for admin@OTUS.LAN:``` - #Указываем Directory Manager password  
```klist``` - #Запросим список билетов Kerberos  
```
Ticket cache: KCM:0  
Default principal: admin@OTUS.LAN  
  
Valid starting Expires Service principal  
08/02/22 18:18:25 08/03/22 17:32:39 krbtgt/OTUS.LAN@OTUS.LAN  
```  
На этом настройка сервера FreeIPA завершена и можно переходить к настройки клиентов.  
  
### Настройка FreeIPA на клиентах
  
Выполнить автоматизированную настройку всех клиентских ВМ можно запуском playbook ```ansible-playbook ./ansible/client_playbook.yaml -i ./ansible/hosts.ini```  
  
Или войдя в консоль клиентской ВМ, используя команды:  
- ```vagrant ssh client.otus.lan``` - Вход в консоль клиента;  
- ```ipa-client-install``` - Запуск матера настройки ipa-client.  
  
Как альтернатива можно воспользоваться командой ```echo -e "yes\nyes" | ipa-client-install --mkhomedir --domain=OTUS.LAN --server=ipa.otus.lan --no-ntp -p admin -w Otus2024```, команда позволяет нам сразу задать требуемые нам параметры:  
- ```--domain``` — имя домена;  
- ```--server``` — имя FreeIPA-сервера;  
- ```--no-ntp``` — не настраивать дополнительно ntp (мы уже настроили chrony);  
- ```-p``` — имя админа домена;  
- ```-w``` — пароль администратора домена (IPA password);  
- ```--mkhomedir``` — создать директории пользователей при их первом логине.  
  
После подключения хостов к FreeIPA-сервер нужно проверить, что мы можем получить билет от Kerberos сервера: ```kinit admin```  
Если подключение выполнено правильно, то мы сможем получить билет, после ввода пароля.  
  
Давайте проверим работу LDAP, для этого на сервере FreeIPA создадим пользователя и попробуем залогиниться к клиенту:
- Авторизируемся на сервере: ```kinit admin```  
- Создадим пользователя otus-user  
```
[root@ipa ~]# ipa user-add otus-user --first=Otus --last=User --password  
Password: #Вводим пароль пользователя otus-user  
Enter Password again to verify: #повторно вводим пароль пользователя otus-user  
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
  User password expiration: 20220802164239Z  
  Email address: otus-user@otus.lan  
  UID: 587200003  
  GID: 587200003  
  Password: True  
  Member of groups: ipausers  
  Kerberos keys available: True  
[root@ipa ~]#   
```  
   
На хосте client1 или client2 выполним команду ```kinit otus-user```  
```
[root@client1 ~]# kinit otus-user  
Password for otus-user@OTUS.LAN:  
Password expired. You must change it now.  
Enter new password:   
Enter it again:   
[root@client1 ~]#  
```
Система запросит у нас пароль и попросит ввести новый пароль.   
  
На этом процесс добавления хостов к FreeIPA-серверу завершен.  
  





