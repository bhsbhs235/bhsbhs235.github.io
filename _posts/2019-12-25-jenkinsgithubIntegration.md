---
layout: post
title: "Jenkins Github 연동"
subtitle: "Jenkins Github Integration"
date: 2019-12-25
background: '/img/posts/bg-img/16.jpg'
comments: true
categories: Jenkins
---

<h1 class="section-heading2" >AWS EC2 Jenkins서버 Github연동</h1>

Jenkins 관리 > Global Tool Configuration

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins8.JPG">	
</div>

서버의 JDK설치 폴더 경로를 추가해줍니다.

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins9.JPG">	
</div>

그 다음 Git 바이너리 파일 경로도 추가해줍니다. 

(RHEL8 버전 Git설치 명령어 : dnf install git)

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins10.JPG">	
</div>

그 다음 Gradle설치 폴더 경로도 추가해줍니다.

(보통 프로젝트내의 gradle을 사용하기 때문에 굳이 할 필요는 없습니다.)

RHEL8버전 Gradle설치
```console
$ wget https://services.gradle.org/distributions/gradle-5.2.1-bin.zip
$ mkdir /opt/gradle
$ unzip -d /opt/gradle gradle-5.2.1-bin.zip
$ export PATH=$PATH:/opt/gradle/gradle-5.2.1/bin
```

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins11.JPG">	
</div>

Jenkins관리 > 시스템 설정 > Jenkins Location > Jenkins 서버 URL

(/etc/sysconfig/jenkins에서 포트번호 설정)

http://ip주소:8090/ (Jenkins서버 포트번호 8090)

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins22.JPG">	
</div>

Jenkins관리 > 플러그인 관리 > (설치가능 탭) GitHub Integration Plugin 설치

홈 사이드바 새로운 Item 클릭 > 프로젝트 이름 입력 > Freestyle project

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins12.JPG">	
</div>

간단하게 설명 입력 > GitHub project 체크 > 해당 github 레포지토리 URL 입력 마지막에 / 붙혀준다

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins13.JPG">	
</div>

소스 코드 관리 > Git 체크 > 해당 github 레포지토리 URL에 .git을 붙혀준다

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins14.JPG">	
</div>

Credentials 추가를 하는 방법은 여러가지 방법이 있는데 저는 token으로 해보겠습니다.

github 계정의 settings(레포지토리 settings X)

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins15.JPG">	
</div>

왼쪽 사이드바 Developer settings > Personal access tokens > Generate new token

간단한 설명 입력 > 범위 지정 (저는 아래 그림과 같이 범위를 지정하였습니다.)

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins16.JPG">	
</div>

토큰이 생성되며, 다시는 볼 수 없기에 다른 곳에 복사를 해두어 가지고 있어야 합니다. 또한 외부로 유출되어서는 안됩니다.

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins17.JPG">	
</div>

다시 Add Credentials로 돌아와서
Domain(Global credentials), Kind(Secret text), Scope(Global), **Secret(아까 생성한 token문자를 여기에 입력)**
ID(아무거나), Description(아무거나) > Add 클릭

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins18.JPG">	
</div>

빌드 유발 > GitHub hook trigger for GITScm polling 체크

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins19.JPG">	
</div>

저는 Gradle 빌드 툴 기반으로 프로젝트를 생성했기 때문에

Build > Invoke Gradle script > Use Gradle Wrapper 체크 > Make gradlew executable 체크 (gradlew를 jenkins에서 실행할 수 있도록하는 권한) > Tasks에 실행할 Task 목록을 입력합니다

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins20.JPG">	
</div>

**※참고 :** Invoke Gradle은 서버에 설치되어있는 Gradle을 사용할때 체크합니다.

이번엔 해당 github 레포지토리의 settings로 이동합니다.

Webhooks > Add Webhook 클릭

Payload URL에 해당 jenkins서버URL/github-webhook/을 입력합니다.

저같은 경우는 http://ip주소:8090/github-webhook/(jenkins서버 포트번호 8090)

Add Webhook클릭

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins21.JPG">	
</div>

그 다음 Jenkins 서비스를 재구동하시고, git push하시면 정상작동 할 것입니다.

참고로 AWS EC2 인스턴스에 해당 포트 접근을 허용하시려면 보안그룹에 규칙을 추가시켜 주어야 합니다.