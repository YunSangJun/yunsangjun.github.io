---
layout: post
title:  "Kubernetes 스터디 #1 Overview"
author: 윤상준
date: 2018-06-30
categories: Kubernetes
---

## Kubernetes 란?

## Kubernetes 컴포넌트

## Kubernetes Object에 대한 이해

### Labels and Selectors

#### Label selectors

##### Equality-based requirement

```
environment = production
tier != frontend
```

```
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia-tesla-p100
```

##### Set-based requirement

```
environment in (production, qa)
tier notin (frontend, backend)
partition
!partition
```

## Kubectl을 사용한 Object 관리
