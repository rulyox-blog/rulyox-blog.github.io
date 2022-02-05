---
layout: post
title: Kubernetes에서 Persistent Volume Claim 내부의 파일 조회
tags: [Kubernetes]
comments: true
---

## Persistent Volume Claim 소개

Kubernetes에서는 컨테이너 속의 삭제되지 않는 스토리지 관리를 위해 Persistent Volume을 이용합니다.

Persistent Volume은 스토리지를 Kubernetes에 제공하는 클러스터 리소스이고, 특정 컨테이너에서 Persistent Volume이 제공하는 볼륨을 이용하기 위해서는 Persistent Volume Claim이라는 요청을 통해 볼륨을 마운팅합니다.

아래는 Persistent Volume Claim을 할당하고 Pod에서 Persistent Volume Claim을 사용하는 간단한 예시입니다.

~~~
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  resources:
    requests:
      storage: 10Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-block-volume
spec:
  containers:
    - name: fc-container
      image: fedora:26
      command: ["/bin/sh", "-c"]
      args: [ "tail -f /dev/null" ]
      volumeDevices:
        - name: data
          devicePath: /dev/xvda
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: block-pvc
~~~

## Persistent Volume Claim 조회

어떤 Persistent Volume을 이용하냐에 따라 할당한 Persistent Volume Claim 내부의 파일들을 조회하는 것이 불편한 경우가 있을 수 있습니다. 이런 상황에는 아래의 명령어를 이용하면 간단하게 내부의 파일들을 조회할 수 있습니다.

~~~
apiVersion: v1
kind: Pod
metadata:
  name: pvc-inspector
spec:
  containers:
  - image: busybox
    name: pvc-inspector
    command: ["tail"]
    args: ["-f", "/dev/null"]
    volumeMounts:
    - mountPath: /pvc
      name: pvc-mount
  volumes:
  - name: pvc-mount
    persistentVolumeClaim:
      claimName: YOUR_CLAIM_NAME_HERE
~~~

위 파일을 `pvc-inspector.yaml`로 저장합니다.

~~~
kubectl apply -f pvc-inspector.yaml
~~~

위 명령어를 실행하면 `pvc-inspector`라는 Pod이 생성됩니다.

~~~
kubectl exec -it pvc-inspector -- sh
cd /pvc
~~~

위 명령어를 실행하면 `pvc-inspector`의 쉘에 접속할 수 있고, `/pvc` 경로에 선택한 Persistent Volume Claim이 마운팅되어 있습니다.

조회를 마친 후에는 아래 명령어로 `pvc-inspector`를 삭제합니다.

~~~
kubectl delete pod pvc-inspector
~~~
