---
layout: post
title: "SpringBoot 폴더구조"
subtitle: "Spring Boot folder structure"
date: 2019-12-25
background: '/img/posts/bg-img/17.jpg'
comments: true
categories: Springboot
---
<h1 class="section-heading2" >SpringBoot 폴더구조</h1>

검색해보니 프로젝트 생성 및 view설정(jsp)은 많은데 css, js를 어디에 위치하는지는 별로 없어서 글을 쓰게 되었습니다.

gradle 빌드툴 기반의 SpringBoot 프로젝트 폴더 구조 입니다.

```
/bin
/gradle
/src
  +-/main
       +-/java
            +-/<source code>
       +-/resources
            +-/public
                +-/error
                    -404.html
            +-/static
                +-/css
                +-/img
                +-/js
            +-/templates
            - application.properties
       +-/webapp
            +-/WEB-INF
                 +-/jsp
  +-/test    
-build.gradle
-gradlew
-gradlew.bat
-settings.gradle    
```

폴더 구조는 위와 같이 하면 되며 application.properties 설정은 아래와 같이 해줍니다.

```yml
spring.mvc.static-path-pattern=/resources/**
```

이미지 경로

```html
<img src="resources/img/welcome.jpg"> ( O )

<img src="resources/static/img/welcome.jpg"> ( X )
```

[해당 문서 링크](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-developing-web-applications)입니다.

<div>
	<img class="img-fluid" src="/img/posts/springboot/springboot1.jpg">	
</div>

[application.properties 설정관련링크](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html)