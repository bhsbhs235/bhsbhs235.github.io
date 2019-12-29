---
layout: post
title: "RHEL8 Docker 설치"
subtitle: "RHEL8 Docker Installation"
date: 2019-12-23
background: '/img/posts/bg-img/27.jpg'
comments: true
categories: Docker
---

<h1 class="section-heading2" >RHEL8 docker-ce 설치</h1>

dnf 명령어로 docker-ce.repo를 갖고 옵니다.

```shell
$ dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
```

docker-ce 리스트를 검색합니다.

```shell
$ dnf list docker-ce
```

해당하는 패키지를 다운 받습니다.

```shell
$ dnf install docker-ce --nobest -y
```

docker서비스를 활성화합니다.

```shell
$ systemctl enable docker
```

다음 명령어로 패키지가 제대로 설치되었는지 확인합니다.

```shell
$ docker --version
```

예제로 hell-world 이미지를 실행해 봅니다.

```shell
$ docker run hello-world
```

<div>
	<img class="img-fluid" src="/img/posts/etc/docker1.jpg">	
</div>

위의 그림과 같이 Hello from Docker! 문구 출력시 정상작동 하는 것입니다.


<h1 class="section-heading2" >RHEL8 docker-compose 설치</h1>

curl 패키지 설치

```shell
$ dnf install curl -y
```

curl -L 옵션으로 해당 URL에서 파일을 가져온다.

```shell
$ curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

실행 권한을 부여한다.

```shell
$ chmod +x /usr/local/bin/docker-compose
```

다음 명령어를 사용해 제대로 설치가 되었는지 확인한다.

```shell
$ docker-compose --version
```
