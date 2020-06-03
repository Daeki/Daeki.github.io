---
title:  "SpringBoot FileUpload"
excerpt: "SpringBoot FileUpload"

categories:
  - springBoot
tags:
  - spring
  - spring boot
  - jsp
  - FileUpload
classes: wide 
---

##### 1. JSP(HTML) 의 form 태그 속성 변경

```html
# method 속성의 값은 post
# enctype 속성의 값은 multipart/form-data

<form action="url주소" method="post" enctype="multipart/form-data">
    <input type="file" name="files">
    ...
</form>
```



##### 2. application.properties

```properties
# Spring Legacy에서 ***-context.xml에 작성했던 설정을
# Spring Boot에서 application.properties에 설정
# 프로젝트 생성시 API는 기본 내장

############################################# fileupload
## multpart 사용 여부
spring.servlet.multipart.enabled=true
## 파일당 최대 파일 크기 설정
spring.servlet.multipart.max-file-size=10MB
## 총 파일 크기 설청
spring.servlet.multipart.max-request-size=100MB
## 파일을 저장할 파일 시스템의 경로
## 배포될 환경이 다를 가능성이 높으므로 쓰지 말자
#spring.servlet.multipart.location=c:/upload

############################################# custom filepath
## 파일을 저장할 폴더를 지정하고 Java에서 호출 해서 사용 할 예정
board.notice.filePath=upload/notice
```

##### 3. Controller

```java
// Controller에서 파라미터로 MultipartFile로 mapping
// 같은 파라미터 이름으로  Filedl 여러개 라면 배열로 선언
// 변수명은 파라미터의 이름과 동일하게
@PostMapping("boardWrite")
public ModelAndView boardWrite(BoardVO boardVO, MultipartFile [] files)throws Exception{
        int result = noticeService.boardWrite(boardVO, files);
}
```



##### 4_1. Service

```java
@Service
public class NoticeService {

    @Autowired
    private NoticeMapper noticeMapper;

  //파일을 저장할 디렉토리 생성
    @Autowired
    private FilePathGenerator filePathGenerator; 

  //파일을 HDD에 저장할 클래스
    @Autowired
    private FileManager fileManager;

  //파일 저장 디렉토리명 application.properties에서 받아오기
    @Value("${board.notice.filePath}")
    private String filePath;

    public int boardWrite(BoardVO boardVO, MultipartFile [] files)throws Exception{
        
        //-------------------  File Save ------------------------------
	int result =noticeRepository.setInsert(boardVO);
     	File file = pathGenerator.getUseClassPathResource(filePath);
		
		for(MultipartFile multipartFile: files) {
			if(multipartFile.getSize()<=0) {
				continue;
			}
			String fileName= fileManager.saveFileCopy(multipartFile, file);
			NoticeFileVO vo = new NoticeFileVO();
			vo.setNum(boardVO.getNum());
			vo.setFileName(fileName);
			vo.setOriName(multipartFile.getOriginalFilename());
			
			result = noticeFileRepostitory.setInsert(vo);
			
			System.out.println(fileName);
		}
		
		return result; 
    }

}

```



##### 4_2. FilePathGenerator

```java
// 다음 세개의 메서드는 파일을 저장할 폴더의 실제 경로를 File객체에 담아서 리턴

@Component
public class FilePathGenerator {
    @Autowired
    private ResourceLoader resourceLoader;

    @Autowired
    private ServletContext servletContext;

    public File getUseResourceLoader(String filePath)throws Exception{

        //ResourceLoader
        //classes 경로를 받아오기 위해 사용
        //생성하려는 디렉토리가 없으면 에러를 발생
        //Default로 만들어진 static 경로를 이용해 File 객체를 생성하고
        //하위 디렉토리를 Child를 사용해 File 클래스의 메서드를 이용해 디렉토리를 생성

        String path="classpath:/static/";

        resourceLoader.getResource(path);

        File file = new File(resourceLoader.getResource(path).getFile(), filePath);

        if(!file.exists()) {
            file.mkdirs();
        }
        return file;
    }

    public File getUseClassPathResource(String filePath)throws Exception{

        //ClassPathResource
        //classes 경로를 받아오기 위해 사용
        //ResourceLoader와 같지만 
        //시작 경로에서 classpath를 제외 함 
        //이후 만들어지는 과정은 동일

        String path="static";

        ClassPathResource classPathResource = new ClassPathResource(path);

        File file = new File(classPathResource.getFile(), filePath);

        if(!file.exists()) {
            file.mkdirs();
        }
        return file;
    }

    public File getUseServletContext(String filePath)throws Exception{

        //ServletContext
        //File 클래스의 메서드를 이용해 디렉토리를 생성
        //다른점은 classpath가 아닌 wepapp 하위에 만들어딤
        //생성할 디렉로리명이 static이라면
        //default로 만들어진 static 디렉토리가 있고 wepapp하위에 하나가 만들어짐

        filePath = servletContext.getRealPath(filePath);
        File file = new File(filePath);
        if(!file.exists()) {
            file.mkdirs();
        }
        System.out.println(filePath);
        return file;
    }
  }
```

##### 4_3. FileManager

```java
// 다음 두 메서드 중 어떤 것을 사용하더라고 결과는 같음
// 파일을 HDD에 저장하고 저장 할 때의 파일이름을 리턴.

@Component
public class FileSaver {

    // FileCopyUtils 클래스의 정적 메서드 copy 사용
    public String saveFileCopy(MultipartFile file, File dest)throws Exception{

        String fileName=null;

        fileName = UUID.randomUUID().toString()+"_"+file.getOriginalFilename();

        dest = new File(dest, fileName);

        FileCopyUtils.copy(file.getBytes(), dest);

        return fileName;
    }

    // MultipartFile 클래스의 멤버메서드 transferTo 사용
    public String saveTransferTo(MultipartFile file, File dest)throws Exception{
        String fileName=null;


        fileName = UUID.randomUUID().toString()+"_"+file.getOriginalFilename();

        dest = new File(dest, fileName);

        file.transferTo(dest);

        return fileName;
    }

}
```

##### 5. Repository(DAO)

```java
//NoticeRepository
@Mapper
public interface NoticeRepository extends BoardRepository {
	public int setInsert(BoardVO boardVO) throws Exception;
}


//NoticeFileRepository
@Mapper
public interface NoticeFileRepostitory {
	public int setInsert(NoticeFileVO noticeFileVO)throws Exception;
}
```



##### 6. Mapper.xml

```xml
<!-- Notice Mapper -->
<mapper namespace="com.iu.s1.board.notice.NoticeRepository">
	<insert id="setInsert" parameterType="NoticeVO" useGeneratedKeys="true" keyProperty="num">
		insert into notice values(#{num}, #{title}, #{writer}, #{contents}, now(), 0)		
	</insert>
</mapper>   


<!--NoticeFile Mapper -->
<mapper namespace="com.iu.s1.board.notice.noticeFile.NoticeFileRepostitory">
	<insert id="setInsert" parameterType="NoticeFileVO">
		insert into noticeFile (num, fileName, oriName)
		values(#{num}, #{fileName}, #{oriName})
	</insert>
</mapper>  
```

