---
layout: post
title:  "Dockerfile을 사용한 tomcat에 War file 배포하기."
date:   2019-02-19 20:20:00 +0900
categories: Docker
comments: true
tags: [Docker, Dockerfile]
---

---

사내에서 사용하는 백오피스 시스템을 Dockerfile를 통해 배포할 일이 생겨 설정 후 간단하게 작성해본다.<br>
오늘 배포 관련해서 로컬에서만 처음 설정해 본 내용이라 학습하는대로 내용은 보충을 좀 해야겠다. 

[도커 명령어 참고 : http://jmlim.github.io/docker/2019/02/24/docker-command/]

## 1. 도커 설치 관련

```
$ sudo apt-get install docker.io  ( 작성자는 이 명령어만 사용하여 설치하였다. )
```
### 도커 설치 후 현재 접속중인 사용자에게 권한을 주어야 사용가능하다.
```
sudo usermod -aG docker $USER 또는 권한을 줄 User의 ID
```

## 2. 소스의 루트경로에 Dockerfile 작성

```
FROM tomcat:8.0.51-jre7-alpine  # docker 안의 tomcat8 셋팅, 사내 백오피스는 자바 7 사용.

ENV TZ=Asia/Seoul ## 타임존 서울로 설정 후 적용
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN rm -Rf /usr/local/tomcat/webapps/ROOT # 기존 도커안의 tomcat의 루트 경로 삭제 
COPY target/배포후 만들어지는 WAR 파일 이름.war /usr/local/tomcat/webapps/ROOT.war # 루트 경로에 복사
ENV JAVA_OPTS="-Dserver.type=dev" # 서버타입 설정, 프로퍼티 파일로 개발 운영 환경 구분..

```

## 3. 소스를 내려받은 후 소스의 루트 경로에서 docker build 하기.
```
$ git clone 소스경로
$ mvn clean package
$ docker build -t ticketmall_backoffice .
```

## 4. 이미지 생성 된것 확인
```
$ docker images

REPOSITORY              TAG                  IMAGE ID            CREATED             SIZE
ticketmall_backoffice   latest               51af9b31cb45        1 hours ago         301MB
tomcat                  8.0.51-jre7-alpine   2304e340e238        10 months ago       145MB
```

## 5. 위에서 만든 도커 컨테이너 실행
```
$ docker container run -p 80:8080 -it ticketmall_backoffice 
```
80 포트로 컨테이너에 접근할 시 8080으로 포워딩 해줌. 톰캣은 잘 실행된다. 

참고자료
 : https://www.youtube.com/watch?v=GdnZkiif64Q


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
