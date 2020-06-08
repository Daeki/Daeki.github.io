---
title:  "Spring Boot Hibernate 연동"
excerpt: "Spring Boot Hibernate 연동"

categories:
  - springBoot
tags:
  - spring
  - spring boot
  - hibernate
classes: wide 
---

### 1. Project 생성

필요 API를 다음과 같이 선택

lombok을 사용하지 않는 다면 제외

### 2. Application.properties

```properties
## DataSource 연결 정보
spring.datasource.username=user02
spring.datasource.password=user02
spring.datasource.url=jdbc:mysql://localhost:3306/user02
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

#--------------------- hibernate -------------------------------

##### table 생성
spring.jpa.hibernate.ddl-auto=update

## create		: 기존 테이블 삭제 후 다시 생성
## create-drop	: create와 같으나 application이 종료 될때 테이블을 drop
## update		: 변경된 부분만 적용
## validate		: 엔티티(Entity, VO)와 테이블이 정상적으로 맵핑 되었는지 확인
## none			: 사용하지 않을 때
## 개발시 편의를 위해 create, create-drop
## 운영시에는 update, validate, none

##### VO클래스의 멤버변수명이 자동으로 DB에 컬럼명과 연결 될때
## regDate 형태면 설정이 필요 (camel case )
## reg_date 형태면 설정이 필요 없음 ( snake case )
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl

##### JPA에서 자동으로 생성되고 실행되는 SQL문을 실시간 출력
logging.level.org.hibernate.sql=debug

##### ? 에 맵핑 되는 파라미터 값 출력
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE

##### 하이버네이트가 실행한 SQL 출력
spring.jpa.show-sql=true

##### 하이버네이트가 실행한 SQL 출력 할 때 보기 쉽게 출력
spring.jpa.properties.hibernate.format_sql=true

##### transaction 처리
spring.aop.proxy-target-class=true 
```



### 3. 예제 Entity(VO) 생성

```java
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;
import javax.persistence.Transient;

import lombok.Data;

@Data //lombok을 쓰지 않는다면 생략 하고 getter/setter 생성
@Entity
@Table(name="member")
public class MemberVO {
	@Id //primary key
	private String id;
	
	@Column //일반 컬럼명
	private String pw;
	
	@Transient //테이블에서 제외
	private String pwCheck;
	
	@Column
	private String name;
	
	@Column
	private String email;
	
	@Column
	private String phone;
}


```



```bash
A. 객체와 테이블 매핑
클래스의 선언부에 작성

	1. @ Entity
	Entity Manager가 관리하는 것을 의미
		
		속성
		1) name
			Entity의 이름을 지정, 다른 패키지나 다른 클래스에 같은 이름이 있으면 에러 발생 생략 하면 클래스의 이름을 기본값으로 지정
			(ex : Member, 첫글자가 소문자가 아님에 주의)
			
	2. @ Table
	Entity와 매핑할 테이블 연결, 생략하면 Entity의 이름과 같은 이름의 테이블과 매핑

		속성
		1) name : 매핑할 테이블 이름
		2) catalog : catalog 기능이 있는 데이터베이스에 catalog 매핑
		3) schema : schema 기능이 있는 데이터베이스에 schema 매핑
		4) UniqueConstraints : DDL 생성시 유니크 제약 조건 생성, 2개 이상의 복합키 가능,


B. 필드와 컬럼 매핑
멤버 변수의 선언부에 작성

	1. @ Column
	멤버 변수와 테이블의 컬럼명과 매핑
	생략 가능
	속성
		1) name : 매핑할 테이블의 컬럼 이름 생략시 변수명이 컬럼명
		2) insertable : true-> insert 허용, false->insert제외(읽기전용, 잘 사용 않함)
		3) updatetable : true-> update 허용, false-> update제외(읽기전용, 잘 사용 않함)
		4) table : 하나의 Entity를 두개 이상의 테이블에 매핑 할때 사용(거의 사용 안함)
		5) nullable(DDL) : null값 허용 여부, false 면 DDL 생성시에 not null 제약 조건 생김
		6) unique : 한 컬럼에 간단한 유니크 제약 조건 생성시 사용, 두 컬럼 이상이면 클래스 선언부에 @Table.uniqueContraints를 사용
		7) columnDefinition(DDL) : 데이터베이스 컬럼 정보를 직접 줄 때 사용
		8) length(DDL) : 문자 길이 제약 조건, String 타입에만 사용
		9) precision, scale(DDL) : BigDecimal 또는 BigInteger 타입에서 아용, precision은 소숫점 포함 전체 자릿수, scale은 소숫점 자릿수, 
		   double, float 타입에는 적용안되고, 큰 숫자나 정밀한 소수를 다뤄야 할 때 사용
		
		(ex) @Column(precision=10, scale=2)
			 private BigDecimmal num;

	2. @ Enumerated
	java enum 타입을 매핑 할때 사용

	속성
		1) EnumType.ORDINAL : enum 순서번호를 데이터베이스에 저장
		2) EnumType.STRING : enum 이름을 데이터베이스에 저장
		
	3. @ Temporal
	날짜 타입(java.util.Date, java.uti.Calendar)을 매핑 할 때 사용

	속성
		1) TemporalType.DATE : 날짜, 데이터베이스 Date타입과 매핑 (2020-01-01)
		2) TemporalType.TIME : 시간, 데이터베이스 Time타입과 매핑 (17:50:50)
		3) TemporalType.TIMESTAMP : 날짜와 시간, 데이터베이스 Timestamp 타입과 매핑 (2020-01-01 17:50:50)
		
	4. @ Lob
	데이터베이스 BLOB, CLOB 타입과 매핑
	CLOB : 멤버변수가 문자면(char [], String)
	BLOB : 멤버변수가 문자 이외(byte [] ...)

	5. @ Transient
	이 변수는 매핑 하지 않음, DB에 저장하거나 조회 하지도 않음
	객체에 임시로 변수를 보관하고 싶을 때 사용

C. 기본키 매핑
	@ Id
	PK로 사용되는 변수에 선언
	생략 가능
```



### 3. 기본키 할당 방식

mysql은 AUTO-INCREMENT, oracle은  SEQUENCE 를 사용하여 기본키를 지정하여 사용 할 수 있다.



##### 1) IDENTIFIED

- 기본키 생성을 데이터베이스에 위임
- Mysql, PostgreSQL, SQL Server, DB2 등에 주로 사용
- AUTO-INCREMENT 기능등을 사용

```
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private int num;
```



##### 2) SEQUENCE

- Oracle, PostgreSQL,H2, DB2 등에 주로 사용
- SEQUENCE 사용

```java
@Entity
@Table(name = "members")
@SequenceGenerator(name = "member", sequenceName = "member_seq", initialValue = 1, allocationSize = 1)
public class MemberVO {
    @Id
    @Column
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "member") //generator=generator이름
    private String id;
```

###### @ SequenceGenerator

​	속성
​		1) name : generator이름
​		2) sequenceName : sequence 이름
​		3) initialValue : sequence 시작 번호 지정 (DDL 생성시)
​		4) allocationSize : 시퀀스 한번 호출에 증가하는 수 5) catalog, schema : catalog, schema 이름



##### 3)그외

###### 3) TABLE

ID키를 생성하는 테이블을 생성하여 사용하는 방식

###### 4) AUTO

특정 데이터베이스에 맞게 자동으로 생성되는 방식