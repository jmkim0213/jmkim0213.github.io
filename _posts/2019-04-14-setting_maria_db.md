---
layout: post
title:  "[SpringBoot] SpringBoot MariaDB 연동"
date:   2019-04-14 16:30:00 +0900
categories: [SpringBoot]
---

# 서론
SpringBoot와 MaridDB를 연동하기 위한 설정 기록용 글입니다 :)

----

### JDBC 플러그인 설치
- build.gradle파일에 의존성 추가

dependencies {
  ...
  ...
  ...
  compile("org.mariadb.jdbc:mariadb-java-client")
}

----

### Application 설정
src/main/resources/application.properties 파일에 다음 내용 추가

spring.datasource.driverClassName=org.mariadb.jdbc.Driver
spring.datasource.url=jdbc:mariadb://{host}:{port}/{db_name}
spring.datasource.username={user_name}
spring.datasource.password={password}