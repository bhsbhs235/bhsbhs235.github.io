---
layout: post
title: "Gradle로 War파일 배포"
subtitle: "War file distribution with Gradle"
date: 2020-01-05
background: '/img/posts/bg-img/28.jpg'
comments: true
categories: Jenkins Springboot
---

<h1 class="section-heading2">들어가기 전에</h1>

혹시 GitHub를 연동해서 쓰실분은 [Jenkins Github연동](https://bhsbhs235.github.io/jenkins/2019/12/25/jenkinsgithubIntegration.html)글을 참고해 주세요.

해당 프로젝트에 따라 설정(예: build.gradle) 내용이 다르기 때문에 최대한 한 줄 한 줄 **의미**를 생각하면서 따라오시면 커스터마이징 하기에 도움이 되실 것으로 예상이 되며,

Gradle 빌드에 대한 기초적인 내용(파일 구성,각 파일 기능)은 이해가 있으시다는 가정하에 글을 쓰게 되었습니다. 

<h1 class="section-heading2">본문</h1>

#### Jenkins 프로젝트 구성
Jenkins 해당 프로젝트 > 구성 > Build > Use Gradle Wrapper 체크 > Make gradlew executable 체크 >
Wrapper location에 ${workspace} > Tasks에 claen build

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins25.jpg">	
</div>

Invoke Gradle은 서버 내부에 설치되어 있는 Gradle을 실행하는 것입니다.

Use Gradle Wrapper은 프로젝트 내의 gradle Wrapper을 이용하는 것입니다.

(저희는 프로젝트내의 gradle설정을 사용할 예정입니다.)

Make gradlew executable을 체크해줘야 권한 에러없이 잘 진행됩니다.

Wrapper location은 Wrapper 경로로 jenkins gradle plugin이 지원하는 변수 ${workspace}로 설정합니다.

workspace는 현제 프로젝트의 workspace경로입니다.

Task의 clean은 workspace/build 레포지터리를 삭제하는 Task이며, build는 말그대로 build하는 Task입니다.

build Task가 성공적으로 수행되면 build 레포지터리가 생성되고 build 설정에 따른 여러가지 파일이 생성됩니다

#### build.gradle

SpringBoot 프로젝트를 구동할 가장 기본적인 의존성만 추가했습니다.

```Groovy
plugins {
	id 'java'
	id 'war'
	id 'eclipse'
	id 'org.springframework.boot' version '2.1.10.RELEASE'
	id 'io.spring.dependency-management' version '1.0.8.RELEASE'
}

group = 'com.springproject'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'     // Java 8버전
targetCompatibility = '1.8'     // Java 8버전

bootWar{                        //bootWar Task시 이름 지정
	archiveBaseName="springproject"
	archiveVersion="0.0.1-SNAPSHOT"
}

repositories {                  // maven 저장소
	mavenCentral()
}

dependencies {
	compile 'org.apache.tomcat.embed:tomcat-embed-jasper'  //내장톰캣
	compile 'javax.servlet:jstl:1.2'    // jsp 의존성
	implementation 'org.springframework.boot:spring-boot-starter-web'       // 기본적인 springboot 의존성
	testImplementation 'org.springframework.boot:spring-boot-starter-test'  // 기본적인 springboot 의존성
}
```

> **Note :** plugin 'war'시 아래의 bootJar, jar Task 말고 bootWar, war Task가 진행되지만 이해를 돕기위해 아래와 같은 설명을 추가했습니다. 그냥 쭈욱 읽어주시면 이해가 될 것으로 예상됩니다.

build.gradle에서 plugin 'java' 추가시 자동적으로 bootJar Task를 실행합니다

```Groovy
bootJar{
    archiveBaseName="springproject"
    archiveVersion="0.0.1-SNAPSHOT"
}
```

위와 같이 설정을 해주지 않으면 기본적으로 artifactID이름.jar가 생성됩니다 

(구글링으로는 item이름이 된다고 하는데 저는 artifactID이름.jar로 생성됐습니다.)

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins26.JPG">	
</div>

```Groovy
jar{
    enabled = true
    baseName = "springproject"
    version = "0.0.1-SNAPSHOT"
}
```

위와 같이 enabled = true시 jar Task를 SKIPPED 하지 않고 진행합니다.

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins27.JPG">	
</div>

만약, 

```Groovy
bootJar{
    archiveBaseName="springproject1"
    archiveVersion="0.0.1-SNAPSHOT"
}

jar{
    enabled = true
    baseName = "springproject2"
    version = "0.0.1-SNAPSHOT"
}
```

위와 같이 설정한다면 bootJar와 jar Task가 실행되기 때문에

"springproject1-0.0.1-SNAPSHOT.jar"와 "springproject2-0.0.1-SNAPSHOT.jar"이 동시에 생성됩니다.

만약 

```Groovy
jar{
    enabled = true
    baseName = "springproject2"
    version = "0.0.1-SNAPSHOT"
}
```

jar 설정만 한다면 bootJar Task를 실행 "artifactID이름.jar"가 생성되고,

jar Task 실행 "springproject2-0.0.1-SNAPSHOT.jar"가 생성됩니다.

war도 마찬가지로 plugin 'war'추가시 bootWar Task가 자동 실행되며, 

war{ enabled = true }시 war Task가 진행됩니다.

또한 위와 같은 방식으로 파일이 생성됩니다.

위의 build.gradle을 이용해 build시

파일 생성 위치는 해당 ```Workspace/build/libs/```

저 같은 경우는 서버에 바로 jenkins 설치후 build를 했기 때문에

```Console
/var/lib/jenkins/workspace/springproject/build/libs/springproject-0.0.1-SNAPSHOT.war
```

위와 같이 생성되었습니다.

지금까지 설명에 대한 기술문서 입니다. 

( 개발의 기초는 기술문서를 읽는 것 입니다. )

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins28.JPG">	
</div>

[Spring Boot Gradle Plugin Reference Guide](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/html/)

#### Springboot 배포, 실행

Springboot는 jsp를 지원하지 않기 때문에 war로 배포해야 하며,

Springboot는 내장 톰캣을 지원하기 때문에 의존성으로 build.gradle에

```Groovy
dependencies { 
    implementation 'org.springframework.boot:spring-boot-starter-web'
    providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat' 
}
```
또는
```Groovy
dependencies {
    compile 'org.apache.tomcat.embed:tomcat-embed-jasper'
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```
를 추가해줍니다.

톰캣 필요없이 ```java -jar 파일명.war``` 명령으로 실행합니다. 

서버에서 수동으로 실행해줘도 되고,

jenkins를 이용해 build후 자동실행시켜주려면

프로젝트 구성 Build탭에 Execute Shell을 추가한 후

```Console
java -jar 파일명.war
```

입력후 저장하면 됩니다.