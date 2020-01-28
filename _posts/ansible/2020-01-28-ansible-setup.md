---
layout: post
title:  "Ansible(앤서블) 설치 및 서버 연결하기. (by Mac)"
date:   2020-01-28 18:30:00 +0900
categories: Ansible
comments: true
tags: [Ansible, 앤서블]
---

---

### Ansible pip3 로 설치
> pip3 를 통해 설치하였다.

~~~
adminui-iMac: jmlim$ pip3 install ansible
~~~

### Ansible 설정하기.
- /etc/ansible 폴더 추가 후 hosts 파일 생성  

~~~
adminui-iMac: jmlim$ cd /etc
adminui-iMac:etc jmlim$ mkdir ansible
adminui-iMac:etc jmlim$ cd ansible
adminui-iMac:ansible jmlim$ cd ansible
adminui-iMac:ansible jmlim$ sudo vim hosts
.....
~~~

- /etc/ansible/hosts 파일 수정

~~~
[jmlim]
10.30.175.66
~~~

### 접속 테스트
- Mac 에서 ansible 설치 및 /etc/ansible/hosts 에 타겟 서버 아이피까지 등록 후 ping 을 시도 하였는데 아래와 같이 에러가 발생하였다.

~~~
adminui-iMac:ansible jmlim$ ansible all -m ping -k
SSH password:
192.168.0.12 | FAILED! => {
    "msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"
}
~~~

위와 같이 IP는 설정해 주었지만 상대방 서버에서는 지금 ansible이 설치되어 있는 서버에서 접속한다고 그냥 받아주지 않는다고 한다. 
지금 서버에서 해당 서버에 연결할 수 있도록 하기 위해 ansible 서버에 sshpass 설치가 필요하다.

다음과 같이 설치가능하다.

### CentOS, Fedora 같은 Redhat 계열
~~~
yum install -y sshpass
~~~

### Ubuntu와 같은 debian 계열
~~~
apt-get install -y sshpass
~~~

### 맥에서는 다음과 같이 설치
- homebrew로 설치가능한데, 디폴트 저장소에는 존재하지 않아 다음의 명령으로 설치한다.

~~~
brew install https://raw.githubusercontent.com/kadwanev/bigboybrew/master/Library/Formula/sshpass.rb
... 설치설치...
~~~

### 재시도 결과 성공.
~~~
adminui-iMac:ansible jmlim$ ansible all -m ping -k --user 계정
SSH password: 패스워드
192.168.0.12 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
~~~
 
참고자료: 
 - https://blog.naver.com/PostView.nhn?blogId=theswice&logNo=221551213336
 - http://egloos.zum.com/mcchae/v/11315324

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
