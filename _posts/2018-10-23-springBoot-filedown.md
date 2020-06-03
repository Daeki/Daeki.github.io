---
title:  "SpringBoot FileDown"
excerpt: "SpringBoot FileDown"

categories:
  - springBoot
tags:
  - spring
  - spring boot
  - jsp
  - FileDown
classes: wide 
---


```bash
# JSP VIew 를 연결하는 InternalViewResolver 대신 FileDown 전용 Customview를 생성해야 함
# Controller에서 return한 ModelAndView에서 View이름을 InternalViewResolver로 보낸다.
# FileDown이면 InternalViewResolver로 보내지 않고 CustomView로 먼저 보내야 함.
# 객체의 이름을 VIew로 하는 BeanNameViewResolver를 작성하는데 AbstractView를 상속받는 클래스 작성 해야 함
# Legacy에서는 BeanNameViewResolver 를 XML에 작성하여 Order 적용(order를 0으로 적용 - 숫자가 낮을수록 우선)
# Boot에서는 별도의 BeanNameViewResolver 없이 자동 적용
```



##### 1. CustomView

```java
// Controller에서 view의 이름을 fileDown으로 보내면
// Bean의 이름이 fileDown 인것을 찾아 실행
// 디렉터리 사용은
// ResourceLoader
// ClassPathResource
// ServletContext
// 개인 취향에 맞게 사용 (사용법은 FileUpload와 동일)


// 이 클래스의 Bean의 아름은 fileDown으로 생성 됨
@Component("fileDown")
public class FileDown extends AbstractView {

    @Autowired
    private ResourceLoader resourceLoader;

    @Override
    protected void renderMergedOutputModel(Map<String, Object> model, HttpServletRequest request,
            HttpServletResponse response) throws Exception {

   		//1. 파일이  저장된  경로 설정
		String path = "classpath:/static/";
		path = path + (String)model.get("path");
		
        //2. 저장된 경로와 저장된 파일명 합치기
         BoardFileVO boardFileVO =(BoardFileVO)model.get("fileVO");
		path = path+boardFileVO.getFileName();
		
        //3. 위의 정보를 담는 파일 객체 생성
		File file = resourceLoader.getResource(path).getFile();
		
		//한글 처리
		response.setCharacterEncoding("UTF-8");
		
		//총 파일의 크기
		response.setContentLengthLong(file.length());
		
		//다운로드시 파일 이름을 인코딩 처리
		String fileName = URLEncoder.encode(boardFileVO.getOriName(), "UTF-8");
		
		//header 설정
		response.setHeader("Content-Disposition", "attachment;filename=\""+fileName+"\"" );
		response.setHeader("Content-Transfer-Encoding", "binary");
		
		//HDD에서 파일을 읽고
		FileInputStream fi = new FileInputStream(file);
		//Client로 전송 준비
		OutputStream os = response.getOutputStream();
		
		//전송
		FileCopyUtils.copy(fi, os);
		
		os.close();
		fi.close();
    }
}   
```



##### 3. Controller

```java
	@GetMapping("fileDown")
	public ModelAndView fileDown(NoticeFileVO noticeFileVO)throws Exception{
		
		ModelAndView mv = new ModelAndView();
		noticeFileVO = noticeService.fileDown(noticeFileVO);
        
        // FileDown 클래스 내에서 사용하는 Model의 키값과 동일한 이름 
		mv.addObject("fileVO", noticeFileVO);
        // FileDown 클래스 내에서 사용하는 Model의 키값과 동일한 이름 
		mv.addObject("path", "upload/notice/");
		
       //FileDown 클래스의 Bean의 이름과 동일한 이름으로 View name을 설정 
        mv.setViewName("fileDown");
		return mv;
	}
```

