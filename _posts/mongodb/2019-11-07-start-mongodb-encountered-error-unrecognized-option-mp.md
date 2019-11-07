---
layout: post
title:  "Windows에서 MongoDB 설치 후 cfg 파일을 통한 실행 시 "Unrecognized option: mp" 오류 발생문제."
date:   2019-11-07 12:33:00 +0900
categories: MongoDB
comments: true
tags: [MongoDB Setup, Windows, MongoDB]
---

---

최근 몽고디비를 통한 테스트가 필요하여 Windows 10에 패키지를 통한 설치를 하였다.

> https://www.mongodb.com/download-center/community 의 msi 파일 설치. (version 4.2 msi 패키지 파일)

Win 10에 MongoDB 설치시 한번에 패키지 설치를 사용할 수 있다고 생각했는데 설치 후에 mongodb 서버 서비스가 활성화 되지 않는 문제가 발생했다.
몽고디비는 윈도우 패키지 설치 파일로 기본 드라이브가 아닌 다른 드라이브에 설치하였고 (F:\) 실행 시 아래와 같은 에러가 발생했다.

~~~
F:\mongodb\bin>mongod -f mongod.cfg
Unrecognized option: mp
try 'mongod --help' for more information
~~~
실행하기전 mongod.cfg 파일에서 bind ip 만 기존 127.0.0.1 허용에서 0.0.0.0 으로 외부망에서도 전부 접근 가능하도록 옵션을 변경하였었다.

<br/>
구글링 한 결과 문제는 mongod.cfg에 있음을 알았고 mongod.cfg를 연 후 파일의 마지막 줄에서 mp라는 단어가 있었다.
mp에 대한 관련 정보는 공식 구성 파일 옵션에서 찾을 수 없어서 mp를 제거하고 Mongodb를 다시 시작하니 성공적으로 서비스가 올라왔다.

### mongod.cfg 파일
~~~
# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: F:\mongodb\data
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path:  F:\mongodb\log\mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0


#processManagement:

#security:

#operationProfiling:

#replication:

#sharding:

## Enterprise-Only Options:

#auditLog:

#snmp:
mp <-- 해당 문자 제거.

~~~


출처
 - https://medium.com/@sleo1104/啟動-mongodb-遇到錯誤-unrecognized-option-mp-27d9561fb96f
 
[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
