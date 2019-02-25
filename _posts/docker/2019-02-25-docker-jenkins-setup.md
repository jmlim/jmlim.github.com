---
layout: post
title:  "Docker를 통한 젠킨스(Jenkins) 설치하기."
date:   2019-02-25 10:02:00 +0900
categories: Docker
comments: true
tags: [Docker, Jenkins]
---

---
도커가 설치되어 있다고 가정한다.

## 1. Jenkins 이미지 내려 받기

Docker Hub 에서 Jenkins 이미지를 내려받을 수 있다.

#### Docker Hub이란? 
 - 도커 이미지를 업로드해서 공유하는 저장소를 도커 레지스트리(Docker Registry)라고 한다. 
 대표적으로는 도커의 공식 레지스트리인 Docker Hub 가 있다. 
 도커 허브에서는 업체에서 제공하는 공식 이미지를 받을 수 있다.
 - Ubuntu 나 CentOS 같은 OS 이미지, MySQL, Redis, MongoDB, Nginx 와 같은 미들웨어, OpenJDK, Golang, NodeJS 와 같은 플랫폼 이미지도 제공한다.

<img src="{{ site.baseurl }}/public/post/docker-jenkins-setup/docker-jenkins-setup-1.png"/> 

내려받을 jenkins 이미지 파일과 pull 명령어를 확인할 수 있다.<br>
여기서는 lts 버전 파일을 내려받았다.

```
docker pull jenkins/jenkins:lts
```

## 2. Jenkins 이미지를 컨테이너로 실행하기

- 내려받은 이미지는 그냥 이미지일 뿐이고 실제 그것을 컨테이너에 올리는 작업까지 해야 의미가 있다. (그것은 마치 클래스를 인스턴스화 시키는것과 비슷하다.(비슷한가?))

docker images 로 내려받아진 이미지 repository 명 확인후에
<img src="{{ site.baseurl }}/public/post/docker-jenkins-setup/docker-jenkins-setup-images.png"/> 

아래 명령어를 실행한다.
```
$ docker run -d -p 8181:8080 -v /jenkins:/var/jenkins_home --name jm_jenkins -u root jenkins/jenkins:lts

// 위 명령어 옵션설명 
-d	detached mode 흔히 말하는 백그라운드 모드
-p	호스트와 컨테이너의 포트를 연결 (포워딩)
-v	호스트와 컨테이너의 디렉토리를 연결 (마운트)
–name	컨테이너 이름 설정
-u 실행할 사용자 지정

맨 마지막 jenkins/jenkins:lts 는 실행할 이미지의 레포지토리 이름이며 만약 이미지가 없을 경우 이미지를 docker hub 에서 땡겨오므로 주의한다.
```

## 3. Jenkins 브라우저로 접속하여 설치작업 계속하기.

docker ps 명령을 통해서 정상적으로 jenkins 컨테이너가 올라온 것을 확인했다면, 브라우저를 통해 해당 포트로 접속한다.
Jenkins 설치를 계속하기 위해선 admin password 를 입력해야 한다. 

원랜 jenkins 도커 이미지에 접근하여 initialAdminPassword 폴더에 접근 후 패스워드를 가져와야 하지만 
"docker logs jenkins" 명령어를 통해 굳이 폴더 접근하지 않아도 확인할 수 있다.

<img src="{{ site.baseurl }}/public/post/docker-jenkins-setup/docker-jenkins-setup-authkey.png"/> 

## Admin password  입력
<img src="{{ site.baseurl }}/public/post/docker-jenkins-setup/docker-jenkins-setup-authkey-browser.png"/> 

## 플러그인 설치
처음 설치할 플러그인들을 선택하는 화면이 나온다. 여기선 기본 권장 플러그인들을 선택하고 설치를 진행한다.
<img src="{{ site.baseurl }}/public/post/docker-jenkins-setup/docker-jenkins-plugin-setting.png"/> 

## 설치중 화면
<img src="{{ site.baseurl }}/public/post/docker-jenkins-setup/docker-jenkins-plugin-setting-2.png"/> 

## 플러그인 설치 후 초기 계정 생성.
<img src="{{ site.baseurl }}/public/post/docker-jenkins-setup/docker-jenkins-account-create.png"/> 

## 설치 완료 후 화면
<img src="{{ site.baseurl }}/public/post/docker-jenkins-setup/docker-jenkins-setup-complete.png"/> 


참고자료: 
 - https://futurecreator.github.io/2018/11/16/docker-container-basics/
 - https://www.leafcats.com/215

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
