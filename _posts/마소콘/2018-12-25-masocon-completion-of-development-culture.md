---
layout: post
title:  "[마소콘 2018] 기술부채의 늪 탈출기 - 개발문화의 완성 강연 내용 후기."
date:   2018-12-25 09:40:00 +0900
categories: 마소콘
comments: true
tags: [MASOCON, 마소콘, 기술부채, 데브옵스, 개발문화]
---

---
이번 후기는 좀 늦었지만.. 일단 정리한 내용이기에 글로 적어본다.
다음 강연은 럭스로보의 데브옵스 방법에 대해 소개하는 강연이었다.

강연내용 정리
--

현재 강연자분은 럭스로보에서 데브옵스 역할을 하고 있음.
#### 럭스로보는? 
 - 교육용 소프트웨어 납품. 아이들이 코딩을 쉽게 배울 수 있도록 도와주는 회사.

### 데브옵스란 무엇인가?
 - 개발, QA, 운영 의 교집합.

### 처음 회사에 들어왔을 때 문제점.
  - 레거시 코드
  - 이슈관리
  - 코드리뷰 X

하나하나 해결해나가기 시작.
 
## 1. 지라(Jira), 컨플루언스(confluence) 도입..

  - 칸반(Kanban) 방법론을 택함.. 
   - 매주 컨퍼런스 콜
    - 이슈를 누구한테 할당을 할 것인지.
   - 데일리 스탠드업 미팅
    - 의사소통 : 슬랙.
    - 매일아침 할일을 공유.
      - 어제한일.
      - 오늘할일.
      - 불편한점.
   - 버전기반으로 스켸줄링.
   - VOC 가 있을 때 Jira에 작성.

## 2. 깃랩(gitlab) 사용.
 - 깃 플로우 (자세한 내용은 링크 참조 : http://woowabros.github.io/experience/2017/10/30/baemin-mobile-git-branch-strategy.html)
    - hotfiex 브랜치 : 버그픽스. 버그 단위로 여러개가 따임.
    - feature 브랜치 : 버그 제외 모든 이슈. 기능단위로 여러개가 따임.
    - release 브랜치 : 릴리즈 할 기능이 모아져있는 브랜치.
    - master 브랜치 : 운영 브랜치.
    - develop 브랜치 : 개발 브랜치.

  - master, develop는 gitlab은 MR을 거치지 않으면(코드리뷰를 거치지 않으면) 들어갈 수 없음. (github의 PR)
  - 번개표시를 달면 코드리뷰 받은대로 처리했다라는 답변과 같음. (LGTM emoji)
    - 어떻게 보면 하나의 재미 요소.

  - 코드리뷰어는 자질구레 한 것들에 집중하는걸 피해야함.
     - 코드리뷰는 코드를 전체적으로 보고 더 큰 체계에 들어맞는지 보는 것.
     - 사소한 코드에 초점을 맞춰서는 안됨.

## 3. Jenkins 
  - 빌드 자동화.
    - Jenkinsfile : groovy 언어로 작성. (스크립트 대로 빌드 되고 배포됨.)
  - 젠킨스 쓰는 이유 : 자동배포.
    - 푸시 이벤트
    - 머지 리퀘스트
    - 코멘트
  - 젠킨스가 처음에 다 확인을 하고 머지까지 해주도록 함. 파이프라인이 깨지면 아예 코드리뷰로 넘어가지 않음.
  - 빌드가 되면 알림이 감. (테스터, 담당자.)
  - 클라우드 프론트에서 캐시 무효화 까지 함.
  - 젠킨스가 각 단계별로 파악을 해줌. (어디서 에러가 났는지 등등.)

## 관리자의 역할은??
 - 개발자가 코드만 편하게 작성할 수 있도록 돕는 것.


## 느낀점 
 - 이전 devops의 강의가 협업의 중요성을 강조했다면 이번 강의는 구체적인 방법에 대한 말씀을 해 주셨다.<br> 강연자분께서 콘퍼런스 마지막에 들어오셔서 지루할 수 도 있는 시간이었는데 설명을 잘 해주셔서 지루하지 않았다.
 스타트업이나 작은 규모의 회사에서 처음 프로세스를 잡을 때 위 내용 그대로 진행해도 좋을것 같다라는 생각이 들었다.

### 관련자료
 - https://www.imaso.co.kr/archives/4481
 - http://it.chosun.com/site/data/html_dir/2018/12/15/2018121500979.html

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
