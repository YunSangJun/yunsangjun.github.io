---
layout: post
title:  "Jenkins CI를 활용한 지속적인 소스코드 통합 환경 구성하기"
author: sj
date: 2019-08-04
categories: cicd
tags:
- jenkinsci
- cicd
- devops
- kubernetes
---

## Overview

소스코드를 지속적으로 통합하기 위해서 TravisCI, CircleCI, Jenkins 등 다양한 툴을 활용할 수 있습니다.

이 문서에서는 Jenkins CI를 활용하여 소스코드를 지속적으로 통합하는 방법을 알아보겠습니다.

Jenkins CI와 아래 서비스를 통합하여 CI 환경을 구성해보겠습니다.

- 소스코드 저장소: Github
- 이미지 저장소: Docker Hub
- 통합(CI): Jenkins
- 애플리케이션 서버: Kubernetes

## 사전 준비

### 소스코드 다운로드 

이 문서에서 사용할 [jenkinsci-demo](https://github.com/YunSangJun/jenkinsci-demo.git) 프로젝트를 복사합니다.

```
git clone https://github.com/YunSangJun/jenkinsci-demo.git
cd jenkinsci-demo
```

### 소스코드 저장소 

Github에 sample-app 저장소를 생성합니다. 

### 이미지 저장소 준비

Docker Hub에 sample-app 저장소를 생성합니다.

### Kubernetes 클러스터 구성

애플리케이션 서버로 사용할 Kubernetes 클러스터를 준비합니다.

### Jenkins 설치

Jenkins를 설치할 namespace를 생성합니다.
```
kubectl create namespace jenkins
```

위에서 다운로드한 jenkinsci-demo 디렉토리로 이동합니다.
```
cd jenkinsci-demo
```

Jenkins Helm chart를 설치합니다.
```
helm install --name jenkins-release --namespace jenkins \
-f jenkins/values.yaml stable/jenkins
```

Maven Build Cache 용도의 pvc를 생성합니다.

```
kubectl create -f jenkins/maven-cache-pvc.yaml -n jenkins
```

Jenkins에 대한 자세한 내용은 [Jenkins 설치하기](/blog/cicd/2018/05/26/installing-jenkins.html) 문서를 참고하세요.

## Jenkins UI 접속

1. Jenkins UI에 접속하기 위해 포트포워딩을 설정합니다. 

    ```
    export POD_NAME=$(kubectl get pods --namespace jenkins -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=jenkins-release" -o jsonpath="{.items[0].metadata.name}")
    kubectl --namespace jenkins port-forward $POD_NAME 8080:8080
    ```

2. Jenkins admin 계정의 암호를 확인합니다.

    ```
    printf $(kubectl get secret --namespace jenkins jenkins-release -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
    xxxxx
    ```

3. 이제 웹 브라우저에서 localhost:8080 로 접속해서 admin 계정으로 로그인합니다.

4. Jenkins 관리 > 시스템 설정 > `# of executors`의 값을 10으로 변경하고 저장합니다.

## Credential 생성하기

DockerHub에 접속하기 위한 credential을 생성해보겠습니다.

1. Jenkins UI에서 왼쪽 메뉴의 Credentials을 선택합니다.

2. 목록에서 Jenkins를 선택합니다. 

3. Global credentials을 선택합니다.

4. 왼쪽 메뉴의 Add Credentials을 클릭합니다.

5. DockerHub의 Username, Password를 입력하고 ID에 docker_credential을 입력합니다.

6. OK 버튼을 선택해 저장합니다.

## Jenkins Pipeline 생성하기

소스코드를 빌드하고 DockerHub에 저장하는 Pipeline을 생성해보겠습니다.

1. 왼쪽 메뉴의 New Item을 선택합니다.

2. Item 이름을 sample-app으로 입력하고 Multibranch Pipeline을 선택합니다.
그리고 OK 버튼을 선택해 다음 화면으로 이동합니다.

3. 화면 상단의 General으로 이동합니다.
Display Name에 sample-app을 입력합니다.

4. Branch Sources로 이동합니다.
Add source > Git을 선택합니다. Project Repository에 앞에서 생성한 sample-app 저장소 주소를 입력합니다.

5. Scan Multibranch Pipeline Triggers로 이동합니다.
Periodically if not otherwise run를 체크하고 Interval을 1 minute로 선택합니다.

6. Save 버튼을 선택해 설정을 저장합니다. 

## sample-app 저장소에 소스코드 업로드

위에서 미리 생성한 sample-app 저장소에 소스코드를 업로드하겠습니다.

1. sample-app 디렉토리를 생성하고 jenkinsci-demo/sample-app의 소스코드를 복사합니다.

    ```
    mkdir sample-app
    cd sample-app
    cp -r ../jenkinsci-demo/sample-app/. ./
    ```

2. Jenkinsfile을 수정합니다.

    `DOCKER_REPOSITORY`를 위에서 생성한 DockerHub 저장소명으로 변경합니다.

    ```
    def dockerCredential = "docker_credential"
    def imageTag = "DOCKER_REPOSITORY:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
    ```

3. 소스코드를 저장소에 업로드합니다.

    ```
    git init
    git add --all
    git commit -m "first commit"
    git remote add origin https://github.com/REPOSITORY_NAME/sample-app.git
    git push -u origin master
    ```

4. Jenkins Pipeline 확인

    소스코드가 업로드되면 Jenkins에서 master branch의 변경사항을 감지하고 Pipeline에 정의된 build job을 수행합니다.

    Job이 성공적으로 수행되면 아래와 같은 화면을 확인할 수 있습니다.

    ![](/blog/assets/images/kubernetes/jenkins/jenkinsci-01.png)

5. Docker build 확인

    Jenkins Pipeline 수행이 완료되면 Docker 이미지 저장소에서 빌드된 이미지를 확인할 수 있습니다.

    이미지의 태그명은 `BRANCH_NAME.BUILD_NUMBER`과 같이 정의됩니다.

    ![](/blog/assets/images/kubernetes/jenkins/jenkinsci-02.png)

6. 애플리케이션 확인

    빌드한 Docker 이미지를 활용하여 애플리케이션을 실행해보겠습니다.

    ```
    docker run -p 8080:8080 DOCKER_REPOSITORY_NAME/sample-app:BRANCH_NAME.BUILD_NUMBER
    ```

    애플리케이션이 실행되면 버전을 확인해봅니다.

    ```
    $ curl localhost:8080/version
    0.1
    ```

## Branch 추가

이번에는 새로운 branch를 만들어 신규 기능을 추가해보겠습니다.

1. `new_feature` branch를 추가합니다.

    ```
    git checkout -b new_feature
    ```

2. Application version을 0.2로 변경합니다.

    ```
    $ vi src/main/resources/application.yaml
    application:
        version: 0.2
    ```

3. 변경사항을 Git 저장소에 반영합니다.

    ```
    git add --all
    git commit -m "Add new feature and Update version as 0.2"
    git push origin new_feature
    ```

4. Jenkins Pipeline 확인

    소스코드가 업로드되면 Jenkins에서 new_feature branch의 변경사항을 감지하고 Pipeline에 정의된 build job을 수행합니다.

    ![](/blog/assets/images/kubernetes/jenkins/jenkinsci-03.png)

    Job이 성공적으로 수행되면 아래와 같은 화면을 확인할 수 있습니다.

    ![](/blog/assets/images/kubernetes/jenkins/jenkinsci-04.png)

5. Docker build 확인

    Jenkins Pipeline 수행이 완료되면 Docker 이미지 저장소에서 빌드된 이미지를 확인할 수 있습니다.

    이미지의 태그명은 `BRANCH_NAME.BUILD_NUMBER`와 같이 정의됩니다.

    ![](/blog/assets/images/kubernetes/jenkins/jenkinsci-05.png)

6. 애플리케이션 확인

    빌드한 Docker 이미지를 활용하여 애플리케이션을 실행해보겠습니다.

    ```
    docker run -p 8080:8080 DOCKER_REPOSITORY_NAME/sample-app:BRANCH_NAME.BUILD_NUMBER
    ```

    애플리케이션이 실행되면 버전을 확인해봅니다.

    ```
    $ curl localhost:8080/version
    0.2
    ```

## Compare & Pull request

`new_feature` branch의 신규 기능을 Pull request 해보겠습니다.

1. 새로운 branch가 추가되면 Compare & Pull request 알림이 나타납니다.
`Compare & Pull request` 버튼을 선택합니다.

    ![](/blog/assets/images/kubernetes/jenkins/jenkinsci-06.png)

2. Merge하는 내용을 비교하고 이상이 없으면 `Create pull request` 버튼을 선택합니다.

    ![](/blog/assets/images/kubernetes/jenkins/jenkinsci-07.png)

## Merge pull request

1. Pull request 목록에 방금 요청한 항목이 나타납니다.

    ![](/blog/assets/images/kubernetes/jenkins/jenkinsci-08.png)

2. 요청 항목에 문제가 없으면 `Merge pull request` 버튼을 선택합니다.

    ![](/blog/assets/images/kubernetes/jenkins/jenkinsci-09.png)

3. master branch에 new_feature 코드가 병합됩니다.
그리고 Jenkins Pipeline이 master branch의 변경사항을 감지하고 애플리케이션을 다시 빌드합니다.


