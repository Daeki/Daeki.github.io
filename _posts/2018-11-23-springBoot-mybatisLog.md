---
title:  "SpringBoot Jsp(View) 연동"
excerpt: "SpringBoot Jsp(View) 연동"

categories:
  - springBoot
tags:
  - spring
  - spring boot
  - jsp
  - jstl
  - EL
classes: wide 
---

Spring Boot에서는 여러가지 VIew를 지원한다.

- JSP
- Thymeleaf
- FreeMarker
- Groovy
- Velocity

Back-end 쪽은 Rest API 서버용으로 사용하고

Front-end 를 독립적으로 개발 하는 형태가 많아 지고 있음.

Boot에서는 JSP를 사용하자 말것을 권고 하고 있고 Thymeleaf 사용을 권장하고 있다

------

##### 1. View(JSP)파일을 생성할 디렉터리 생성

- Legacy와 동일하게 /WEB-INF/views/ 경로를 설정하려 하지만 디렉토리가 없음
- 개발자가 src/main/webapp/ 하위에 /WEB-INF/views/ 디렉터리를 생성
- 프로젝트 생성시 Packaging을 war로 선택해야 webapp 디렉터리가 만들어지니 꼭 war로 선택 !!

##### 2. application.properties 작성

- Legacy에서는 InternalViewResolver를 servlet-context.xml에 명시되어 있다.
- Boot에서는 servlet-context.xml 대신 application.properties에 prefix와 suffix를 설정한다

```properties
# application.properties 에 추가 
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

##### 3. pom.xml Defendency 추가

- JSP를 기본 제공하지 않기 때문에 JSTL등이 포함되어 있지 않다.,

```xml
<!-- pom.xml -->
<!-- version은 명시 하지 않는다 -->

<!--tomcat-jasper for JSP in Sprng boot -->
<dependency>
	<groupId>org.apache.tomcat.embed</groupId>
	<artifactId>tomcat-embed-jasper</artifactId>
</dependency>

<!-- jstl -->
<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>jstl</artifactId>
</dependency>

```

