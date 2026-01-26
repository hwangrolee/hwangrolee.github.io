---
layout: post
title: 2019 Google Study Jam - 두번째이야기 Hello Node Kubernetes
date: 2019-01-10
description: 2019 Google Study Jam - 두번째이야기 Hello Node Kubernetes
categories: [DevOps]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /2019-Google-Study-Jam_Hello-Node-Kubernetes/
author: hwangrolee
tags: [Kubernetes]
---

오늘은 두번째 스터디를 진행했습니다.

주제는 **Hello Node Kubernetes**

주제에서 알 수 있듯이 kubenetes의 기본을 살펴보는 시간이었습니다.

kubenetes의 pod, service, deployments 의 개념을 명령어를 직업 쳐보면서 배우고 Kubernetes cluster, up scaling 작업을 통해 clustering의 동작 방식을 알게되었습니다.

마지막으로 쿠버네티스가 잘 동작하는지 시각적으로 보여주는 Dashboard설정법을 배웁니다.

기존에 kubernetes를 개념적으로 알고 있었지만 이렇게 실습해보니 더욱 쉽게 이해가 되고 좋았습니다.

그럼 오늘의 스터디를 시작해보겠습니다!!

오늘 스터디를 진행하면 아래의 아키텍처를 만나실수 있습니다.

오늘 진행할 사항

- Create a Node.js server.
- Create a Docker container image.
- Create a container cluster
- Create a Kubernetes pod.
- Scale up your services.

위 5가지를 수행하며 쿠버네티스 스터디를 진행했습니다.

리눅스 기반으로 스터디가 진행되며 vim, emacs, nano와 같은 리눅스기반 편집기를 사용할 줄 알아야합니다.

저는 vim을 예전부터 사용해왔기 때문에 vim을 사용했습니다.

### Create a Node.js server

> 쿠버네티스내에서 동작할 Nodejs server를 제작합니다.

```bash
$ vi server.js
```

```javascript
var http = require("http");
var handleRequest = function (request, response) {
  response.writeHead(200);
  response.end("Hello World!");
};
var www = http.createServer(handleRequest);
www.listen(8080);
```

위와같이 파일을 만듭니다.
위파일을 실행한 NodeJS 서버는은 8080포트로 접근가능하며 "Hello World!" 를 리턴합니다.

### Create a Docker container image.

> 쿠버네티스내 continer에서 동작할 Docker Image 를 만들어봅니다.

```bash
$ vi Dockerfile
```

```bash
FROM node:6.9.2     # Nodejs V6.9.2 이미지를 불러온다.
EXPOSE 8080         # 외부와 8080 포트를 연결한다.
COPY server.js .    # Build 위에 있는 server.js를 복사해온다.
CMD node server.js  # 컨테이너 실행시 node server.js를 실행한다.
```

```bash
# Dockerfile을 Image로 빌드합니다.
# image 이름은 gcr.io/PROJECT_ID/hello-node 입니다.
# 이미지에 v1 태그를 등록합니다.
$ docker build -t gcr.io/PROJECT_ID/hello-node:v1 .
Sending build context to Docker daemon  20.99kB
Step 1/4 : FROM node:6.9.2
6.9.2: Pulling from library/node
75a822cd7888: Pull complete
57de64c72267: Pull complete
4306be1e8943: Pull complete
871436ab7225: Pull complete
0110c26a367a: Pull complete
1f04fe713f1b: Pull complete
ac7c0b5fb553: Pull complete
Digest: sha256:2e95be60faf429d6c97d928c762cb36f1940f4456ce4bd33fbdc34de94a5e043
Status: Downloaded newer image for node:6.9.2
 ---> faaadb4aaf9b
Step 2/4 : EXPOSE 8080
 ---> Running in 3b9b5a4618dc
Removing intermediate container 3b9b5a4618dc
 ---> 46752700727f
Step 3/4 : COPY server.js .
 ---> 7a9afff80010
Step 4/4 : CMD node server.js
 ---> Running in 61bdb8747991
Removing intermediate container 61bdb8747991
 ---> 4e86e4736626
Successfully built 4e86e4736626
Successfully tagged gcr.io/qwiklabs-gcp-a22c2abe0529392c/hello-node:v1
```

build된 이미지를 실행해보고 결과를 확인해봅니다.

```bash
# -d : 백그라운드 모드
# -p : 포트 포워딩, 컨테이너의 8080 포트를 외부 8080와 연결
$ docker run -d -p 8080:8080 gcr.io/PROJECT_ID/hello-node:v1
fa0d785f94cb9eaf9fa73b0211057fe482e6c8f0a2265e503a904285c9a8e3c5

# GET 방식으로 요청합니다.
$ curl -XGET localhost:8080
Hello World!

# 실행중인 컨테이너를 확인합니다.
$ docker ps -a
CONTAINER ID        IMAGE                             COMMAND
fa0d785f94cb        gcr.io/PROJECT_ID/hello-node:v1   "/bin/sh -c…"

# 컨테이너를 중지합니다.
$ docker stop fa0d785f94cb
fa0d785f94cb # 중지된 컨테이너 ID를 반환합니다.
```

생성된 이미지를 Kubernetes에서 사용하기 위해 GCP registry에 저장해 봅니다.

```bash
# 생성된 이미지를 Google Container Registry에 push 합니다.
# 해당 이미지는 gcp web console -> Container Registry에서 확인가능합니다.
$ gcloud docker -- push gcr.io/PROJECT_ID/hello-node:v1
WARNING: `gcloud docker` will not be supported for Docker client versions above 18.03.

As an alternative, use `gcloud auth configure-docker` to configure `docker` to
use `gcloud` as a credential helper, then use `docker` as you would for non-GCR
registries, e.g. `docker pull gcr.io/project-id/my-image`. Add
`--verbosity=error` to silence this warning: `gcloud docker
--verbosity=error -- pull gcr.io/project-id/my-image`.

See: https://cloud.google.com/container-registry/docs/support/deprecation-notices#gcloud-docker

The push refers to repository [gcr.io/qwiklabs-gcp-a22c2abe0529392c/hello-node]
026f0f5e78a3: Pushed
381c97ba7dc3: Pushed
604c78617f34: Pushed
fa18e5ffd316: Pushed
0a5e2b2ddeaa: Pushed
53c779688d06: Pushed
60a0858edcd5: Pushed
b6ca02dfe5e6: Pushed
v1: digest: sha256:af6ecd3b8087d1fc49c34ff02910a1a63131e74510922e0c3353e930249d7999 size: 2002
```

GCP Registry에 저장된 Docker Image를 확인해봅니다.

### Create a container cluster

> 쿠버네티스 내의 컨테이너 클러스터를 생성합니다.

```bash
# gcloud 명령어를 사용하기 위해 자신의 PROJECT_ID를 세팅합니다.
$ gcloud config set project PROJECT_ID
Updated property [core/project].

# gcloud명령어로 클러스터를 생성합니다.
# 클러스터명 : hello-world
# 노드수 : 2
# 머신유형: n1-standard-1
# 클러스터 위치: us-central1-a
$ gcloud container clusters create hello-world \
                --num-nodes 2 \
                --machine-type n1-standard-1 \
                --zone us-central1-a
Creating cluster hello-world in us-central1-a...done.
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-a22c2abe0529392c/zones/us-central1-a/clusters/hello-wor
ld].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-central1-a
/hello-world?project=qwiklabs-gcp-a22c2abe0529392c
kubeconfig entry generated for hello-world.
NAME         LOCATION       MASTER_VERSION  MASTER_IP       MACHINE_TYPE   NODE_VERSION  NUM_NODES  STATUS
hello-world  us-central1-a  1.10.9-gke.5    35.188.172.182  n1-standard-1  1.10.9-gke.5  2          RUNNING
```

생성된 클러스터를 확인합니다.

### Create your pod

> 컨테이너 그룹인 포드를 생성하고 확인해봅니다.

pod를 실행하고 실행상태를 확인해봅니다.

```bash
$ kubectl run hello-node \
    --image=gcr.io/PROJECT_ID/hello-node:v1 \
    --port=8080
deployment.apps "hello-node" created

$ kubectl get deployments
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   1         1         1            1           40s

$ kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
hello-node-7db8fd78df-jz4pv   1/1       Running   0          1m

$ kubectl cluster-info
Kubernetes master is running at https://35.188.172.182
GLBCDefaultBackend is running at https://35.188.172.182/api/v1/namespaces/kube-system/services/default-http-backend:http/proxy
Heapster is running at https://35.188.172.182/api/v1/namespaces/kube-system/services/heapster/proxy
KubeDNS is running at https://35.188.172.182/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://35.188.172.182/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: REDACTED
    server: https://35.188.172.182
  name: gke_qwiklabs-gcp-a22c2abe0529392c_us-central1-a_hello-world
contexts:
- context:
    cluster: gke_qwiklabs-gcp-a22c2abe0529392c_us-central1-a_hello-world
    user: gke_qwiklabs-gcp-a22c2abe0529392c_us-central1-a_hello-world
  name: gke_qwiklabs-gcp-a22c2abe0529392c_us-central1-a_hello-world
current-context: gke_qwiklabs-gcp-a22c2abe0529392c_us-central1-a_hello-world
kind: Config
preferences: {}
users:
- name: gke_qwiklabs-gcp-a22c2abe0529392c_us-central1-a_hello-world
  user:
    auth-provider:
      config:
        access-token: ya29.GqMBjQZLvWCTZr0MQ4s5TCQK3NhaZFbpRqsMLG_11fyEs62wVbKfb4fnfFwA9hjT80eRUleTtwKH-lIhsIext2ewa9XPvFkno5eSJGQ5VOramQNB
JeiROmuAczSL5Tbgs4MviP_hva4X6CERtpKrOG4hjqzykjes2wYBVYibz8EKadcj1djlhsUI1DqenSQ5FjSRSsj4-5cAvAnsp1ZeLLCKeRmLKw
        cmd-args: config config-helper --format=json
        cmd-path: /google/google-cloud-sdk/bin/gcloud
        expiry: 2019-01-10T03:30:04Z
        expiry-key: '{.credential.token_expiry}'
        token-key: '{.credential.access_token}'
      name: gcp

$ kubectl get events

$ kubectl logs hello-node-7db8fd78df-jz4pv
```

### Allow external traffic

> 외부 네트워크와의 연결을 허용하며 IP를 부여받습니다.

```bash
$ kubectl expose deployment hello-node --type="LoadBalancer"
service "hello-node" exposed

$ kubectl get services
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)          AGE
hello-node   LoadBalancer   10.31.241.99   35.193.233.235   8080:32509/TCP   1m
kubernetes   ClusterIP      10.31.240.1    <none>           443/TCP          11m
```

### Scale up your service

> 배포된 pod를 scale up 하여 수를 증가시켜봅니다.

```bash
$ kubectl scale deployment hello-node --replicas=4
deployment.extensions "hello-node" scaled
$ kubectl get deployment
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   4         4         4            4           9m
$ kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
hello-node-7db8fd78df-6vn6h   1/1       Running   0          57s
hello-node-7db8fd78df-jz4pv   1/1       Running   0          10m
hello-node-7db8fd78df-r245c   1/1       Running   0          57s
hello-node-7db8fd78df-zdc9v   1/1       Running   0          57s
```

### Roll out an upgrade to your service

> 기존 배포된 nodejs를 버전업하여 배포해봅니다.

```bash
$ vi server.js
```

```javascript
var http = require("http");
var handleRequest = function (request, response) {
  response.writeHead(200);
  // response.end("Hello World!");
  response.end("Hello Kubernetes World!");
};
var www = http.createServer(handleRequest);
www.listen(8080);
```

```bash
$ docker build -t gcr.io/PROJECT_ID/hello-node:v2 .
Sending build context to Docker daemon  2.397MB
Step 1/4 : FROM node:6.9.2
 ---> faaadb4aaf9b
Step 2/4 : EXPOSE 8080
 ---> Using cache
 ---> 46752700727f
Step 3/4 : COPY server.js .
 ---> 82af2e80dcaf
Step 4/4 : CMD node server.js
 ---> Running in e54605d268f2
Removing intermediate container e54605d268f2
 ---> 71124401746d
Successfully built 71124401746d
Successfully tagged gcr.io/qwiklabs-gcp-a22c2abe0529392c/hello-node:v2
```

```bash
$ gcloud docker -- push gcr.io/PROJECT_ID/hello-node:v2
The push refers to repository [gcr.io/qwiklabs-gcp-a22c2abe0529392c/hello-node]
d7dd5a8d4a66: Pushed
381c97ba7dc3: Layer already exists
604c78617f34: Layer already exists
fa18e5ffd316: Layer already exists
0a5e2b2ddeaa: Layer already exists
53c779688d06: Layer already exists
60a0858edcd5: Layer already exists
b6ca02dfe5e6: Layer already exists
v2: digest: sha256:c2d1066f18f99955c9ab57557ae1aa2210cdfbed2ec55cfceb86b36bfbb378d8 size: 2002
```

```bash
$ kubectl edit deployment hello-node
```

spec.template.spec.containers.image 위치에 있는 gcr.io/PROJECT_ID/hello-node:v1 -> gcr.io/PROJECT_ID/hello-node:v2로 바꿈

```bash
# 저장시 아래와 같은 메세지를 확인
deployment.extensions "hello-node" edited
```

```bash
$ kubectl get deployments
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   4         4         4            4           19m
```

```bash
$ kubectl get services
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)          AGE
hello-node   LoadBalancer   10.31.241.99   35.193.233.235   8080:32509/TCP   16m
kubernetes   ClusterIP      10.31.240.1    <none>           443/TCP          25m
```

### Kubernetes graphical dashboard (optional)

> 대쉬보드 설정법을 알아봅니다.

```bash
$ kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)
Your active configuration is: [cloudshell-2007]
clusterrolebinding.rbac.authorization.k8s.io "cluster-admin-binding" created
```

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
secret "kubernetes-dashboard-certs" created
serviceaccount "kubernetes-dashboard" created
role.rbac.authorization.k8s.io "kubernetes-dashboard-minimal" created
rolebinding.rbac.authorization.k8s.io "kubernetes-dashboard-minimal" created
deployment.apps "kubernetes-dashboard" created
service "kubernetes-dashboard" created
```

```bash
$ kubectl -n kube-system edit service kubernetes-dashboard
Edit cancelled, no changes made.
```

```bash
$ kubectl -n kube-system describe $(kubectl -n kube-system \
get secret -n kube-system -o name | grep namespace) | grep token:
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlci10b2tlbi00NjRmeiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjdlZGI4ZDhlLTE0ODEtMTFlOS05Y2VmLTQyMDEwYTgwMDEyYiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpuYW1lc3BhY2UtY29udHJvbGxlciJ9.U7IGwSM66LwfjJQYVdvHPzK0DcK4t6wXGdvw55Vlq_SVFTeo7Uq1uYTDeQ-dvwX5JHsvYKrRMOwNfGB1PLYknAo_62PdzBwuZ7Nf7zMsOqJ1A25HC7JZQ6H2VOESfHBWrHYI-M6O3F7cagoTqFT8HAkGh_YRG6X_oY0OjMA5f5FyVr07AERtqXGqsx9w6SYi2rLQ9NcO2Y0cSWRBBIRE4MxsX35gldzjd1YrhE8cHkUvtj8HfsHKMwl6F39CtDVQAec4ZuOJr_Ols8CVxxFkcJdIHDvWW81TkdixFIza7RpKddGCiwyndHOnBP-LEwTPrPz5z8vdWGOCdDa2G97qtw
```

```bash
$ kubectl proxy --port 8081
Starting to serve on 127.0.0.1:8081
```

위와 같이 실행하면 대쉬보드가 실행됩니다.

기본 링크에
**/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/cronjob?namespace=default**를 이어 붙이면 대쉬보드로 이동가능합니다.

위에서 얻은 토큰으로 로그인을 시도하면 대쉬보드로 이동한다.
