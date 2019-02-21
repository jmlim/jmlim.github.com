---
layout: post
title:  "Jenkins pipeline을 통한 AWS Elastic Beanstalk 에 배포하기 샘플"
date:   2019-02-21 18:30:00 +0900
categories: Jenkins
comments: true
tags: [Jenkins, Git, Maven, AWS, Elastic Beanstalk]
---

---
> 필자 본인이 기억하기 위한 용도로 짧게 작성하였다. 

> pipeline 에 대한 대략적인 내용 참고 : https://www.youtube.com/watch?v=56jtwSrNvrs

### 사전 준비되어 있는 부분
- Github 사용.
- jenkins 2.x 이상이 필요하며 파이프라인 플러그인 설치가 필요하다.
- Elastic Beanstalk로 환경구성이 이미 되어 있다고 가정한다. 


## 1. Jenkins 새로운 아이템 선택.
<img src="{{ site.baseurl }}/public/post/jenkins-pipeline-beanstalk/jenkins-new-item.png"/> 

## 2. Jenkins 에서 아이템 네임 입력 후 Pipeline 선택.
<img src="{{ site.baseurl }}/public/post/jenkins-pipeline-beanstalk/jenkins-pipeline-project-create.png"/> 

## 3. General 탭에서 깃헙 프로젝트 선택 후 프로젝트 url 입력.
<img src="{{ site.baseurl }}/public/post/jenkins-pipeline-beanstalk/jenkins-pipeline-general.png"/> 

## 4. Build Trigger 탭에서 언제 빌드가 되게 할 것인지 선택 
 > Github hook trigger GITSem polling을 선택함.

<img src="{{ site.baseurl }}/public/post/jenkins-pipeline-beanstalk/jenkins-pipeline-build-triggers.png"/> 

## 5. Pipeline 설정.

### 5.1. Repositories 에 URL 및 계정 추가.

### 5.2. 레포지토리의 branch 선택 
   > ex) */master

### 5.3. 실행될 파이프라인 스크립트 경로 설정. 
   > 이 프로젝트에선 프로젝트 root 경로에 Jenkinsfile 이름의 파일에 스크립트를 지정하였다.
 
<img src="{{ site.baseurl }}/public/post/jenkins-pipeline-beanstalk/jenkins-pipeline-pipeline.png"/> 

## 6. Jenkinsfile 스크립트 작성 후 git commit 후 push

#### 6.1. 환경관련 이미지 참고용
 -  s3 버킷명 (여기선 S3버킷이름으로 표시)
 <img src="{{ site.baseurl }}/public/post/jenkins-pipeline-beanstalk/jenkins-uploads3-bucket.png"/> 

 - Beanstalk 어플리케이션이름 및 환경이름 참고 (해당 경로에 있는 이름대로 아래 스크립트에 포함시키면 된다.)
 <img src="{{ site.baseurl }}/public/post/jenkins-pipeline-beanstalk/jenkins-deploy-beanstalk-name.png"/> 

#### 6.2. 작성한 Jenkinsfile 스크립트 파일

```groovy
pipeline {
    agent any
    environment {
        // Slack configuration
        SLACK_COLOR_DANGER  = '#E01563'
        SLACK_COLOR_INFO    = '#6ECADC'
        SLACK_COLOR_WARNING = '#FFC300'
        SLACK_COLOR_GOOD    = '#3EB991'
    } // environment
    stages {
        // WAR 파일로 빌드 (테스트 부분 스킵..)
        stage('Build War') {
            steps {
                sh 'pwd'
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing'
            }
        }
        // S3에 먼저 업로드 후 Deploy 진행
        stage('Upload S3') {
            steps {
                echo 'Uploading'
                sh 'aws s3 cp /var/lib/jenkins/workspace/프로젝트명/target/프로젝트.war s3://S3버킷이름/${JOB_NAME}-${GIT_BRANCH}-${BUILD_NUMBER}.war \
                    --acl public-read-write \
                    --region ap-northeast-2' //서울리전
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying'
                sh 'aws elasticbeanstalk create-application-version \
                    --region ap-northeast-2 \ 
                    --application-name 어플리케이션이름 \
                    --version-label ${JOB_NAME}-${BUILD_NUMBER} \
                    --description ${BUILD_TAG} \
                    --source-bundle S3Bucket="S3버킷이름",S3Key="${JOB_NAME}-${GIT_BRANCH}-${BUILD_NUMBER}.war"'
                sh 'aws elasticbeanstalk update-environment \
                    --region ap-northeast-2 \
                    --environment-name 환경이름 \
                    --version-label ${JOB_NAME}-${BUILD_NUMBER}'
            }
        }
    }
    post {
        always {
            echo 'This will always run'
        }
        // 성공 시 슬랙 #tickets 채널에 성공 메세지 보내기
        success {
            echo 'This will run only if successful'
            echo 'Pushing EBS'
            slackSend (color: "${env.SLACK_COLOR_GOOD}",
                channel: "#tickets",
                message: "*SUCCESS:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} by ${env.USER_ID}\n More info at: ${env.BUILD_URL}")

        }
        // 실패 시 슬랙 #tickets 채널에 성공 메세지 보내기
        failure {
            echo 'This will run only if failed'
            //슬랙 #tickets 채널에 실패 메세지 보내기
            slackSend (color: "${env.SLACK_COLOR_DANGER}",
                channel: "#tickets",
                message: "*FAILED:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} by ${env.USER_ID}\n More info at: ${env.BUILD_URL}")
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, if the Pipeline was previously failing but is now successful'
        }
    }
}

```

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
