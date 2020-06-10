---
title:  "Spring Boot Hibernate OneToOne"
excerpt: "Spring Boot Hibernate OneToOne"

categories:
  - springBoot
tags:
  - spring
  - spring boot
  - hibernate
classes: wide 
---

## 1. OneToOne

- @OneToOne을 사용하지 않는 것을 권장
- @OneToOne은 Lazy Loading에 심각한 문제가 있을 수 있다.
- 1:1 관계이면 두개의 테이블을 생성하는 것 보다 하나의 테이블로 관리 하는것이...????
- @OnetoMany, @ManytoOne을 이용해서 0 번 인덱스만 꺼내 쓰는 걸로....

### A. 기본 구성

- 회원 한명당 하나의 이미지 파일을 가지고 있다는 가정
- 회원:이미지파일은 1:1의 관계 임..
- Members Table : 회원정보를 담고 있고
- MemberFiles Table : 이미지 파일을 담고 있고, id 컬럼은 Memers Table을 참조하는 Foreign Key 임
- Members Table이 Parent
- MemberFiles Table이 Child
- FK를 소유하고 있는 Table을 Owner라고 칭함(FK 관리 할 수 있는 테이블, MemberFiles)
- Select 시 id column 으로 Join (inner or outer) 실행



![img](https://wikidocs.net/images/page/64561/005_1_hibernate_2_onetoone.png)



```
CREATE TABLE  members (
  `id` VARCHAR(32) NOT NULL,
  `pw` VARCHAR(32) NOT NULL,
  `name` VARCHAR(255) NULL,
  `email` VARCHAR(255) NULL,
   PRIMARY KEY (`id`))
   ENGINE = InnoDB;


CREATE TABLE memberFiles (
  `num` int auto_increment,
  `id` VARCHAR(32) NOT NULL unique,
  `fname` VARCHAR(255) NULL,
  `oname` VARCHAR(255) NULL,
   PRIMARY KEY (`num`),
   constraint memberFiles_fk foreign key (`id`) references members (`id`) on delete cascade 
   )
   ENGINE = InnoDB;
```





### B. VO(DTO) 구성

- MemberVO 의 멤버변수로 MemberFilesVO를 선언 하는것이 전통방식의 VO 구조
- JPA 에서는 MemberVO내에 MemberFilesVO를 선언하고
- MemberFilesVO의 멤버변수로 MemberVO도 선언한다.
- 단, MemberFilesVO내에 id 컬럼은 작성하지 않는다.(MemberVO로 대체, 부모를 참조하는 FK 선온 안함)
- id 컬럼을 명시하면 error 발생!!!!!!!!
- FK가 PK역활도 같이 하면 생략 하면 안대!!!





```java
@Entity
@Table(name = "members") //맵핑할 테이블명 지정, 클래스명과 테이블명이 동일하면 생략 가능
public class MemberVO {

    @Id
    private String id;
    private String pw;
    @Transient
  //테이블의 칼럼에 매핑되지 않는 속성을 지 정하며 임시로 값을 저장하는 용도로 사용
  //테이블의 컬럼에는 없는 속성, 폼검증등 Java에서 사용할 목적등으로 필요할때.
    private String pw2;
    private String name;
    private String email;

    @OneToOne(mappedBy = "memberVO", cascade = CascadeType.ALL)
  //------------ mappedBy ------------
  // mappedBy="자식 VO(DTO)에 선언된 부모 VO(DTO) 의 변수명"
  // MemberFilesVO클래스의 멤버변수명인 memberVO
  // 테이블명도 아니고, 컬럼명도 아니다, Java의 멤버변수명이다..
  //------------------------------------------------------------------------------------------
  //------------ cascade ------------
  // 부모테이블을 삭제 할때 자식 테이블은 어떻게 할 것인가...
  // 테이블 생성시 cascade 옵션을 지정 하지 않았다면 
  // jpa에서 할 수 있다.
 // 테이블 생성시에 하는게 좋을 듯... 좀더 알아 보자!!!
    private MemberFilesVO memberFilesVO;

   ...getter, setter

   }


@Entity
@Table(name = "memberFiles")
public class MemberFilesVO {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer num;
    private String fname;
    private String oname;

    @OneToOne
    @JoinColumn(name="id")
   //------------ name ------------
   // 부모 테이블을 참조하는 멤버변수명
   // 생략된 변수명(컬럼명과 일치)을 명시
   // 테이블의 컬럼으로 있지만 VO에 생략된 멤버 변수명을 적용.
   //------------------------------------------------------------------------------------------
   //------------ nullable ------------
   // false : Outer Join
   // true  : Inner Join


    private MemberVO memberVO;

...getter, setter
```





### C. Repository(DAO) 구성





```java
public interface MemberRepository extends JpaRepository<MemberVO, String>{ }

public interface MemberFilesRepository extends JpaRepository<MemberFilesVO, Integer>{ }
```





### D. Test

##### 1) Select Test

- 양방향 관계
- 부모를 Select하면 자식도 같이 Select한다.
- 자식을 Select해도 부모도 같이 Select한다.





```java
@SpringBootTest
class MemberServiceTest {

    @Autowired
    private MemberRepository memberRepository;
    @Autowired
    private MemberFilesRepository memberFilesRepository;


    @Test
    public void MemberSelectTest()throws Exception{

        MemberVO memberVO = memberRepository.findById("admin").get();
        System.out.println("MemberVO_ID : "+memberVO.getId());
        System.out.println("MemberVO_Email :" + memberVO.getEmail());
        System.out.println("Files_Num : "+ memberVO.getMemberFilesVO().getNum());
        System.out.println("Files_Fname : "+ memberVO.getMemberFilesVO().getFname());
    }
  ----------------------------- Result   ----------------------------- 
  MemberVO_ID : admin
  MemberVO_Email :admin@admin.com
  MemberFilesVO_Num : 1
  MemberFilesVO_Fname : adminfname.jpg

    @Test   
    public void MemberFileTest()throws Exception{
        MemberFilesVO memberFilesVO = memberFilesRepository.findById(1).get();
        System.out.println("memberFilesVO_Num : "+memberFilesVO.getNum());
        System.out.println("memberFilesVO_Fname :" + memberFilesVO.getFname());
        System.out.println("memberVO_Id : "+ memberFilesVO.getMemberVO().getId());
        System.out.println("memberVO_Email : "+ memberFilesVO.getMemberVO().getEmail());
    }
 ----------------------------- Result   ----------------------------- 
  MemberFilesVO_Num : 1
  MemberFilesVO_Fname :adminfname.jpg
  MemberVO_Id : admin
  MemberVO_Email : admin@admin.com
```





##### 2) Insert Test

- 양방향 관계
- 부모를 select 해서 해당 PK 값이 있으면 update 없으면 insert 실행
- 자식 insert
- 부모에 자식 데이터, 자식에 부모 데이터를 주입하고 insert
- 또는 각각 insert
- Test1 또는 Test4 insert success
- 부모의 cascade 속성에 따라 Test4 가 실패 할 수도 있다.





```java
@SpringBootTest
//@Transactional
class MemberServiceTest {

    @Autowired
    private MemberService memberService;
    @Autowired
    private MemberRepository memberRepository;
    @Autowired
    private MemberFilesRepository memberFilesRepository;

//  1. Members, MemberFiles 에 각각 Insert 
//  @Test 
    public void MemberInserTest1()throws Exception{
        MemberVO memberVO = new MemberVO();
        memberVO.setId("user01Id");
        memberVO.setPw("user01Pw");
        memberVO.setName("user01Name");
        memberVO.setEmail("user01Email");
        memberRepository.save(memberVO);

        MemberFilesVO memberFilesVO = new MemberFilesVO();
        memberFilesVO.setFname("user01Fname");
        memberFilesVO.setOname("user01Oname");
        memberFilesVO.setMemberVO(memberVO);
        memberFilesRepository.save(memberFilesVO);
        //Members Table Insert 
        //MemberFiles Table Insert
        //둘다 성공
    }

//  2. MemberFileVO에 MemberVO를 주입하고 members에 Insert     
//  @Test
    public void MemberInserTest2()throws Exception{
        MemberVO memberVO = new MemberVO();
        memberVO.setId("user01Id");
        memberVO.setPw("user01Pw");
        memberVO.setName("user01Name");
        memberVO.setEmail("user01Email");
        MemberFilesVO memberFilesVO = new MemberFilesVO();
        memberFilesVO.setFname("user01Fname");
        memberFilesVO.setOname("user01Oname");

        memberFilesVO.setMemberVO(memberVO);

        memberRepository.save(memberVO);
        //Members Table Insert 성공
        //MemberFiles Table Insert 안됌.
        //따로 MemberFiles를 Insert를 한번 더 해야 함
        //Error는 아님
    }

//  3. MeberVO에 MemberFilesVO  주입하고  members에 Insert        
//  @Test
    public void MemberInserTest3()throws Exception{
        MemberVO memberVO = new MemberVO();
        memberVO.setId("user01Id");
        memberVO.setPw("user01Pw");
        memberVO.setName("user01Name");
        memberVO.setEmail("user01Email");

        MemberFilesVO memberFilesVO = new MemberFilesVO();
        memberFilesVO.setFname("user01Fname");
        memberFilesVO.setOname("user01Oname");

        memberVO.setMemberFilesVO(memberFilesVO);

        memberRepository.save(memberVO);
        //MemberFiles의 모든 변수가  Null이라 Insert Error 발생
    }   


//  4. MemberFileVO에 MemberVO를 주입하고 , MeberVO에 MemberFilesVO 서로 주입하고  members에 Insert       
    @Test
    public void MemberInserTest4()throws Exception{
        MemberVO memberVO = new MemberVO();
        memberVO.setId("user01Id");
        memberVO.setPw("user01Pw");
        memberVO.setName("user01Name");
        memberVO.setEmail("user01Email");

        MemberFilesVO memberFilesVO = new MemberFilesVO();
        memberFilesVO.setFname("user01Fname");
        memberFilesVO.setOname("user01Oname");

        memberFilesVO.setMemberVO(memberVO);
        memberVO.setMemberFilesVO(memberFilesVO);

        memberRepository.save(memberVO);
        //cascade = CascadeType.ALL 이면 성공
        //cascade = CascadeType.MERGE 성공

        //cascade = CascadeType.DETACH error
        //cascade = CascadeType.PERSIST error
        //cascade = CascadeType.REFRESH error
        //cascade = CascadeType.REMOVE error
        //MemberFiles의 모든 변수가  Null이라 Insert Error 발생
    }
//  5. MemberFileVO에 MemberVO를 주입하고  memberFiles에 Insert        
//  @Test
    public void MemberInserTest5()throws Exception{
        MemberVO memberVO = new MemberVO();
        memberVO.setId("user01Id");
        memberVO.setPw("user01Pw");
        memberVO.setName("user01Name");
        memberVO.setEmail("user01Email");

        MemberFilesVO memberFilesVO = new MemberFilesVO();
        memberFilesVO.setFname("user01Fname");
        memberFilesVO.setOname("user01Oname");

        memberFilesVO.setMemberVO(memberVO);

        memberFilesRepository.save(memberFilesVO);
        //MemberFiles의 Id 가 Null이라 Insert Error 발생
    }

//  6.  MemberVO에 MemberFileVO를 주입하고  memberFiles에 Insert   
//  @Test
    public void MemberInserTest6()throws Exception{
        MemberVO memberVO = new MemberVO();
        memberVO.setId("user01Id");
        memberVO.setPw("user01Pw");
        memberVO.setName("user01Name");
        memberVO.setEmail("user01Email");


        MemberFilesVO memberFilesVO = new MemberFilesVO();
        memberFilesVO.setFname("user01Fname");
        memberFilesVO.setOname("user01Oname");

        memberVO.setMemberFilesVO(memberFilesVO);

        memberFilesRepository.save(memberFilesVO);
        //MemberFiles의 Id 가 Null이라 Insert Error 발생
    }   

//  7.  MemberFileVO에 MemberVO를 주입하고 , MeberVO에 MemberFilesVO 서로 주입하고  memberFiles에 Insert  
//  @Test
    public void MemberInserTest7()throws Exception{
        MemberVO memberVO = new MemberVO();
        memberVO.setId("user01Id");
        memberVO.setPw("user01Pw");
        memberVO.setName("user01Name");
        memberVO.setEmail("user01Email");


        MemberFilesVO memberFilesVO = new MemberFilesVO();
        memberFilesVO.setFname("user01Fname");
        memberFilesVO.setOname("user01Oname");

        memberFilesVO.setMemberVO(memberVO);
        memberVO.setMemberFilesVO(memberFilesVO);

        memberFilesRepository.save(memberFilesVO);
        //MemberFiles의 Id 가 Null이라 Insert Error 발생
    }   
```



##### 3) Update Test

- Update 메서드는 따로 없다.
- save 메서드가 insert와 update 동시 사용
- 부모든 자식이든 PK 값은 입력 해줘야 함
- PK를 제외한 모든 컬럼을 Update 하므로 값을 다 넣어 줘야 함.
- 사용법은 Insert와 동일



```java
@SpringBootTest
class MemberServiceTest {

    @Autowired
    private MemberRepository memberRepository;
    @Autowired
    private MemberFilesRepository memberFilesRepository;


    @Test 
    public void MemberUpdateTest1()throws Exception{
        MemberVO memberVO = new MemberVO();
        memberVO.setId("user01Id");
        memberVO.setPw("Updateuser01Pw");
        memberVO.setName("Updateuser01Name");
        memberVO.setEmail("Updateuser01Email");
        memberRepository.save(memberVO);

        MemberFilesVO memberFilesVO = new MemberFilesVO();
        memberFilesVO.setNum(18);
        memberFilesVO.setFname("Updateuser01Fname");
        memberFilesVO.setOname("Updateuser01Oname");
        memberFilesVO.setMemberVO(memberVO);
        memberFilesRepository.save(memberFilesVO);
        //Members Table Insert 
        //MemberFiles Table Insert
        //둘다 성공
    }

  @Test
    public void MemberUpdateTest4()throws Exception{
        MemberVO memberVO = new MemberVO();
        MemberFilesVO memberFilesVO = new MemberFilesVO();

        memberVO.setId("user01Id");
        memberVO.setPw("user01Pw");
        memberVO.setName("user01Name");
        memberVO.setEmail("user01Email");

        memberFilesVO.setFname("user01Fname");
        memberFilesVO.setOname("user01Oname");
        memberFilesVO.setNum(18);
        memberFilesVO.setMemberVO(memberVO);
        memberVO.setMemberFilesVO(memberFilesVO);

        memberRepository.save(memberVO);
        //cascade = CascadeType.ALL 이면 성공
        //cascade = CascadeType.MERGE 성공

        //cascade = CascadeType.DETACH error
        //cascade = CascadeType.PERSIST error
        //cascade = CascadeType.REFRESH error
        //cascade = CascadeType.REMOVE error
        //MemberFiles의 모든 변수가  Null이라 Insert Error 발생
    }
  }
```





##### 4) Delete Test

- 부모를 삭제 하면 자식도 같이 삭제

```java
@SpringBootTest
class MemberServiceTest {

    @Autowired
    private MemberRepository memberRepository;
    @Autowired
    private MemberFilesRepository memberFilesRepository;


//  1. Member Id로 Members Table에서 삭제
//  @Test
    public void MemberDeletTest1()throws Exception{
        MemberVO memberVO = new MemberVO();
        memberVO.setId("user03Id");
        memberRepository.deleteById(memberVO.getId());

        // 둘다 삭제 성공,
        // table에 cascade 조건을 주지 않았지만 VO에서 cascade 옵션이 진행
        //Members Select
        //MemberFiles Select
        //MemberFiles Delete
        //Members Delete
    }

  }
```