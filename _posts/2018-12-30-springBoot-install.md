---
title:  "SpringBoot 설치와 환경설정"
excerpt: "SpringBoot 설치와 환경설정"

categories:
  - springBoot
tags:
  - spring
  - spring boot
  - spring boot 설치
classes: wide 
---

### Spring Boot 설치

##### 1. STS DOWN

[Spring Boot Down](https://spring.io/tools): 

##### 2. 압축 풀기

다운 받은 파일은 jar로 압축 되어 있다. cmd 를 이용해 압축 해제

```bash
# ex) C:\springBoot 폴더에 sts 다운 

# cd 명령어로 sts가 있는 폴더로 이동
C:\> cd springBoot

# Java 명령어 실행
C:\springBoot> java -jar  spring-tool-suite-4-4.6.1.....

# 결과 확인
C:\springBoot> dir
2018-12-30  오전 09:23    <DIR>          sts-4.6.1.RELEASE
```



##### 3. 환경설정

```bash

#### Web Browser 설정 ###
# window > references > general > web browser
# Use External Web Brower 선택
# Chrome 선택 
# Apply


#### Encoding 설정 ####
# window > references > general > worksapce > text file encoding > UTF-8 변경
# window > references > Web > CSS > encoding > UTF-8 변경
# window > references > Web > HTML > encoding > UTF-8 변경
# window > references > Web > JSP > encoding > UTF-8 변경
# Apply

#### Properties 파일 한글 깨짐 해결 ####
# window > references > general > Content type > Text > Java Properties File 선택
# 하단에 Default encoding 입력창에 UTF-8로 수정
# Update 버튼 클릭 ( Update 하지 않으면 취소)
# Apply 

```







