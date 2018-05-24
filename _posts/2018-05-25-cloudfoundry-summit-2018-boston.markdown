---
layout: post
title:  "CloudFoundry Summit in 2018 Boston"
author: 윤상준
date: 2018-05-25 01:45:05 +0900
categories: cloudfoundry
---

## CF^3 - Putting a Kubernetes Behind CF - Julz Friedman, Andrew Edgar & Julian Skupnjak, IBM

### Link
https://www.youtube.com/watch?v=9l3GgW95GmQ&list=PLhuMOCWn4P9hJD3wsstF8gJIxOnJ_CTot&index=17

### Summary
- CloudFoundry의 핵심 가치는 개발자의 경험이다.(e.g cf push, service broker)
- Container Orchestrator(Diego + Garden)는 다양하게 지원하게 어떨까?(Kubernetes, Swarm, Mesos..)
- 이를 위해 아래와 같은 것들을 개발 중
  1. OPI(Orchecstrator Provider Interface) : 다양한 Orchestrator를 지원하도록 추상화
  2. Sync : CF로 배포한 앱을 Kubernetes에 맞게 변환. Staged app을 Docker image로 변환.
  3. Registry : CF droplet에 기반한 OCI(Open Container Initiative) registry?
  4. St8ge : Kubernetes에 staging 실행. Buildpack 탐색 및 다운로드. Droplet 업로드


