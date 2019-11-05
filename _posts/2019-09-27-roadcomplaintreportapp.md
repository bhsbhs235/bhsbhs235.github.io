---
layout: post
title: "도로 불편사항 신고 앱"
subtitle: "Road Complaint Report App"
date: 2019-09-27 09:00:00
background: '/img/posts/bg-img/21.jpg'
comments: true
categories: Projects
---
<style>
	li {
		font-weight: bold;
	}
</style>
<h1 class="section-heading2" >작품개요</h1>
차량용 스마트 불편신고 어플리케이션은 라즈베리파이와 카메라 모듈, 버튼형 블루투스, 비콘 모바일 기기를 이용하여 동작한다 버튼형 블루투스 비콘은 사용자가 운전 중에 
간편하게 기능을 동작시킬 수 있도록 간단한 버튼이 부착된 비콘 기기이다 비콘 기기에 있는 버튼을 클릭하면 상황에 맞는 비콘 데이터가 주변의 블루투스 장치로 전달된다 이 
데이터를 받은 라즈베리파이는 비콘 데이터를 인식하여 카메라 모듈을 실행시켜 사진 또는 동영상을 촬영한다 이와 같은 방법으로 촬영된 사진 또는 영상 데이터는 사용자의 
모바일 기기로 전달되고 개발하는 스마트 불편 신고 어플리케이션을 통해 신고를 진행한다 차량용 스마트 불편신고 어플리케이션이 동작하는 과정은 다음 그림과 같다.

<img class="img-fluid" src="/img/posts/projects/senierproject2.PNG">

본 과제에서는 위의 구조를 바탕으로 내용을 수행하기 위하여 버튼형태의 비콘과 라즈베리파이 카메라 모듈을
이용하며 안드로이드 기반의 불편신고 어플리케이션을 개발한다. 라즈베리파이에서는 버튼형 비콘으로부터  
비콘 데이터를 수신하는 기능과 수신한 비콘 데이터를 확인하는 부분, 확인된 비콘 데이터에 대해 카메라 
모듈을 실행하여 사진을 촬영하는 부분으로 기능을 구성한다. 모바일 어플리케이션에서는 라즈베리파이로부터 
전달 받은 사진 데이터를 현재 시간과 모바일 기기의 GPS상 위치 등을 바탕으로 신고 문자를 작성하는 등의 기능을 수행한다.

<h2 class="section-heading2">중요성과 필요성</h2>

최근 서울특별시에서는 시민들이 생활 중에 느끼는 각종 불편사항이나 신고사항을  손쉽게 신고할 수 있도록 ‘서울 스마트 불편신고’ 모바일 어플리케이션을 서비스하고 있다.
이와 같은 신고 방법은 기존의 민원을 제기하는 방법의 복잡함과 불편함을 해소하기 위한 방안으로 등장한 해결방안이다. 실제로 일상생활에서의 불편한 점이나 사건･사고에 대한 
신고의 필요성을 느끼는 사람에 비해 사진이나 영상을 촬영하고 신고를 하는 사람은 극히 드물다. 특히, 차량을 운행하는 운전자들은 사고 현장이나 불법 주･정차 차량을 발견하는 등 
다양한 현장을 목격하지만 운전 중이라는 문제로 인하여 신고를 하기 어려움이 있다. 또한, 최근에는 차량용 블랙박스 이용자가 늘어남에 따라 블랙박스를 이용하여 영상을 녹화하고 이후 
녹화된 영상으로 신고를 하는 사람들도 있다. 하지만 이는 즉각 신고를 해야하는 사고 현장에 대해 신고가 어렵다는 점과 지난 뒤 신고를 잊게 되는 문제점이 있다.
본 과제에서는 운전자들이 운전 중에 간편하게 사건･사고 현장을 신고할 수 있는 차량용 스마트 불편신고 어플리케이션을 개발한다. 개발하는 어플리케이션은 라즈베리파이와 버튼형 블루투스 
비콘을 이용하여 운전 중인 운전자들도 버튼클릭 한 번으로 손쉽게 사진을 촬영하고 신고를 할 수 있는 서비스를 제공한다.

<h3 class="section-heading2">시스템 구성</h3>

<img class="img-fluid" src="/img/posts/projects/senierproject3.PNG">
<div style="text-align: center;">
<img class="img-fluid" src="/img/posts/projects/senierproject4.PNG" align="center">
</div>

<h4 class="section-heading2">기능</h4>

<ol>
	<li>자동 신고 기능</li>
		<p style="margin: 0;">버튼형 비콘을 누르면 라즈베리파이가 비콘 데이터를 받아 편집된 동영상, 사진을 스마트폰으로 전달하고 구글맵 데이터와 신고내용을 통합해 MMS창으로 띄운다 
		그리하여 사용자가 보내기만 할 수 있도록 도와준다</p>
		<br>
		<div style="text-align: center;">
		<img class="img-fluid" src="/img/posts/projects/senierproject8.jpg" align="center">
		</div>
		<br>
	<li>환경 설정 기능</li>
		<p style="margin: 0;">사용자가 동영상을 첨부할지,사진으로 첨부할지를 선택할 수 있고, 세부내용으로 사진은 3장, 5장, 8장 동영상은 45초, 60초, 90초로 사용자가 원하는 것으로 
		선택가능하다. 사용자가 자동신고 기능을 원하지 않을 수 있으므로 설정할 수 있도록 하였다</p>
		<br>
		<div style="text-align: center;">
		<img class="img-fluid" src="/img/posts/projects/senierproject4.jpg" align="center">
		</div>
		<br>
	<li>블루투스 페어링 기능</li>
		<p style="margin: 0;">라즈베리파이와 모바일로의 데이터 송수신을 블루투스 소켓 프로그래밍으로 구현하여 다른 네트워크 지원이 없어도 사용자가 쉽게 사용할 수 있다</p>
		<br>
	<li>비콘 uuid 등록, 관리</li>
		<p style="margin: 0;">앱에서 버튼형 비콘 수신에 필요한 UUID를 등록 할 수 있으며 관리 할 수 있다.</p>
		<br>
	<li>구글맵, 플레이스 기능</li>
		<p style="margin: 0;">사용자가 수동으로 신고를 할 경우 원하는 위치를 쉽게 찾을 수 있도록 
		GooglePlaceAPI, 정확한 위치의 위도와 경도를 가져올 수 있도록 GoogleMapAPI를 사용하였다.</p>
		<br>
		<div style="text-align: center;">
		<img class="img-fluid" src="/img/posts/projects/senierproject9.jpg" align="center">
		<img class="img-fluid" src="/img/posts/projects/senierproject10.jpg" align="center">
		<img class="img-fluid" src="/img/posts/projects/senierproject11.jpg" align="center">
		</div>
		<br>
	<li>모바일 앱 기능</li>
		<p style="margin: 0;">위의 내용을 제외한 앱의 기능으로 수동신고기능, 신고내역 조회하기가 있다.</p>
		<br>
		<div style="text-align: center;">
		<img class="img-fluid" src="/img/posts/projects/senierproject5.jpg" align="center">
		<img class="img-fluid" src="/img/posts/projects/senierproject6.jpg" align="center">
		<img class="img-fluid" src="/img/posts/projects/senierproject7.jpg" align="center">
		</div>
		<br>
</ol>

# 참고자료

- [소스 코드](https://github.com/bhsbhs235/RoadComplaintReportApp)

- [실생활 활용 영상](https://www.youtube.com/watch?v=NmmcdeuiXrk)

- [앱기능 영상](https://www.youtube.com/watch?v=CQEOzQg9Bb8)






	