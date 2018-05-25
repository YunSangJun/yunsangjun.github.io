---
layout: post
title:  "Helm chart repository 호스팅"
author: 윤상준
date: 2018-05-26 02:44:00 +0900
categories: helm
---

Helm chart를 개발한 후 외부에서 helm cli를 통해 다운로드 및 설치 하려면 Helm chart repository를 만들어야 합니다.

kubernetes helm [가이드](https://github.com/kubernetes/helm/blob/master/docs/chart_repository.md)에서 지원하는 방식은 3가지 입니다.

- Github Pages
- Object Storage(GCS, AWS S3)
- Ordinary Web Servers

## Github Pages

1. Github에 Repository 생성 및 docs 폴더 생성

    [샘플 코드](https://github.com/YunSangJun/sj-charts)

    ```
    Chart Repository
        | ㅡ docs
    ```

2. Github Page 설정

    `Repository -> Settings -> Github Pages` 메뉴 클릭

    Source를 `/docs` 로 지정. 지정 후 아래와 같은 site 주소 확인

    https://yunsangjun.github.io/sj-charts

3. Helm Package 생성

    ```
    # helm create mychart
    # helm package mychart
    # mv mychart-0.1.0.tgz docs
    # helm repo index docs --url https://yunsangjun.github.io/sj-charts
    # git add --all
    # git commit -m 'init'
    # git push origin master
    ```

4. Helm repository에 추가

    ```
    # helm repo add mychart https://yunsangjun.github.io/sj-charts
    ```
