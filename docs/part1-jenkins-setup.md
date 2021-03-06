# Part 1: Jenkins Setup

## Jenkins Master

1. Configure locale in ubuntu
```
curl -sL https://raw.githubusercontent.com/prabhatpankaj/ubuntustarter/master/initial.sh | sh
```

2. Install Docker
```
sudo su

cd

curl -sSL https://get.docker.com/ | sh
```

3. Add `ubuntu` user to `docker` group
```
usermod -aG docker ubuntu

exit    # make sure after exit user should be ubuntu 

newgrp docker

sudo service docker restart

docker ps # test docker is running or not

```

4. Run Jenkins container and tail logs
```
mkdir ~/jenkins
cd ~/jenkins
docker pull jenkins/jenkins:lts
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v $(pwd):/var/jenkins_home --restart always jenkins/jenkins:lts
docker logs -f jenkins
```

5. Open web UI to jenkins and install following plugins:
- AnsiColor
- Blue Ocean
- Self-Organizing Swarm Plug-in Modules
- Throttle Concurrent Builds

## Jenkins Agent

1. Configure locale in ubuntu
```
curl -sL https://raw.githubusercontent.com/prabhatpankaj/ubuntustarter/master/initial.sh | sh
```

2. Install Docker
```
sudo su

cd

curl -sSL https://get.docker.com/ | sh
```

3. Add `ubuntu` user to `docker` group
```
usermod -aG docker ubuntu

exit    # make sure after exit user should be ubuntu 

newgrp docker

sudo service docker restart

docker ps # test docker is running or not

```

4. Install `openjdk-8-jdk`
```
sudo apt install openjdk-8-jdk
```

5. Download Jenkins Swarm Client (https://wiki.jenkins.io/display/JENKINS/Swarm+Plugin)
```
sudo -s
mkdir -p /usr/local/jenkins
cd /usr/local/jenkins
wget https://repo.jenkins-ci.org/releases/org/jenkins-ci/plugins/swarm-client/3.7/swarm-client-3.7.jar
touch swarm.sh
chmod +x swarm.sh
```

6. Edit the file `/usr/local/jenkins/swarm.sh` so that it contains the following:
```
#!/bin/bash
cd $(dirname $0)

JENKINS_IP="3.83.216.238"
USERNAME="admin"
PASSWORD="pass1234"

java -jar swarm-client-3.7.jar -name "$(hostname)" -executors 8 -labels docker -master "http://$JENKINS_IP:8080" -username "$USERNAME" -password "$PASSWORD" -fsroot /tmp
```

7. Edit the file `/etc/systemd/system/jenkins.service` so that it contains the following:
```
[Unit]
Description=Jenkins
After=network.target

[Service]
User=ubuntu
Restart=always
Type=simple
ExecStart=/usr/local/jenkins/swarm.sh

[Install]
WantedBy=multi-user.target
```

8. Start the service:
```
systemctl enable jenkins
systemctl start jenkins
```

9. Configure Throttle Concurrent Builds in jenkins
* go to Manage jenkins > Configure System > Throttle Concurrent Builds
* provide Category Name	as throttleDocker
* provide Throttled Node Label as docker
* provide Maximum Concurrent Builds Per Node Labeled As Above	to 1

![Throttle Concurrent Builds](/docs/images/Throttle-Concurrent-Builds.png)

10. Download docker-compose and vegeta binaries to `/usr/local`
```
cd /usr/local/bin

sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x docker-compose

wget https://github.com/tsenart/vegeta/releases/download/cli%2Fv12.1.0/vegeta-12.1.0-linux-amd64.tar.gz
tar xf *.gz
rm *.gz

```
