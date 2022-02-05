---
layout: post
title: AWS Linux에 Docker 설치
tags: [Tutorials]
comments: true
---

AWS Linux에 Docker와 Docker Compose를 설치하고 `ec2-user`에게 권한을 주는 명령어입니다.

~~~
sudo yum -y upgrade



# install docker
sudo yum -y install docker

# give permission
sudo usermod -aG docker ec2-user

# start service
sudo service docker start

# start service on boot
sudo systemctl enable docker



# install docker-compose
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/bin/docker-compose

# give permission
sudo chmod +x /usr/bin/docker-compose
~~~

위 작업을 완료한 후, AMI로 만들어두면 필요할 때마다 Docker가 미리 설치된 인스턴스를 쉽게 만들 수 있습니다.
