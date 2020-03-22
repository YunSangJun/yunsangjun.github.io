---
layout: post
title:  "Istio를 활용해 Multi Cluster 환경에 Service Mesh 구성하기"
author: sj
date: 2019-08-11
categories: istio
tags:
- istio
- servicemesh
- multicluster
- kubernetes
---

## Overview

Istio를 활용하여 여러개의 Kubernetes Cluster 환경에 Service Mesh를 구성할 수 있습니다.

인프라 및 Kubernetes Cluster 환경에 따라 3가지 구성 방법이 있습니다.

이 문서에서는 1번 방법을 활용하여 Service Mesh를 구성해보겠습니다.

1. [Shared control plane (multi-network)](https://istio.io/docs/setup/kubernetes/install/multicluster/shared-gateways/)

    이 방법은 여러개의 Cluster가 하나의 Istio Control Plane을 공유합니다.
    Istio Gateway를 통해 Cluster간 통신하므로 각 Cluster의 네트워크가 분리되어 있고 VPN 또는 Direct 네트워크로 연결되어 있지 않아도 됩니다. 

    ![](/assets/images/kubernetes/istio/istio-shared-multi.svg)

2. [Shared control plane (single-network)](https://istio.io/docs/setup/kubernetes/install/multicluster/shared-vpn/)

    1번과 마찬가지로 여러개의 Cluster가 하나의 Istio Control Plane을 공유합니다.
    별도의 Gateway가 없기 때문에 각 Cluster의 네트워크가 VPN 등을 통해 연결성이 있어야합니다.
    각 Cluster의 네트워크에서 Pod와 Service의 CIDR은 중복되서는 안되고 서로간의 라우팅이 가능해야합니다. 

    ![](/assets/images/kubernetes/istio/istio-shared-single.svg)

3. [Multiple control planes](https://istio.io/docs/setup/kubernetes/install/multicluster/gateways/)

    1번과 마찬가지로 Istio Gateway를 통해 Cluster간 통신을 하지만 Istio Control Plance을 공유하지 않고 각각의 Cluster에 설치합니다.

    ![](/assets/images/kubernetes/istio/istio-multiple-multi.svg)

## 준비하기

### Kubernetes Cluster 준비

1.12, 1.13, 1.14 버전의 Kubernetes Cluster를 2개 이상 준비합니다.
이 문서에서는 1.12 버전의 2개의 Cluster를 활용하겠습니다.

편의상 2개의 Cluster를 아래와 같이 환경 변수로 설정합니다.

```
$ kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO       NAMESPACE
*         cluster1   cluster1   user@foo.com   default
          cluster2   cluster2   user@foo.com   default

$ export CTX_CLUSTER1=$(kubectl config view -o jsonpath='{.contexts[0].name}')
$ export CTX_CLUSTER2=$(kubectl config view -o jsonpath='{.contexts[1].name}')
$ echo CTX_CLUSTER1 = ${CTX_CLUSTER1}, CTX_CLUSTER2 = ${CTX_CLUSTER2}
CTX_CLUSTER1 = cluster1, CTX_CLUSTER2 = cluster2
```

### Istio 다운로드

[Istio Release](https://github.com/istio/istio/releases) 페이지에서 원하는 버전을 다운로드합니다.

```
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=x.x.x sh -
```

### Platform 설정

[platform-specific setup](https://istio.io/docs/setup/kubernetes/platform-setup/) 페이지를 참고하여 각 클라우드별로
필요한 설정을합니다.

### Helm 설치

[Helm 설치하기](/helm/2018/05/27/installing-helm.html) 문서를 참고하여 Helm Client를 설치합니다.

## Multi Cluster에 Service Mesh 구성하기

### Primary Cluster 설정

1. Primary Cluster의 Istio deployment yaml 생성합니다.

    위에서 다운로드 받은 Istio 디렉토리로 이동합니다.

    ```
    $ cd istio-x.x.x
    ```

    helm template 기능을 사용하여 istio deployment yaml 파일을 생성합니다.

    ```
    $ helm template --name=istio --namespace=istio-system \
    --set global.mtls.enabled=true \
    --set security.selfSigned=false \
    --set global.controlPlaneSecurityEnabled=true \
    --set global.proxy.accessLogFile="/dev/stdout" \
    --set global.meshExpansion.enabled=true \
    --set 'global.meshNetworks.network1.endpoints[0].fromRegistry'=Kubernetes \
    --set 'global.meshNetworks.network1.gateways[0].address'=0.0.0.0 \
    --set 'global.meshNetworks.network1.gateways[0].port'=443 \
    --set gateways.istio-ingressgateway.env.ISTIO_META_NETWORK="network1" \
    --set global.network="network1" \
    --set 'global.meshNetworks.network2.endpoints[0].fromRegistry'=n2-k8s-config \
    --set 'global.meshNetworks.network2.gateways[0].address'=0.0.0.0 \
    --set 'global.meshNetworks.network2.gateways[0].port'=443 \
    install/kubernetes/helm/istio > istio-auth.yaml
    ```

2. Primary Cluster에 Istio를 설치합니다. 

    ```
    $ kubectl create --context=$CTX_CLUSTER1 ns istio-system
    $ kubectl create --context=$CTX_CLUSTER1 secret generic cacerts -n istio-system --from-file=samples/certs/ca-cert.pem --from-file=samples/certs/ca-key.pem --from-file=samples/certs/root-cert.pem --from-file=samples/certs/cert-chain.pem
    $ for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply --context=$CTX_CLUSTER1 -f $i; done
    $ kubectl apply --context=$CTX_CLUSTER1 -f istio-auth.yaml
    ```

    아래와 같이 모든 Pod의 Running 상태가 될 때까지 기다립니다.

    ```
    $ kubectl get pods --context=$CTX_CLUSTER1 -n istio-system
    NAME                                      READY   STATUS      RESTARTS   AGE
    istio-citadel-9bbf9b4c8-nnmbt             1/1     Running     0          2m8s
    istio-cleanup-secrets-1.1.0-x9crw         0/1     Completed   0          2m12s
    istio-galley-868c5fff5d-9ph6l             1/1     Running     0          2m9s
    istio-ingressgateway-6c756547b-dwc78      1/1     Running     0          2m8s
    istio-pilot-54fcf8db8-sn9cn               2/2     Running     0          2m8s
    istio-policy-5fcbd55d8b-xhbpz             2/2     Running     2          2m8s
    istio-security-post-install-1.1.0-ww5zz   0/1     Completed   0          2m12s
    istio-sidecar-injector-6dcc9d5c64-7hnnl   1/1     Running     0          2m8s
    istio-telemetry-57875ffb6d-n2vmf          2/2     Running     3          2m8s
    prometheus-66c9f5694-8pccr                1/1     Running     0          2m8s
    ```

3. Ingress Gateway를 생성합니다. 

    ```
    $ kubectl apply --context=$CTX_CLUSTER1 -f - <<EOF
    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
    name: cluster-aware-gateway
    namespace: istio-system
    spec:
    selector:
        istio: ingressgateway
    servers:
    - port:
        number: 443
        name: tls
        protocol: TLS
        tls:
        mode: AUTO_PASSTHROUGH
        hosts:
        - "*.local"
    EOF
    ```

4. Primary Cluster의 Ingress IP와 Port를 확인합니다.

    Primary Cluster에서 Ingress Gateway의 Service를 조회합니다.

    ```
    $ kubectl config use-context $CTX_CLUSTER1
    $ kubectl get svc istio-ingressgateway -n istio-system
    NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                      AGE
    istio-ingressgateway   LoadBalancer   172.x.x.1        130.x.x.1       80:31380/TCP,443:31390/TCP,31400:31400/TCP   17h
    ```

    Ingress의 Host 주소와 Secure port를 확인합니다.

    ``` 
    $ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    $ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
    $ echo The ingress gateway of cluster1: address=$INGRESS_HOST, port=$SECURE_INGRESS_PORT
    130.x.x.1, 443
    ```

5. Istio Configmap의 mesh nework 설정에서 gateway 주소를 위에서 조회한 Ingress 정보로 변경합니다.

    data.mesh.meshNetworks.networks.network1.address를 0.0.0.0에서 INGRESS_HOST로 변경합니다.
    data.mesh.meshNetworks.networks.network1.port를 443에서 SECURE_INGRESS_PORT 변경합니다.

    ```
    $ kubectl edit cm -n istio-system --context=$CTX_CLUSTER1 istio
    apiVersion: v1
    data:
        mesh:
        ...
            meshNetworks: "networks:\n  network1:\n    endpoints:\n    - fromRegistry: Kubernetes\n
        \   gateways:\n    - address: 0.0.0.0\n      port: 443\n  ...
    ```

### Second Cluster 설정

1. Primary Cluster의 Ingress Gateway 주소를 확인합니다.

    ```
    $ export LOCAL_GW_ADDR=$(kubectl get --context=$CTX_CLUSTER1 svc --selector=app=istio-ingressgateway \
    -n istio-system -o jsonpath='{.items[0].status.loadBalancer.ingress[0].ip}') && echo ${LOCAL_GW_ADDR}
    ```

2. Second Cluster의 Istio deployment yaml 생성합니다.

    ```
    $ helm template --name istio-remote --namespace=istio-system \
    --values install/kubernetes/helm/istio/values-istio-remote.yaml \
    --set global.mtls.enabled=true \
    --set gateways.enabled=true \
    --set security.selfSigned=false \
    --set global.controlPlaneSecurityEnabled=true \
    --set global.createRemoteSvcEndpoints=true \
    --set global.remotePilotCreateSvcEndpoint=true \
    --set global.remotePilotAddress=${LOCAL_GW_ADDR} \
    --set global.remotePolicyAddress=${LOCAL_GW_ADDR} \
    --set global.remoteTelemetryAddress=${LOCAL_GW_ADDR} \
    --set gateways.istio-ingressgateway.env.ISTIO_META_NETWORK="network2" \
    --set global.network="network2" \
    install/kubernetes/helm/istio > istio-remote-auth.yaml
    ```

3. Second Cluster에 Istio를 설치합니다.

    ```
    $ kubectl create --context=$CTX_CLUSTER2 ns istio-system
    $ kubectl create --context=$CTX_CLUSTER2 secret generic cacerts -n istio-system --from-file=samples/certs/ca-cert.pem --from-file=samples/certs/ca-key.pem --from-file=samples/certs/root-cert.pem --from-file=samples/certs/cert-chain.pem
    $ kubectl apply --context=$CTX_CLUSTER2 -f istio-remote-auth.yaml
    ```

    Istio의 모든 Pod가 Running 상태가 될때까지 기다립니다. 
    Primary Cluster와 다르게 일부 컴포넌트만 설치됩니다.

    ```
    $ kubectl get pods --context=$CTX_CLUSTER2 -n istio-system -l istio!=ingressgateway
    NAME                                     READY   STATUS      RESTARTS   AGE
    istio-citadel-75c8fcbfcf-9njn6           1/1     Running     0          12s
    istio-cleanup-secrets-1.1.0-vtp62        0/1     Completed   0          14s
    istio-sidecar-injector-cdb5d4dd5-rhks9   1/1     Running     0          12s
    ```

4. Second Cluster의 Ingress IP와 Port를 확인합니다.

    Second Cluster에서 Ingress Gateway의 Service를 조회합니다.

    ```
    $ kubectl config use-context $CTX_CLUSTER2
    $ kubectl get svc istio-ingressgateway -n istio-system
    NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                      AGE
    istio-ingressgateway   LoadBalancer   172.x.x.2        130.x.x.2       80:31380/TCP,443:31390/TCP,31400:31400/TCP   17h
    ```

    Ingress의 Host 주소와 Secure port를 확인합니다.

    ``` 
    $ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    $ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
    $ echo The ingress gateway of cluster2: address=$INGRESS_HOST, port=$SECURE_INGRESS_PORT
    130.x.x.2, 443
    ```

5. Istio Configmap의 mesh nework 설정에서 gateway 주소를 위에서 조회한 Ingress 정보로 변경합니다.

    data.mesh.meshNetworks.networks.network2.address를 0.0.0.0에서 INGRESS_HOST로 변경합니다.
    data.mesh.meshNetworks.networks.network2.port를 443에서 SECURE_INGRESS_PORT 변경합니다.

    ```
    $ kubectl edit cm -n istio-system --context=$CTX_CLUSTER2 istio
    apiVersion: v1
    data:
        mesh:
        ...
        meshNetworks: "networks:\n  network1:\n    endpoints:\n    - fromRegistry: Kubernetes\n
            \   gateways:\n    - address: 0.0.0.0\n      port: 443\n  network2:\n    endpoints:\n
            \   - fromRegistry: n2-k8s-config\n    gateways:\n    - address: 0.0.0.0\n
            \     port: 443\n  "
    ```

6. n2-k8s-config 설정 파일을 생성하기 위한 환경 변수를 설정합니다.

    ```
    $ CLUSTER_NAME=$(kubectl --context=$CTX_CLUSTER2 config view --minify=true -o jsonpath='{.clusters[].name}')
    $ SERVER=$(kubectl --context=$CTX_CLUSTER2 config view --minify=true -o jsonpath='{.clusters[].cluster.server}')
    $ SECRET_NAME=$(kubectl --context=$CTX_CLUSTER2 get sa istio-multi -n istio-system -o jsonpath='{.secrets[].name}')
    $ CA_DATA=$(kubectl get --context=$CTX_CLUSTER2 secret ${SECRET_NAME} -n istio-system -o jsonpath="{.data['ca\.crt']}")
    $ TOKEN=$(kubectl get --context=$CTX_CLUSTER2 secret ${SECRET_NAME} -n istio-system -o jsonpath="{.data['token']}" | base64 --decode)
    ```

7. n2-k8s-config 파일을 생성합니다. 

    ```
    $ cat <<EOF > n2-k8s-config
    apiVersion: v1
    kind: Config
    clusters:
    - cluster:
        certificate-authority-data: ${CA_DATA}
        server: ${SERVER}
        name: ${CLUSTER_NAME}
    contexts:
    - context:
        cluster: ${CLUSTER_NAME}
        user: ${CLUSTER_NAME}
        name: ${CLUSTER_NAME}
    current-context: ${CLUSTER_NAME}
    users:
    - name: ${CLUSTER_NAME}
        user:
        token: ${TOKEN}
    EOF
    ```

### Primary Cluster와 Second Cluster 동기화

1. Primary Cluster에서 Second Cluster를 동기화합니다.

    ```
    $ kubectl create --context=$CTX_CLUSTER1 secret generic n2-k8s-secret --from-file n2-k8s-config -n istio-system
    $ kubectl label --context=$CTX_CLUSTER1 secret n2-k8s-secret istio/multiCluster=true -n istio-system
    ```

2. Second Cluster의 Ingress Gateway가 Running 상태가 될때까지 기다립니다.

    ```
    $ kubectl get pods --context=$CTX_CLUSTER2 -n istio-system -l istio=ingressgateway
    NAME                                    READY     STATUS    RESTARTS   AGE
    istio-ingressgateway-5c667f4f84-bscff   1/1       Running   0          16m
    ```

이제 Primary Cluster와 Seconde Cluster가 Service Mesh로 구성되었습니다.

다음으로 샘플 서비스를 배포하여 2개의 Cluster에서 Service Mesh가 어떻게 동작하는지 확인해보겠습니다.

## 샘플 서비스 배포하기

Overview의 구성도와 같이 helloworld 애플리케이션을 Primary와 Second 클러스터에 각각 배포할것입니다.

각각의 인스턴스는 동일한 애플리케이션이지만 이미지의 버전이 v1, v2로 다릅니다.

### Second Cluster에 helloworld v2 버전 배포

1. sample namespace를 생성하고 istio proxy가 자동으로 injection 되도록 설정합니다.

    ```
    $ kubectl create --context=$CTX_CLUSTER2 ns sample
    $ kubectl label --context=$CTX_CLUSTER2 namespace sample istio-injection=enabled
    ```

2. helloworld v2를 배포합니다.

    ```
    $ kubectl create --context=$CTX_CLUSTER2 -f samples/helloworld/helloworld.yaml -l app=helloworld -n sample
    $ kubectl create --context=$CTX_CLUSTER2 -f samples/helloworld/helloworld.yaml -l version=v2 -n sample
    ```

3. helloworld v2가 Running 상태가 될때까지 기다립니다.

    ```
    $ kubectl get po --context=$CTX_CLUSTER2 -n sample
    NAME                             READY     STATUS    RESTARTS   AGE
    helloworld-v2-7dd57c44c4-f56gq   2/2       Running   0          35s
    ```

### Primary Cluster에 helloworld v1 버전 배포

1. sample namespace를 생성하고 istio proxy가 자동으로 injection 되도록 설정합니다.

    ```
    $ kubectl create --context=$CTX_CLUSTER1 ns sample
    $ kubectl label --context=$CTX_CLUSTER1 namespace sample istio-injection=enabled
    ```

2. helloworld v1을 배포합니다.

    ```
    $ kubectl create --context=$CTX_CLUSTER1 -f samples/helloworld/helloworld.yaml -l app=helloworld -n sample
    $ kubectl create --context=$CTX_CLUSTER1 -f samples/helloworld/helloworld.yaml -l version=v1 -n sample
    ```

3. helloworld v1이 Running 상태가 될때까지 기다립니다.

    ```
    $ kubectl get po --context=$CTX_CLUSTER1 -n sample
    NAME                            READY     STATUS    RESTARTS   AGE
    helloworld-v1-d4557d97b-pv2hr   2/2       Running   0          40s
    ```

### Cluster간 트래픽 흐름 확인

2개의 Cluster에서 트래픽을 확인하기 위해 sleep 애플리케이션에서 helloworld 애플리케이션을 호출해보겠습니다.

1. sleep 애플리케이션을 2개의 Cluster에 배포하고 Running 상태가 될때까지 기다립니다.

    ```
    $ kubectl apply --context=$CTX_CLUSTER1 -f samples/sleep/sleep.yaml -n sample
    $ kubectl apply --context=$CTX_CLUSTER2 -f samples/sleep/sleep.yaml -n sample
    ```

2. Primary Cluster에서 helloworld를 여러번 호출합니다. 

    ```
    kubectl exec --context=$CTX_CLUSTER1 -it -n sample -c sleep $(kubectl get pod --context=$CTX_CLUSTER1 -n sample -l app=sleep -o jsonpath='{.items[0].metadata.name}') -- curl helloworld.sample:5000/hello
    ```

    각 클러스터의 v1, v2 버전이 번갈아가면서 호출되는 것을 확인할 수 있습니다.

    ```
    Hello version: v2, instance: helloworld-v2-758dd55874-6x4t8
    Hello version: v1, instance: helloworld-v1-86f77cd7bd-cpxhv
    ```

    Primary Cluster의 sleep pod의 istio-proxy 컨테이너의 로그를 조회해봅니다.
    한번은 Second Cluster의 Gateway를 통하여 Second Cluster에 있는 helloworld v2가 호출되었고
    다른 한번은 같은 Cluster의 helloworld v1이 호출되었습니다.
    
    ```
    $ kubectl logs --context=$CTX_CLUSTER1 -n sample $(kubectl get pod --context=$CTX_CLUSTER1 -n sample -l app=sleep -o jsonpath='{.items[0].metadata.name}') istio-proxy
    [2018-11-25T12:37:52.077Z] "GET /hello HTTP/1.1" 200 - 0 60 190 189 "-" "curl/7.60.0" "6e096efe-f550-4dfa-8c8c-ba164baf4679" "helloworld.sample:5000" "130.x.x.2:15443" outbound|5000||helloworld.sample.svc.cluster.local - 10.20.194.146:5000 10.10.0.89:59496 -
    [2018-11-25T12:38:06.745Z] "GET /hello HTTP/1.1" 200 - 0 60 171 170 "-" "curl/7.60.0" "6f93c9cc-d32a-4878-b56a-086a740045d2" "helloworld.sample:5000" "10.10.0.90:5000" outbound|5000||helloworld.sample.svc.cluster.local - 10.20.194.146:5000 10.10.0.89:59646 -
    ```

지금까지 살펴본 내용을 보면 Istio의 Service Mesh 구성을 통해 서로 다른 네트워크에 구성한 Cluster에서 실행중인 서비스간의 통신이
가능하다는 것을 확인할 수 있었습니다.
