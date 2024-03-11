# **“Бесшовная миграция шардированного и реплицированного кластера с версии 3.2/3.6/4.0/4.4 на актуальную версию монги.” (основной этап)**


Настроить шардированный и реплицированный кластер mongodb и выполнить апгрейд шардов до последней версии mongodb.

Подготовительный этап показал, что на данный момент можно выполнить последовательные скользящие миграции только от версии 4.4 до 5.0, от версии 5.0 до 6.0, от версии 6.0 до 7.0. Это мы и проделаем сегодня.

Кроме того, апгрейд шардов до последней версии mongodb можно выполнить через бэкап-рестор БД, учитывая размер БД и простой системы на весь процесс обновления.  Для этого необходимо выполнить следующие действия:

•	Настроить шардированный кластер на Percona Server for MongoDB 4.4 

•	Настроить резервное копирование и сделать бэкап БД в шардированном кластере

•	Остановить кластер и выполнить апгрейд до Percona Server for MongoDB 7.0 через удаление Percona Server for MongoDB 4.4 с каталогом данных

•	Выполнить восстановление БД из бэкапа на Percona Server for MongoDB 7.0  

**Cоздаем 4 вм c Ubuntu22 в YandexCloud**

vm1: mongo1 (6vCPU, 6GB ram)

vm2: mongo2 (6vCPU, 6GB ram)

vm3: mongo3 (6vCPU, 6GB ram)

vm4: mongo4 (6vCPU, 6GB ram)

**Конфигурация шардированного кластера**

vm1=config1 + primary1 + slave2 + slave3

vm2=config2 + slave1 + primary2 + slave3

vm3=config3 + slave1 + slave2 + primary3 + mongos2

vm4=mongos1


#local workplace

ssh-keygen -t ed25519

Generating public/private ed25519 key pair.
Enter file in which to save the key (C:\Users\admkaa/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\admkaa/.ssh/id_ed25519.
Your public key has been saved in C:\Users\admkaa/.ssh/id_ed25519.p


for ($i = 1; $i -le 4; $i++) {
    yc compute instance create --name mongo$i --hostname mongo$i --cores 6 --memory 6G --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts --network-interface subnet-name=mongo-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a --ssh-key C:/Distr/.ssh/id_ed25519.pub
}


yc compute instance list

+----------------------+--------+---------------+---------+---------------+-------------+
|          ID          |  NAME  |    ZONE ID    | STATUS  |  EXTERNAL IP  | INTERNAL IP |
+----------------------+--------+---------------+---------+---------------+-------------+
| fhm420rs3aflret8sj2h | mongo4 | ru-central1-a | RUNNING | 51.250.86.234 | 10.128.0.11 |
| fhm6qfiqcoa0q1fvh8os | mongo3 | ru-central1-a | RUNNING | 51.250.73.255 | 10.128.0.32 |
| fhm7ibhsr9573o284mkd | mongo2 | ru-central1-a | RUNNING | 51.250.13.75  | 10.128.0.21 |
| fhmrd8t5ho37p7as0p7i | mongo1 | ru-central1-a | RUNNING | 51.250.90.20  | 10.128.0.26 |
+----------------------+--------+---------------+---------+---------------+-------------+


for ($i = 1; $i -le 4; $i++) {
    yc compute instance stop --name mongo$i
}


for ($i = 1; $i -le 4; $i++) {
    yc compute instance start --name mongo$i
}


for ($i = 1; $i -le 4; $i++) {
    yc compute instance delete --name mongo$i
}




yc compute instance get mongo1
51.250.90.20
ssh -i "C:/Distr/.ssh/id_ed25519" yc-user@51.250.90.20
123


#mongo1
ssh-keygen
#/home/yc-user/.ssh
id_rsa
id_rsa.pub

#не работает c mongo1
ssh-copy-id yc-user@mongo2



vi /home/yc-user/.ssh/id_rsa.pub
ssh-rsa ************** yc-user@mongo1


yc compute instance get mongo2
51.250.13.75
ssh -i "C:/Distr/.ssh/id_ed25519" yc-user@51.250.13.75
123

yc compute instance get mongo3
51.250.73.255
ssh -i "C:/Distr/.ssh/id_ed25519" yc-user@51.250.73.255
123


yc compute instance get mongo4
51.250.86.234
ssh -i "C:/Distr/.ssh/id_ed25519" yc-user@51.250.86.234
123


#mongo1-4

vi /home/yc-user/.ssh/authorized_keys
#добавить с новой строки public key id_rsa.pub с mongo1


#mongo1

sudo apt install sshpass

#работает

sshpass -P assphrase -p "123" ssh yc-user@mongo1 'hostname; ps -aef | grep mongo | grep -v grep'
sshpass -P assphrase -p "123" ssh yc-user@mongo1 'hostname; sudo dpkg -l | grep mongod'




#install percona mongo 4.4 & backup pbm on vm1-4
#mongo1

sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-44 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb
ssh yc-user@mongo2 'sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-44 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb'
ssh yc-user@mongo3 'sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-44 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb'
ssh yc-user@mongo4 'sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-44 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb'


#без запроса пароля

for i in {1..4}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-44 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb"
done


#с запросом пароля

for i in {1..4}
do ssh -p 22 yc-user@mongo$i "sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-44 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb"
done


ps -aef | grep mongo | grep -v grep
sudo dpkg -l | grep mongod
ssh yc-user@mongo1 'ps -aef | grep mongo | grep -v grep'
ssh yc-user@mongo2 'sudo dpkg -l | grep mongod'

#mongo1

for i in {1..4}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "hostname; sudo dpkg -l | grep mongod"
done

#mongo1

for i in {1..4}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "hostname; ps -aef | grep mongo | grep -v grep"
done


#stop mongo service localy

ps -aef | grep mongo | grep -v grep |awk {'print $2'} | sudo xargs kill -9

#stop mongo service 1-4

for i in {1..4}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print \$2}' | sudo xargs kill -9"
done

#create folders 1-4

for i in {1..4}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "hostname; sudo rm -rf /home/mongo && sudo mkdir -p /home/mongo/{dbc1,db1,db2,db3} && sudo chmod 777 /home/mongo/{dbc1,db1,db2,db3}"
done

#check existing folders

for i in {1..4}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "ls -la /home/mongo"
done

#run config servers 1-3

#ok
for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "sudo mongod --configsvr --bind_ip '0.0.0.0' --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid"
done

#check

for i in {1..4}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "hostname; ps -aef | grep mongo | grep -v grep"
done


netstat -tlpn

#initiate config srv on 1 (config server doensn't have an arbitr)

sudo su 
mongo --port 27001 

rs.initiate({"_id" : "RScfg", configsvr: true, members : [{"_id" : 0, host : "mongo1:27001"},{"_id" : 1, host : "mongo2:27001"},{"_id" : 2, host : "mongo3:27001"}]});

#rs.add( { host: "mongo1:27001" } )

use admin
db.createUser({user: "UserClusterAdmin",pwd: "123", roles: [ "clusterAdmin" ]})

--Владелец всех БД

db.createUser({user: "UserdbOwner",pwd: "123", roles: [ { role: "dbOwner", db: "*" } ]})

-- Супер ROOT

db.createUser({user: "UserRoot",pwd: "123", roles: [ "root" ]})

#stop mongo service 1-3
#ok

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print \$2}' | sudo xargs kill -9"
done

#try run mongo with auth

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "hostname; mongod --auth --configsvr --bind_ip --bind_ip 0.0.0.0 --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid"
done
-- BadValue: security.keyFile is required when authorization is enabled with replica sets
						
#create folders for mongo-security

for i in {1..4}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "hostname; sudo mkdir /home/mongo/mongo-security && sudo chmod 777 /home/mongo/mongo-security"
done
 
#generate keyfile on 1-st instance

openssl rand -base64 756 > /home/mongo/mongo-security/keyfile
chmod 400 /home/mongo/mongo-security/keyfile

scp /home/mongo/mongo-security/keyfile yc-user@mongo2:/home/mongo/mongo-security/keyfile
scp /home/mongo/mongo-security/keyfile yc-user@mongo3:/home/mongo/mongo-security/keyfile
scp /home/mongo/mongo-security/keyfile yc-user@mongo4:/home/mongo/mongo-security/keyfile

for i in {2..4}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "hostname; chmod 400 /home/mongo/mongo-security/keyfile"
done

#run mongo with auth

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "hostname; sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --configsvr --bind_ip '0.0.0.0' --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid"
done

#test connection

mongo --port 27001 -u "UserClusterAdmin" -p 123 --authenticationDatabase "admin"


rs.config()
rs.status()

#########################################################################################################

#create replicas on different nodes (no auth)

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'sudo mongod --shardsvr --dbpath /home/mongo/db1 --bind_ip '0.0.0.0' --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/dbrs1.log --pidfilepath /home/mongo/db1/dbrs1.pid;'
done

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'sudo mongod --shardsvr --dbpath /home/mongo/db2 --bind_ip '0.0.0.0' --port 27021 --replSet RS2 --fork --logpath /home/mongo/db2/dbrs2.log --pidfilepath /home/mongo/db2/dbrs2.pid;'
done

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'sudo mongod --shardsvr --dbpath /home/mongo/db3 --bind_ip '0.0.0.0' --port 27031 --replSet RS3 --fork --logpath /home/mongo/db3/dbrs3.log --pidfilepath /home/mongo/db3/dbrs3.pid;'
done

##kill incorrect mongo pocesses on specific ports netstat -tlpn

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "hostname; sudo ps -aef | grep  27011| grep -v grep |awk '{print \$2}' | sudo xargs kill -9"
done

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "hostname; ps -aef | grep  27021| grep -v grep |awk '{print \$2}' | sudo xargs kill -9"
done

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "hostname; ps -aef | grep  27031| grep -v grep |awk '{print \$2}' | sudo xargs kill -9"
done


#create replicasets on mastert nodes

mongo --host mongo1 --port 27011
rs.initiate({"_id" : "RS1", members : [{"_id" : 0, priority : 3, host : "mongo1:27011"},{"_id" : 1, host : "mongo2:27011"},{"_id" : 2, host : "mongo3:27011"}]});
use admin
db.createUser({user: "UserDBAdmin",pwd: "123", roles: [ { role: "dbAdmin", db: "*" } ]})
db.createUser({user: "UserDBRoot",pwd: "123", roles: [ "root" ]})

mongo --host mongo2 --port 27021
rs.initiate({"_id" : "RS2", members : [{"_id" : 0, host : "mongo1:27021"},{"_id" : 1, priority : 3, host : "mongo2:27021"},{"_id" : 2, host : "mongo3:27021"}]});
use admin
db.createUser({user: "UserDBAdmin",pwd: "123", roles: [ { role: "dbAdmin", db: "*" } ]})
db.createUser({user: "UserDBRoot",pwd: "123", roles: [ "root" ]})

mongo --host mongo3 --port 27031
rs.initiate({"_id" : "RS3", members : [{"_id" : 0, host : "mongo1:27031"},{"_id" : 1, host : "mongo2:27031"},{"_id" : 2, priority : 3, host : "mongo3:27031"}]});
use admin
db.createUser({user: "UserDBAdmin",pwd: "123", roles: [ { role: "dbAdmin", db: "*" } ]})
db.createUser({user: "UserDBRoot",pwd: "123", roles: [ "root" ]})

#stop replicas with data

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i "ps -aef | grep shardsvr | grep -v grep | awk '{print \$2}'| sudo xargs kill -9"
done

#start replicas with auth

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db1 --bind_ip '0.0.0.0' --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/dbrs1.log --pidfilepath /home/mongo/db1/dbrs1.pid;'
done

mongo --port 27011 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db2 --bind_ip '0.0.0.0' --port 27021 --replSet RS2 --fork --logpath /home/mongo/db2/dbrs2.log --pidfilepath /home/mongo/db2/dbrs2.pid;'
done

mongo --port 27021 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db3 --bind_ip '0.0.0.0' --port 27031 --replSet RS3 --fork --logpath /home/mongo/db3/dbrs3.log --pidfilepath /home/mongo/db3/dbrs3.pid;'
done

mongo --port 27031 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"

###############################################################################################################################################################################

#create mongos with auth

for i in {3,4}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'sudo mkdir -p /home/mongo/dbms && sudo chmod 777 /home/mongo/dbms'
done

for i in {3,4}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'mongos --keyFile /home/mongo/mongo-security/keyfile --configdb RScfg/mongo1:27001,mongo2:27001,mongo3:27001 --bind_ip '0.0.0.0' --port 27000 --fork --logpath /home/mongo/dbms/dbs.log --pidfilepath /home/mongo/dbms/dbs.pid'
done

#####sharded cluster is ready

mongo --port 27000 --host mongo4 -u "UserRoot" -p 123 --authenticationDatabase "admin"
db.a.insert({"a":1})
MongoBulkWriteError: Database test could not be created :: caused by :: No shards found

##add shards

use admin
sh.addShard("RS1/mongo1:27011,mongo2:27011,mongo3:27011")
sh.addShard("RS2/mongo1:27021,mongo2:27021,mongo3:27021")
sh.addShard("RS3/mongo1:27031,mongo2:27031,mongo3:27031")
sh.status()

-- Для того чтобы реплика была доступна на чтение:

rs.secondaryOk(true)


#get db size

#under yc-user user

for i in {1..4}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'hostname; sudo du -m -s /home/mongo/*'
done



ssh mongo4
#download collection

wget https://dl.dropboxusercontent.com/s/p75zp1karqg6nnn/stocks.zip 
sudo apt install unzip
unzip -qo stocks.zip 
cd dump/stocks/
ls -l

#restore collection

mongorestore --port 27000 -u "UserRoot" -p 123 --authenticationDatabase "admin" values.bson

mongo --port 27000 -u "UserRoot" -p 123 --authenticationDatabase "admin"
db.values.find().limit(1)



db.values.count()
show databases


-- запустим долгий запрос

-- db.values.find({$where: '(this.open - this.close > 100)'},{"stock_symbol":1,"open":1,"close":1})

-- через 20 минут надоело) посмотрим загрузку - так как уехало на 2 RS - то его мастер и будет страдать

-- посмотрим план

use test
db.values.find({$where: '(this.open - this.close > 100)'}).explain("executionStats")

-- и его тоже не смогли

use test
db.values.createIndex({stock_symbol: 1})

-- меньше 1 минуты индекс

-- сделал машинки мощнее и теперь 6 секунд 0_0, была e2_small

sh.enableSharding("test")
sh.shardCollection("test.values",{ stock_symbol: 1 })
sh.status()

-- Тоже самое, но через mapreduce.

-- Работает за 10 секунд

var map=function(){
   if (this.open - this.close > 100)  {
    emit(this.stock_symbol, this.open - this.close);
   }
};

var reduce=function(key, values) {
     return Array.sum(values);
};
db.values.mapReduce(map, reduce, {out: 'o' })
db.o.find()

##############################################################################################################################################

**#percona dump**

pbm

-- https://docs.percona.com/percona-backup-mongodb/initial-setup.html

#add user for every shard and config server

mongo --port 27001 -u "UserRoot" -p 123 --authenticationDatabase "admin"
mongo --host mongo1 --port 27011 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"
mongo --host mongo2 --port 27021 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"
mongo --host mongo3 --port 27031 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"

use admin
db.getSiblingDB("admin").createRole({ "role": "pbmAnyAction",
      "privileges": [
         { "resource": { "anyResource": true },
           "actions": [ "anyAction" ]
         }
      ],
      "roles": []
   });

db.getSiblingDB("admin").createUser({user: "pbmuser",
       "pwd": "secretpwd",
       "roles" : [
          { "db" : "admin", "role" : "readWrite", "collection": "" },
          { "db" : "admin", "role" : "backup" },
          { "db" : "admin", "role" : "clusterMonitor" },
          { "db" : "admin", "role" : "restore" },
          { "db" : "admin", "role" : "pbmAnyAction" }
       ]
    });
exit

#check file

sudo nano /etc/default/pbm-agent


############################################################################################################################################

#check file

sudo nano /etc/default/pbm-agent

#create backup folders

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'sudo mkdir -p /home/mongo/backups && sudo chmod 777 /home/mongo/backups && sudo mkdir -p /home/mongo/pbm && sudo chmod 777 /home/mongo/pbm'
done

#create config file

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'cat > temp.cfg << EOF 
storage:
  type: filesystem
  filesystem:
    path: /home/mongo/backups
EOF
cat temp.cfg | sudo tee -a /home/yc-user/pbm_config.yaml
' 
done

#apply configs

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'pbm config --file /home/yc-user/pbm_config.yaml --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27001/?authSource=admin&replicaSet=RScfg"'
done

-- стандартный путь, описанный в документации как обычно не работает

-- sudo systemctl start pbm-agent

-- sudo systemctl start pbm-agent

-- sudo systemctl status pbm-agent

-- sudo systemctl stop pbm-agent

-- journalctl -u pbm-agent.service

ssh mongo1
export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27001/?authSource=admin&replicaSet=RScfg"
pbm list

#run pbm agent

xVBIpPGc6D7dYCVC

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'nohup pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27001/?authSource=admin&replicaSet=RScfg" > /home/mongo/pbm/agent.$(hostname -s).27001.log 2>&1 &'
done;

-- запускаем агент бэкап для остальных реплик

for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'nohup pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27011/?authSource=admin&replicaSet=RS1" > /home/mongo/pbm/agent.$(hostname -s).27011.log 2>&1 &'
done
for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'nohup pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27021/?authSource=admin&replicaSet=RS2" > /home/mongo/pbm/agent.$(hostname -s).27021.log 2>&1 &'
done
for i in {1..3}
do sshpass -P assphrase -p "123" ssh -p 22 yc-user@mongo$i 'nohup pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27031/?authSource=admin&replicaSet=RS3" > /home/mongo/pbm/agent.$(hostname -s).27031.log 2>&1 &'
done


ps -xf

pbm status
pbm logs
pbm backup 
pbm list


############################################################################################################################################

#После выполнения бэкапа можно провести апгрейды монги


#Апгрейд до версии до версии 5.0

#Проходим по условию актуальной версии 4.4

Чтобы обновить существующее развертывание MongoDB до версии 5.0 , вам необходимо использовать выпуск серии 4.4 .

Для обновления версии, предшествующей серии 4.4 , необходимо последовательно обновлять основные выпуски, пока не будет выполнено обновление до серии 4.4 . 

Например, если вы используете версию 4.2 , вам необходимо сначала выполнить обновление до версии 4.4, прежде чем вы сможете выполнить обновление до версии 5.0 .

Чтобы обновить сегментированный кластер до версии 5.0 , все члены кластера должны быть версии не ниже 4.4 . 

В процессе обновления проверяются все компоненты кластера и выдаются предупреждения, если какой-либо компонент работает под управлением версии более ранней, чем 4.4 .

db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )

Все члены должны возвращать результат, включающий файлы "featureCompatibilityVersion" : { "version" : "4.4" }

Для сегментов и серверов конфигурации убедитесь, что ни один член набора реплик не находится в ROLLBACK рабочем RECOVERING состоянии.

#на mongo3,mongo4 отключаем балансировщик

vm3=config3 + slave1 + slave2 + primary3 + mongos2
vm4=mongos1

ssh -p 22 yc-user@mongo3
mongo --port 27001 -u "UserRoot" -p 123 --authenticationDatabase "admin"
sh.stopBalancer()
sh.getBalancerState()

ssh -p 22 yc-user@mongo4
mongo --port 27001 -u "UserRoot" -p 123 --authenticationDatabase "admin"
sh.stopBalancer()
sh.getBalancerState()


#Обновляйте второстепенные члены набора реплик по одному:

vm1=config1 + primary1 + slave2 + slave3
vm2=config2 + slave1 + primary2 + slave3
vm3=config3 + slave1 + slave2 + primary3 + mongos2

#в случае нахождения всех участников на разных вм, нужно выбрать стратегию обновления сначала второстепенных участников и далее обновления первичных путем переключения: rs.stepDown()

#Учитывая специфику нашей конфигурации шардированного и реплецированного кластера, то останавливать процессы и обновлять монгу мы будем в следующем порядке:

#mongo3

ssh -p 22 yc-user@mongo3

#убиваем процессы монги

hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print $2}' | sudo xargs kill -9
hostname; sudo dpkg -l | grep hostname; sudo ps -aef | grep mongo | grep -v grep

#удаляем старые пакеты mongodb 4.4

sudo apt purge percona-server-mongodb* && sudo apt purge percona-mongodb* && dpkg --purge percona-server-mongodb-server && sudo percona-release disable all && apt-get update
sudo apt-cache madison percona-server-mongodb

#ставим пакеты mongodb 5.0

sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-50 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb

#запускаем процессы монги как было ранее на вм

sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --configsvr --bind_ip '0.0.0.0' --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid;
mongo --port 27001 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"
sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db1 --bind_ip '0.0.0.0' --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/dbrs1.log --pidfilepath /home/mongo/db1/dbrs1.pid;
mongo --port 27011 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"
sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db2 --bind_ip '0.0.0.0' --port 27021 --replSet RS2 --fork --logpath /home/mongo/db2/dbrs2.log --pidfilepath /home/mongo/db2/dbrs2.pid;
mongo --port 27021 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"
sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db3 --bind_ip '0.0.0.0' --port 27031 --replSet RS3 --fork --logpath /home/mongo/db3/dbrs3.log --pidfilepath /home/mongo/db3/dbrs3.pid;
mongo --port 27031 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"
sh.status()
rs.status()

#Обновляйте шарды один за другим

#В нашем случае далее обновим монгу на mongo2, далее на mongo1, затем уже перейдем к mongos на вм mongo4, mongo3


#mongo2

ssh -p 22 yc-user@mongo2

#убиваем процессы монги

hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print $2}' | sudo xargs kill -9
hostname; sudo dpkg -l | grep hostname; sudo ps -aef | grep mongo | grep -v grep

#удаляем старые пакеты mongodb 4.4

sudo apt purge percona-server-mongodb* && sudo apt purge percona-mongodb* && dpkg --purge percona-server-mongodb-server && sudo percona-release disable all && apt-get update
sudo apt-cache madison percona-server-mongodb

#ставим пакеты mongodb 5.0

sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-50 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb
#запускаем процессы монги как было ранее на вм
sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --configsvr --bind_ip '0.0.0.0' --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid;
mongo --port 27001 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"
sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db1 --bind_ip '0.0.0.0' --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/dbrs1.log --pidfilepath /home/mongo/db1/dbrs1.pid;
mongo --port 27011 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"
sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db2 --bind_ip '0.0.0.0' --port 27021 --replSet RS2 --fork --logpath /home/mongo/db2/dbrs2.log --pidfilepath /home/mongo/db2/dbrs2.pid;
mongo --port 27021 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"
sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db3 --bind_ip '0.0.0.0' --port 27031 --replSet RS3 --fork --logpath /home/mongo/db3/dbrs3.log --pidfilepath /home/mongo/db3/dbrs3.pid;
mongo --port 27031 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"
sh.status()
rs.status()



#mongo1
ssh -p 22 yc-user@mongo1
#убиваем процессы монги
hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print $2}' | sudo xargs kill -9
hostname; sudo dpkg -l | grep hostname; sudo ps -aef | grep mongo | grep -v grep
#удаляем старые пакеты mongodb 4.4
sudo apt purge percona-server-mongodb* && sudo apt purge percona-mongodb* && dpkg --purge percona-server-mongodb-server && sudo percona-release disable all && apt-get update
sudo apt-cache madison percona-server-mongodb
#ставим пакеты mongodb 5.0
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-50 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb
#запускаем процессы монги как было ранее на вм
sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --configsvr --bind_ip '0.0.0.0' --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid;
mongo --port 27001 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"
sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db1 --bind_ip '0.0.0.0' --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/dbrs1.log --pidfilepath /home/mongo/db1/dbrs1.pid;
mongo --port 27011 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"
sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db2 --bind_ip '0.0.0.0' --port 27021 --replSet RS2 --fork --logpath /home/mongo/db2/dbrs2.log --pidfilepath /home/mongo/db2/dbrs2.pid;
mongo --port 27021 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"
sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db3 --bind_ip '0.0.0.0' --port 27031 --replSet RS3 --fork --logpath /home/mongo/db3/dbrs3.log --pidfilepath /home/mongo/db3/dbrs3.pid;
mongo --port 27031 -u "UserDBRoot" -p 123 --authenticationDatabase "admin"
sh.status()
rs.status()


#mongo4 (mongos1)
ssh -p 22 yc-user@mongo4
#убиваем процессы монги
hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print $2}' | sudo xargs kill -9
hostname; sudo dpkg -l | grep hostname; sudo ps -aef | grep mongo | grep -v grep
#удаляем старые пакеты mongodb 4.4
sudo apt purge percona-server-mongodb* && sudo apt purge percona-mongodb* && dpkg --purge percona-server-mongodb-server && sudo percona-release disable all && apt-get update
sudo apt-cache madison percona-server-mongodb
#ставим пакеты mongodb 5.0
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-50 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb
#запускаем процессы монги как было ранее на вм
mongos --keyFile /home/mongo/mongo-security/keyfile --configdb RScfg/mongo1:27001,mongo2:27001,mongo3:27001 --bind_ip '0.0.0.0' --port 27000 --fork --logpath /home/mongo/dbms/dbs.log --pidfilepath /home/mongo/dbms/dbs.pid
mongo --port 27000 -u "UserRoot" -p 123 --authenticationDatabase "admin"
sh.startBalancer()


#mongo3 (mongos2)
ssh -p 22 yc-user@mongo3
mongos --keyFile /home/mongo/mongo-security/keyfile --configdb RScfg/mongo1:27001,mongo2:27001,mongo3:27001 --bind_ip '0.0.0.0' --port 27000 --fork --logpath /home/mongo/dbms/dbs.log --pidfilepath /home/mongo/dbms/dbs.pid
mongo --port 27000 -u "UserRoot" -p 123 --authenticationDatabase "admin"
sh.startBalancer()

#Поднятие версии в кластере до 5.0
#Установка FeatureCompatibilityVersion (fCV): «5.0» неявно выполняет replSetReconfigдля каждого сегмента, чтобы добавить termполе в документ конфигурации реплики сегмента, и блокируется до тех пор, пока новая конфигурация не распространится на большинство членов набора реплик.
#mongo4
ssh -p 22 yc-user@mongo4
mongo --port 27000 -u "UserRoot" -p 123 --authenticationDatabase "admin"
db.adminCommand( { setFeatureCompatibilityVersion: "5.0" } )


#mongo3
ssh -p 22 yc-user@mongo3
mongo --port 27000 -u "UserRoot" -p 123 --authenticationDatabase "admin"
db.adminCommand( { setFeatureCompatibilityVersion: "5.0" } )



#Далее аналогично проводим апгрейд от версии 5.0 до 6.0
#учитываем, что mongo --> mongosh для подключения к экземпляру монги


#Далее аналогично проводим апгрейд от версии 6.0 до 7.0
#учитываем, что mongo --> mongosh для подключения к экземпляру монги
#не забываем выполнить команду db.adminCommand( { setFeatureCompatibilityVersion: "7.0" } )




Используемые инструкции:
1.	https://docs.percona.com/percona-server-for-mongodb/3.6/install/apt.html
2.	https://docs.percona.com/percona-server-for-mongodb/4.4/install/upgrade-from-mongodb.html
3.	https://docs.percona.com/percona-server-for-mongodb/4.4/install/uninstall.html
4.	https://docs.percona.com/percona-server-for-mongodb/7.0/install/apt.html
5.	https://www.mongodb.com/docs/v6.0/release-notes/5.0-upgrade-replica-set/
6.	https://www.mongodb.com/docs/v6.0/release-notes/6.0-upgrade-replica-set/
7.	https://www.mongodb.com/docs/v6.0/release-notes/5.0-upgrade-sharded-cluster/#std-label-5.0-upgrade-sharded-cluster

