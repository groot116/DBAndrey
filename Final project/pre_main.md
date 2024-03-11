“Бесшовная миграция шардированного и реплицированного кластера с версии 3.2/3.6/4.0/4.4 на актуальную версию монги.” (подготовительный этап)


Заходим в репозиторий перконы https://repo.percona.com/
Проверям какие версии есть, что есть версия минимальная в теории, которая может стоять у нас на проде, версия 4.4. Далее будем обновляться до последней версии в репозитории 7.0.

Cоздаем 4 вм c Ubuntu22 в YandexCloud
vm1: mongo1 (2vCPU, 2GB ram)


Подготовительная работа
1.1.Установка Yandex Cloud (CLI)
https://cloud.yandex.ru/ru/docs/cli/quickstart#windows_1

Для установки с помощью PowerShell:
#iex (New-Object System.Net.WebClient).DownloadString('https://storage.yandexcloud.net/yandexcloud-yc/install.ps1')

Получение OAuth-токена
https://oauth.yandex.ru/verification_code#access_token=**************&token_type=bearer&expires_in=31536000



начать настройку профиля CLI
#yc init
Your profile default Compute zone has been set to 'ru-central1-a'.
#yc config list

token: y0_AgAAAAA_FzUYAATuwQAAAAD9HgWHAABOwTYbMEhI_6CeKTdn1r1cGJXNtA
cloud-id: b1g11c516j07ju6qv5f3
folder-id: b1gmtrf647q735i4l8uq
compute-default-zone: ru-central1-a

создайте облачную сеть my-yc-network
#yc vpc network create --name my-yc-network --labels my-label=my-value --description "mongo yc"

id: enplu14q4n6bp8vngre3
folder_id: b1gmtrf647q735i4l8uq
created_at: "2024-03-04T23:40:42Z"
name: my-yc-network
description: mongo yc
labels:
  my-label: my-value
default_security_group_id: enp661a74g7av9cp3igk

создайте подсеть mongo-ru-central1-a в облачной сети my-yc-network
yc vpc subnet create --name mongo-ru-central1-a --network-name my-yc-network --zone 'ru-central1-a' --range 10.128.0.0/24	

id: e9bjtj5r8vrdbvetpl2v
folder_id: b1gmtrf647q735i4l8uq
created_at: "2024-03-04T23:49:24Z"
name: mongo-ru-central1-a
network_id: enplu14q4n6bp8vngre3
zone_id: ru-central1-a
v4_cidr_blocks:
  - 10.128.0.0/24

yc vpc network list
yc vpc network list --format yaml




Настройка ssh ключей для подключения к вм в облаке
https://cloud.yandex.ru/ru/docs/compute/operations/vm-connect/ssh
Создайте новый ключ
#ssh-keygen -t ed25519
kPJf9HfcbXYbuZMC
Your identification has been saved in C:\Users\admkaa/.ssh/id_ed25519.
Your public key has been saved in C:\Users\admkaa/.ssh/id_ed25519.pub.
The key fingerprint is:
SHA256:3CFn8EKRWDCymsVctsZW4qLBQqy9kZdGmpoF1dAsU5g alfaleasing\admkaa@AMMOSN1378

Создание вм в YC
https://cloud.yandex.ru/ru/docs/cli/cli-ref/managed-services/compute/instance/create

yc compute instance create –help


yc compute instance create --name mongo1 –hostname mongo1 --network-interface subnet-name= mongo-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a --ssh-key ~/.ssh/id_ed25519.pub

#работает
yc compute instance create --hostname mongo1 --network-interface subnet-name=mongo-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a

yc compute instance create --name mongo1 --hostname mongo1 --network-interface subnet-name=mongo-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a

yc compute instance create --name mongo1 --hostname mongo1 --cores 2 --memory 2G --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts --network-interface subnet-name=mongo-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a

yc compute instance create --name mongo1 --hostname mongo1 --cores 2 --memory 2G --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts --network-interface subnet-name=mongo-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a --ssh-key C:/Distr/.ssh/id_ed25519.pub


yc compute instance get mongo1

primary_v4_address:
      address: 10.128.0.34
      one_to_one_nat:
        address: 158.160.39.38
        ip_version: IPV4

yc compute instance get --full mongo1


#connect to VM
ssh yc-user@158.160.39.38

ssh -i "C:\Distr\.ssh\id_ed25519"  yc-user@158.160.39.38

sudo su


yc compute instance delete mongo1


Выполняем первый этап на вм mongo1 (создание вм и подключение к вм выполнили выше)



1.	Поставить standalone mongodb 4.4 и провести апгрейд версии до 7.0. 
На данном этапе мы проведем апгред с 4.4 сразу до 7.0 и увидим, что это приведет к неработоспособному состоянию монги. А также далее проведем серию апгрейдов с 4.4 до 5.0, с 5.0 до 6.0, с 6.0 до 7.0  с работоспособным конечным состоянием монги.
wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
sudo percona-release enable psmdb-44 release
sudo apt update
List available versions:
sudo apt-cache madison percona-server-mongodb


sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-44 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb

dpkg -l |grep mongo
 
mongod --version
sudo systemctl status mongod
 
cat /etc/mongod.conf
mongo
show dbs
Изменяем конфиг, как будет далее в репликасетах и шардах
Проверяем наличие процесса монги
hostname; sudo ps -aef | grep mongo | grep -v grep
Убиваем процесс монги
hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print $2}' | sudo xargs kill -9
Cоздадим каталог монги
sudo rm -rf /home/mongo && sudo mkdir -p /home/mongo/dbc && sudo chmod 777 /home/mongo/dbc
hostname; mongod --bind_ip localhost,$(hostname) --dbpath /home/mongo/dbc --port 27001 --fork --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid
mongo --port 27001 --host mongo1
Включение аутентификации
sudo mkdir /home/mongo/mongo-security && sudo chmod 777 /home/mongo/mongo-security
openssl rand -base64 756 > /home/mongo/mongo-security/keyfile
chmod 400 /home/mongo/mongo-security/keyfile
Создаем привилегированных пользователей монги
mongo --port 27001
use admin
--Администратор кластера
db.createUser({user: "UserClusterAdmin",pwd: "Andrey@123", roles: [ "clusterAdmin" ]})
--Владелец всех БД
db.createUser({user: "UserdbOwner",pwd: "Andrey@123", roles: [ { role: "dbOwner", db: "*" } ]})
-- Супер ROOT
db.createUser({user: "UserRoot",pwd: "Andrey@123", roles: [ "root" ]})
show users;
запускаем с аутентификацией И КЛЮЧОМ
hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print $2}' | sudo xargs kill -9
hostname; sudo ps -aef | grep mongo | grep -v grep
hostname; mongod --auth --keyFile /home/mongo/mongo-security/keyfile --bind_ip localhost,$(hostname) --dbpath /home/mongo/dbc --port 27001 --fork --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid
mongo --port 27001 -u "UserClusterAdmin" -p Andrey@123 --authenticationDatabase "admin"
mongo --port 27001 -u "UserRoot" -p Andrey@123 --authenticationDatabase "admin"
mongo --port 27001 -u "UserdbOwner" -p Andrey@123 --authenticationDatabase "admin"

#db.dropUser("UserClusterAdmin", {w: "majority", wtimeout: 5000});
#db.dropUser("UserRoot", {w: "majority", wtimeout: 5000});
#db.dropUser("UserdbOwner", {w: "majority", wtimeout: 5000});

Залить данные (коллекция котировок с nasdaq 750Мб, 4,3кк записей)
wget https://dl.dropboxusercontent.com/s/p75zp1karqg6nnn/stocks.zip
/home/yc-user/stocks.zip
sudo apt install unzip
unzip -qo stocks.zip 
cd dump/stocks/
ls -l
 
cd /home/yc-user/dump/stocks/
mongorestore --port 27001 -u "UserRoot" -p Andrey@123 --authenticationDatabase "admin"  --db nasdaq values.bson
4308303 document(s) restored successfully. 0 document(s) failed to restore.

Обновление до 7.0
Прежде чем начать обновление, обновите файл конфигурации MongoDB ( /etc/mongod.conf), включив в него следующие параметры.
processManagement:
   fork: true
   pidFilePath: /var/run/mongod.pid

Выполнить бэкап 
Cоздадим каталог для бэкапов
sudo mkdir -p /home/mongo/backups && sudo chmod 777 /home/mongo/backups
mongo --port 27001 -u "UserRoot" -p Andrey@123 --authenticationDatabase "admin"
use admin
--УЗ для РК
db.createUser({user: "UserBackup",pwd: "secretpwd",roles: [ { role: "backup", db: "admin" } ]})

mongodump --uri "mongodb://UserBackup:secretpwd@localhost:27001/nasdaq?authSource=admin" --out /home/mongo/backups/$(date +'%m-%d-%y')
-- done dumping nasdaq.values (4308303 documents)

sudo systemctl status mongod
hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print $2}' | sudo xargs kill -9
hostname; sudo ps -aef | grep mongo | grep -v grep
sudo mv /etc/mongod.conf /etc/mongod.conf.bkp

sudo dpkg -l | grep mongod
-- Remove the installed packages:

sudo apt remove percona-server-mongodb* 

Удалить пакет, помеченный dpkg как rc.
Для percona-server удаляем файлы конфигурации и двоичные файлы.
dpkg --purge percona-server-mongodb-server 
Для percona-backup удаляем только двоичные файлы пакета. Конфигурационные файлы пакета не удаляются.
dpkg --purge percona-backup-mongodb
Remove log files:
sudo rm -r /var/log/mongodb
sudo rm -r /home/mongo/dbc/dbc.log*

Устанавливаем пакеты Percona Server for MongoDB 7.0

wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
sudo percona-release enable psmdb-70 release
sudo apt update
List available versions:
sudo apt-cache madison percona-server-mongodb


sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-70 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb

sudo dpkg -l | grep mongod
sudo systemctl status mongod
hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print $2}' | sudo xargs kill -9
hostname; sudo ps -aef | grep mongo | grep -v grep
Шаг, по инструкции но для fork процесса не надо делать
--sudo chown -R mongod:mongod /home/mongo/dbc 

«видим цель не видим преград»=))	
hostname; mongod --bind_ip localhost,$(hostname) --dbpath /home/mongo/dbc --port 27001 --fork --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid
Ошибка:
mongo1
about to fork child process, waiting until server is ready for connections.
forked process: 4505
ERROR: child process failed, exited with 1
To see additional information in this output, start without the "--fork" option.

hostname; mongod --bind_ip localhost,$(hostname) --dbpath /home/mongo/dbc --port 27001 --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid
Ошибка:
mongo1
{"t":{"$date":"2024-03-06T17:06:49.785Z"},"s":"I",  "c":"CONTROL",  "id":20697,   "ctx":"main","msg":"Renamed existing log file","attr":{"oldLogPath":"/home/mongo/dbc/dbc.log","newLogPath":"/home/mongo/dbc/dbc.log.2024-03-06T17-06-49"}}

hostname; mongod --bind_ip localhost,$(hostname) --dbpath /home/mongo/dbc --port 27001 --pidfilepath /home/mongo/dbc/dbc.pid
Ошибка:
mongo1
{"t":{"$date":"2024-03-06T17:32:21.846+00:00"},"s":"F",  "c":"CONTROL",  "id":20573,   "ctx":"initandlisten","msg":"Wrong mongod version","attr":{"error":"UPGRADE PROBLEM: Found an invalid featureCompatibilityVersion document (ERROR: Location4926900: Invalid featureCompatibilityVersion document in admin.system.version: { _id: \"featureCompatibilityVersion\", version: \"4.4\" }. See https://docs.mongodb.com/master/release-notes/6.0-compatibility/#feature-compatibility. :: caused by :: Invalid feature compatibility version value '4.4'; expected '6.0' or '6.3' or '7.0'. See https://docs.mongodb.com/master/release-notes/6.0-compatibility/#feature-compatibility.). If the current featureCompatibilityVersion is below 6.0, see the documentation on upgrading at https://docs.mongodb.com/master/release-notes/6.0/#upgrade-procedures."}}
{"t":{"$date":"2024-03-06T17:32:21.846+00:00"},"s":"I",  "c":"REPL",     "id":4784900, "ctx":"initandlisten","msg":"Stepping down the ReplicationCoordinator for shutdown","attr":{"waitTimeMillis":15000}}
Ошибка связана с тем, что версия mongodb несовместима с файлами данных, смонтированными в формате /data/db.
Разрыв между версиями слишком велик. Вам нужно будет обновить его поэтапно:
•	От 4,4 до 5,0 https://www.mongodb.com/docs/v6.0/release-notes/5.0-upgrade-replica-set/
•	От 5.0 до 6.0 https://www.mongodb.com/docs/v6.0/release-notes/6.0-upgrade-replica-set/
Если размер набора данных позволяет, вы можете сократить весь процесс, создав резервную копию данных из версии 4.4, запустив версию 6 с новым пустым томом, подключенным к /data/db, и восстановив базу данных из резервной копии.

На этот случай у нас есть бэкап в /home/mongo/backups/.
Чистим каталог с данными
sudo rm -rf /home/mongo/dbc/*
hostname; mongod --bind_ip localhost,$(hostname) --dbpath /home/mongo/dbc --port 27001 --fork --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid

hostname; sudo ps -aef | grep mongo | grep -v grep
--В 7.0 mongosh вместо shellника mongo
mongosh --port 27001
use admin
--Администратор кластера
db.createUser({user: "UserClusterAdmin",pwd: "Andrey@123", roles: [ "clusterAdmin" ]})
--Владелец всех БД
db.createUser({user: "UserdbOwner",pwd: "Andrey@123", roles: [ { role: "dbOwner", db: "*" } ]})
-- Супер ROOT
db.createUser({user: "UserRoot",pwd: "Andrey@123", roles: [ "root" ]})
--УЗ для РК
db.createUser({user: "UserBackup",pwd: "secretpwd",roles: [ { role: "backup", db: "admin" } ]})
show users;

hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print $2}' | sudo xargs kill -9
hostname; sudo ps -aef | grep mongo | grep -v grep

hostname; mongod --auth --keyFile /home/mongo/mongo-security/keyfile --bind_ip localhost,$(hostname) --dbpath /home/mongo/dbc --port 27001 --fork --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid

mongorestore --port 27001 -u "UserRoot" -p Andrey@123 --authenticationDatabase "admin"  --db nasdaq_new --drop /home/mongo/backups/03-06-24/nasdaq/
4308303 document(s) restored successfully. 0 document(s) failed to restore.
mongosh --port 27001 -u "UserRoot" -p Andrey@123 --authenticationDatabase "admin"
test> show dbs;
admin       180.00 KiB
config      108.00 KiB
local        72.00 KiB
nasdaq_new  225.62 MiB

Теперь проделаем последовательные скользящие миграции только от версии 4.4 до 5.0 и от 5.0 до 6.0. А также от 6.0 до 7.0.
 Делаем полную очистку пакетов mongodb
hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print $2}' | sudo xargs kill -9
hostname; sudo ps -aef | grep mongo | grep -v grep
sudo dpkg -l | grep mongod
sudo apt remove percona-server-mongodb* && sudo apt remove percona-mongodb* && sudo apt remove percona-backup* && dpkg --purge percona-server-mongodb-server && dpkg --purge percona-backup-mongodb && sudo rm -rf /home/mongo/dbc/* && sudo percona-release disable all && apt-get update

и установку версии 4.4
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-44 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb
sudo dpkg -l | grep mongod

sudo systemctl status mongod
sudo systemctl stop mongod

hostname; mongod --bind_ip localhost,$(hostname) --dbpath /home/mongo/dbc --port 27001 --fork --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid
hostname; sudo ps -aef | grep mongo | grep -v grep
mongo --port 27001
use admin
--Администратор кластера
db.createUser({user: "UserClusterAdmin",pwd: "Andrey@123", roles: [ "clusterAdmin" ]})
--Владелец всех БД
db.createUser({user: "UserdbOwner",pwd: "Andrey@123", roles: [ { role: "dbOwner", db: "*" } ]})
-- Супер ROOT
db.createUser({user: "UserRoot",pwd: "Andrey@123", roles: [ "root" ]})
--УЗ для РК
db.createUser({user: "UserBackup",pwd: "secretpwd",roles: [ { role: "backup", db: "admin" } ]})
show users;

hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print $2}' | sudo xargs kill -9
hostname; sudo ps -aef | grep mongo | grep -v grep

hostname; mongod --auth --keyFile /home/mongo/mongo-security/keyfile --bind_ip localhost,$(hostname) --dbpath /home/mongo/dbc --port 27001 --fork --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid

cd /home/yc-user/dump/stocks/
mongorestore --port 27001 -u "UserRoot" -p Andrey@123 --authenticationDatabase "admin"  --db nasdaq values.bson
mongo --port 27001 -u "UserRoot" -p Andrey@123 --authenticationDatabase "admin"

Выполняем обновление с версии 4.4 до версии 5.0.
Удаляем версию 4.4
hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print $2}' | sudo xargs kill -9
hostname; sudo ps -aef | grep mongo | grep -v grep

•	apt remove will only remove the packages and leave the configuration and data files.
•	apt purge will remove all the packages with configuration files and data.

sudo apt purge percona-server-mongodb* && sudo apt purge percona-mongodb* && sudo apt purge percona-backup* && dpkg --purge percona-server-mongodb-server && sudo percona-release disable all && apt-get update && sudo apt purge percona-backup-mongodb

dpkg --purge percona-backup-mongodb
sudo dpkg -l | grep mongod
sudo apt-cache madison percona-server-mongodb

Ставим версию 5.0
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-50 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb
sudo dpkg -l | grep mongod
mongod --version

sudo systemctl status mongod
sudo systemctl stop mongod

hostname; mongod --auth --keyFile /home/mongo/mongo-security/keyfile --bind_ip localhost,$(hostname) --dbpath /home/mongo/dbc --port 27001 --fork --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid
mongo --port 27001 -u "UserRoot" -p Andrey@123 --authenticationDatabase "admin"

Выполняем обновление с версии 5.0 до версии 6.0.
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
db.adminCommand( { setFeatureCompatibilityVersion: "5.0" } )

Удаляем версию 5.0
hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print $2}' | sudo xargs kill -9
hostname; sudo ps -aef | grep mongo | grep -v grep
sudo apt purge percona-server-mongodb* && sudo apt purge percona-mongodb* && sudo apt purge percona-backup* && dpkg --purge percona-server-mongodb-server && dpkg --purge percona-backup-mongodb && sudo percona-release disable all && apt-get update

sudo dpkg -l | grep mongod

sudo apt-cache madison percona-server-mongodb

Ставим версию 6.0
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-60 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb

sudo dpkg -l | grep mongod
mongod --version
sudo systemctl status mongod
sudo systemctl stop mongod

hostname; mongod --auth --keyFile /home/mongo/mongo-security/keyfile --bind_ip localhost,$(hostname) --dbpath /home/mongo/dbc --port 27001 --fork --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid
mongosh --port 27001 -u "UserRoot" -p Andrey@123 --authenticationDatabase "admin"
show dbs

Выполняем обновление с версии 6.0 до версии 7.0.
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
db.adminCommand( { setFeatureCompatibilityVersion: "6.0" } )

Удаляем версию 6.0
hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print $2}' | sudo xargs kill -9
sudo apt purge percona-server-mongodb* && sudo apt purge percona-mongodb* && sudo apt purge percona-backup* && dpkg --purge percona-server-mongodb-server && dpkg --purge percona-backup-mongodb && sudo percona-release disable all && apt-get update

sudo dpkg -l | grep hostname; sudo ps -aef | grep mongo | grep -v grep
mongod
sudo apt-cache madison percona-server-mongodb

Ставим версию 7.0
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-70 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb

sudo dpkg -l | grep mongod
mongod --version
sudo systemctl status mongod
sudo systemctl stop mongod

hostname; mongod --auth --keyFile /home/mongo/mongo-security/keyfile --bind_ip localhost,$(hostname) --dbpath /home/mongo/dbc --port 27001 --fork --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid
mongosh --port 27001 -u "UserRoot" -p Andrey@123 --authenticationDatabase "admin"
show dbs

db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
db.adminCommand( { setFeatureCompatibilityVersion: "7.0" } )

MongoServerError: Once you have upgraded to 7.0, you will not be able to downgrade FCV and binary version without support assistance. Please re-run this command with 'confirm: true' to acknowledge this and continue with the FCV upgrade.
db.adminCommand( { setFeatureCompatibilityVersion: "7.0", confirm: true } )
use nasdaq
show collections


yc compute instance delete mongo1
На этом первый этап тестирований и проверок закончен успешно.







Используемые инструкции:
1.	https://docs.percona.com/percona-server-for-mongodb/3.6/install/apt.html
2.	https://docs.percona.com/percona-server-for-mongodb/4.4/install/upgrade-from-mongodb.html
3.	https://docs.percona.com/percona-server-for-mongodb/4.4/install/uninstall.html
4.	https://docs.percona.com/percona-server-for-mongodb/7.0/install/apt.html
5.	https://www.mongodb.com/docs/v6.0/release-notes/5.0-upgrade-replica-set/
6.	https://www.mongodb.com/docs/v6.0/release-notes/6.0-upgrade-replica-set/
7.	https://www.mongodb.com/docs/v6.0/release-notes/5.0-upgrade-sharded-cluster/#std-label-5.0-upgrade-sharded-cluster







