---
title:  "3. SpringBoot DI & IOC"
excerpt: "3. SpringBoot DI & IOC"

categories:
  - springBoot
tags:
  - springboot
  - Spring boot
  - DI
  - IOC	
classes: wide 
---

### 1. Annotation을 이용한 DI & IOC

##### 1) Bean 등록(객체 생성) Annotation

Bean을 생성하려 하는 Class 선언부에 역할에 맞는 Annotation 선언

```bash
# @Controller		: 객체 생성, Controller 역할 부여 
# @Service		: 객체 생성, Service 역할 부여 
# @Repository		: 객체 생성, Repository(DAO) 역할 부여 
# @Component		: 객체 생성, 그외 나머지 역할
# @RestController	 : 객체 생성, 비동기 처리방식을 사용할 때 모든 메서드에 @ResponseBody 추가 해줌
# @AdviceController	 : 객체 생성, 예외 처리 Controller

# ex)
@Component
public class Sample{}
```

Java에서 객체 생성시 참조변수명을 설정하는것처럼 Bean container를 이용해 객체를 생성 할때 이름을 만들 수 있다
이름을 따로 지정하지 않으면 클래스명의 첫글자를 소문자로 한것이 이름이 되고, 다른 이름을 사용하고 싶은면 다음과 같이 진행

```bash
# @Autowired로 Inject 시 사용할 이름 지정
@Component
@Qualifier("name")
public class Sample{}


# @Resource로 Inject 시 사용할 이름 지정 
@Component("name")
public class Sample{}

# @Qualifier("name") 와 @Component("name") 의 이름은 @Autowired 와 @Resource 둘다 사용 가능
```



##### 2) Bean Injection (객체 주입) Annotation

생성된 객체를 필요한 곳에 주입

- Setter
- Constructor
- Field

```bash
# Autowried는 같은 Type을 Inject
# 같은 Type이 있다면 같은 이름을 Inject
# 다른 이름을 명시하고 싶으면 @Qualifier("다른이름") 으로 지정

@Autowired
@Qualifier("name")
private Sample sample;

# Resource는 이름으로 Inject
# 이름은 같지만 Type이 다르면 error
@Resource(name="name")
private Sample sample;
```



##### 3) 예시

```java
// Arm super class
public abstract class Arm {  }   


// Type : PartArmRight or Arm
// Name : partArmRight 라는 이름으로 객체 생성
@Component    
public class PartArmRight extends Arm{    }    


// Type : PartArmLeft or Arm
// Name : partArmLeft 라는 이름으로 객체 생성
@Component    
public class PartArmLeft extends Arm{    }    


// 객체 주입 - 1
@Component
public class RobotPengHa {
	// PartArmRight Type을 찾아 주입
    @Autowired
    private PartArmRight partArmRight;

	// PartArmLeft type을 찾아 주입
    @Autowired
    private PartArmLeft partArmLeft;
}

// 객체 주입 - 2 
// 부모형의 데이터타입 을 선언
@Component
public class RobotPengHa {
    // partArmRight, partArmLeft 둘 모두 Arm 타입 이지만
    // Name으로 구분해서 Inject 해줌
    @Autowired
    private Arm partArmRight;
    @Autowired
    private Arm partArmLeft;
}


// 객체 주입 - 3
// 부모형의 데이터타입 을 선언을 하고 이름이 다를 때는 Error 발생
// 멤버변수가 Arm 타입(부모) 으로 선언되어 있고 같은 타입의 Bean이 두개 있다.
// 이름으로 구분하려 하지만 해당 멤버변수명으로 선언된 이름이 없어 에러 발생
@Component
public class RobotPengHa {
    @Autowired
    private Arm right;
    @Autowired
    private Arm left;
}

//--------------------------------------------------------------------

// error 해결 
@Component
public class RobotPengHa {

    // @Qualifier 를 이용한 error 해결
    // 부모 타입 중 이름이 partArmRight 인것을 Injcect
    @Autowired
    @Qualifier("partArmRight")
    private Arm right;
    
    // @Resource 를 이용한 error 해결
    // 이름이 partArmLeft 인것을 Injcect
    @Resource(name = "partArmLeft")
    private Arm left;
}

```



### 2. Java를 이용한 DI & IOC

##### 1) 클래스 선언부에 @Configuration 선언

- 이 클래스는 객체를 정의하는 class를 의미

##### 2) 객체를 만들 메서드를 선언하고 선언부에 @Bean을 선언

- Annotation을 쓰는것과 달리 기본 이름이 만들어 지지 않는다. 
- @Qulitifier("이름") 이나 @Bean("이름")을 사용하여 이름을 지정 할 수 있다.

##### 3) 객체를 생성하고 리턴

##### 4) 예시

```java
public abstract class Arm {}    

public class PartArmLeft extends Arm{}     

public class PartArmRight extends Arm{}

-----------------------------------------
    
public class RobotPengHa {
  
   //partArmRight는 @Autowired를 이용한 주입 
   @Autowired
   private PartArmRight partArmRight;

    //partArmLeft는 configuration에서 setter 를 이용한 주입
    private PartArmLeft partArmLeft;

    public void setPartArmLeft(PartArmLeft partArmLeft) {
        this.partArmLeft = partArmLeft;
    }
    public PartArmLeft getPartArmLeft() {
        return partArmLeft;
    }
    public PartArmRight getPartArmRight() {
        return partArmRight;
    }
    public void setPartArmRight(PartArmRight partArmRight) {
        this.partArmRight = partArmRight;
    }
}

----------------------------------------------------------
@Configuration
public class RobotPengHaConfig {

    //PartArmLeft 객체를 la 라는 이름으로 생성
    @Bean("la")
    public PartArmLeft getPartArmLeft() {
        return new PartArmLeft();   
    }

    //PartArmRight 객체를 ra 라는 이름으로 생성
    @Bean("ra")
    public PartArmRight getPartArmRight() {
        return new PartArmRight();
    }

    @Bean("robotPengHa")
    public RobotPengHa getRobotPengHa(PartArmLeft partArmLeft) {
        RobotPengHa robotPengHa = new RobotPengHa();
        // PartArmRight 객체는 @Autowired로 자동 주입
        // PartArmLeft 객체는 setter 주입
        // 매개변수로 선언된 PartArmLeft는 생성된 PartArmLeft 객체를 주입 받는다.
        // 메서드 선언부에 @Autowried를 선언 해야 하지만 Spring 4 이후 부터는 생략 가능 하다
        robotPengHa.setPartArmLeft(partArmLeft);
        return robotPengHa;
    }

}    

```





