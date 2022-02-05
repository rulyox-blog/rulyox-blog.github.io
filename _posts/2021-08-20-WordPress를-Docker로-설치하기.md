---
layout: post
title: WordPress를 Docker로 설치하기
thumbnail-img: /assets/img/posts/2021-08-20-WordPress를-Docker로-설치하기/thumb.png
share-img: /assets/img/posts/2021-08-20-WordPress를-Docker로-설치하기/thumb.png
tags: [Tutorials]
comments: true
---

안녕하세요. 전 Wordpress를 이용해 블로그를 이용하다가 이번에 Static Hosting을 이용하기 위해 Jekyll로 바꾸게 되었습니다. 하지만 전에 Wordpress를 구동시켰던 방식을 공유하기 위해 포스트를 작성하였습니다.

Jekyll 블로그에서 작성하는 Wordpress 설치 튜토리얼입니다.

WordPress를 직접 설치할 수도 있지만, Docker를 이용하면 빠르고 쉽게 설치할 수 있고 volume binding을 이용해 사이트를 통째로 쉽게 백업할 수도 있습니다.

## 컨테이너 생성하기

WordPress를 구동시키기 위해선 웹서버 컨테이너와 DB 컨테이너 2개가 필요합니다. 여러 개의 컨테이너를 실행하야 할때, Docker Compose를 이용하면 쉽게 관리할 수 있습니다.

~~~
version: '3.1'

services:

  wordpress:
    container_name: wordpress-web
    image: wordpress
    restart: always
    ports:
      - 80:80
      - 443:443
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress_user
      WORDPRESS_DB_PASSWORD: PASSWORD
      WORDPRESS_DB_NAME: wordpress_db
    volumes:
      - ./wordpress:/var/www/html

  db:
    container_name: wordpress-db
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: wordpress_user
      MYSQL_PASSWORD: PASSWORD
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - ./db:/var/lib/mysql
~~~

{: .box-error}
물론 PASSWORD는 안전한 비밀번호로 대체해야 합니다.

위 내용을 `docker-compose.yml`에 저장한 뒤, 아래 명령을 실행하면 컨테이너들을 구동시킬 수 있습니다.

~~~
docker-compose up -d
~~~

위 명령을 실행하시면 `wordpress-web`과 `wordpress-db` 컨테이너 2개가 구동됩니다. `wordpress-web`은 Apache 웹서버가 돌아가는 컨테이너이고, `wordpress-db`에는 MySQL 데이터베이스가 돌아가고 있습니다.

또한, 현재 디렉토리에 폴더 2개가 생성됩니다. `./wordpress`에는 `wordpress-web`의 웹서버 파일들이 저장되고, `./db`에는 `wordpress-db`의 데이터베이스 파일들이 저장됩니다. 백업을 위해서는 이 두 개의 폴더들을 안전한 곳에 저장하면 됩니다.

## HTTPS 활성화학기

처음 컨테이너들을 구동시키고 웹사이트에 접속해보면 HTTP로만 접속할 수 있고 HTTPS로는 접속이 불가능합니다. 사이트의 신뢰성을 높이기 위해선 TLS 인증서를 발급받고 HTTPS를 활성화해야 합니다. HTTPS를 활성화하는 것도 여러 방법이 있고, 이를 위한 유료 WordPress 플러그인도 존재합니다. 하지만 저는 쉽고 무료인 [Certbot](https://certbot.eff.org)을 이용했습니다.

~~~
docker exec -it wordpress-web bash
~~~

위 명령을 실행하시면 `wordpress-web` 컨테이너의 쉘에 접속할 수 있습니다. 컨테이너에 접속하여 Certbot의 튜토리얼을 따라 설치하신 뒤, 컨테이너를 재시작하시면 웹사이트에 HTTPS로 접속하실 수 있습니다.
