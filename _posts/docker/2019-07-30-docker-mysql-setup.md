---
layout: post
title:  "Docker를 통한 MySQL 설치하기. (macOS)"
date:   2019-07-30 11:00:00 +0900
categories: Docker
comments: true
tags: [Docker, docker, MySQL, 도커, Mysql Docker 한글문제]
---

---
 > 2020-02-26 에 한글깨짐 관련 내용 추가 함.

이 글에선 Mac 용 docker에서 MySQL을 설정하는 방법과 MAC OS에서 MySQL에 접근 방법에 대해 알아본다.

Docker가 무엇인지 알고, MySQL을 사용하는 방법을 이해하고 SQL 명령을 사용하여 사용자를 생성하고, 데이터베이스를 만들고, 권한을 부여하는 방법을 이해하고 있다고 가정한다.
> docker가 설치되어 있다고 가정한다.

## 1. 도커 버전 확인

~~~
docker --version
~~~

## 2. MySQL Docker 이미지 다운로드

아래 명령어를 통해 MySQL 8.0.17 태그 이미지를 다운로드한다. <br/>
태그에는 MySQL 버전을 명시하며. 만약 태그에 버전을 명시하지 않으면, 최신 버전인 latest를 가져온다.

~~~
docker pull mysql:8.0.17
~~~

> docker hub에서 내려받을 수 있는 mysql 버전 참고 : https://hub.docker.com/_/mysql/

Docker 이미지 확인

~~~
docker images
~~~

## 3. Docker MySQL 컨테이너 생성 및 실행

 - 호스트의 /Users/{내계정}/datadir 디렉토리를 컨테이너의 /var/lib/mysql 디렉토리로 마운트
 - docker에 mysql과 같은 DB를 설치하는 경우 컨테이너 삭제와 함께 데이터도 날라가므로, 저장소는 반드시 외부 저장소를 사용한다.

 > ex)  -v /Users/jmlim/datadir:/var/lib/mysql

~~~
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password --name jmlim-mysql -v /Users/jmlim/datadir:/var/lib/mysql mysql:8.0.17
~~~

 - 하지만 위 명령어로 실행하여 mysql db를 생성하여 개발 시 한글문제가 살생할 것이다.
 - 한글이 깨지지 않도록 설정하려면 아래 인자값을 넣어주어야 한다. 
 
 ~~~
 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
 ~~~
###  ex ) 
~~~
 docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password --name jmlim-mysql -v /Users/jmlim/datadir:/var/lib/mysql mysql:8.0.17 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
~~~

위 명령어를 실행하여 컨테이너를 올려도 되지만 아래와 같이 docker-compose.yml 파일로 만들어서 실행할 수 있다.
 - 장황환 도커 옵션을 한눈에 볼 수 있어 편하다.

### docker-compose.yml 파일 생성.
~~~
version: "3" # 파일 규격 버전
services: # 이 항목 밑에 실행하려는 컨테이너 들을 정의
  db: # 서비스 명
    image: mysql:8.0.17 # 사용할 이미지
    container_name: jmlim-mysql # 컨테이너 이름 설정
    ports:
      - "3306:3306" # 접근 포트 설정 (컨테이너 외부:컨테이너 내부)
    environment: # -e 옵션
      MYSQL_ROOT_PASSWORD: "password"  # MYSQL 패스워드 설정 옵션
    command: # 명령어 실행
      - --character-set-server=utf8mb4 
      - --collation-server=utf8mb4_unicode_ci
    volumes:
      - /Users/jmlim/datadir:/var/lib/mysql # -v 옵션 (다렉토리 마운트 설정)
~~~

### 실행 docker-compose 파일 실행
 - docker-compose.yml 작성한 위치에서 실행
 - 백그라운드로 실행 시 옵션 -d 붙이면 됨.
   - 자세한건 옵션 참고
~~~
docker-compose up -d
~~~

## 4. Docker 컨테이너 목록 출력
~~~
docker ps -a
~~~

## 5. MySQL 컨테이너 bash 쉘 접속
> docker exec 명령을 사용하여 docker 컨테이너에 접근한 다음 MySQL에 로그인한다.

~~~
docker exec -it jmlim-mysql bash
~~~

## 6. MySQL 서버 접속
로컬에서 mysql를 설치하고 접속하는 방법과 동일. 
패스워드는 MySQL 컨테이너를 실행할 때 지정한 정보를 입력한다.
 - MYSQL_ROOT_PASSWORD = password

~~~
root@f3af78fa6428:/#mysql -u root -p

mysql>
~~~

## 7. 데이터베이스와 사용자를 생성하고 (컨테이너 내에서) MySQL에서 권한을 부여한다.

- jmlim이라는 사용자를 생성하고, 모든 권한을 부여한다.
- 변경된 권한 적용

> 중요 : 컨테이너 외부에서 MySQL에 로그인도 가능해야 하므로 jmlim@localhost에서 localhost 대신 %를 사용한다.

~~~
mysql> CREATE USER 'jmlim'@'%' IDENTIFIED BY 'password';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON *.* TO 'jmlim'@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> quit
~~~

## 8. 연결 테스트를 해본다.
<img src="{{ site.baseurl }}/public/post/docker-mysql-setup/docker-mysql-connect-test.png"/> 

> 외부에 있는 윈도우 피시의 HeidiSQL 툴로 접근 테스트 결과 잘 된다.


참고자료: 
 - https://jayden-lee.github.io/post/docker/mysql-install
 - https://dzone.com/articles/docker-for-mac-mysql-setup
 - https://hub.docker.com/_/mysql
 - https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
