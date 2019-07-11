---
layout: post
title:  "AWS EC2 Linux(Amazon Linux) 에 메이븐(maven) 설치하기"
date:   2019-07-11 18:50:00 +0900
categories: ETC
comments: true
tags: [Amazon Linux, EC2, maven, mvn, install]
---

---

## AWS EC2 Linux(Amazon Linux) 에 메이븐(maven) 설치하기

아주 간단한 방법으로 Maven을 설치 가능하다.

```
sudo wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo
sudo sed -i s/\$releasever/6/g /etc/yum.repos.d/epel-apache-maven.repo
sudo yum install -y apache-maven
mvn --version

```
출처: 
 - https://gist.github.com/sebsto/19b99f1fa1f32cae5d00

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---

