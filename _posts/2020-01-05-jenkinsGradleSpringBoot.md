---
layout: post
title: "Jenkins,Springboot 배포(with Gradle)"
subtitle: "Jenkins,Springboot deploy(with Gradle)"
date: 2020-01-05
background: '/img/posts/bg-img/28.jpg'
comments: true
categories: Jenkins Springboot
---
#### Jenkins 프로젝트 구성
Jenkins 해당 프로젝트 > 구성 > Build > Use Gradle Wrapper 체크 > Make gradlew executable 체크 >
Wrapper location에 ${workspace} > Tasks에 claen build

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins25.jpg">	
</div>

Use Gradle Wrapper은 프로젝트 내의 gradle Wrapper을 이용하는 것입니다.

Make gradlew executable을 체크해줘야 권한 에러없이 잘 진행됩니다.

Invoke Gradle은 서버 내부에 설치되어 있는 Gradle을 실행하는 것입니다.

Wrapper location은 Wrapper 경로로 jenkins gradle plugin이 지원하는 변수 ${workspace}로 설정합니다.

workspace는 현제 프로젝트의 workspace경로입니다.

Task의 clean은 workspace/build 레포지터리를 삭제하는 Task이며, build는 말그대로 build하는 Task입니다.

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

build.gradle에서 plugin 'java'추가시 자동적으로 bootJar Task를 실행합니다

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

파일 생성 위치는 해당 ```Workspace/build/libs/```

저 같은 경우는 서버에 바로 jenkins 설치후 build를 했기 때문에

```Console
/var/lib/jenkins/workspace/springproject/build/libs/springproject-0.0.1-SNAPSHOT.war
```

위와 같이 생성되었습니다.

#### War 파일이름

war도 마찬가지로 plugin 'war'추가시 bootWar Task가 자동 실행되며, 

war{ enabled = true }시 war Task가 진행됩니다.

또한 위와 같은 방식으로 파일이 생성됩니다.

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
