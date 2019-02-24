---
layout: post
title:  "Docker 기본 명령어 정리."
date:   2019-02-25 06:10:00 +0900
categories: Docker
comments: true
tags: [Docker]
---

---

자주 사용하거나 자주 사용하게 될 명령어들을 다른 블로그를 참고하여 작성하였다.

## 버전 확인
```
docker -v
```
## 이미지 내려 받기
```
docker pull [이미지 이름]
```
> ex) docker pull centos:7

## 컨테이너 실행
```
docker start mycentos
``` 

## 컨테이너 내부 접근
```
docker attach mycentos
```
## 컨테이너 생성 및 내부접근
```
docker run -i -t --name centos7 centos:7
```

## 컨테이너 정지
```
docker stop centos7
```

## 컨테이너 정지 후 빠져나오기
```
exit
``` 
> or
Ctrl + D

## 컨테이너 정지하지 않고 빠져나오기
> Ctrl + P, Q

## 컨테이너 실행 중인 목록 확인
```
docker ps
```

## 컨테이너 모든 목록 확인
```
docker ps -a
```

## 컨테이너 이름 변경
```
docker rename [변경 전 컨테이너 이름] [변경 후 컨테이너 이름]
``` 

## exec 명령어사용을 통한 백그라운드 실행 컨테이너 내부 접속
```
docker exec -i -t wordpressdb /bin/bash
```

# Docker container & image 삭제관련 명령어

## 컨테이너 삭제
```
docker rm [컨테이너id]
```

## 컨테이너 모두 삭제 
```
docker rm `docker ps -a -q`
```

## 이미지 삭제
### 현재 이미지 확인
```
docker images
```
### 이미지 삭제
```
docker rmi [이미지id]
```
### 컨테이너를 삭제하기 전에 이미지를 삭제
```
docker rmi -f [이미지id]
```

### 모든 도커 컨테이너 삭제(remove all docker containers) 
- 구동중인 모든 도커 컨테이너들을 중지시키고, 삭제한다.
```
 docker stop `docker ps -a -q`
 docker rm `docker ps -a -q`
```

### 모든 도커 이미지 삭제(remove all docker images)
```
- docker rmi `docker images -q`
```

## docker image 실행 및 옵션설명 (ex : jenkins image 실행)
> $ docker run -d -p 8080:8080 -v /jenkins:/var/jenkins_home --name jenkins -u root jenkins

### 위 실행 커멘드에 대한 옵션 간단 설명
```
-d	detached mode 흔히 말하는 백그라운드 모드
-p	호스트와 컨테이너의 포트를 연결 (포워딩)
-v	호스트와 컨테이너의 디렉토리를 연결 (마운트)
–name	컨테이너 이름 설정
-u 실행할 사용자 지정
```

### 옵션에 대한 자세한 부분은 아래 참고
 - http://pyrasis.com/book/DockerForTheReallyImpatient/Chapter20/28

참고자료
 - https://joonyon.tistory.com/38
 - https://brunch.co.kr/@hopeless/10

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
