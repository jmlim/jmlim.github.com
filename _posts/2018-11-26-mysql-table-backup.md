---
layout: post
title:  "[MySQL] 특정 테이블만 백업하기"
date:   2018-11-26 15:09:00 +0900
categories: mysql
comments: true
tags: [Mysql, DB]
---

---
### [MySQL] 특정 테이블만 백업하기 

형식) 내계정~]$mysqldump -u [사용자명] -p [데이타베이스명] [백업받을 테이블명] > [백업받을파일명]
```html
jmlim-linux]$ mysqldump -u testuser -p testDB testTable > test.sql
Enter password:
```
위에서는 testDB에 있는 testTable만 test.sql이란 파일로 백업받고 있다. <br/>
Enter password: 란에 비밀번호를 입력하면 test.sql이란 파일에 testTable만 백업된다. <br/>

[MySQL 백업된 테이블 복구]

형식) 내계정~]$mysq -u [사용자명] -p [데이타베이스명] < [백업받은 파일명]
```html
jmlim-linux]$ mysql -u testuser -p testDB < test.sql
Enter password:
```


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
