---
title:  "SpringBoot Mybatis"
excerpt: "SpringBoot Mybatis"

categories:
  - springBoot
tags:
  - spring
  - spring boot
  - mybatis
classes: wide 
---

##### 1. Project 생성

![Project 생성](/assets/images/2018-11-30-springBoot-myBatis.PNG)

- oracle 이 아닌 mysql을 쓴다면 mysql driver를 선택

##### 2. application.properties

SpringBoot에서 tomcat-jdbc를 기본 Datasource 로 제공되었지만

2.0부터 HikariCP가 기본으로 변경

```properties
# HikariCP 가 기본 제공 되어 있어 따로 Bean 생성 없이 기본 정보만 넣어주면 객체가 생성됨    
# SqlSessionTemplate, SqlSession은 기본 제공 되므로 생성 할 필요가 없다.    

#------------------------- Oracle -------------------------#
spring.datasource.hikari.username=spring01
spring.datasource.hikari.password=spring01
spring.datasource.url=jdbc:oracle:thin:@192.168.184.4:1521:xe
# -- hikari.jdbc-url 속성명은 연결되지 않는다. 그래서 일반 url 사용
#spring.datasource.hikari.jdbc-url=jdbc:oracle:thin:@192.168.184.4:1521:xe
spring.datasource.hikari.driver-class-name=oracle.jdbc.driver.OracleDriver

# ------- 또는 
spring.datasource.username=spring01
spring.datasource.password=spring01
spring.datasource.url=jdbc:oracle:thin:@192.168.184.4:1521:xe
spring.datasource.driver-class-name=oracle.jdbc.driver.OracleDriver

#------------------------- My Sql -------------------------#
spring.datasource.username=spring01
spring.datasource.password=spring01
spring.datasource.hikari.driver-class-name=com.mysql.cj.jdbc.Driver
# mysql 8 이후 : com.mysql.cj.jdbc.Driver
# mysql 8 이전 : com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/user

# ------- 또는 
spring.datasource.username=spring01
spring.datasource.password=spring01
spring.datasource.url=jdbc:mysql://localhost:3306/user
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

```



##### 3. VO(DTO) & Mapper(DAO) 클래스 생성

```java 
------------------------------- VO
package com.iu.s1.board;
public class BoardVO {
	
	private int num;
	private String title;
	private String writer;
	private String contents;
	private Date regDate;
	private int hit;
  ---- Getter Setter 
  }


-------------------------------- Mapper

package com.iu.s1.board;
//@Repositpry 선언은 생략 해도 상관 없음
@Mapper
public interface BoardMapper {
	
	public BoardVO boardSelect(BoardVO boardVO) throws Exception;
	
	public int boardWrite(BoardVO boardVO)throws Exception;

}

------------------------------------------------------------------------------------
@Mapper

Legacy 에서는 Mapper(DAO) 를 사용시 @Repository 를 이용해 객체 생성하고 사용
Boot 에서는 Mapper(DAO)가 Interface이기 때문에 @Repository만 단독 사용하면 객체 생성이 안됨
@Mapper는 Interface를 구현 클래스로 만들어 객체를 생성해 주는 역활
@Mapper만 선언해도 되고 @Repository를 같이 선언해도 무방
```



##### 4. Mapper.xml의 파일 위치

1) src/main/java하위의 Mapper(DAO)와 같은 패키지 내

- application.properties 파일에 추가 작업 필요 X

2) src/main/resource 하위 (추가 패키지 생성 상관  X)

![Project 생성](/assets/images/2018-11-30-springBoot-myBatis2.png)

- application.properties 파일에 Mapper.xml 파일의 경로를 명시

```properties
## application.properties
## mapper.xml 파일의 경로 작성
mybatis.mapper-locations=classpath:/data/**/*Mapper.xml
```



##### 5. Mapper.xml 파일 생성

```bash
# namespace의 이름은 사용하는 Mapper(DAO)의 패키지명.클래스명 과 동일한 이름으로 작성
<mapper namespace="com.iu.s1.board.notice.NoticeRepository">
	# id 이름은 Mapper(DAO)의 메서드명과 동일한 이름으로 작성
	<select id="boardSelect" ...> select 쿼리문 </select>
	<insert id="boardWrite" ...> insert 쿼리문 </insert>
```



##### 6. Mybatis Config 파일 생성

- Alias 등록 
- src/main/resources 하위에 생성
- ex) src/main/resources/database/mybatisConfig.xml을 생성한 경우

```bash
# src/main/resources/database/mybatisConfig.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
	<typeAliases>
		<typeAlias type="java.util.List" alias="List"/>
		<typeAlias type="com.iu.s1.board.notice.NoticeVO" alias="NoticeVO"/>
	</typeAliases>
</configuration>  
```



```properties
# application.properties
# applocation.properties 파일에 config 파일의 위치를 지정
mybatis.config-location=classpath:database/mybatisConfig.xml
```



##### 7. Tip - Insert 시 자동 생성 키 사용

```xml
---- Mysql
# keyProperty="num" -> BoardVO의 멤버변수명
# AutoIncement로 작성된 번호를 num에 대입 후 쿼리문 실행

<insert id="boardWrite" parameterType="BoardVO" useGeneratedKeys="true"  keyProperty="num">
  insert into notice values (#{num}, #{title},....)
</insert>


---- Oracle
# keyProperty="num" -> BoardVO의 멤버변수명
# Sequence로 작성된 번호를 num에 대입 후 쿼리문 실행

<insert id="boardWrite" parameterType="BoardVO">
 <selectKey keyProperty="num" resultType="int" order="BEFORE">
    select board_seq.nextval FROM DUAL
  </selectKey>
    insert into notice values (#{num}, #{title},....)
</insert>
```



##### 7. Tip - like 검색

```xml
---- MySql
<select id="boardWrite" parameterType="Pager" resultType="BoardVO">
	...
    where contents like like concat(‘%’, #{search},’%’)
</select>

---- Oracle
<select id="boardWrite" parameterType="Pager" resultType="BoardVO">
	...
    where contents like '%'||#{search}||'%'
</select>

```

##### 7. Tip - pageing 

```xml
-- oracle은 limit 명령어가 없어서 rownum울 사용
-- mysql은 limit 명령어 사용

-- Mysql
-- limit 시작번호, 갯수
-- where 절에 쓰는게 아니라 order by 뒤에 씀
<select id="boardWrite" parameterType="Pager" resultType="BoardVO">
	select * from notice where num > 0 order by num desc limit 10, 5
</select>

-- Oracle
-- rownum 사용
<select id="boardWrite" parameterType="Pager" resultType="BoardVO">
    select * from
    (select N.*, rownum R from
    (select * from notice where num > 0 order by num desc) N)
    where R between 1 and 10;
</select>
```

