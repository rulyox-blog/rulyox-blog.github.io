---
layout: post
title: Docker 컨테이너 안에서 Host의 root 이용
tags: [Docker]
comments: true
---

일반적으로 Docker는 Host와 분리된 환경을 이용하기 위해 사용합니다.

흔히 있는 일은 아니지만, Docker 컨테이너 안에서 Host의 파일시스템에 접속해야 하거나, Host 시스템에서 어떤 프로그램을 실행시키는 것이 필요할 수 있습니다.

이럴 때 Docker 컨테이너에 `/`를 통째로 마운트할 수 없기 때문에, 아래 명령어와 같이 `chroot`을 응용하여 Host의 root를 이용하는 방식으로 해결할 수 있습니다.

~~~
docker run -it --privileged --net=host --pid=host --ipc=host --volume /:/host busybox chroot /host
~~~

위 명령어에 대한 해설은 아래와 같습니다.

- `privileged` 옵션은 기본적으로 접근이 차단된 일부 시스템 자원에 접근할 수 있도록 하는 옵셥입니다.
- 3개의 `=host` 옵션들은 컨테이너의 Network, PID, IPC namespace를 Host와 같게하는 옵션입니다.
- Host의 `/`를 컨테이너 속 특정 디렉토리에 마운팅하였습니다.
- `busybox` 이미지를 이용한 이유는 크기가 최대한 작으면서 `chroot` 명령어가 탑재된 이미지를 이용하기 위해서 입니다.
- `chroot /host`를 실행하면, 현재 프로세스 그룹의 root가 해당 디렉토리로 변경됩니다.

본 포스트는 [zwischenzugs의 "The Most Pointless Docker Command Ever" 포스트](https://zwischenzugs.com/2015/06/24/the-most-pointless-docker-command-ever/)를 참고하여 작성되었습니다.
