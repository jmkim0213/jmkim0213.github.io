---
layout: post
title:  "[AWS] Tomcat 설치 & 설정 및 War 배포 (with yum)"
date:   2019-03-31 22:30:00 +0900
categories: [AWS, SpringBoot, Tomcat]
---

# 서론
개인 프로젝트를 진행하면서 Spring Web Service를 EC2에 Deploy하기 위해 Tomcat을 설치했습니다.
간단하게 기록용으로 글을 작성합니다 :) 

----

### JDK 설치하기
- JDK8 설치  
$ yum list | grep openjdk  
$ sudo yum install java-1.8.0-openjdk

- 기존 버전 설치되어있을 시  
$ sudo /usr/sbin/alternatives --config java  
$ sudo yum remove java-1.7.0-openjdk

- 버전확인  
$ java -version

----

### 톰캣 설치하기
- 명령어  
$ yum list | grep tomcat8  
$ sudo yum install tomcat8  
$ sudo yum install tomcat8-admin-webapps  
$ sudo yum install tomcat8-webapps

----

### 관리자 로그인 기능 활성화
- 명령어  
$ cd /usr/share/tomcat8/conf/Catalina/localhost  
$ sudo vim ./manager.xml

{% highlight xml %}
<Context privileged="true" antiResourceLocking="false" docBase="${catalina.home}/webapps/manager"> 
  <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />
</Context>
{% endhighlight %}

----

### 톰켓 관리자 설정
- 명령어  
$ cd /usr/share/tomcat8/conf  
$ sudo vim ./tomcat-users.xml

----

### 톰캣 실행하기
- 명령어  
$ sudo service tomcat8 start

----

### War파일 배포하기
- http://{host}:8080/manager/html

----