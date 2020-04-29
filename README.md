# razin92_infra
[![Travis](https://img.shields.io/travis/com/Otus-DevOps-2019-08/razin92_infra?style=flat-square)](https://travis-ci.com/Otus-DevOps-2019-08/razin92_infra)

Репозиторий для работы над домашними заданиями в рамках курса **"DevOps практики и инструменты"**

**Содрежание:**
<a name="top"></a>
1. [ДЗ#3 - Знакомство с облачной инфраструктурой](#hw3)
2. [ДЗ#4 - Деплой тестового приложения](#hw4)
3. [ДЗ#5 - Сборка образов VM при помощи Packer](#hw5)
4. [ДЗ#6 - Практика IaC с использованием Terraform](#hw6)
5. [ДЗ#7 - Принципы организации инфраструктурного кода и работа над инфраструктурой в команде на примере Terraform](#hw7)
6. [ДЗ#8 - Управление конфигурацией. Основные DevOps инструменты. Знакомство с Ansible](#hw8)
7. [ДЗ#9 - Деплой и управление конфигурацией с Ansible](#hw9)
8. [ДЗ#10 - Ansible: работа с ролями и окружениями](#hw10)
9. [ДЗ#11 - Разработка и тестирование Ansible ролей и плейбуков](#hw11)
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
[Содержание](#top)
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
gcloud compute --project=some-project-1234 \
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
 # Приоритет правила
 # Сеть виртуальных машин
 # Действия брандмауэера
 # Порты
 # Сеть источника обращения
 # Теги
```
---
<a name="hw5"></a>
[Содержание](#top)
# Домашнее задание 5
## Сборка образов VM при помощи Packer
[Репозиторий Packer'a](https://www.packer.io/downloads.html)

Создание учетных данных для приложения в GCP
```
$ gcloud auth application-default login
```
Просмотр токена
```
$ gcloud auth application-default print-access-token
```
## Создание baked-образа
Шаблон представлен в `.json`-файлах

Пример параметров builders
```
{
    "builders": [
        {
            "type": "googlecompute",
            "project_id": "some-project-1234",
            "image_name": "some-image-{{timestamp}}",
            "image_family": "some-image",
            "source_image_family": "ubuntu-1604-lts",
            "zone": "europe-west1-b",
            "ssh_username": "sshuser",
            "machine_type": "f1-micro"
        }
    ]
}
```
Пример параметров provisioners
```
# Описывает дополнительные действия по настройке необходимого окружения

{
    "provisioners": [
        {
            "type": "shell",
            "script": "scripts/some_script.sh",
            "execute_command": "sudo {{.Path}}"
        }
    ]
}
```
Проверка шаблона Packer
```
$ packer validate ./some_template.json
```
Сборка образа
```
$ packer build some_template.json
```
После сборки образ доступен для установки из консоли управления GCP
## Самостоятельные задания
1. Пример шаблона, принимающий пользовательские переменные
```
{
    "variables": {
        "project_id": null,
        "source_image_family": null,
        "machine_type": "f1-micro",
        "disk_type": "pd-standard",
        "description": "OTUS edu image",
        "network": "default"
    },
    "builders": [
        {
          "type": "googlecompute",
          "project_id": "{{ user `project_id` }}",
          "image_name": "reddit-base-{{timestamp}}",
          "image_family": "reddit-base",
          "source_image_family": "{{ user `source_image_family` }}",
          "zone": "europe-west3-a",
          "ssh_username": "appuser",
          "machine_type": "{{ user `machine_type` }}",
          "disk_size": 10,
          "disk_type": "{{ user `disk_type` }}",
          "image_description": "{{ user `description` }}",
          "network": "{{ user `network` }}",
          "tags": ["puma-server"]
        }
    ],
    "provisioners": [
        {
          "type": "shell",
          "script": "scripts/install_ruby.sh",
          "execute_command": "sudo {{.Path}}"
        },
        {
          "type": "shell",
          "script": "scripts/install_mongodb.sh",
          "execute_command": "sudo {{.Path}}"
        }
    ]
}
```
Стоит отметить, что переменные без определенных значений по-умолчанию является отличным способом использовать секреты.

2. Пример указания переменных в файле
```
{
    "project_id": "some-project-1234",
    "source_image_family": "ubuntu-1604-lts"
}
```
Для использования переменных следует выполнять следующие команды
```
# Переменные без файла
packer build -var "some_var_name=some_value" template.json

# Переменные в отдельном файле
packer buld -var-file="/path/to/variables.json" template.json

```

3. Дополнительные параметры шаблона для GCP VM

Указаны в примере выше. Полный список доступен по [ссылке](https://www.packer.io/docs/builders/googlecompute.html#account_file)
## Задания со *
Подготовка скрипта установки приложения с запуском через systemctl

`packer/files/deploy_puma.sh`

```
#! /bin/sh

# Deploy Reddit App

git clone -b monolith https://github.com/express42/reddit.git /etc/reddit/
cd /etc/reddit && bundle install
adduser puma --disabled-password --gecos ""

# Startup Script

mkdir /usr/local/puma
wget https://gist.githubusercontent.com/razin92/06870f9eb9e448d3bde12b17113e2233/raw/244fbf48a49954c49967ca620d25eda99686bbad/start.sh -O /usr/local/puma/start.sh
chmod +x /usr/local/puma/start.sh

# Service Puma

wget https://gist.githubusercontent.com/razin92/06870f9eb9e448d3bde12b17113e2233/raw/244fbf48a49954c49967ca620d25eda99686bbad/puma.service -O /etc/systemd/system/puma.service
systemctl daemon-reload
systemctl start puma
systemctl enable puma
```
Скрипт запуска приложения для сервиса
```
#! /bin/sh

# Puma startup script

puma --dir /etc/reddit/
```
Cервис приложения Puma
```
[Unit]
Description=Puma

[Service]
Type=simple
User=puma
Group=puma

ExecStart=/usr/local/puma/start.sh
ExecStop=/bin/kill -15 $MAINPID

[Install]
WantedBy=multi-user.target
```
Скрипт создания инстанса из ранее созданного образа
```
#! /bin/sh

set -x

project_id=$1
instance_name=$2
image_name=$3

# Правило брандмауэра для работы приложения

gcloud compute --project=$project_id \
firewall-rules create puma-server \
--direction=INGRESS \
--priority=1000 \
--network=default \
--action=ALLOW \
--rules=tcp:9292 \
--source-ranges=0.0.0.0/0 \
--target-tags=puma-server

# Создания инстанса VM с конкретным образом

gcloud compute instances create $instance_name \
--project=$project_id \
--boot-disk-size=10GB \
--image=$image_name \
--image-project=$project_id \
--machine-type=g1-small \
--zone=europe-west3-a \
--tags puma-server \
--restart-on-failure

# С последним созданным образом
# --image=$image_name заменить на --image-family=reddit-full 
```
<a name="HW6"></a>
[Содержание](#top)
# Домашнее задание 6
## Практика IaC с использованием Terraform
### Terraform-1
**Список основных комманд для работы с Terraform** 

[Команды в документации](https://www.terraform.io/docs/cli-index.html)
1. Инициализация проекта 
```
terraform init
```
2. Проверка синтаксиса конфигруционных файлов .tf 
```
terraform validate
```
3. Проверка изменений 
```
terraform plan
```
4. Применение изменений 
```
terraform apply
```
5. Просмотр текущего состояния проекта 
```
terraform show
```
6. Просмотр вывода при настроенной секции `outputs`
```
terraform output
terraform output <переменная>
# terraform output app_external_ip
```
7. Пересоздание ресурса
```
terraform taint <имя ресурса>
# terraform taint google_compute_instance.app
```
8. Удаление всех ресурсов
```
terraform destroy
```
9. Форматирование конфигурационных файлов для повышения "читаемости"
```
terraform fmt
``` 

**Основные секции в конфигурационных файлах**

[Конфигурация в документации](https://www.terraform.io/docs/configuration/index.html)
1. terraform
```
terraform {
    # Описывает конфигурацию самого terraform
    # Например: требуемая версия
    required_version = "0.12.8"
}
```
2. [provider](https://www.terraform.io/docs/providers/index.html)
```
provider "<имя_провайдера>" {
    # Конфигурация используемого провайдера
}

# Пример
provider "google" {
  # Версия провайдера
  version = "2.15.0"
  project = "project-id"
  region  = "europe-west1"
}
```
3. [resource](https://www.terraform.io/docs/configuration/resources.html)
    - [provisioner](https://www.terraform.io/docs/provisioners/index.html)
```
resource "<тип_ресурса>" "<имя_ресурса>" {
    # Описывает конфигурацию ресурса
}
```
```
provisioner "<тип>" {
    # Описывает действия после создания ресурса
    # Является частью конфигурации ресурса
}
# Примеры
# Поставщик типа "файл" копирует файл из локального хранилища в ресурс
provisioner "file" {
    source      = "files/puma.service"
    destination = "/tmp/puma.service"
}
# Поставщик типа "remote-exec" производит выполенение команды в ресурсе
provisioner "remote-exec" {
    script = "files/deploy.sh"
}
```
***Полный пример ресурсов на примере [GCP](https://www.terraform.io/docs/providers/google/index.html) (Инстанса и Правила брандмауэра)***
```
resource "google_compute_instance" "app" {
  name         = "reddit-app"
  machine_type = "g1-small"
  zone         = var.zone
  tags         = ["reddit-app"]
  boot_disk {
    initialize_params {
      image = var.disk_image
    }
  }

  network_interface {
    network = "default"
    access_config {}
  }

  metadata = {
    # Путь до публичного ключа
    ssh-keys = "appuser:${file(var.public_key_path)})"
  }

  connection {
    type        = "ssh"
    host        = self.network_interface[0].access_config[0].nat_ip
    user        = "appuser"
    agent       = false
    private_key = file(var.private_key_path)
  }

  provisioner "file" {
    source      = "files/puma.service"
    destination = "/tmp/puma.service"
  }

  provisioner "remote-exec" {
    script = "files/deploy.sh"
  }
}

resource "google_compute_firewall" "firewall_puma" {
  name = "allow-puma-default"
  # Сеть, в которй действует правило
  network = "default"
  # Список разрешений
  allow {
    protocol = "tcp"
    ports    = ["9292"]
  }
  # Каким адресам рарешается доступ
  source_ranges = ["0.0.0.0/0"]
  # Правило применимо для инстансов с перечисленными тэгами
  target_tags = ["reddit-app"]
}
```
4. [output](https://www.terraform.io/docs/configuration/outputs.html)
```
output "<имя_переменной>" {
  # Выводит выбранные параметры состояния проекта
}

# Пример, внешний IP-адрес инстанса GCP
output "app_external_ip" {
  value = google_compute_instance.app.network_interface[0].access_config[0].nat_ip
}
```
5. [variable](https://www.terraform.io/docs/configuration/variables.html)
```
variable <имя_переменной> {
    # Пользовательская переменная, используемая в проекте
}

# Пример
variable region {
  # Описание переменной
  description = "Region"
  # Значение по умолчанию
  default = "europe-west1"
}

# Пример использования в конфигурации
# var.<имя_переменной>
...
provider "google" {
  # Версия провайдера
  version = "2.15.0"
  project = "project-id"
  region  = var.region # <-- вызов переменной
}
# Переменные без значения по-умолчанию хранятся в файле *.tfvars
```
`*.tfvars.example`
```
project = "your_project_id"
disk_image = "reddit-base"
```
Рекомендуемые файлы для .gitignore
```
*.tfstate
*.tfstate.*.backup
*.tfstate.backup
*.tfvars
.terraform/
```
## Задание со *
Для добавление SSH-ключей в проект необходимо добавить следующую конфигурацию
```
# Метаданные ssh-keys
resource "google_compute_project_metadata" "ssh_keys" {
    metadata = {
    ssh-keys = "appuser1:${file(var.public_key_path)}"
  }
}

# Также для блокировки ssh-ключей внутри инстанса
# Указывается в разделе metadata {} конфигурации инстанса
...
metadata = {
    block-project-ssh-keys = false
  }
...
```
Для указания нескольких ключей стоит указать их в одну строку
```
resource "google_compute_project_metadata" "ssh_keys" {
    metadata = {
    ssh-keys = "appuser1:${file(var.public_key_path)}appuser2:${file(var.public_key_path)}"
  }
}
# или для читаемости
resource "google_compute_project_metadata" "ssh_keys" {
    metadata = {
    ssh-keys = <<EOF
    appuser1:${file(var.public_key_path)}
    appuser2:${file(var.public_key_path)}
    EOF
  }
}
```
**! Внимание**

При применении данных конфигураций из проекта GCP удалятся ключи, созданные вручную (через WEB или утилиту gloud). Поэтому все ключи стоит хранить в конфигурации terraform.
## Задание с **
### Создание балансировщика
Необходимые компоненты для работы балансировщика

Группа инстансов
```
resource "google_compute_instance_group" "app-cluster" {
  name = "app-cluster"
  description = "Reddit-app instance group"
  project = var.project
  # Список инстансов, входящих в группу
  instances = "${google_compute_instance.app.*.self_link}"

  # Именованный порт приложения
  named_port {
    name = "app-http"
    port = "9292"
  }

  zone = var.zone
}
```
Мониторинг доступности ресурсов
```
resource "google_compute_health_check" "app-healthcheck" {
  name = "app-healthcheck"
  check_interval_sec = 1
  timeout_sec = 1
  tcp_health_check {
    port = "9292"
  }
}
```
Бэкенд сервис
```
resource "google_compute_backend_service" "app-backend" {
  name = "app-backend"
  protocol = "HTTP"
  port_name = "app-http"
  timeout_sec = 10

  backend {
    # Группа инстансов
    group = "${google_compute_instance_group.app-cluster.self_link}"
  }
  # Ссылка на мониторинг доступности
  health_checks = ["${google_compute_health_check.app-healthcheck.self_link}"]
}
```
URL-map
```
resource "google_compute_url_map" "app-map" {
  name = "app-map"
  description = "Reddit-app LB"
  # Бекенд-сервис по-умолчанию для неопознанных запросов
  default_service = "${google_compute_backend_service.app-backend.self_link}"

  # Правило перенаправления на бекенд
  host_rule {
    hosts = ["*"]
    path_matcher = "allpaths"
  }

  # Условия срабатывания перенаправления на бекенд
  path_matcher {
    name = "allpaths"
    default_service = "${google_compute_backend_service.app-backend.self_link}"
  }
}
```
Правило перенаправления трафика
```
resource "google_compute_global_forwarding_rule" "app-rule"{
  name       = "app-rule"
  # Все запросы на указанный порт отправляем на Proxy
  target     = "${google_compute_target_http_proxy.app-proxy.self_link}"
  port_range = "80"
}
```
http-proxy для входящего трафика
```
resource "google_compute_target_http_proxy" "app-proxy" {
  name        = "app-proxy"
  # Все принятые запросы перенаправляются на URL-map
  url_map     = "${google_compute_url_map.app-map.self_link}"
}
```
Использование `count` и вынесение количества циклов в переменные позволяет управлять количеством необходимых ресурсов централизованно, также можно гарантировать их идентичность. 

Использование одинаковых приложений, предоставленных для домашней работы, в балансировке применимо только для статичных данных. При работе с динамическими данными, при падении одного из инстансов, вся его база данных будет потеряна для работы. Для корректной балансировки необходимо синхронизировать базы данных всех инстансов.

<a name="HW7"></a>
[Содержание](#top)
# Домашнее задание 7
## Принципы организации инфраструктурного кода и работа над инфраструктурой в команде на примере Terraform
### Terraform-2

Добавление уже имеющихся ресурсов у провайдера
```
$ terraform import <ресурс>

# Пример
$ terraform import google_compute_firewall.firewall_ssh default-allow-ssh
```
Это говорит о том, что Terraform имеет иноформацию о всех ресурсах провайдера и их всегда можно добавить в конфигурацию.

Создание ресурса GCP ip-адрес
```
resource "google_compute_address" "app_ip" {
  name = "reddit-app-ip"
}
```

В конфигурации можно ссылаться на уже указанные ресурсы. При этом порядок создания ресурсов будет согласно зависимостям. Это пример ***Неявной зависимости***.
```
network_interface {
 network = "default"
 access_config = {
   # Использование вышеуказанного ресурса
   nat_ip = google_compute_address.app_ip.address
 }
}
```

Для удобства управлением проекта конфигурация может быть разбита на несколько `*.tf`-файлов. Также, для многократного использования конфигурации применяются ***Модули***. Конфигурация модулей такая же, как и у обычного проекта, за исключением того, что они могут представлять только какую-то его часть. Например, только один ресурс. 

**Пример использования модулей**
```
provider "google" {
  version = "~> 2.15"
  project = var.project
  region  = var.region
}

module "app" {
  # Расположение директории с модулем
  source          = "modules/app"
  # Переменные, передаваемые в модуль
  public_key_path = var.public_key_path
  zone            = var.zone
  app_disk_image  = var.app_disk_image
}

module "db" {
  source          = "modules/db"
  public_key_path = var.public_key_path
  zone            = var.zone
  db_disk_image   = var.db_disk_image
}
```
**Загрузка указанных модулей в проект (в директорию .terraform)**
```
$ terraform get

# Результат выполнения
Get: file:///Users/user01/hw09/modules/app
Get: file:///Users/user01/hw09/modules/db

# Пример расположения
$ tree .terraform
.terraform
├── modules
│ ├── 9926d1ca5a4ce00042725999e3b3a90f -> /Users/user01/hw09/modules/db
│ └── dea8bdea57c956cc3317d254e5822e13 -> /Users/user01/hw09/modules/app
└── plugins
└── darwin_amd64
├── lock.json
└── terraform-provider-google_v0.1.3_x4

```
**Получение Output-переменных из модуля**
```
# Output модуля
output "app_external_ip" {
  value = google_compute_instance.app.network_interface.0.access_config.0.assigned_nat_ip
}

# Использование переменной в проекте
output "app_external_ip" {
  value = module.app.app_external_ip
}
``` 
[Реестр модулей для Terraform](https://registry.terraform.io/)

[Модули для провайдера Google](https://registry.terraform.io/browse?provider=google)

**Пример использования модуля из реестра.**

[Модуль storage-bucket](https://registry.terraform.io/modules/SweetOps/storage-bucket/google)

После инициализации и применения создает бакет в указанном проекте
```
provider "google" {
  version = "~> 2.15"
  project = var.project
  region  = var.region
}

module "storage-bucket" {
  source  = "SweetOps/storage-bucket/google"
  version = "0.3.0"

  # Наименования бакета
  name = "storage-bucket-test"
}

output storage-bucket_url {
  value = module.storage-bucket.url
}
```
## Задание со *
Для хранения состояния проекта можно использовать удаленные бакеты, что будет полезно для работы в команде или на разных рабочих станциях. Место хранения стейт-файла указывается в конфигурации вида:
```
# backend.tf

terraform {
  backend "gcs" {
    # Имя бакета
    bucket = "storage-bucket-test"
    # Имя директории
    prefix = "test1"
  }
}
```
При применении изменений одновременно с нескольких расположений конфигурации, но с единым местом хранения стейт-файла то применение, что выполнялось первым заблокирует стейт до окончания применений. При этом будет выведена информация о блокировке.
```
$ terraform apply
Acquiring state lock. This may take a few moments...

Error: Error locking state: Error acquiring the state lock: writing "gs://storage-bucket-reddit-app/stage/default.tflock" failed: googleapi: Error 412: Precondition Failed, conditionNotMet
Lock Info:
  ID:        1570462114386010
  Path:      gs://storage-bucket-reddit-app/stage/default.tflock
  Operation: OperationTypeApply
  Who:       user@hostname
  Version:   0.12.8
  Created:   2019-10-07 15:28:34.17645946 +0000 UTC
  Info:


Terraform acquires a state lock to protect the state from being written
by multiple users at the same time. Please resolve the issue above and try
again. For most commands, you can disable locking with the "-lock=false"
flag, but this is not recommended.
```
## Задание с **
При использовании провиженеров внутри модулей необходимо учесть расположение файлов для развертывания.

Например, если скрипты находятся в директории с модулем можно использовать переменную `${path.module}`, которая укажет на расположения директории модуля.
```
provisioner "file" {
    # Путь до скрипта
    source      = "${path.module}/puma.service"
    destination = "/tmp/puma.service"
  }
```
Так как во время выполнения домашнего задания VM тестового приложения была отделена от VM базы данных необходимо указать настройки подключения к БД.

Получение внутреннего ip-адреса VM БД
```
# modules/db/outputs.tf

output "db_internal_ip" {
  value = google_compute_instance.db.network_interface.0.network_ip
}
```
Передача внутреннего ip-адреса VM БД в VM приложения
```
# prod/main.tf

module "app" {
  # ...
  db_ip = module.db.db_internal_ip
}
```
Использование адреса в конфигурации
```
# modules/app/main.tf

provisioner "remote-exec" {
    inline = [
      "echo 'export DATABASE_URL=${var.db_ip}' >> /home/appuser/.profile"
    ]
  }
```
Так как конфигурация MongoDB стандартная, то монго слушает обращения только с localhost.

Изменение конфигурации MongoDB для работы приложения
```
# modules/db/main.tf

provisioner "remote-exec" {
    inline = [
      "sudo sed -i 's/127.0.0.1/0.0.0.0/' /etc/mongod.conf",
      "sudo systemctl restart mongod"
    ]
  }

# sed - замена строк в файле
# -i - сохранение изменений
# s/127.0.0.1/0.0.0.0/ - строка/поиск значения/новое значение
```
<a name="HW8"></a>
[Содержание](#top)
# Домашнее задание 8
## Управление конфигурацией. Основные DevOps инструменты. Знакомство с Ansible
### Ansible-1

***Установка Ansible***
```
$ python -m pip install ansible
```
***Запуск***
```
$ ansible all|host|group -i inventory -m ping

# -m - вызов модуля
# -i - вызов инвентаря
```

***Конфигурационный файл***
```
# ansible.cfg

[defaults]
inventory = ./inventory
remote_user = appuser
private_key_file = ~/.ssh/appuser
host_key_checking = False
retry_files_enabled = False
```
<a name="inventory"></a>
***Файлы инвентаря***

[Документация](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
1. .ini
```
# inventory

[app]
appserver ansible_host=app.example.com

[db]
dbserver ansible_host=db.example.com
```
2. .yml
```
# inventory.yml

all:
  children:
    app:
      hosts:
        app.example.com:
    db:
      hosts:
        db.example.com:
```
3. .json static
```
# inventory.json

{
  "all": {
    "children": {
      "app": {
        "hosts": {
          "app.example.com": null
        }
      },
      "db": {
        "hosts": {
          "db.example.com": null
        }
      }
    }
  }
}
```
4. .json dynamic
```
# результат выполнения скрипта

{
  "app": {
    "hosts": ["app.example.com"]
    }, 
  "db": {
    "hosts": ["db.example.com"]
    }
}
```
***Выполнение команд на удаленном хосте***

Модуль `command`
```
$ ansible app -m command -a 'ruby -v'
appserver | SUCCESS | rc=0 >>
ruby 2.3.1p112 (2016-04-26) [x86_64-linux-gnu]

$ ansible app -m command -a 'bundler -v'
appserver | SUCCESS | rc=0 >>
Bundler version 1.11.2

$ ansible db -m command -a 'systemctl status mongod'
dbserver | SUCCESS | rc=0 >>
● mongod.service - High-performance, schema-free document-oriented database

# не может выполнить несколько команд за раз
# не использует оболочку окружения (sh, bash)
$ ansible app -m command -a 'ruby -v; bundler -v'
appserver | FAILED | rc=1 >>
ruby: invalid option -; (-h will show valid options) (RuntimeError)non-zero
return code
```
Модуль `shell`
```
$ ansible app -m shell -a 'ruby -v; bundler -v'
appserver | SUCCESS | rc=0 >>
ruby 2.3.1p112 (2016-04-26) [x86_64-linux-gnu]
Bundler version 1.11.2

$ ansible db -m shell -a 'systemctl status mongod'
dbserver | SUCCESS | rc=0 >>
● mongod.service - High-performance, schema-free document-oriented database
```
Модуль `systemd`
```
$ ansible db -m systemd -a name=mongod
dbserver | SUCCESS => {
    "changed": false,
    "name": "mongod",
    "status": {
        "ActiveState": "active", ...
```
Модуль `service`
```
$ ansible db -m service -a name=mongod
dbserver | SUCCESS => {
    "changed": false,
    "name": "mongod",
    "status": {
        "ActiveState": "active", ...
```
Модуль `git`
```
$ ansible app -m git -a \
    'repo=https://github.com/express42/reddit.git dest=/home/appuser/reddit'
appserver | SUCCESS => {
    "after": "61a7f75b3d3e6f7a8f279896fb4e9f0556e1a70a",
    "before": null,
    "changed": true
}
$ ansible app -m git -a \
    'repo=https://github.com/express42/reddit.git dest=/home/appuser/reddit'
appserver | SUCCESS => {
    "after": "61a7f75b3d3e6f7a8f279896fb4e9f0556e1a70a",
    "before": "5c217c565c1122c5343dc0514c116ae816c17ca2",
    "changed": false,
    "remote_url_changed": false
}
```
### Плейбук
Последовательно выполняет все описанные действия
```
# clone.yml

---
- name: Clone
  hosts: app
  tasks:
    - name: Clone repo
      git:
        repo: https://github.com/express42/reddit.git
        dest: /home/appuser/reddit
```
Запуск
```
ansible-playbook clone.yml
```
Результат выполнения плейбука при уже имеющихся данных у хоста
```
PLAY RECAP **********************************************************************************************************
app.example.com               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
Результат выполнения плейбука после удаления директории `/home/appuser/reddit`
```
TASK [Clone repo] ***************************************************************************************************
changed: [app.example.com]

PLAY RECAP **********************************************************************************************************
app.example.com               : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
Флаг `changed` указывает на то, что был изменен один хост, а именно, выполнена комманда клонирования git-репозитория в директорию, указанную в playbook, так как при запуске Ansible он при проверке ее не нашел

## Задание со *
---
Примеры инвентори учебного проекта см. [выше](#inventory)

Статический инвентори .json повторяет структуру .yml файла.

Динамический инвентори отличается по структуре. Формируется только при выполнении скрипта (bash, python, ruby и др.). При использовании параметра `--list` возвращает полный список хостов в виде инветори. Также есть опциональный параметр `--host <hostname>`, при котором скрипт должен вернуть список с переменными для этого хоста. Секция `_meta` может помочь получить все нужные переменные хостов за одну команду.
```
{
    "group": {
        "hosts": [
            "host1.example.com",
            "host2.example.com"
        ],
        "vars": {
            "ansible_ssh_user": "appuser",
            "ansible_ssh_private_key_file": "~/.ssh/appuser",
            "example_variable": "value"
        }
    },
    # Список всех переменных хостов в инвентори
    "_meta": {
        "hostvars": {
            "host1.example.com": {
                "host_specific_var": "bar"
            },
            "host2.example.com": {
                "host_specific_var": "foo"
            }
        }
    }
}
```
Для реализации демонстрации динамического инвентори на основе статического проделаны следующие шаги:

Формирование JSON
```
{
  "app": {
    "hosts": ["app.example.com"]
    }, 
  "db": {
    "hosts": ["db.example.com"]
    }
}
```
Скрипт, читающий JSON для Ansible. Скрипт должен не возвращать переменную со значением, а возвращать само значение.
```
#!/usr/bin/python

import sys
import json

# чтение параметров командной строки
script, action = sys.argv

def read_static():
    # чтение сформированного файла
    with open('inventory.json') as source:
        inventory = json.load(source)
        return json.dumps(inventory)


if __name__ == '__main__':
    # захват команды --list
    if action == '--list':
        print(read_static())
```
Изменение конфигурационного файла Ansible
```
# ansible.cfg


[defaults]
inventory = ./inventory.py
remote_user = appuser
private_key_file = ~/.ssh/appuser
host_key_checking = False
retry_files_enabled = False
```
Результат выполнения `$ ansible all -m ping`
```
app.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}

db.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```
<a name="HW9"></a>
[Содержание](#top)
# Домашнее задание 9
## Деплой и управление конфигурацией с Ansible
### Ansible-2

Все хосты, до которых не смог достучаться Ansible записываются в файл `*.retry`. Этот файл можно использовать для перезапуска плейбука для довыполнения всей работы командой
```
$ ansible-playbook playbook.yml --limit @/path/to/*.retry
```
***Плейбук*** может состоять из нескольких ***сценариев***. ***Сценарий*** может содержать набор ***заданий*** (`tasks`). Для разделения ***сценариев*** и ***заданий*** можно использовать ***теги*** (`tags`) для дальнейшего выборочного запуска необходимых задач.

***Пример***
```
---
- name: Configure db hosts
  hosts: db
  tags: db-tag
  become: true
  vars:
    mongo_bind_ip: 0.0.0.0
  tasks:
    - name: Change mongo config file
      template:
        src: templates/mongod.conf.j2
        dest: /etc/mongod.conf
        mode: 0644 # Права на файл
      notify: restart mongod

  handlers:
    - name: restart mongod
      become: true
      service: name=mongod state=restarted

- name: Configure app hosts
  hosts: app
  tags: app-tag
  become: true
  vars:
    db_host: 10.132.15.212
  tasks:
    - name: Add unit file for Puma
      copy:
        src: files/puma.service
        dest: /etc/systemd/system/puma.service

    - name: Add config file for Puma
      template:
        src: templates/db_config.j2
        dest: /home/appuser/db_config
        owner: appuser
        group: appuser
      notify: reload puma

    - name: enable puma
      systemd: name=puma enabled=yes

  handlers:
    - name: reload puma
      service: name=puma state=restarted

- name: Deploy app
  hosts: app
  tags: deploy-tag
  become: true
  tasks:
    - name: Fetch the latest version of application code
      git:
        repo: 'https://github.com/express42/reddit.git'
        dest: /home/appuser/reddit
        version: monolith
      notify: reload puma

    - name: Bundle install
      bundler:
        state: present
        chdir: /home/appuser/reddit

  handlers:
    - name: reload puma
      service: name=puma state=restarted
```
Для большего удобства рекомендуется использовать отдельные ***плейбуки*** для разных задач. Это повышает читаемость и уменьшает количество ошибок при редактировании. При использовании нескольких ***плейбуков*** для выполнения одной глобальной задачи можно использовать файл ***плейбука*** с импортом всех используемыех файлов.

***Пример***
```
---
- import_playbook: db.yml
- import_playbook: app.yml
- import_playbook: deploy.yml
```
Для провервки работоспособности ***плейбуков*** можно использовать следующую команду
```
$ ansible-playbook playbook.yml --check

# можно выбрать отдельные хосты или группы: --limit <host|group>
# можно выбрать отдельные задачи тегами: --tags <tag_name>
```
Для поиска ошибок также можно использовать `debug`
```
$ ansible all -m debug -a var=ansible_host

# данный пример выведет значения всех переменных ansible_host
```
Для динамического создания файлов используются шаблоны jinja2
```
# Задача

---
- name: Configure db hosts
  hosts: db
  become: true
  vars:
    # Переменная для шаблона
    mongo_bind_ip: 0.0.0.0
  tasks:
    - name: Change mongo config file
      # Использование шаблона
      template:
        src: templates/mongod.conf.j2
        dest: /etc/mongod.conf
        mode: 0644 # Права на файл
```
```
# Шаблон
# mongod.conf.j2

# Where and how to store data.
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# network interfaces
net:
  # Переменные из задания Ansible
  port: {{ mongo_port | default('27017') }}
  bindIp: {{ mongo_bind_ip }}
```
Для проверки отрисованного шаблона, а также просмотр изменений применить команду
```
$ ansible-playbook -C playbook.yml --diff
```
Пример вывода
```
TASK [Change mongo config file] ******************************************************************************************
--- before: /etc/mongod.conf
+++ after: /home/slav/.ansible/tmp/ansible-local-7240VlQ881/tmpwntXut/mongod.conf.j2
@@ -1,16 +1,8 @@
-# mongod.conf
-
-# for documentation of all options, see:
-#   http://docs.mongodb.org/manual/reference/configuration-options/
-
 # Where and how to store data.
 storage:
   dbPath: /var/lib/mongodb
   journal:
     enabled: true
-#  engine:
-#  mmapv1:
-#  wiredTiger:

 # where to write logging data.
 systemLog:
@@ -21,21 +13,5 @@
 # network interfaces
 net:
   port: 27017
-  bindIp: 127.0.0.1
+  bindIp: 0.0.0.0

-
-#processManagement:
-
-#security:
-
-#operationProfiling:
-
-#replication:
-
-#sharding:
-
-## Enterprise-Only Options:
-
-#auditLog:
-
-#snmp:
```
После выполненных заданий могут быть вызваны дополнительные задачи. Отдают репорт об успешном выполнении задачи `notify`, перехватывают `handlers`.
```
tasks:
    - name: Change mongo config file
      template:
        src: templates/mongod.conf.j2
        dest: /etc/mongod.conf
        mode: 0644 # Права на файл
      # Уведомить handler с именем `restart mongod`
      notify: restart mongod

# Выполняют дополнительные задачи при успешном выполнении task
handlers:
  - name: restart mongod
    become: true
    service: name=mongod state=restarted
```

## Задание со *
Для использования Dynamic Inventory для GCP можно использовать плагин Ansible gcp_compute. [Документация](https://docs.ansible.com/ansible/latest/plugins/inventory/gcp_compute.html)

Требования:
```
# requirements.txt
...
requests>=2.18.4
google-auth>=1.3.0
```

Инвентарные файлы должны заканчиваться на `.gcp.(yml|yaml)` или `gcp_compute.(yml|yaml)`

Пример ***inventory*** GCP
```
---
# Имя плагина
plugin: gcp_compute
# Управляемые проекты
projects:
  - project.id
# Файл для подключения к консоли GCP
service_account_file: ~/path/to/key.json
# Тип аутентификации
auth_kind: serviceaccount
# Отображаемая информация о хостах
hostnames:
  - name
# Параметры инстанса добавления в группу
# Делятся на группы путем поиска строки в Имени
groups:
  app: "'-app' in name"
  db: "'-db' in name"
# Переменные hostsvars
compose:
  # Внешние IP адреса. Используются для подключения к хостам
  ansible_host: networkInterfaces[0].accessConfigs[0].natIP
  # Внутренние IP адреса хостов 
  internal_ip: networkInterfaces[0].networkIP

```
Для просмотра сформированного `.json` инвентори применить следующую команду
```
$ ansible-inventory -i inventory.gcp.yml --list
```
Для просмотра сформированного дерева инвентори
```
$ ansible-inventory -i inventory.gcp.yml --graph

@all:
  |--@app:
  |  |--reddit-app
  |--@db:
  |  |--reddit-db
  |--@ungrouped:
```
Переменные `hostvars` можно использовать в ***плейбуках***
```
- name: Configure app hosts
  hosts: app
  become: true
  vars:
    # Использование переменной из hostvars как строку
    db_host: "{{ hostvars['reddit-db'].internal_ip }}"
  tasks:
    - name: Add config file for Puma
      template:
        src: templates/db_config.j2
        dest: /home/appuser/db_config
        owner: appuser
        group: appuser
      notify: reload puma

# Использование переменной из hostvars как список
#    db_host: 
#      - "{{ hostvars['reddit-db'].internal_ip }}"
```
```
# Использование Dynamic Inventory по-умолчанию
# ansible.cfg

[defaults]
inventory = ./inventory.gcp.yml
...
```
## Самостоятельное задание
Использования Ansible как провиженер для Packer
```
...
"provisioners": [
	{
    "type": "ansible",
    "playbook_file": "ansible/packer_app.yml"
	}
]
...
```
Использование модулей `apt` (устанавливает пакеты), `apt-key` (добавляет ключи репозиториев), `apt-repository` (добавляет репозитории) для установки необходимого ПО
```
---
- name: Install MongoDB
  hosts: all
  become: true
  tasks:
    - name: Add MongoDB repo key
      apt_key:
        # путь до ключа
        url: https://www.mongodb.org/static/pgp/server-3.2.asc
        state: present
    - name: Add MongoDB repo
      apt_repository:
        # репозиторий
        repo: deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse
        state: present
        # файл с информацией о добавляемом репозитории
        filename: mongodb-org-3.2.list
    - name: Install & run MongoDB
      apt:
        update_cache: yes
        # Имя устанавливаемого пакета
        pkg: mongodb-org
        state: present 
      notify: start mongod
    - name: Enable service Mongod
      systemd: name=mongod enabled=yes 
  handlers:
    - name: start mongod
      systemd: name=mongod state=started
```
```
---
- name: Install Ruby & Bundler
  hosts: all
  become: true
  tasks:
    - name: Using apt
      apt:
        # Установка нескольких пакетов
        name: "{{ packages }}" 
        state: present       
      vars:
        # Переменная со списком пакетов
        packages:
          - ruby-full
          - ruby-bundler
          - build-essential
```
[Содержание](#top)
<a name="hw10"></a> 
# Домашнее задание 10
## Ansible: работа с ролями и окружениями

***Роли*** - сгруппированные в единое целое описания конфигурации (таски, хендлеры, файлы, шаблоны, переменные).

**Ansible Galaxy** - инструмент для работы с ролями, созданных сообществом
```
$ ansible-galaxy -h
```
Создание структуры пустой роли
```
$ ansible-galaxy init <name>
```
Структура роли
```
db
├── defaults           # Переменные по-умолчанию
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta               # Информация о роли, создателе и зависимостях
│   └── main.yml
├── README.md
├── tasks              # Директория для задач
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars               # Директория для переменных, которые не должны
    └── main.yml       #         переопределяться пользователем
```
**Ansible Vault** - инструмент, используемый для шифрования секретов. При выполнении плейбука секреты расшифровываются и могут быть выведены через дебаг. Следует учесть этот момент.

Для шифрования используется строковый ключ. Может храниться в файли и быть указан в конфигурации.
```
[defaults]
...
vault_password_file = vault.key
```
Шифрование
```
$ ansible-vault encrypt /path/to/file
```
Редактирование
```
$ ansible-vault edit /path/to/file
```
Рашифрование
```
$ ansible-vault decrypt /path/to/file
```
## Задание со *
Аналогично заданию из прошлого ДЗ
## Задание с **
Конфигурация TravisCI сохраняется в корне репозитория в файле `.travis.yml`. [Документация](https://docs.travis-ci.com/)

Пример
```
# Дистрибутив тестового окружения
dist: trusty 
# Указывает необходимость явно вызывать sudo
sudo: required
# Язык
language: bash
before_install:
# Скрипт с условием запуска в зависимости от бранча
- if [ "$TRAVIS_BRANCH" = "master" ]; then sh ./play-travis/requirements.sh; fi
- if [ "$TRAVIS_BRANCH" = "master" ]; then sh ./play-travis/test.sh; fi
- curl https://raw.githubusercontent.com/express42/otus-homeworks/2019-08/run.sh |
  bash
notifications:
  slack:
    rooms:
      secure: <зашифрованный или открытый ключ>
```
<a name="hw11"></a>
[Содержание](#top)
# Домашнее задание 11
## Разработка и тестирование Ansible ролей и плейбуков

**VirtualBox**

**!** Не устанавливать в виртуальную среду внутри виртуальной машины. Для нормальной работы требует:
- Intel VT-x/EPT или AMD-V/RVI
- Счетчики производительности виртуализации процессора

Работает внутри виртуальной машины VMware Workstation.

**Vagrant**

- [Базовые команды Vagrant](https://gist.github.com/wpscholar/a49594e2e2b918f4d0c4)
- [Поддерживаемые провайдеры](https://www.vagrantup.com/docs/providers/)
- [Установка](https://www.vagrantup.com/downloads.html)
- [Конфигурация](https://www.vagrantup.com/docs/vagrantfile/)
- [Репозиторий Box-образов](https://app.vagrantup.com)

Конфигурация хранится в `Vagrantfile`. Примеры:
```
Vagrant.configure("2") do |config|

  # Указание провайдера и его настроек
  config.vm.provider :virtualbox do |v|
    # Кол-во оперативной памяти VM
    v.memory = 1024
  end

  # Описание и настройки VM
  config.vm.define "dbserver" do |db|
    # Наименование образа
    db.vm.box = "ubuntu/xenial64"
    db.vm.hostname = "dbserver"
    db.vm.network :private_network, ip: "10.10.10.10"

    # Провиженеры и их настройки
    db.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbooks/site.yml"
      ansible.groups = {
        "db" => ["dbserver"],
        # Переменные для Ansible
        "db:vars" => {"mongo_bind_ip" => "0.0.0.0"}
      }
      end
  end
end
```
Путь до рабочих папок Vagrant-a
```
# В директории, где находится Vagrantfile
.vagrant/
# Автоматически сгенерированный инвентори Ansible
.vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory
```
**Ansible**

Установка Python, если его нет на целевой машине
```
---
- name: Check && install python
  hosts: all
  become: true
  # Пропуск сбора фактов, т.к. требует Python
  gather_facts: False

  tasks:
    - name: Install python for Ansible
      # Исполнение команд, используя shell
      raw: test -e /usr/bin/python || (apt -y update && apt install -y python-minimal)
      changed_when: False
```
Запуск задания в зависимости от типа виртуализации
```
# Модификация systemctl внутри Docker-a
# так как он не поддерживает его "из коробки"
---
- name: Install systemctl when docker
  copy:
    src: systemctl.py
    dest: /bin/systemctl
  # Запуск, если соблюдено условие
  when: "ansible_virtualization_type == 'docker'"
```
Просмотр всех переменных Ansible
```
$ ansible all -m setup
```
! При большом количестве заданий рекомендуется группировать их и распределять в отдельные файлы. При групперовке при помощи `include` важен порядок. Задачи выполняются по порядку сверху вниз.

Дебаг сообщения. Отображаются перед выполнением заданий
```
- name: Show info about the env this host belongs to
  debug:
    msg: "This host is in {{ env }} environment!!!"
```
## Задание со *
При создании VM с помощью Vagrant не настраивался Nginx.

Конфиг для его работы:
```
...
    app.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbooks/site.yml"
      ansible.groups = {
        "app" => ["appserver"],
        "app:vars" => {"db_host" => "10.10.10.10"}
      }
      # Дополнительные настройки Nginx и пользователя
      ansible.extra_vars = {
        "deploy_user" => "vagrant",
        "nginx_sites" => { "default" => [
          'listen 80',
          'server_name "reddit"',
          'location / {
            proxy_pass http://127.0.0.1:9292;
          }'
        ]}
      }
      end
    end
...
```
**Molecule и Testinfra**

- [Документация Molecule](https://molecule.readthedocs.io/en/stable/)
- [Документация Testinfra](https://testinfra.readthedocs.io/en/latest/)

Используются для локального тестирования ролей Ansible.

Зависимости
```
molecule>=2.6
testinfra>=1.10
python-vagrant>=0.5.15
``` 
Создание сценария и инициализация Molecule
```
$ molecule init scenario --scenario-name <name> -r <role_name> -d vagrant
# -d - Драйвер
```
Путь до тестов
```
molecule/<scenario_name>/tests/test_default.py
```
Путь до описания тестовой машины
```
molecule/<scenario_name>/molecule.yml
```
Создание тестовой машины
```
$ molecule create
```
Список созданных инстансов
```
$ molecule list
```
Подключение к инстансу по SSH
```
$ molecule login -h instance
```
Путь до плейбука для тестируемой роли
```
molecule/<scenario_name>/playbook.yml
```
Применение конфигурации (провиженинг)
```
$ molecule converge
```
Запуск тестов
```
$ molecule verify 
```
### Самостоятельное задание
Проверка портов при помощи [модулей](http://testinfra.readthedocs.io/en/latest/modules.html) Testinfra
```
def test_listening_port(host):
    assert host.socket("tcp://0.0.0.0:27017").is_listening
```
Указание дополнительных аргументов и переменных окружения Ansible в конфигурации Packer
```
{
	"type": "ansible",
	"playbook_file": "ansible/playbooks/packer_app.yml",
  # Переменные окружения
	"ansible_env_vars": ["ANSIBLE_ROLES_PATH={{ pwd }}/ansible/roles"],
  # Дополнительные аргументы
	"extra_arguments": ["--tags", "ruby"]
}
```
## Задание со *
Необходимые пакеты:
```
ansible==2.9.0
molecule>=2.6
ansible-lint>=4.1.0
testinfra>=1.10
apache-libcloud>=0.20.0
```
Установка драйвера GCE для molecule
```
pip install 'molecule[gce]'
```
Необходимые переменные для работы GCE
```
# Переменные окружения (глобальные)
GCE_SERVICE_ACCOUNT_EMAIL
GCE_CREDENTIALS_FILE
GCE_PROJECT_ID
```
Переменные кастомизации инстанса задаются в блоке `platforms` файла `molecule.yml`
```
# Пример
platforms:
  - name: instance-travis
    zone: europe-west1-b
    machine_type: f1-micro
    image: ubuntu-1604-xenial-v20170919
```
Шаги для интеграции Travis CI (взято из примера [Artemmkin](https://gist.github.com/Artemmkin/e1c845e96589d5d71476f57ed931f1ac))
```
wget https://raw.githubusercontent.com/vitkhab/gce_test/c98d97ea79bacad23fd26106b52dee0d21144944/.travis.yml
# генерируем ключ для подключения по SSH
ssh-keygen -t rsa -f google_compute_engine -C 'travis' -q -N ''
# Создаем ключ в метадате проекта infra в GCP

# Должен быть предварительно создан сервисный аккаунт и скачаны креды (credentials.json)
# Должна быть предварительно подключена данная репа в тревисе
travis encrypt GCE_SERVICE_ACCOUNT_EMAIL='ci-test@infra-179032.iam.gserviceaccount.com' --add
# Не рекомендуется выполнять следующий шаг, так как пушится абсолютный локальный путь. Оставлен для примера
travis encrypt GCE_CREDENTIALS_FILE="$(pwd)/credentials.json" --add
travis encrypt GCE_PROJECT_ID='infra-179032' --add

# шифруем файлы
tar cvf secrets.tar credentials.json google_compute_engine
# Интеграция с travis-ci.org
travis login
# Альтернативно с travis-ci.com
# travis login --pro
travis encrypt-file secrets.tar --add

# пушим и проверяем изменения
git commit -m 'Added Travis integration'
git push
```
Так как в Travis CI секреты отправляются в зашифрованном виде и в архиые, необходимо расположить их по нужным местам
```
# .travis.yml
...
- tar xvf secrets.tar
# GCE ищет приватный SSH-ключ по этому пути
- mv google_compute_engine ~/.ssh/google_compute_engine
# Расположение сервисных аутентификационных данных GCE
- export GCE_CREDENTIALS_FILE="$(pwd)/credentials.json"
```
Для тестирования роли использовать следующие шаги
```
- pip install -r requirements.txt
- molecule create
- molecule converge
- molecule verify
- molecule destroy
```
