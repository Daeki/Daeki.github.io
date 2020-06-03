---
title:  "Console창에 SpringBoot Mybatis SQL Log 출력"
excerpt: "SpringBoot Mybatis SQL Log"

categories:
  - springBoot
tags:
  - spring
  - spring boot
  - mybatis
  - log4j
classes: wide 
---

- MyBatis sql문 실행시 SQL문의 실행 쿼리와 결과물을 Console에 출력
- 개발시 어떤 쿼리가 실행되고 어떤 값이 들어가는지 확인 할 수가 있어 개발에 용이

##### 1. API

pom.xml에 다음 API를 추가

```xml
<!-- https://mvnrepository.com/artifact/org.bgee.log4jdbc-log4j2/log4jdbc-log4j2-jdbc4.1 -->
<!-- 버전은 최신 버전으로 -->
<dependency>
	<groupId>org.bgee.log4jdbc-log4j2</groupId>
	<artifactId>log4jdbc-log4j2-jdbc4.1</artifactId>
	<version>1.16</version>
</dependency>
```



##### 2. Application.properties 수정

```properties
## 수정 전 
spring.datasource.username=user01
spring.datasource.password=user01
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/user

## 수정 후 
## spring.datasource.hikari... 속성을 사용하면 error 발생
spring.datasource.username=user01
spring.datasource.password=user01
spring.datasource.driver-class-name=net.sf.log4jdbc.sql.jdbcapi.DriverSpy
spring.datasource.url=jdbc:log4jdbc:mysql://localhost:3306/user

## 다음을 추가
## resources 폴더에 log4jdbc.log4j2.properties 파일을 생성하고 다음의 내용을 추가 해도 되고
## 별도의 파일 생성 없이 application.properties 파일에 추가 해도 됌
log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
```



##### 3. 출력할 log 설정

```xml
<!--기본 설정은 모든 로그를 출력 -->
<!--로그 출력 형식을 설정해서 원하는 로그만  출력 가능 -->
<!-- resources 폴더 내에 logback.xml 을 생성 -->
<!-- Google에 검색하면 많은 설정 형식이 존재. 맘에 드는것을 사용 -->
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{yyyyMMdd HH:mm:ss.SSS} [%thread] %-3level %logger{5} - %msg %n</pattern>
    </encoder>
  </appender>

  <logger name="jdbc" level="OFF"/>
  <logger name="jdbc.sqlonly" level="OFF"/>
  <logger name="jdbc.sqltiming" level="DEBUG"/>
  <logger name="jdbc.audit" level="OFF"/>
  <logger name="jdbc.resultset" level="OFF"/>
  <logger name="jdbc.resultsettable" level="DEBUG"/>
  <logger name="jdbc.connection" level="OFF"/>

  <root level="INFO">
    <appender-ref ref="STDOUT" />
  </root>
    
</configuration>

```



```properties
jdbc.sqlonly : SQL문만을 로그로 남기며, PreparedStatement일 경우 관련된 argument 값으로 대체된 SQL문을 출력
jdbc.sqltiming : SQL문과 해당 SQL을 실행한 시간 정보(milliseconds)를 포함.
jdbc.audit : ResultSet을 제외한 모든 JDBC 호출 정보를 출력, 로그 양이 많아서 특별한 문제가 아니면 사용을 권장하지 않음
jdbc.resultset : ResultSet을 포함한 모든 JDBC 호출 정보를 출력 매우, 방대한 양의 로그가 생성
jdbc.resultsettable : SQL 결과 조회된 데이터의 table을 로그로 출력
```