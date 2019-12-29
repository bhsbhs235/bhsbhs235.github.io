---
layout: post
title: "nginx tomcat 연동"
subtitle: "nginx tomcat integration"
date: 2019-12-24
background: '/img/posts/bg-img/26.jpg'
comments: true
categories: Nginx
---

<h1 class="section-heading2" >nginx tomcat 연동</h1>

nginx.conf 수정

```shell
$ vi /etc/nginx/nginx.conf
```


```Bash
##추가
upstream tomcat {
        ip_hash;
        server 127.0.0.1:8080;
} 

##수정
server {
        listen       80;
        server_name  localhost;
 
        location / {
            proxy_set_header    Host $http_host;
            proxy_set_header    X-Real-IP $remote_addr;
            proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header    X-Forwarded-Proto $scheme;
            proxy_set_header    X-NginX-Proxy true;
 
            proxy_pass http://tomcat;
            proxy_redirect      off;
            charset utf-8;
         }
}
```

nginx 서비스를 실행한 후 다음과 같은 오류가 뜬다면

```console
2016/07/01 09:14:00 [crit] 1938#0: *4 connect() to 127.0.0.1:8080 failed (13: Permission denied) while connecting to upstream, ….
```

nginx가 proxy로 접근시 접근권한을 풀어주는 명령어 실행

```shell
$ setsebool -P httpd_can_network_connect 1
```

nginx포트로 접근시 tomcat으로 연동성공

<div>
	<img class="img-fluid" src="/img/posts/etc/nginx3.jpg">	
</div>