---
layout: post
title: Kubernetes에서 Bastion Pod을 만들어 접근이 제한된 DB에 접속하기
tags: [Kubernetes]
comments: true
---

AWS 클라우드 상의 Kubernetes 클러스터를 이용하여 시스템이 구성되어 있을 때, 특정 서비스는 Kubernetes 내에서만 접근 가능한 경우가 있습니다.

대표적인 예시로는 AWS 위에 띄워진 Kubernetes 클러스터 내의 서비스가 AWS RDS 데이터베이스를 이용한다고 할 때, RDS 인스턴스에 대한 접근 권한을 AWS 네트워크로만 설정하여 AWS 외부에서는 접근이 불가능한 경우가 있습니다.

이런 경우, Kubernetes 안에 임시로 Bastion Host 역할을 하는 Pod을 생성하고 포트포워팅을 하여, 일반 컴퓨터에서도 해당 서비스에 접근할 수 있습니다.

## Bastion Host란

Bastion Host란 내부 네트워크에서만 접근 가능한 서비스에 외부 네트워크에서도 접속할 수 있도록, 내부 네트워크에 연결되어 있으면서 외부 네트워크에 개방되어 게이트 역할을 하는 서버입니다.

일반적으로 Bastion Host에는 아무나 내부 네트워크에 접속할 수 없도록 인증 방식이 필요하지만, 로컬에 포트포워딩하는 경우에는 본인만 접속할 수 있기 때문에 필요하지 않습니다.

## Bastion Pod 만들기

아래 예시는 NGINX Reverse Proxy를 이용하는 Bastion Pod을 만들어 외부에서 RDS에 떠 있는 PostgreSQL(기본 포트 5432)에 접속하는 예시입니다.

NGINX의 설정은 ConfigMap으로 생성하고 volumeMounts를 통해 적용합니다.

~~~
apiVersion: v1
kind: Pod
metadata:
  name: my-bastion
spec:
  containers:
    - name: my-bastion
      image: nginx:stable-alpine
      volumeMounts:
        - name: my-bastion-config
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
      ports:
        - containerPort: 5432
  volumes:
    - name: my-bastion-config
      configMap:
        name: my-bastion-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-bastion-config
data:
  nginx.conf: |-
    worker_processes auto;
    error_log stderr info;
    events {
      worker_connections 1024;
    }
    stream {
      server {
        listen 5432 so_keepalive=on;
        proxy_pass my-database.rds.amazonaws.com:5432;
        proxy_socket_keepalive on;
      }
    }
~~~

위 파일을 `my-bastion.yaml`로 저장합니다.

~~~
kubectl apply -f my-bastion.yaml
~~~

위 명령어를 통해 필요한 리소스를 생성합니다.

~~~
kubectl port-forward my-bastion 5432:5432
~~~

위 명령어를 통해 호스트의 5432 포트를 `my-bastion` Pod의 5432 포트로 포워딩 합니다.

포트포워딩된 상태에서는 호스트에서 localhost:5432에 연결하는 것으로 RDS 인스턴스에 접속할 수 있습니다.

작업이 모두 끝났으면, 아래 명령어를 통해 생성했던 리소스를 모두 삭제합니다.

~~~
kubectl delete -f my-bastion.yaml
~~~
