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
158.160.98.98
ssh -i "C:/Distr/.ssh/id_ed25519" yc-user@158.160.98.98
123


#mongo1
ssh-keygen
#/home/yc-user/.ssh
id_rsa
id_rsa.pub

#не работает
ssh-copy-id yc-user@mongo2



vi /home/yc-user/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDIlAm7h46a6WxVfO9eesWQDfoxw8CYvq2oasU7/fRBrRQ9bWe7j0vQJk2OQO3nGeXyuSLKoEW2vOoTTYAhE4W8mNK6p6Y2yktg8S+Z3QlPxZyh/iOimWLlDAfpXNq7c+alXZ5hVYXrIgOpH7LoqImg56XhRH+tCX2egXx+sX0CFQyGPud8fQMTSjNmYQQFbZbaEri7bJAnwptklZ3YPjUnKF4EaXxNbew6JrJFEvUNYrokET6BGcyD66BlTXKwK+kvJE9LXmTnmEpVep+Md1zKCJLUTUtnV5LcxHiSwgz23K7jZZVvTjA31UQT/FTU2NOzY7avL2UvBopZzMw3wcEeOzsihH478VUZL3YaMU32M59qeqG9c1w7Tnfak6KBtQ1h31FwyjTfIW2HfIYnXbMRRoy/VRiw+1gkpqTzkoyS5EmsBYQqYdG3cDe1IEFWa8z94hpI6lgvdUrNdCQD9xrtinO2kbniYoWRI8xHBlOARCWoOBvu9LPOSf1hl/H2kFc= yc-user@mongo1


yc compute instance get mongo2
158.160.33.164
ssh -i "C:/Distr/.ssh/id_ed25519" yc-user@158.160.33.164
123

yc compute instance get mongo3
84.252.130.89
ssh -i "C:/Distr/.ssh/id_ed25519" yc-user@84.252.130.89
123


yc compute instance get mongo4
178.154.200.124
ssh -i "C:/Distr/.ssh/id_ed25519" yc-user@178.154.200.124
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




33333




















