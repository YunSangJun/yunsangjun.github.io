---
layout: post
title:  "OAuth Proxy를 활용한 애플리케이션 인증 및 인가"
author: sj
date: 2018-05-23
categories: authentication
---

OIDC(OpenID Connect identity providers)와 연계하여 애플리케이션의 인증을 해야하는 경우가 있습니다. 자체 개발한 애플리케이션의 경우 OIDC와 연동하기 위한 인증 기능을 개발하면 됩니다. 하지만 오픈소스를 사용하는 경우 해당 오픈소스에서 OIDC와의 연동기능을 제공하지 않으면 오픈소스의 코드를 수정해야합니다(예 : Kibana). 하지만 오픈소스의 코드를 수정하면 업스트림과의 일관성을 유지해야하는 부분이 부담스러울수 있습니다. 이 경우 [Keycloak Proxy](https://github.com/kubernetes/charts/tree/master/incubator/keycloak-proxy)와 같은 OAuth Proxy를 사용하여 애플리케이션의 인증/인가를 처리할 수 있습니다.

이 페이지에서는 Keycloak Proxy를 활용하여 애플리케이션의 인증을 처리하는 방법에 대해 설명합니다.
아래의 "데모 애플리케이션"과 "Keycloak Proxy" 배포 따라하기를 통해 Keycloak Proxy를 활용한 인증방법을 쉽게 이해할 수 있습니다.

## 사전 준비

### Kubernetes Cluster 준비

이 데모 애플리케이션과 Keycloak Proxy는 Kubernetes 환경을 위해 만들어졌습니다.

퍼블릭 클라우드에서 제공하는 Kubernetes(GKE, IKS, AKS, EKS 등)나 Private Kubernetes, Minikube 등의 환경을 준비합니다.

### Keycloak(OpenID Connect Provider) 환경 준비

Keycloak Proxy는 OIDC인 Keycloak과의 연동을 위해 디자인 되었지만 다른 OIDC 와도 연동할 수 있습니다.

[Keycloak Helm Chart](https://github.com/kubernetes/charts/tree/master/incubator/keycloak)를 활용하여 Keycloak을 쉽게 설치할 수 있습니다.

또는 이미 설치되어 있는 Keycloak이나 다른 OIDC(OpenID Connect identity providers)를 사용할 수 있습니다.

### Realm, Client, Role 생성

Keycloak Proxy와 Keycloak을 연동하기 위해서는 Keycloak 관리자 대시보드에서 Realm, Client, Role을 미리 생성해야합니다.

[Keycloak 가이드](https://github.com/YunSangJun/keycloak-proxy-demo/blob/master/keycloak-guide.md)를 참고하여 Realm, Client, Role을 생성합니다.

## 데모 애플리케이션 배포하기

### 구조

전제 조건 : 이 데모 애플리케이션에서는 Keycloak Proxy가 `/admin` endpoint에 대해서만 권한을 체크하도록 설정했습니다.

- Keycloak Proxy는 `/user` endpoint에 대한 클라이언트 요청을 허용합니다.

![Keycloak Proxy Flow Allow](/assets/images/keycloak_proxy_flow_user_allow.png)

- Keycloak Proxy는 클라이언트가 admin 권한을 가지고 있지 않다면 `/admin` endpoint에 대한 클라이언트 요청을 거부합니다.

![Keycloak Proxy Flow Deny](/assets/images/keycloak_proxy_flow_admin_deny.png)

- Keycloak Proxy는 클라이언트가 admin 권한을 가지고 있다면 `/admin` endpoint에 대한 클라이언트 요청을 허용합니다.

![Keycloak Proxy Flow Deny](/assets/images/keycloak_proxy_flow_admin_allow.png)

### 다운로드

아래 github 저장소에서 데모 애플리케이션을 다운로드합니다.

```
$ git clone https://github.com/YunSangJun/keycloak-proxy-demo
$ cd keycloak-proxy-demo
```

### 배포

아래 명령을 실행하여 애플리케이션과 서비스를 배포합니다.

```
$ kubectl apply -f deployment.yaml
$ kubectl apply -f service.yaml
```

### 배포 확인

아래 명령을 실행하여 정상적으로 배포되었는지 확인합니다.

```
$ kubectl get po,svc
NAME                       READY     STATUS    RESTARTS   AGE
po/demo-85cdbcc8c7-6pkbv   1/1       Running   0          10s

NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
svc/demo-service   ClusterIP   172.21.189.11   <none>        80/TCP    10s
```

### 접속 확인

데모 애플리케이션에 접속 할 수 있도록 `port-forward` 설정을 합니다.

```
$ kubectl port-forward demo-85cdbcc8c7-6pkbv 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
```

이제 `http://127.0.0.1:8080/user`와 `http://127.0.0.1:8080/admin`엡 접속해봅니다.
"Hello User!"와 "Hello Admin!" 메세지를 볼 수 있습니다.
현재는 `/user`와 `/admin` endpoint에 별도의 인증없이 접속 할 수 있습니다.

다음 과정에서는 Keycloak Proxy를 배포하고 이를 Keycloak(OpenID Connect Provider)과 연계하여 admin 권한을 가진 사용자만 `/admin` endpoint에 접속할 수 있도록 설정해보겠습니다.

## Keycloak Proxy 배포하기

아래 가이드를 참고하여 Keycloak Proxy를 배포해보겠습니다.

Keycloak Proxy에 대한 자세한 설명은 [Keycloak Proxy Helm Chart](https://github.com/kubernetes/charts/tree/master/incubator/keycloak-proxy)를 참고하세요.

1. `values.yaml` 작성하기

아래와 같이 `values.yaml`을 작성합니다.

참고: 사전 준비에서 Keycloak 관리자 대시보드에서 Realm, Client, Role을 생성했습니다. 이 정보를 아래 `configmap` 속성에 작성해야합니다.
이 페이지에서는 `NodePort`를 사용해 서비스를 외부에 노출시켰습니다. `LoadBalancer`나 `Ingress`를 사용할 수도 있습니다.

```
service:
  type: NodePort
  nodePort: 32589
  port: 80

configmap:
  targetUrl: http://demo-service
  realm: demo
  realmPublicKey: "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsLa2YmPyakziINoUgRLrEHLCKcyz62LhLU4JQsbJXMa1Zj4u/bU5D4dau1WkF94ivKV1osvqJqtQ4jCJYfRYrhCYeYOZWB7YFxELj+zMyP72Gxqg/YfWXKrzVnI5MYdZNx52dWAvBVDsDrxiZzJ0Xc92qCdKnEbvpK50XCh15KjWSjucbcJPwGX6kclLCmX0V47ziSo83FjH3ddFP81Kmza3on569Xi0QAAx3g/ZgPgZOSuF9OWwh3aMTwkfx9DlGeU5pY7uqvjuM9v33g0tdpOEelRAqu0aH/HEFXk9Mn74U1GQU/drflQVWEbv+9YvnUJN4cGt0oqmwQYU+Ix4qwIDAQAB"
  authServerUrl: http://url-to-keycloak.example.com/auth
  resource: demo
  secret: 2b2c17f0-245e-4978-a663-9a02a268a8f4
  pattern: /admin
  rolesAllowed: admin
```

2. Keycloak Proxy 배포하기

아래 명령을 실행하여 Keycloak Proxy를 배포합니다.

```
$ helm install --name keycloak-proxy -f values.yaml incubator/keycloak-proxy
NAME:   keycloak-proxy
LAST DEPLOYED: Wed May 16 01:17:25 2018
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/Deployment
NAME            DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
keycloak-proxy  1        1        1           0          0s

==> v1/Pod(related)
NAME                             READY  STATUS             RESTARTS  AGE
keycloak-proxy-67df99bbd5-ckfx7  0/1    ContainerCreating  0         0s

==> v1/ConfigMap
NAME                      DATA  AGE
keycloak-proxy-configmap  1     0s

==> v1/Service
NAME                    TYPE      CLUSTER-IP     EXTERNAL-IP  PORT(S)       AGE
keycloak-proxy-service  NodePort  172.21.215.70  <none>       80:32589/TCP  0s


NOTES:
1. Keycloak Proxy can be accessed:

   * Within your cluster, at the following DNS name at port 80:

     keycloak-proxy.demo.svc.cluster.local

   * From outside the cluster, run these commands in the same shell:

     export NODE_PORT=$(kubectl get --namespace demo -o jsonpath="{.spec.ports[0].nodePort}" services keycloak-proxy-service)
     export NODE_IP=$(kubectl get nodes --namespace demo -o jsonpath="{.items[0].status.addresses[0].address}")
     echo http://$NODE_IP:$NODE_PORT
```

## 무엇이 일어났는지 확인하기

이제 Keycloak Proxy에 접속하여 admin 권한을 가진 사용자만 `/admin` endpoint에 접속할 수 있는지 확인해보겠습니다.

1. `http://$NODE_IP:$NODE_PORT/user` 접속

    먼저 `/user` endpoint에 접속해봅니다. 이전과 마찬가지로 인증없이 접근이 가능합니다. Keycloak Proxy 배포 시 `pattern` 설정에 `/admin` 패턴만 설정했기 때문에 `/user` endpoint는 bypass 합니다.

    ![Authentication and Authorization](./img/page_user.png)

2. Connect to `http://$NODE_IP:$NODE_PORT/admin`

    Keycloak Proxy will redirect to Keycloak login page because Keycloak Proxy check authentication and authorization about `/admin` endpoint.
    ![Authentication and Authorization](./img/login1.png)

    If you don't have a Keycloak account, create user. Click "Register" button and insert user information such as below.
    ![Authentication and Authorization](./img/add_user.png)

    Login with the user.
    ![Authentication and Authorization](./img/login2.png)

    If you success to login, now you become valid user.
    However, you will may get a "HTTP 403 error" because you don't have authorization about `/admin` endpoint.
    ![Authentication and Authorization](./img/page_admin_error.png)

3. Map "admin" role to user account

    Select "admin" role in "Available Roles" and click "Add selected" button.
    ![Authentication and Authorization](./img/role_mapping.png)

    Now, you have an authorization about `/admin` endpoint so you can access to `/admin` endpoint.
    ![Authentication and Authorization](./img/page_admin_success.png)
