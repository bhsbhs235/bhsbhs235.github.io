---
layout: post
title: "docker-compose,springboot무중단배포"
subtitle: "Springboot continuous deployment with docker-compose"
date: 2020-02-02
background: '/img/posts/bg-img/32.jpg'
comments: true
categories: Jenkins Springboot Docker
---

<h1 class="section-heading2">들어가기 전에</h1>

가장 기본이 되는 것은 공식 문서입니다. 

제 글은 참고용이며, 해당 프로젝트에 따라 설정이 모두 다르기 때문에 최대한 한 줄 한 줄 자세하게 적도록 노력하였습니다 **의미**를 생각하시면서 봐주시면 적용하기 쉬울 것으로 예상됩니다.

또한, 저의 블로그에 Gradle로 war파일배포, Shellscript로 War자동배포/실행, Docker로 springboot실행(Gradle) 등 이 글을 읽기 전에 보면 좋은 내용들이 많이 있으니 혹시 보다가 이해가 안되시는 부분이 있으시다면 이전의 글들을 보면 많은 도움이 될 것으로 예상이 됩니다.

감사합니다.

<h1 class="section-heading2">본문</h1>

#### 시스템 구성도

<div>
	<img class="img-fluid" src="/img/posts/docker/docker5.jpg">	
</div>


#### Jenkins 프로젝트 > 구성

##### General

<div>
	<img class="img-fluid" src="/img/posts/docker/docker1.jpg">	
</div>

설명 : 아무거나 > Github project 체크 > Project url : https://github.com/bhsbhs235/springproject/ (해당 github repository 주소)

##### 소스 코드 관리

해당 형상관리 툴 Git에 따라 다를 수 있습니다. 저는 github를 사용했으며, Credentials 추가 방법은 여러가지 방법이 있으므로 따로 설명하지 않겠습니다 (저는 github token으로 인증했습니다.)

Git체크 > Repository URL : https://github.com/bhsbhs235/springproject.git > Credentials추가 > Branches to build : */master

<div>
	<img class="img-fluid" src="/img/posts/docker/docker2.jpg">	
</div>

##### 빌드 유발

어느 시점에서 빌드를 할 건지 선택하는 항목으로 저는 Giuhub 저장소에 push에 의한 hook 이벤트 발생시 빌드를 유발하도록 GitHub hook trigger for GITScm polling에 체크를 하였습니다.

<div>
	<img class="img-fluid" src="/img/posts/docker/docker3.jpg">	
</div>

##### Build

프로젝트 내의 gradle wrapper를 사용할 것이므로, User Gradle Wrapper 체크

Wrapper location : ${workspace} ( jenkins 내부 변수로 해당 workspace 경로를 나타냄 )

Tasks : clean build ( clean는 build디렉토리를 삭제하는 Task명령어 build는 말그대로 build하는 Task 명령어)

Execute shell 추가 후 Command에 bash ${WORKSPACE}/deploy.sh 로 빌드 후 실행 할 명령어를 쉘스크립트로 구동

<div>
	<img class="img-fluid" src="/img/posts/docker/docker4.jpg">	
</div>

#### Nginx

가장 먼저 springproject_blue,green을 배포할 웹서버를 구축해야 합니다.

##### nginx.conf

``` Nginx
http {

    upstream springproject{
        least_conn;
        server 127.0.0.1:8081 weight=5 max_fails=3 fail_timeout=10s;
        server 127.0.0.1:8082 weight=10 max_fails=3 fail_timeout=10s;
    }

    server {

        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
                proxy_pass http://springproject;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
        }

    }
}
```

nginx.conf > http 섹터에 위와 같이 추가 합니다. 

http://springproject로 proxy pass를 지정합니다.

그러면 위에 upstream springproject의 설정에 따라 Load-balancing 됩니다.

설정내용은 localhost(127.0.0.1)의 8081번 포트 또는 8082번 포트로 연결시켜주는 내용이며,

8081포트에 응답이 없을 경우(구동 중지)에는 8082로 연결시켜 줄 것이며, 

반대로 8082포트에 응답이 없을 경우(구동 중지)에는 8081로 연결시켜 줄 것입니다.

**위의 내용과 blue_green 배포 방식과 연결시켜 설명 하자면**

8082(blue)가 구동 중일 때, github에 push > 변경된 내용 build > build한 war 파일을 8081(green)에 배포 > 8082 서버를 내림 > Load-balancing에 의해 8081(green)을 실행 > 현재 8081(green) 구동 중 > github에 push > 변경된 내용 build > build한 war 파일을 8082(blue)에 배포 > 8081 서버를 내림 > Load-balancing에 의해 8082(blue)을 실행 > 무한 반복

위와 같은 방식으로 무중단 배포가 가능한 것입니다.

##### Nginx Docker 컨테이너로 실행

따로 서버에 Nginx를 다운받아서 nginx.conf를 위와 같이 수정하여 서비스를 구동해주어도 되지만 저는 Nginx도 Docker 컨테이너로 구동하여 서비스를 제공하도록 하였습니다.

**여기서 주의할 점이 있습니다.**

Docker에서 지원되는 네트워크로 컨테이너 간의 통신이 가능합니다. 

하지만 Load-balancing을 하기 위해서는 기본( default로 컨테이너이름_default로 자동으로 생성됨)또는 여러 네트워크( bridge,none 등)가 아닌 host(서버)로 네트워크를 수동적으로 연결해줘야 한다는 것입니다.

왜냐하면 기타의 네트워크로 연결하면 blue, green서버를 찾는 데에 어려움이 생깁니다.

예를 들어 제가 실습한 두가지 방법에 대해 설명드리겠습니다.

첫번째, 컨테이너 이름으로 찾을 시

``` Nginx

upstream springproject{
        least_conn;
        server springproject_green:8081 weight=5 max_fails=3 fail_timeout=10s;
        server springproject_blue:8082 weight=10 max_fails=3 fail_timeout=10s;
}

```

같은 네트워크에 연결되어 있다고 가정했을 때 위와 같이 컨테이너이름으로 접근할 수 있습니다.

하지만 nginx 컨테이너 구동시 오류가 생깁니다 왜냐하면 위와 같은 설정으로 구동할 때는 두 서버 blue,green이 동시에 이미 돌아가고 있어야 하기 때문입니다. 

( 이유는 잘 모르겠습니다 제 추측상 해당 서버를 찾지 못하면 그냥 작동이 안되는 것 같습니다.  )

하지만 blue_green 배포 특성상 동시에 돌아가고 있기엔 불가능 하기 때문에 입니다. ( 하나가 돌아가면 하나는 죽어있는 상태입니다.)

두번째, 특정 ip로 찾을 시

예를 들어 web_bridge 라는 네트워크를 임의로 생성합니다. 

docker network inspect web_bridge 명령어를 통해 subnet이 192.168.107.0/20을 확인합니다.

``` Nginx

upstream springproject{
        least_conn;
        server 192.168.107.3:8081 weight=5 max_fails=3 fail_timeout=10s;
        server 192.168.107.4:8082 weight=10 max_fails=3 fail_timeout=10s;
}

```

따라서 위와 같이 설정합니다

자동적으로 192.168.107.2 는 nginx 서버 그 다음 순서대로 3, 4에 생성되는 것을 확인하고 임의로 3에 green, 4에 blue로 임의로 지정했습니다.

이렇게 하고 컨테이너 구동시 아무 오류 없이 동작하며 blue,green 무중단 배포도 정상적으로 잘 됩니다. 하지만 ip를 예상만으로 임의대로 설정한다는 것은 좋은 방법이 결코 아니기 때문에 이 방법은 그냥 안하는 것으로 하겠습니다.

그래서 **마치 localhost에서 Nginx를 구동하는 것과 같이 host서버 네트워크로 연결합니다.**

( 이해가 안되신다면 'Docker 네트워크'에 대해 알아보시기 부탁드립니다.)

**docker-compose.nginx.yml :**

``` Yml
version: '3.7'

services:
    nginx:
        build:
            context: .
            dockerfile: Dockerfile_nginx
        image: bhsbhs235/nginx:0.1
        ports: 
            - "80:80"
        container_name: nginx_springproject
        network_mode: "host"
```

`build`는 docker build 명령어 (이미지 생성)에 관한 설정으로 context는 Dockerfile이 위치하는 경로를 지정하며, build할 dockerfile 이름은 'Dockerfile_nginx'로 설정해주었습니다.

image 이름 (\--tag 옵션)은 `dockerhub계정이름/nginx:태그`로 설정해 주었습니다. ( dockerhub에 push하려면 계정이름을 붙혀서 만들어줘야 합니다.)

`ports`는 docker run 명령어 (컨테이너 구동) -p 옵션과 같고, container_name 컨테이너이름 (\--name옵션)은 nginx_springproject로 설정해주었습니다. 

그리고 네트워크를 localhost 네트워크로 연결했습니다. network_mode: "host"

**Dockerfile_nginx :**

``` Dockerfile
FROM nginx

COPY nginx.conf /etc/nginx/nginx.conf

VOLUME /var/log/nginx/log

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

`COPY`로 설정파일인 nginx.conf를 컨테이너로 복사합니다.

`VOLUME`으로 log를 서버에서 확인할 수 있도록 컨테이너에 저장하지 않고 호스트에 저장하도록 합니다.

`EXPOSE`는 호스트와 연결시킬 포트 번호 설정입니다.

`CMD`는 컨테이너가 시작되었을 때 명령을 실행하는 옵션입니다.

웹서버인 nginx 부터 실행합니다.

`docker-compose -p nginx -f docker-compose.nginx.yml up -d`

(만약 docker-compse 명령어를 인식하지 못하면 바이너리 파일을 경로를 직접 불러오면 됩니다.)

-p 는 프로젝트 이름을 지정

-f 는 compose 파일을 지정

-d 는 백그라운드로 실행

#### Springboot

##### Blue

**docker-compose.blue.yml :**

``` Yml
version: '3.7'

services:
    springproject:
        build:
            context: .
            dockerfile: Dockerfile_springproject
            args:
                WAR_FILE: build/libs/springproject-0.0.1-SNAPSHOT.war
        image: bhsbhs235/springproject:0.1
        ports: 
            - "8082:8080"
        container_name: springproject_blue
```

일부분은 위에서 설명했으므로 생략하겠습니다.

`args`는 docker build 명령어 (이미지 생성)에서 \--build-arg 옵션과 같고 이미지를 생성할 때 변수를 전달하는 옵션입니다. 

gradle로 build시 war파일이 build/libs/ 디렉토리 밑에 생성되므로 위와 같이 변수값을 전달하였습니다.

`ports` , `docker run -p` 에 대해 이해가 필요하신 분은 [Docker로 springboot실행(Gradle)](https://bhsbhs235.github.io/jenkins/springboot/docker/2020/01/19/jenkinsDockerSpringboot.html)를 참고해주시기 바랍니다.

**Dockerfile_springproject :**

``` Dockerfile
FROM openjdk:8-jdk-alpine
ARG WAR_FILE=springproject-0.0.1-SNAPSHOT.war
COPY ${WAR_FILE} springproject.war
ENTRYPOINT ["java","-jar","-Dserver.port=8080","springproject.war"]
```

Springboot를 java 자바 기반으로 실행시킬 것이므로 FROM openjdk:8-jdk-alpine

`ARG`는 위에서 args: WAR_FILE: 로 건내준 변수값을 받는 변수입니다.

`COPY`로 변수(WAR_FILE)의 값(${WAR_FILE})을 `springproject.war`이름으로 컨테이너에 복사합니다.

`ENTRYPOINT`도 또한 컨테이너가 시작되었을 때 명령을 실행하는 옵션입니다. springboot 프로젝트를 "8080"포트로 실행해줍니다.


##### Green

**docker-compose.green.yml :**

``` Yml
version: '3.7'

services:
    springproject:
        build:
            context: .
            dockerfile: Dockerfile_springproject
            args:
                WAR_FILE: build/libs/springproject-0.0.1-SNAPSHOT.war
        image: bhsbhs235/springproject:0.1
        ports: 
            - "8081:8080"
        container_name: springproject_green
```

port 연결을 8081로 연결해주는 것 말고는 blue와 큰 차이는 없습니다.

#### deploy.sh

``` Bash
#!/bin/bash

DOCKER_APP_NAME=springproject

EXIST_BLUE=$(/usr/local/bin/docker-compose -p ${DOCKER_APP_NAME}-blue -f docker-compose.blue.yml ps | grep Up)

if [ -z "$EXIST_BLUE" ]; then
    echo "blue up"
    /usr/local/bin/docker-compose -p ${DOCKER_APP_NAME}-blue -f docker-compose.blue.yml up -d --build

    sleep 10

    /usr/local/bin/docker-compose -p ${DOCKER_APP_NAME}-green -f docker-compose.green.yml down
else
    echo "green up"
    /usr/local/bin/docker-compose -p ${DOCKER_APP_NAME}-green -f docker-compose.green.yml up -d --build

    sleep 10

    /usr/local/bin/docker-compose -p ${DOCKER_APP_NAME}-blue -f docker-compose.blue.yml down
fi
```

EXIST_BLUE로 blue 서버가 동작하고 있는 지 확인합니다.

if문 -z 옵션은 문자열의 길이가 0이면 `true` 입니다.

즉, 길이가 0이라는 뜻은 blue 서버가 동작중이지 **않다는** 것이고, blue 서버를 **up** green 서버 **down**

반대로 길이가 0이 아니라는 것은 blue 서버가 동작 중이라는 뜻이며, green 서버 **up** blue 서버 **down**

<h1 class="section-heading2">Workflow</h1>

현재 nginx 웹서버 실행 중입니다.

1. 코드 수정 후 github에 push한다.

1. hook 이벤트 발생 (jenkins)

1. Invoke Gradle script clean, build Task 진행

1. build에 의해 ${workspace}/build/libs/springproject-0.0.1-SNAPSHOT.war 생성 (build.gradle참고)

1. Execute shell로 deploy.sh 쉘스크립트 실행

1. 상황에 따라 blue, green 서버를 down 또는 up

<h1 class="section-heading2">참고 문서</h1>

[Compose file version 3 reference](https://docs.docker.com/compose/compose-file/) - docker-compose 명령어관련 공식문서

[Springboot with Docker](https://spring.io/guides/gs/spring-boot-docker/) - Springboot 공식문서 

[Docker commandline](https://docs.docker.com/engine/reference/commandline/docker/) - Docker 명령어관련 공식문서

[Dockerfile reference](https://docs.docker.com/engine/reference/builder/) - Dockerfile 설정관련 공식문서