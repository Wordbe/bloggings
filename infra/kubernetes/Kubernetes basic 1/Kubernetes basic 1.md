# Kubernetes basic 1



```shell
# 설치
brew install kubectl

# 버전 확인
kubectl version --client --output=yaml
clientVersion:
  buildDate: "2022-05-03T13:46:05Z"
  compiler: gc
  gitCommit: 4ce5a8954017644c5420bae81d72b09b735c21f0
  gitTreeState: clean
  gitVersion: v1.24.0
  goVersion: go1.18.1
  major: "1"
  minor: "24"
  platform: darwin/arm64
kustomizeVersion: v4.5.4
```

<br /><br />

# Cluster 생성

쿠버네티스는 효율적인 방식으로 클러스터간 애플리케이션 컨테이너들의 배포와 스케쥴링을 자동화한다.

![](https://d33wubrfki0l68.cloudfront.net/283cc20bb49089cb2ca54d51b4ac27720c1a7902/34424/docs/tutorials/kubernetes-basics/public/images/module_01_cluster.svg)

- Control Plane 은 클러스터를 관리한다.
  - 애플리케이션의 스케쥴링, 바라는 상태(desired state)로 유지, 스케일링, 새로운 변경으로 rolling out 등을 담당한다.

<br />

- 노드는 쿠버네티스 클러스터에서 워커로 일하는 가상머신(VM) 또는 물리적 컴퓨터이다.
  - 각 노드는 kubelet 을 가지는데, 노드를 관리하는 대리자 역할을 한다.
  - 노드는 컨트롤 플레인과 쿠버네티스 API 를 통해 통신한다.



<br />

## 클러스터 실습

minikube 를 설치하고, minikube 를 통해 가상머신을 실행시켜 그 안에 쿠버네티스 클러스터를 생성한다.

```shell
minikube version
minikube start
```

쿠버네티스 작업을 위해 kubectl 을 이용한다.

```shell
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.4", GitCommit:"e87da0bd6e03ec3fea7933c4b5263d151aafd07c", GitTreeState:"clean", BuildDate:"2021-02-18T16:12:00Z", GoVersion:"go1.15.8", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.2", GitCommit:"faecb196815e248d3ecfb03c680a4507229c2a56", GitTreeState:"clean", BuildDate:"2021-01-13T13:20:00Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"linux/amd64"}
```

- 클라이언트 버전은 kubectl 버전이고, 서버 버전은 마스터에 설치된 쿠버네티스 버전이다.

```shell
$ kubectl cluster-info
Kubernetes control plane is running at https://10.0.0.33:8443
KubeDNS is running at https://10.0.0.33:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

$ kubectl get nodes
NAME       STATUS   ROLES                  AGE    VERSION
minikube   Ready    control-plane,master   5m3s   v1.20.2
```

- 위 명령어는 현재 하나의 노드가 ready 상태임을 보여준다. (배포할 애플리케이션을 승인할 준비가 되었다는 뜻이다.)

# App 배포







# App 탐색







# 앱을 퍼블릭으로 노출







# Scale your app







# 앱 변경





<br />

<br />

<br />

<br />

<br />
