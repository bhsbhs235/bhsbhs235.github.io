---
layout: post
title: "Shellscript로 War자동배포/실행(Gradle)"
subtitle: "Auto Deploy/Run War with Shellscript(Gradle)"
date: 2020-01-12
background: '/img/posts/bg-img/30.jpg'
comments: true
categories: Jenkins Springboot 
---

<h1 class="section-heading2">들어가기 전에</h1>

build 툴은 Gradle을 사용하였으며 기본적으로 build가 무슨 뜻인지 따로 설명하지는 않겠습니다.

build시 Springboot 프로젝트를 war로 배포하도록 설정하였습니다. 다음 글을 참고해주세요. [Gradle로 War파일 배포](https://bhsbhs235.github.io/jenkins/springboot/2020/01/05/jenkinsGradleSpringBoot.html)

한 줄 한 줄 **의미**를 생각하면서 글을 읽어야 프로젝트 환경에 따라 커스터마이징 할 수 있습니다.

감사합니다.

<h1 class="section-heading2">본문</h1>

#### Jenkins 프로젝트 구성


Jenkins 프로젝트 > 구성 build 탭

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins29.JPG">	
</div>

Invoke Gradle script > Use Gradle Wrapper (프로젝트 내의 gradle사용) > Make gradlew executable 체크 > Wrapper location 에 ${workspace} > Tasks 에 clean build

gradle build Task로 war 생성 후 bash로 shell script를 실행해줍니다.

Add build step에서 Execute shell을 추가 후 아래 명령어를 입력해줍니다.

```Console
bash ${WORKSPACE}/deploy.sh 8080 ${JOB_NAME}
```

> **Note :** 위의 ${WORKSPACE},${JOB_NAME}은 jenkins에서 지원하는 내부 변수입니다.

WORKSPACE 변수를 사용해서 해당 workspace안에 있는 deploy.sh 실행

> **Note :** WORKSPACE경로 - /var/lib/jenkins/workspace/프로젝트이름

실행할 포트 번호 8080과 프로젝트 이름을 변수로 던져줍니다.

[Jenkins Set Environment Variables](https://wiki.jenkins.io/display/JENKINS/Building+a+software+project)

**deploy.sh**

```Bash
#!/bin/bash
# sudo bash {PATH}/deploy.sh 8080 springproject

# Server Port
# Ex) 8080
SERVER_PORT=$1
# Service Name
# Ex) springproject
PROJECT_NAME=$2
 
PROJECT_PATH=/var/lib/jenkins/workspace/$PROJECT_NAME/build/libs
WAR_FILE=$PROJECT_PATH/$PROJECT_NAME-0.0.1-SNAPSHOT.war
TMP_PATH_NAME=/tmp/$PROJECT_NAME-pid
 
# Function
function stop(){
    sudo echo " "
    sudo echo "Stoping process on port: $SERVER_PORT"
    sudo fuser -n tcp -k $SERVER_PORT 
 
    if [ -f $TMP_PATH_NAME ]; then
        sudo rm $TMP_PATH_NAME
    fi
 
    sudo echo " "
}
 
function start(){
    sudo echo " "
    sudo nohup java -jar -Dserver.port=$SERVER_PORT $WAR_FILE /tmp 2>> /dev/null >> /dev/null &
    sudo echo " "
}
 
# Function Call
stop
 
start

```

```Bash
sudo fuser -n tcp -k $SERVER_PORT
```

tcp $serverPort(8080)에 해당하는 port를 Kill 하는 명령어 이며, 

이 명령어를 사용하기 위해서는 아래의 패키지를 다운받습니다.

```Console
$ yum install -y psmisc
```

nohup은 백그라운드로 실행하는 명령어입니다.

(&와는 다르게 사용자가 터미널 세션을 끊어도 실행을 종료하지 않습니다)

```Bash
/tmp 2 >> dev/null >> /dev/null 
```

위의 내용은 /tmp내용을 log에 남기지 않겠다는 뜻입니다.

```Console
sudo: no tty present and no askpass program specified
```

만약 위와 같은 오류가 발생할 경우,

jenkins user에 sudo 권한을 줍니다.

```Console
$ vi /etc/sudoers
```
편집기로 /etc/sudoers 에 아래 내용을 추가해줍니다.
```Bash
jenkins		ALL=(ALL)		NOPASSWD=(ALL)
```

위의 오류를 정상적으로 해결한 후 

git 툴과 연동작업이 되어있다면 git push 만으로 알아서 build하고 배포까지 정상작동 할 것입니다.

**출처 :**

[https://heowc.tistory.com/75](https://heowc.tistory.com/75)


