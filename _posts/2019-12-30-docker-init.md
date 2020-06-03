---
title:  "SpringBoot Junit Test"
excerpt: "SpringBoot Junit Test"

categories:
  - springBoot
tags:
  - spring
  - spring boot
  - Junit
classes: wide 
---



#####  1. Junit

Spring boot 2.0 이상 부터는 Junit 5 가 기본 내장되어 있다.
Junit4 도 지원 가능 하지만 Junit 4 선택시 추가 API를 다운로드 하라는 메세지를 출력

**귀찮으니 기본 내장 되어 있는 Junit 5 를 사용 하도록 하자. **



##### 2. Junit Test Case 생성

- src/test/java 패키지에 작성

```bash
# new > Junit Test case 선택  
# 기본 버전은 new Junit jupitor Test 선택  
# 패키지는 프로젝트 생성시 기본 패키지 하위로 만들어야 한다. 
# boot는 legecy 처럼 따로 component-scan을 설정하는 servlet.xml 이 없다.
# 기본 scan 범위는 기본 패키지 하위로 정해져 있다.
# 기본 패키지나 하위패키지의 범위를 벗어나면 Annotation을 읽지 못한다.  
```



- Junit Test case 선언부에 @SpringBootTest 선언

```java
@SpringBootTest
class SpringBoot1ApplicationTests {

	@Test
	void contextLoads() {
	}

}
```



##### 3.  SpringBoot1ApplicationTests 상속 받아서 Test case 생성 

```bash
# SpringBoot1ApplicationTests의 지정자가 default로 되어 있어 다른 패키지 상속 해도 접근이 불가
# SpringBoot1ApplicationTests의 지정자를 public으로 수정
# 상속받은 case 에서는 @SpringBootTest 선언을 하지 않고 사용 가능
```

```java
@SpringBootTest
public class SpringBoot1ApplicationTests {}

public class SampleTest extends SpringBoot1ApplicationTests {  }
```

##### 4, Test Annotation

```java
//@BeforeAll(BeforeClass)  : 테스트 클래스 시작시 딱 1번 호출
@BeforeAll
public static void beforeAllTest() {}

//@AfterAll(AfterClass)      : 테스트 클래스 종료시 딱 1번 호출
@AfterAll
public static void AfterAllTest() {}

//@BeforeEach(Before)      : 각각의 테스트 메서드 실행전 호출
@BeforeEach
public void beforeEachTest(){}

//@AfterEach(After)          : 각각의 테스트 메서드 실행후 호출
@AfterEach
public void afterEachTest(){}

//@Test						: 테스트를 수행할 메서드
@Test
public void test() {}

```

##### 5. assert

- assert의 상세내용은 여기서 [Assert Reference](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/Assertions.html)