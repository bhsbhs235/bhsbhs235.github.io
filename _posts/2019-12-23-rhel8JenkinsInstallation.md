---
layout: post
title: "RHEL8 jenkins설치"
subtitle: "RHEL8 Jenkins Installation"
date: 2019-12-23
background: '/img/posts/bg-img/24.jpg'
comments: true
categories: Jenkins
---
<h1 class="section-heading2" >AWS EC2 RHEL8버전에서 Jenkins설치하기</h1>

- wget : 웹 서버로부터 콘텐츠를 가져오는 컴퓨터 프로그램 HTTP,HTTPS,FTP 프로토콜 사용

yum install 명령을 이용해 wget 설치

```shell
$ yum install wget
```

jenkins.repo 가져오기

```shell
$ wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
```

redhat-stable(안전버전) 설치파일 다운로드

```shell
$ rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```

jenkins 설치

```shell
$ yum install jenkins
```

사용할 jenkins 서버 포트 변경

```shell
$ vi /etc/sysconfig/jenkins
```

```Bash
## Type:        integer(0:65535)
## Default:     8080
## ServiceRestart: jenkins
#
# Port Jenkins is listening on.
# Set to -1 to disable
#
JENKINS_PORT="8090"
```

jenkins 실행

```shell
$ service jenkins start
```

만약 서비스 실행시 오류가 난다면 systemctl 명령어로 오류원인을 검색해봅니다.

```shell
$ systemctl status jenkins.service
```

<div>
	<img class="img-fluid" src="/img/posts/etc/jenkins2.jpg">	
</div>

오류를 보면 jenkins실행에 필요한 java파일이 없다고 나옵니다.

jdk 파일을 설치해 보겠습니다 설치가능한 jdk 리스트를 검색합니다.

```shell
$ yum list java*jdk-devel
```

<div>
	<img class="img-fluid" src="/img/posts/etc/jenkins3.jpg">	
</div>

jdk 1.8 버전 64bit용을 다운로드합니다.

```shell
$ yum install java-1.8.0-openjdk-devel.x86_64
```

jenkins 서비스를 실행합니다.

```shell
$ service jenkins start
```

<div>
	<img class="img-fluid" src="/img/posts/etc/jenkins4.jpg">	
</div>

aws ec2에서 해당포트에 접근하려면 보안그룹에 추가해줘야 합니다.

왼쪽 사이드바에서 **네트워크 및 보안 > 보안 그룹** 클릭

<div>
	<img class="img-fluid" src="/img/posts/etc/jenkins5.JPG">	
</div>

해당 EC2서비스 인스턴스에 해당하는 보안 그룹을 클릭한 후 밑의 탭  **인바운드 > 편집** 클릭

규칙추가 클릭 후 다음과 같이 추가합니다.

<div>
	<img class="img-fluid" src="/img/posts/etc/jenkins6.JPG">	
</div>

브라우저에서 해당하는 http://ip or URL:8090 으로 접속합니다.

※참고로 ip는 인스턴스에 들어가면 퍼블릭 DNS(IPv4)라고 있습니다.

아래그림과 같은 화면이 뜨면 성공!
<div>
	<img class="img-fluid" src="/img/posts/etc/jenkins7.JPG">	
</div>