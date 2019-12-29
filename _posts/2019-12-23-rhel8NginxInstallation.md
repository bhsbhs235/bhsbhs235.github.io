---
layout: post
title: "RHEL8 nginx설치"
subtitle: "RHEL8 Nginx Installation"
date: 2019-12-23
background: '/img/posts/bg-img/25.jpg'
comments: true
categories: Nginx
---

<h1 class="section-heading2" >AWS EC2 RHEL8버전에서 Nginx설치하기</h1>

nginx 설치
```shell
$ yum install nginx
```

service 활성화
```console
$ systemctl enable nginx
```

nginx 실행
```shell
$ service nginx start
```

aws ec2에서 해당포트에 접근하려면 보안 그룹에 추가해줘야 합니다.

왼쪽 사이드바에서 **네트워크 및 보안 > 보안 그룹** 클릭

<div>
	<img class="img-fluid" src="/img/posts/etc/jenkins5.JPG">	
</div>

해당 EC2서비스 인스턴스에 해당하는 보안 그룹을 클릭한 후 밑의 탭  **인바운드 > 편집** 클릭

규칙추가 클릭 후 다음과 같이 추가합니다.

<div>
	<img class="img-fluid" src="/img/posts/etc/nginx1.JPG">	
</div>

브라우저에서 해당하는 http://ip or URL:80 으로 접속합니다.

※참고로 ip는 인스턴스에 들어가면 퍼블릭 DNS(IPv4)라고 있습니다.

아래그림과 같은 화면이 뜨면 성공!

<div>
	<img class="img-fluid" src="/img/posts/etc/nginx2.jpg">	
</div>