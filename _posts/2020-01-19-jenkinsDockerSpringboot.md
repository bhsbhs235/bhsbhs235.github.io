---
layout: post
title: "Docker로 springboot실행(Gradle)"
subtitle: "Run springboot with Docker(Gradle)"
date: 2020-01-19
background: '/img/posts/bg-img/29.jpg'
comments: true
categories: Jenkins Springboot
---

<h1 class="section-heading2">들어가기 전에</h1>

가장 기본이 되는 것은 공식 문서입니다. 

제 글은 참고용이며, 해당 프로젝트에 따라 설정이 모두 다르기 때문에 최대한 한 줄 한 줄 자세하게 적도록 노력하였습니다. **의미**를 생각하시면서 봐주시면 적용하기 쉬울 것으로 예상됩니다.

감사합니다.

<h1 class="section-heading2">본문</h1>

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins30.JPG">	
</div>

Invoke Gradle script > Use Gradle Wrapper 에서 Make gradlew executable 체크 > Wrapper location ${workspace} > Tasks clean build

build Task로 War파일을 배포합니다

> **Tip :** [Gradle로 War파일 배포](https://bhsbhs235.github.io/jenkins/springboot/2020/01/05/jenkinsGradleSpringBoot.html)참고

Add build step에서 Execute shell을 추가 후 아래 명령어를 입력해줍니다.

```Console
docker build --build-arg WAR_FILE=build/libs/springproject-0.0.1-SNAPSHOT.war -t bhsbhs235/springproject:0.1 .
```

--build-arg는 이미지를 생성할 때 변수를 전달하는 옵션입니다.

-t 는 이미지이름/태그를 정해주는 옵션입니다. 보통 "이름:태그" 로 만들어 줍니다.

> **Tip :**  Dockerhub로 push할 때는 username/tagname 형태가 되어야 합니다.

```Console
docker run --name springproject -d -p 8080:8080 bhsbhs235/springproject:0.1
```

--name 는 컨테이너 이름을 지정하는 옵션입니다.

-d 는 백그라운드로 실행하는 옵션입니다.

-p 는 포트를 지정하는 옵션입니다.

-p 옵션은 헷갈리기 떄문에 자세하게 설명해드리겠습니다.

```-p 8080:8080```에서 앞의 8080은 호스트의 포트번호입니다. 즉 **서버의 구동포트번호**입니다. 뒤의 8080은 **컨테이너 내부의 포트번호**입니다. 따라서 8080:8080 뜻은 컨테이너 내부의 포트와 서버의 포트를 연결해준다는 뜻입니다.

뒤에서 설명드리겠지만 Springboot를 구동시킬 컨테이너 내부에서 

```Console
java -jar -Dserver.port=8080 springproject.war
```

다음과 같이 포트 8080으로 구동시키기 때문에 뒤에 8080이며, 

서버의 8080포트로 서비스 예정이므로 앞에 8080이 됩니다.

뒤에서 한 번 더 설명드리도록 하고 우선 넘어가겠습니다.

이미지를 생성할 Dockerfile을 생성합니다.

```Dockerfile
FROM openjdk:8-jdk-alpine
ARG WAR_FILE=springproject-0.0.1-SNAPSHOT.war
COPY ${WAR_FILE} springproject.war
ENTRYPOINT ["java","-jar","-Dserver.port=8080","springproject.war"]
```

FROM - 생성할 이미지의 기반을 설정합니다.(이 설정을 왜 하시는지 처음엔 이해가 안 되실 수 있으나, 많은 설정들을 보면 자연스럽게 이해가 가실겁니다.)

ARG - 변수값을 지정해주는 설정으로 위의 springproject-0.0.1-SNAPSHOT.war은 아무의미가 없는 Default Value값으로 그냥 정보만 적어둔 것입니다.

위에 이미지를 생성해주는 명령어에서 ```--build-arg WAR_FILE=build/libs/springproject-0.0.1-SNAPSHOT.war```로 변수값으을 전달하면 ARG WAR_FILE에서 값을 받아 적용되는 것입니다.

즉, DockerFile에서 ```ARG WAR_FILE``` 로 변수만 만들어주면 오류는 없습니다. 하지만 값을 받을 변수가 없으면 오류가 납니다.

COPY - 호스트의 파일을 컨테이너로 복사하는 설정입니다. WAR_FILE 변수값을 이용해 "springproject.war" 이름으로 Copy합니다.

ENTRYPOINT - 컨테이너에서 명령어를 실행하는 설정입니다. 

```Console
java -jar -Dserver.port=8080 springproject.war
```

springproject.war를 8080 포트로 실행하는 명령어입니다. springboot는 내장톰캣을 지원하기 때문에 따로 톰캣으로 구동하지 않고도, 다음과 같이 자바명령어를 이용해 실행할 수 있습니다.

Dockerfile에 대해 설명해드렸으니 아까 말한 docker run -p 옵션에 대해 다시 설명해드리겠습니다.

만약 Dockerfile에 

```Dockerfile
ENTRYPOINT ["java","-jar","-Dserver.port=90","springproject.war"]
```

설정하고,

```docker run --name springproject -d -p 50:90 springproject:0.1```으로 컨테이너를 실행했다면 

호스트(서버)의 50번포트와 컨테이너의 90번포를 연결시킨다는 뜻으로 웹서비스에 접근하려면

http://domainname(ipv4):50/ 로 검색해야 합니다.

> **Note :** http://domainname(ipv4):90/ ( X )

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins31.JPG">	
</div>

위의 그림과 같이 표현할 수 있습니다.

만약 다른이름으로 컨테이너를 하나 더 생성한다면 어떻게 될까요?

```Console
docker run --name webservice -d -p 600:90 springproject:0.1
```

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins32.JPG">	
</div>

위의 그림과 같이 표현할 수 있을 것입니다.

이렇게까지 자세하게 설명하는 이유는 컨테이너가 많고 이후에 네트워크를 연결해 컨테이너 간에 통신을 하거나 데이터를 주고받으려면 이 개념을 확실하게 이해해야 하기 때문입니다.

> **Tip :** 만약 AWS EC2서버를 이용하신다면 보안그룹에서 50,600번 포트를 개방해야 접근이 가능합니다.

만약 docker daemon permission denied 에러가 발생한다면,

```Console
usermod -aG docker jenkins
```

jenkins 서버 재실행, 오류가 재발생하면

```Console
usermod -aG root jenkins
```

jenkins 서버 재실행, 오류가 재발생하면

```Console
setfacl -m u:jenkins:rwx /var/run/docker.sock
```

명령어를 실행해줍니다.

<h1 class="section-heading2">참조 문서</h1>

[Springboot with Docker](https://spring.io/guides/gs/spring-boot-docker/) - Springboot 공식문서 

[Docker commandline](https://docs.docker.com/engine/reference/commandline/docker/) - Docker 명령어관련 공식문서

[Dockerfile reference](https://docs.docker.com/engine/reference/builder/) - Dockerfile 설정관련 공식문서