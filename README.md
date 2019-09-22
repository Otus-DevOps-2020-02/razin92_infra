# razin92_infra
Репозиторий для работы над домашними заданиями в рамках курса **"DevOps практики и инструменты"**
**Содрежание:**

1. [ДЗ 3 - Знакомство с облачной инфраструктурой](#hw3)
2. [ДЗ 4 - Деплой тестового приложения ](#hw4)
---
<a name="hw3"></a> 
# Домашнее задание 3
## Знакомство с облачной инфраструктурой
**Необходимые данные для проверки ДЗ**
```
bastion_IP = 35.207.131.174
someinternalhost_IP = 10.156.0.3
```

**Команда для создания ключей SSH**

```
$ ssh-keygen -t rsa -f ~/.ssh/$USERNAME -C $COMMENT -P ""

# -t rsa - тип создаваемого ключа
# -f - путь сохранения ключа
# -С - комментарий
# -P - пароль создаваемого ключа
```
***Приватный ключ***
```
$ ~/.ssh/$USERNAME
```
***Публичный ключ***
```
$ ~/.ssh/$USERNAME.pub
```
Публичный ключ используется для доступа в виртуальные машины всего проекта через SSH. Устанавливается ключ через GCE --> Метаданные --> SSH-Ключи. При желании может быть переназначен.

***Создание виртуальной машины с внешней сетью - Экземпляры ВМ***

**Подключение к виртуальной машине с использованием приватного ключа**
```
$ ssh -i ~/.ssh/$USERNAME $USERNAME@<внешний IP VM>

# -i - путь до ключа
```

***Создание виртуальной машины без доступа к внешенй сети - Экземпляры ВМ***

**Настройка SSH Forwarding**

```
# Проверка текущих настроек
$ ssh-add -L

# -L - список параметров публичных ключей
```
При получении ошибки
> Could not open a connection to your authentication agent.

Следует выполнить следующую команду
```
$ eval "$(ssh-agent)"
# Запуск ssh-agent, который требует ssh-add
```
***Добавление ключа***
```
$ ssh-add ~/.ssh/$USERNAME
```
***Подключение по SSH с включенным SSH Forwarding***
```
ssh -i ~/.ssh/$USERNAME -A $USERNAME@<IP адрес сервера>

# -A - SSH Forwarding
```
### Самостоятельное задание
**Подключение к локальной машине в удаленной приватной сети через bastion-host**
1. Подключение через ***Pseudo-terminal*** ! НЕ рекомендуется
    ```
    $ ssh -i ~/.ssh/$USERNAME $USERNAME@<IP-адрес bastion-host'а> 'ssh <IP-адрес локальной машины>'

    # "консоль" является 'stdin' выполненной команды
    ```
2. Подключение с использованием bastion-host'а как Proxy
    ```
    $ ssh -i ~/.ssh/$USERNAME $USERNAME@<IP-адрес локальной машины> -o "proxycommand ssh -W %h:%p $USERNAME@<IP-адрес bastion-host'а>"

    # -o - использование дополнительных опций SSH
    # -W - проброс stdin/stdout через Proxy
    # %h:%p - Hostname:Port машины $USERNAME@<IP-адрес локальной машины>
    ```
**Подключение к локальной машине в удаленной приватной сети через bastion-host при помощи команды вида ***ssh someinternalhost*** и по алиасу ***someinternalhost*****

Для создания простого подключения по SSH необходимо добавить следующую конфигурацию в файл `~/.ssh/config`
```
Host someinternalhost    # Наименование хоста (используется для вызова)
  Hostname <IP-адрес локальной машины>
  User $USERNAME
  ProxyCommand ssh -W %h:%p <IP-адрес bastion-host'а>   # Использования bastion-host'а как Proxy
  IdentityFile ~/.ssh/$USERNAME    # путь до ключа
```
Для создания алиаса необходимо выполнить следующую команду
```
$ alias <имя алиаса>='команда'

# Пример:
$ alias someinternalhost='ssh someinternalhost'
```

## VPN-сервер для серверов GCP

1. Установка VPN-сервера pritunl и базы mongodb
```
sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list << EOF
deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/4.0 multiverse
EOF

sudo tee /etc/apt/sources.list.d/pritunl.list << EOF
deb http://repo.pritunl.com/stable/apt xenial main
EOF

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv 7568D9BB55FF9E5287D586017AE645C0CF8E292A
sudo apt-get update
sudo apt-get --assume-yes install pritunl mongodb-org
sudo systemctl start pritunl mongod
sudo systemctl enable pritunl mongod
```
2. Запуск настройки через web-интерфейс https://<адрес сервера>/setup

3. Добавление User'а, организации и сервера.

При прикреплении User'a к серверу сгенерируется конфигурация для подключения через OpenVPN, доступная для скачивания.

***Пример конфигурации***
```
#{
# "push_auth": false,
# "sync_secret": "CMNLPCrfMgEFScVtadB3V3ANN1nRz4aC",
# "token_ttl": 172800,
# "organization_id": "5d827c58bbd3b54fa33a629b",
# "user": "test",
# "disable_reconnect": false,
# "sync_token": "mGdwSnoS5XNw2JO3rUEOznwwIKWyW9H0",
# "sync_hash": "6fec97399e8a7d5e2b7925f30b4ed999",
# "server_public_key": [
#  "-----BEGIN RSA PUBLIC KEY-----",
#  "MIICCgKCAgEArUloHbGwM21W60vKFjBf9LoiJvTy5e4z+32rGORiOH2Gl2aIVpdL",
#  "TaK9f4ksSWTBjsP9mLgW+Xrti3aZZ6GT0CFhT00Gs7sA1uTNKx682DWjCM8fDGJU",
#  "6fJ1rDlLYhULaMrWh7shi5OBc7cXODsVrTQkdGM8Be6Q+mAgOJzDjfkokOnlPTMk",
#  "nvkczM1vDjGuTS7DJNvObHJqKpiGgUdK3oZqugbLWYGLCrE9g8terUb93rR5TNsR",
#  "OH6tpCQDSSym6KC7r+s4PBOJ0s+XszMR7TEGdm8v1Ou5ct8ktbzZrKrRbhxru5Vg",
#  "sHVZwfM7fZBla3YxDcZORTm9KKKuAVjVBGVgVDTiW2ruuRsz+HlSNdPmc5eH1kNa",
#  "7ZyriYQop/MlPyNEdV35mzL/k3FHd7U4Y3kX2/EFZc5dMBAN8HSAsbhQNJ0cOJh1",
#  "LQV48j0e/VzU6uYpPhrbZYDA4+GSAFHyU27qtq4P1LWywNeTGLWWjTw7BiT/wsFH",
#  "rcHtKxQRwlIx7IW9kF+vjgrLSgzR10qbTHt+DjxErn5tXtuI0sJgF7GW7k6CdJBq",
#  "V+VCJ83kYWkNOtCMmsK/BlXuktlzwXiWG6Nj3JtY3b9VQSB240/XqLdhSEL5r/ZE",
#  "UcGMf+SqidE6DQyStZTxqY+w6fgff9D7Y5bba16H2izBA7aXyohJxAECAwEAAQ==",
#  "-----END RSA PUBLIC KEY-----"
# ],
# "server_id": "5d827e01bbd3b54fa33a6406",
# "user_id": "5d827c59bbd3b54fa33a62a9",
# "server": "Bastion",
# "token": false,
# "version": 1,
# "push_auth_ttl": 172800,
# "sync_hosts": [
#  "https://35.207.131.174"
# ],
# "server_box_public_key": "eHFhIgPi+Z17CKHMlryqA8Am3dzEskvVWk52+mz37xQ=",
# "organization": "HW3",
# "password_mode": "pin"
#}
setenv UV_ID 5aa81c289d8d42a2822b898c561b7717
setenv UV_NAME restless-refuge-8788
client
dev tun
dev-type tun
remote 35.207.131.174 10855 udp
nobind
persist-tun
cipher AES-128-CBC
auth SHA1
verb 2
mute 3
push-peer-info
ping 10
ping-restart 60
hand-window 70
server-poll-timeout 4
reneg-sec 2592000
sndbuf 393216
rcvbuf 393216
max-routes 1000
remote-cert-tls server
comp-lzo no
auth-user-pass
key-direction 1
<ca>
</ca>
<tls-auth>
#
# 2048 bit OpenVPN static key
#
-----BEGIN OpenVPN Static key V1-----
-----END OpenVPN Static key V1-----
</tls-auth>
<cert>
</cert>
<key>
</key>
```
---
<a name="HW4"></a>
# Домашнее задание 4
## Деплой тестового приложения
**Необходимые данные для проверки ДЗ**
```
testapp_IP = 35.198.171.185
testapp_port = 9292
```
***Создание виртуальной машины в GCP с помощью gcloud***
```
gcloud compute instances create reddit-app \
--boot-disk-size=10GB \  
--image-family ubuntu-1604-lts \  
--image-project=ubuntu-os-cloud \  
--machine-type=g1-small \ 
--tags puma-server \  
--restart-on-failure  

# Имя виртуальной машины
# Размер загрузочного диска
# Образ ОС
# Семейство образа ОС
# Тип виртуальной машины
# Теги для сети
# Дополнительные опции
```
***Необходимые компоненты для тестового приложения***

**Ruby & Bundler**
```
$ sudo apt install -y ruby-full ruby-bundler build-essential 
```
**MongoDB**
```
# Репозиторий
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
$ sudo bash -c 'echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" > /etc/apt/sources.list.d/mongodb-org-3.2.list'
# Установка
$ sudo apt install -y mongodb-org
```
* ! При проблемах с GPG и ключем для получения установки с репозитория Mongo
  ```
  # 1. Можно проигнорировать проблему при установке, добавив --allow-unauthenticated
  $ sudo apt install -y mongodb-org --allow-unauthenticated

  # 2. Использовать официальную инструкцию по установке
  $ wget -qO - https://www.mongodb.org/static/pgp/server-3.2.asc | sudo apt-key add -
  $ echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
  $ sudo apt-get update
  $ sudo apt install -y mongodb-org
  ```
**Тестовое приложение**
 ([Ссылка на Github](https://github.com/express42/reddit/tree/monolith))

***Установка***

```
$ git clone -b monolith https://github.com/express42/reddit.git
$ cd reddit && bundle install
```
***Запуск***
```
$ puma -d
```
## Самостоятельная работа 1
**Скрипты для установки необходимых компонентов**

Вышеперечисленные команды завернуты в скрипты:
1. `install_ruby.sh`
2. `install_mongodb.sh`
3. `deploy.sh`

Чтобы данные скрипты добавились в репозиторий Git как исполняемые файлы необходимо после команды `git add` и до `git commit` выполнить следующую команду для всех файлов:
```
git update-index --chmod=+x some_script.sh
``` 
## Дополнительное задание 1
**startup-script для VM**

***Передача локального файла в качестве startup-script***
```
gcloud compute instances create reddit-app-test \
--boot-disk-size=10GB \
--image-family ubuntu-1604-lts \
--image-project=ubuntu-os-cloud \
--machine-type=g1-small \
--tags puma-server \
--restart-on-failure \
--metadata-from-file startup-script=./startup_script.sh
```
***Альтернативные варианты передачи startup_script.sh***
1. Код скрипта непосредственно в команде
```
gcloud compute instances create example-instance --tags http-server \
--metadata startup-script='#! /bin/bash
# Installs apache and a custom homepage
sudo su -
apt-get update
apt-get install -y apache2
cat <<EOF > /var/www/html/index.html
<html><body><h1>Hello World</h1>
<p>This page was created from a simple start up script!</p>
</body></html>
EOF'
```
2. Код скрипта загружается с заданного URL
```
gcloud compute instances add-metadata example-instance \
    --metadata startup-script-url=gs://bucket/file
```
***Полный состав startup-script***
```
#! /bin/sh
apt update
apt install -y ruby-full ruby-bundler build-essential
wget -qO - https://www.mongodb.org/static/pgp/server-3.2.asc | sudo apt-key add -
echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
apt update
apt install -y mongodb-org
systemctl start mongod
systemctl enable mongod
git clone -b monolith https://github.com/express42/reddit.git
cd reddit && bundle install
puma -d
```
## Дополнительное задание 2
**Создание правила брандмауэра удаленно средствами gcloud**
```
gcloud compute --project=wise-cycling-253214 \
firewall-rules create puma-server \
--direction=INGRESS \
--priority=1000 \
--network=default \
--action=ALLOW \
--rules=tcp:9292 \
--source-ranges=0.0.0.0/0 \
--target-tags=puma-server

 # Указание проекта для создания правила
 # Имя правила
 # Напрвление трфика
 # Сеть виртуальных машин
 # Действия брандмауэера
 # Порты
 # Сеть источника обращения
 # Теги
```
