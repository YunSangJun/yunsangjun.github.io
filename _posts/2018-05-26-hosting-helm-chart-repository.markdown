---
layout: post
title:  "Helm chart repository 호스팅"
author: sj
date: 2018-05-26
categories: helm
tags:
- helm
- chart
- repository
- kubernetes
- container
---

내가 만든 Helm chart를 다른 사용자에게 공유하기 위해서는 Helm repository가 필요합니다.

Official repository(https://github.com/helm/charts)에 Contribution하는 방법과 자체 repository를 사용하는 두 가지 방식 있습니다.

자체 repository는 크게 3가지 형태로 지원합니다.(웹 서버 기능 필요)

- Github Pages
- Object Storage(GCS, AWS S3, ICOS 등)
- Ordinary Web Servers(Nginx, Apache)

자세한 내용은 [The Chart Repository Guide](https://github.com/kubernetes/helm/blob/master/docs/chart_repository.md)를 참고하세요.

## Github Pages

Github Pages를 사용하는 방식은 쉽고 무료로 사용할 수 있습니다.
Github Pages는 웹 서버 기능을 지원합니다.

1. Github에 Repository 생성

    이 페이지에서는 `my-charts`라고 생성하겠습니다.

2. Github Page 설정

    `Repository -> Settings -> Github Pages > Source > master branch 선택 > Save 선택`

    설정 완료 후 나타나는 `https://example.github.io/my-charts/` 주소가 웹 서버 주소입니다.

3. git repository checkout 및 stable 디렉토리 생성

    ```
    $ git checkout  https://github.com/example/my-charts.git
    $ cd my-charts & mkdir stable
    ```

4. `demo` helm chart 생성 및 package

    ```
    $ helm create demo
    Creating demo

    # helm package demo
    Successfully packaged chart and saved it to: PATH/demo-0.1.0.tgz
    ```

5. helm 파일을 stable 디렉토리에 복사(앞에서 생성한 demo-0.1.0.tgz 사용)

    ```
    # mv demo-0.1.0.tgz stable
    ```

6. Index 파일 생성

    stable 디렉토리 하위에 index.yaml 파일이 생성됩니다.

    ```
    $ helm repo index [INDEX_FILE_PATH] --url [CHART_REPO_URL]
    $ helm repo index stable --url https://example.github.io/my-charts/stable
    ```

7. 변경사항 git에 commit

    ```
    $ git add --all & git commit -m 'init' & git push origin master
    ```

8. 웹 서버 주소에 접속해 index 파일 확인

    https://example.github.io/my-charts/stable/index.yaml

    ```
    apiVersion: v1
    entries:
      demo:
      - apiVersion: v1
        appVersion: "1.0"
        created: 2018-10-03T12:14:26.693334507+09:00
        description: A Helm chart for Kubernetes
        digest: c677efc778b455ad3b9d53c03d37cf0002d067ac80da35d0921672a77f3c9bc3
        name: demo
        urls:
        - https://example.github.io/my-charts/stable/demo-0.1.0.tgz
        version: 0.1.0
    generated: 2018-10-03T12:14:26.690034532+09:00
    ```

9. Index 파일의 helm chart 패키지 다운로드 확인

    https://example.github.io/my-charts/stable/demo-0.1.0.tgz


* [Sample Repository 참고](https://github.com/YunSangJun/my-charts)
