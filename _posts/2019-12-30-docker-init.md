---
title:  "Docker 설치 와 Mysql"
excerpt: "Docker 설치 및 활용"

categories:
  - docker
tags:
  - docker
  - spring boot
  - mysql
classes: wide 
---



#####  1. 설치

windows에 docker 설치

[Docker](https://www.docker.com/) 에서 자신의 OS에 맞는 설치 프로그램 다운

Windows 10 Pro 미만은 다른 프로그램을 설치 해야 함



##### 2. mysql Image 다운

cmd 에서 다음 명령어로 다운

```bash
# Mysql Image 다운
# docker pull [다운받을 이미지명]
c\:> docker pull mysql

# 자신의 PC에 다운 받은 이미지 목록들 확인
c\:> docker images
REPOSITORY  TAG     IMAGE ID        CREATED       SIZE
mysql       latest  30f937e841c8    12 days ago   541MB

# 필요시 다운 받은 이미지 삭제
# docker rmi [삭제할 이미지명]
c\:> docker rmi mysql
```



##### 3. Image를 기반으로 Container 생성 및 실행

```bash
# docker run [option 들] [이미지명]
c\:> docker run -d --name db -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql

# -d : 백그라운드로 실행
# --name [container 이름] : 다른 컨테이너와 구분하는 이름(직접 작성)
# -p [Host port]:[Container port] : port Foward
# -e MYSQL_ROOT_PASSWORD=[비밀번호] : root 유저의 비밀번호 지정(직접 작성)
```



##### 4. Conatiner 실행 명령어들

```bash
# 현재 실행 중인 Container 들의 목록
c\:> docker ps

# 현재 실행 중인 것과 종료된  Container 들의 목록
c\:> docker ps -a

# Container 종료
c\:> docker stop [Container 이름 or Container ID]

# Container 시작
c\:> docker start [Container 이름 or Container ID]

# Container 재실행
c\:> docker restart [Container 이름 or Container ID]

# Container 삭제
c\:> docker rm [Container 이름 or Container ID]

```

##### 5. 실행 중인 Container 접속

```bash
# docker exec -it [Container 이름 or Container ID] [실행할 shell]
c\:> docker exec -it db bin/bash
root@33c5a1309b7d:/#

# mysql 접속
root@33c5a1309b7d:/# mysql -u -root -p
Enter password:#Container 생성시 만든 password 입력 
mysql>
```





