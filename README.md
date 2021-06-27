## Установка стека LEMP и Wordpress на CentOS 7 с помощью Ansible

### Первоначальная настройка сервера

Добавить пользователя

`adduser sb-user`

Назначить пользователю пароль

`passwd sb-user`

Добавить пользователя в группу wheel

`gpasswd -a sb-user wheel`

Войти в систему под новым пользователем

`su - sb-user`

Создать каталог для ключей ssh

`mkdir .ssh && chmod 700 .ssh`

Добавить публичный ключ пользователя

`vi .ssh/authorized_keys`

Назначить права на файл

`chmod 600 .ssh/authorized_keys`

Bнести изменения в конфигурационный файл sshd_config

`sudo vi /etc/ssh/sshd_config`

Запретить доступ пользователю root через ssh

`PermitRootLogin no`

Перезапустить сервис sshd

`sudo systemctl reload sshd`

Настроить фаервол

`sudo yum install firewalld`

`sudo systemctl start firewalld`

Добавить необходимые сервисы

```
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
```

Для добавления дополнительных портов:

`sudo firewall-cmd --permanent --add-port=4444/tcp`

Посмотреть список доступных сервисов:

`sudo firewall-cmd --get-services`

Посмотреть текущие настройки:

`sudo firewall-cmd --permanent --list-all`

Перечитать конфигурацию:

`sudo firewall-cmd --reload`

Включить фаервол при запуске системы

`sudo systemctl enable firewalld`

Установить и включить NTP сервис

```
sudo yum install ntp
sudo systemctl start ntpd
sudo systemctl enable ntpd
```

### Устанвка стека LEMP и Wordpress

Склонировать репозиторий командой:

`git clone https://github.com/sanchos2/sb-hw1.git && cd sb-hw1`

Переименовать файл ansible.cfg.example в ansible.cfg
Необходимо изменить настройки пользователя и путь к закрытому ключу:

```shell
remote_user = sb-user
private_key_file = ~/.ssh/sb-user
```

Переименовать файл inventory/inventory.yaml.example в inventory.yaml
Необходимо изменить адрес хоста, порт и название сервера:
```shell
web-server:
      ansible_host: test.example.com
      ansible_port: 22
```

В папке playbooks создать файл wp_install.yml со следующим содержанием:

```shell
---
- name: Configure web server with LEMP stack and install Wordpress
  hosts: all
  become: true
  gather_facts: true
  vars:
    mysql_root_password: "Pa$$w0rd" # Здесь указываем пароль для пользователя root MariaDB
    server_name: test.example.com # Указываем DNS имя хоста
    certbot_admin_email: admin@example.com # e-mail администратора сайта
    certbot_certs:
      - domains: # Ниже прописываем DNS имена для сертификата Let's Encrypt
          - test.example.com
          - www.test.example.com 
    wp_db_name: "wordpress" # Имя базы данных для Wordpress
    wp_db_user: "wp_admin" # Имя пользователя для базы данных Wordpress
    wp_db_password: "Pa$$w0rd" # Пароль пользователя для базы данных Wordpress
    wp_db_read_user: "wp_read" # Имя пользователя для базы данных Wordpress c правами на чтение
    wp_db_read_user_password: "Pa$$word" # Пароль пользователя для базы данных Wordpress c правами на чтение

  roles: # Запускаемые роли. Для установки только LEMP стека можно использовать db, memcached, nginx, php
    - db # Установка MariaDB
    - memcached # Установка memcached
    - nginx # Установка nginx
    - geerlingguy.certbot # Запрос сертификатов Let's Encrypt
    - https # Конфигурирование Nginx для использования протокола https
    - php # Установка php-fpm
    - wp # Установка и конфигурирование Wordpress
```

Запустить установку можно командой:

`ansible-playbook playbooks/wp_install.yml -K`
