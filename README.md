# anubis
Build out a nexus:oss Repository/Registry Server

yum install -y curl tmux python-pip python-dev build-essential lynx elinks tree zip unzip java-1.8.0-openjdk ctop lvm2

adduser devops
cd /home/devops
mkdir .ssh

#As devops user
# ssh-keygen -t rsa -b 1024 -C "ioexcept@gmail.com"
# save as : ~/.ssh/nexus_devops

cat << EOF > /home/devops/.ssh/config
Host *
    StrictHostKeyChecking no

Host github.com
        User git
        PreferredAuthentications publickey
        IdentityFile ~/.ssh/nexus_devops
EOF

wget https://raw.githubusercontent.com/ssgeejr/config/master/.tmux.conf -O /etc/tmux.conf

curl -fsSL https://get.docker.com/ | sh

pip install -U pip
pip install --ignore-installed docker-compose

systemctl start docker
systemctl enable docker

usermod -aG docker devops

mkdir /opt/nexus
pvcreate /dev/xvdb 
vgcreate nexusvg /dev/xvdb
lvcreate --name lv1 -l 100%FREE nexusvg
mkfs.ext4 /dev/nexusvg/lv1
mkdir /opt/nexus
mount /dev/nexusvg/lv1 /opt/nexus
echo "/dev/nexusvg/lv1 /opt/nexus ext4 defaults 0 0" >> /etc/fstab

chgrp -R devops /opt
chown devops:devops /opt/nexus

#as devops
cd /opt/nexus
mkdir data

### POSSIBLY NEEDED ... 
```
/etc/docker/daemon.json:

{
  "insecure-registries": [
    "your-repo:8082",
    "your-repo:8083"
  ],
  "disable-legacy-registry": true
}
```

cat << EOF > docker-compose.yml
#docker -ti run -d -p 80:8081 --name nexus -v /opt/nexus/data:/nexus-data sonatype/nexus:oss
version: '3'
services:
    nginx:
        image: sonatype/nexus:oss
        container_name: nexusoss
        ports:
            - "80:8081"
        tty: true
        stdin_open: true
        restart: always
        volumes:
            - /opt/nexus/data:/nexus-data
 
EOF



