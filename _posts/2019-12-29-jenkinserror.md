---
layout: post
title: "Jenkins 빌드 에러"
subtitle: "Jenkins build error"
date: 2019-12-29
background: '/img/posts/bg-img/22.jpg'
comments: true
categories: Jenkins
---

<h1 class="section-heading2">Jenkins 빌드 에러</h1>

AWS EC2에서 jenkins서버 구동시 다음과 같은 error를 보실수 있습니다.

```Console
OpenJDK 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000eaaa0000, 178978816, 0) failed; error='Cannot allocate memory' (errno=12)
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (mmap) failed to map 178978816 bytes for committing reserved memory.
# An error report file with more information is saved as:
# /var/lib/jenkins/.gradle/daemon/5.6.4/hs_err_pid2055.log
```

원인은 JVM 구동시 메모리가 부족하다는 것입니다.

이유는 두가지가 있습니다

- 시스템의 물리적 RAM 또는 SWAP 공간의 부족

- 프로세스 크기 제한에 도달 

**해결방법 :**

- 시스템의 메모리 로드 줄이기.

- 물리적 메모리 또는 SWAP 공간 늘리기.

- SWAP 백킹 저장소가 가득 차 있는지 확인.

- 64 비트 OS에서 64 비트 Java 사용.

- Java 힙 크기 줄이기 (-Xmx / -Xms).

- Java 스레드 수 줄이기

- Java 스레드 스택 크기 줄이기 (-Xss)

- -XX:ReservedCodeCacheSize= 를 사용하여 더 큰 코드 캐시 설정

EC2 인스턴스는 64bit ( $ getconf LONG_BIT ) JDK버전도 64bit이므로 아무 문제 없었고,

구글링으로 Java 힙 크기, 스레드 스택 크기 줄이기를 시도 해보았지만 오류 해결에는 도움이 되지 않았습니다.

또한 물리적 메모리는 늘릴수 없기 때문에, **SWAP 공간** 늘리는 방법을 시도해 보았습니다.

SWAP 공간이 없는 것을 확인하고

```Console
$ free -h
```

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins23.JPG">	
</div>

SWAP 파일을 생성합니다.

크기는 1.5 ~ 2배정도로 설정하는 것을 권장합니다.

```Console
$ touch /var/spool/swap/swapfile 
$ dd if=/dev/zero of=/var/spool/swap/swapfile count=2048000 bs=1024
```

시스템에서 접근 가능하도록 600 권한을 줍니다. 파일 소유권은 root:root(chown root:root)

```Shell
$ chmod 600 /var/spool/swap/swapfile
```

파일 포맷을 SWAP으로 변환 후 SWAP file등록

```Console
$ mkswap /var/spool/swap/swapfile
$ swapon /var/spool/swap/swapfile
```

파일시스템테이블(/etc/fstab)에 등록

```Console
$ vi /etc/fstab
```
```Vim
#
# /etc/fstab
# Created by anaconda on Tue Jun 18 17:03:37 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#

#추가
/var/spool/swap/swapfile    none    swap    defaults    0 0
```

SWAP파일이 정상적으로 등록되었는지 확인합니다.

```Console
$ free -h
```

<div>
	<img class="img-fluid" src="/img/posts/jenkins/jenkins24.JPG">	
</div>


성공적으로 등록이 되었으며 빌드시 error없이 진행이 됩니다.