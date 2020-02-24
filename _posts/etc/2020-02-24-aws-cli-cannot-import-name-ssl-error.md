---
layout: post
title:  "macOS 에서 aws cli 실행 시 ImportError: cannot import name ‘ssl’ from ‘urllib3.util.ssl_’ 발생 문제 해결."
date:   2020-02-24 13:10:00 +0900
categories: ETC
tags: [macOS, aws, awscli, pip]
---

---


## 현상
 - mac os awscli 실행을 위해 스크립트를 실행 하였는데 아래 에러가 발생하였다. 
~~~
adminui-iMac:.aws jmlim$ aws
Traceback (most recent call last):
  File "/Users/jmlim/Library/Python/3.7/bin/aws", line 19, in <module>
    import awscli.clidriver
  File "/Users/jmlim/Library/Python/3.7/lib/python/site-packages/awscli/clidriver.py", line 17, in <module>
    import botocore.session
  File "/Users/jmlim/Library/Python/3.7/lib/python/site-packages/botocore/session.py", line 30, in <module>
    import botocore.credentials
  File "/Users/jmlim/Library/Python/3.7/lib/python/site-packages/botocore/credentials.py", line 42, in <module>
    from botocore.utils import InstanceMetadataFetcher, parse_key_val_file
  File "/Users/jmlim/Library/Python/3.7/lib/python/site-packages/botocore/utils.py", line 31, in <module>
    import botocore.httpsession
  File "/Users/jmlim/Library/Python/3.7/lib/python/site-packages/botocore/httpsession.py", line 7, in <module>
    from urllib3.util.ssl_ import (
ImportError: cannot import name 'ssl' from 'urllib3.util.ssl_' (/Users/jmlim/Library/Python/3.7/lib/python/site-packages/urllib3/util/ssl_.py)
~~~

> urllib3.util.ssl_ 에러 이름을 호출할 수 없다는데..뭐지? 

아래 참조 링크를 건 블로그 확인 결과 pip 이 고장 났기 때문이라고 한다.<br>
기존 파이선 버전을 업그레이드 후에 pip를 최신버전으로 적용하면 된다고 하여 그대로 진행하였다. <br/>
실제 아래 명령어를 실행하여 업그레이드 및 설치 결과 이후 위 에러는 발생하지 않았다.

## 파이선 업그레이드 
~~~
$ brew install python3
~~~

## pip 설치 
 - 내 경우엔 pip 명령어 실행 시 실행명령어를 찾지 못했었다.
~~~
sudo easy_install pip
~~~

## aws 명령어 실행 결과 정상작동 확인.
~~~
adminui-iMac:.aws jmlim$ aws
usage: aws [options] <command> <subcommand> [<subcommand> ...] [parameters]
To see help text, you can run:

  aws help
  aws <command> help
  aws <command> <subcommand> help
aws: error: the following arguments are required: command
adminui-iMac:.aws jmlim$
~~~

참고: 
 - https://www.nickkou.me/2019/12/importerror-cannot-import-name-ssl-from-urllib3-util-ssl_/
 - https://howchoo.com/g/mze4ntbknjk/install-pip-python


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
