# Kubernetes basic 1

<br />

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

- kubernetes deployment 를 통해 애플리케이션 인스턴스를 생성하면, deployment controller 가 지속적으로 인스턴스들을 모니터링한다.
- 이 때 만약 인스턴스를 호스팅하는 노드가 내려가거나 삭제되면, 디플로이먼트 컨트롤러는 클러스터 내의 다른 노드 안에 인스턴스로 대체한다.
- 이런 self-healing 메커니즘은 머신의 동작 실패나 유지를 다룬다.

기존 시스템에서는 애플리케이션을 시작하는 스크립트가 사용되지만, 이런 스크립트로 machine failure 를 복구하지는 못한다. 반면 쿠버네티스 디플로이먼트는 가용성을 지원한다.

![](https://d33wubrfki0l68.cloudfront.net/8700a7f5f0008913aa6c25a1b26c08461e4947c7/cfc2c/docs/tutorials/kubernetes-basics/public/images/module_02_first_app.svg)

## Deployment 실습

```shell
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1/1     1            1           2m8s
```

배포 과정

- 클러스터에서 앱을 배포할 적절한 노드를 찾는다.
- 그 노드에서 앱이 스케쥴된다.
- 필요시 새로운 노드에서 인스턴스가 다시 스케쥴 되도록 설정된다.

<br />

**프록시 연결**

```shell
# 다른 탭에서 실행 (아웃풋 없이 실행되고 있는 상태가 됨)
$ kubectl proxy
Starting to serve on 127.0.0.1:8001

# 원래 탭에서 시도
$ curl http://localhost:8001/version
{
  "major": "1",
  "minor": "20",
  "gitVersion": "v1.20.2",
  "gitCommit": "faecb196815e248d3ecfb03c680a4507229c2a56",
  "gitTreeState": "clean",
  "buildDate": "2021-01-13T13:20:00Z",
  "goVersion": "go1.15.5",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

- proxy 를 생성할 수 있고, 이를 통해 클라이언트 터미널에서 클러스터 간 private 망과 연결할 수 있다.
- 클러스터 컨트롤 플레인에 접근하여 쿠버네티스 API 를 사용할 수 있다.
- 프록시 없이 접근하려면 `Service` 를 사용하면 된다.

<br />

**실행중인 pod 확인**

```shell
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-57978f5f5d-szq4f

$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/
{
  "kind": "Pod",
  "apiVersion": "v1",
  ...
}
```

<br />

# App 탐색

## Pods

- 디플로이먼트를 생성하면, 쿠버네티스가 애플리케이션을 호스팅하기 위해 파드를 만든다.
- 파드는 하나이상의 컨테이너(docker 등)가 모인 그룹을 뜻하는 쿠버네티스 추상이다. 그리고 아래 리소스를 공유한다.
  - volumes 같은 저장소
  - 고유한 클러스터 IP 주소 네트워크
  - 각 컨테이너를 어떻게 실행할지에 대한 정보 (컨테이너 이미지 버전, 사용할 특정 포트 등)

![](https://d33wubrfki0l68.cloudfront.net/fe03f68d8ede9815184852ca2a4fd30325e5d15a/98064/docs/tutorials/kubernetes-basics/public/images/module_03_pods.svg)

<br />

## Nodes

- 파드는 항상 노드 안에서 실행된다.
- 노드는 worker 머신이다. 클러스터 구성에 따라 가상머신 또는 물리머신이 될 수 있다.
- 한 노드는 여러 파드를 가질 수 있고, 쿠버네티스 컨트롤 플레인이 자동으로 클러스터에서 노드 간 파드의 스케쥴링을 담당한다.

노드가 실행하는 것

- kubelet : 쿠버네티스 컨트롤 플레인과 노드 간 통신을 담당한다. 머신에서 실행되는 파드와 컨테이너들을 관리한다.
- 레지스트리로부터 컨테이너 이미지를 pull 하고, 컨테이너를 unpacking 하고 애플리케이션을 실행하는 역할을 하는 container runtime(Docker 같은 거임) 을 실행한다.

![](https://d33wubrfki0l68.cloudfront.net/5cb72d407cbe2755e581b6de757e0d81760d5b86/a9df9/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg)

공통적으로 사용되는 명령어

```shell
kubectl get # resources list
kubectl describe # resource information
kubectl logs # container logs in a pod
kubectl exec # execute a command on a container in a pod
```

<br />

## Exploring App 실습

```shell
kubectl get pods
kubectl describe pods
```

- 파드는 격리된 private 네트워크에서 실행되기 때문에, 디버그와 통신을 위해 프록시 접근이 필요하다.

```shell
# terminal tab2
kubectl proxy
...
Starting to serve on 127.0.0.1:8001

# terminal tab1
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-fb5c67579-kg5sj

$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-fb5c67579-kg5sj | v=1
```

- 애플리케이션이 전송하는 `STDOUT` 은 전부 파드 안 컨테이너의 로그가 된다.
- 아래 명령어로 로그를 확인할 수 있다. 컨테이너를 명시하면 해당 컨테이너만 로그를 볼 수 있다.

```shell
$ kubectl logs $POD_NAME
Kubernetes Bootcamp App Started At: 2022-12-20T09:23:29.369Z | Running On:  kubernetes-bootcamp-fb5c67579-kg5sj 

Running On: kubernetes-bootcamp-fb5c67579-kg5sj | Total Requests: 1 | App Uptime: 533.266 seconds | Log Time: 2022-12-20T09:32:22.635Z
```

또한 파드 컨테이너 안에서 직접 명령어를 실행할 수도 있다.

```shell
$ kubectl exec $POD_NAME -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=kubernetes-bootcamp-fb5c67579-kg5sj
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
NPM_CONFIG_LOGLEVEL=info
NODE_VERSION=6.3.1
HOME=/root

# bash 세션 시작
$ kubectl exec -ti $POD_NAME -- bash
```



# 앱을 퍼블릭으로 노출

## Services

- 서비스는 파드의 논리적 집합을 정의하고, 접근 제어 정책을 정의하는 추상(abstraction)이다.
  - 파드는 생명주기가 있고, 언젠가는 죽는다.(mortal) 예를 들어 노드가 죽으면 그 안의 파드도 같이 없어진다.
  - 서비스는 파드 간 느슨한 결합을 가능하게 한다.

`type`

- ClusterIP (default) : 클러스터 안에 내부 IP 에 서비스를 노출한다.
- NodePort : 클러스터 안 각 선택된 노드의 같은 port 에 서비스를 노출한다. NAT(네트워크 주소변환) 을 사용한다. `NodeIP:NodePort` 형식으로 클러스터 바깥에서 안으로 서비스 접근할 수 있게 한다. ClusterIP 를 포함하는 확대집합(superset)이다.
- LoadBalancer : 클라우드 안에 로드밸런서를 생성하고, 고정된 외부 IP 를 서비스에 할당한다. NodePort 를 포함하는 확대집합이다.
- ExternalName : 정의한 `externalName` (예. foo.bar.example.com)에 서비스를 매핑한다. `externalName` 에 해당하는 CNAME 을 반환한다.

`selector` 없이 정의된 서비스는 해당 엔드포인트 오브젝트를 만들지 않는다. 따라서 사용자가 직접 특정 엔드포인트를 서비스와 매핑할 수 있다.

## Labels

![](https://d33wubrfki0l68.cloudfront.net/7a13fe12acc9ea0728460c482c67e0eb31ff5303/2c8a7/docs/tutorials/kubernetes-basics/public/images/module_04_labels.svg)

label 을 통해 파드들과 서비스를 연결하여 구분할 수 있다. label 은 키-값 쌍이고, 오브젝트에 붙여서 다양한 방법으로 사용된다.

- development, test, productioon 구분 가능
- 버전 태그
- 태그를 통해 오브젝트 분류

## Expose 실습

```shell
$ kubectl get pods
NAME                                  READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-fb5c67579-646sq   1/1     Running   0          103s
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2m7s
```

이제 명령어를 통해 expose 해보자. (선언형 yaml 로 하길 권장하지만 실습이라 명령어 활용)

```shell
$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080 service/kubernetes-bootcamp exposed

$ kubectl get services
NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes            ClusterIP   10.96.0.1      <none>        443/TCP          3m8s
kubernetes-bootcamp   NodePort    10.107.168.6   <none>        8080:30923/TCP   33s

$ kubectl describe services/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   app=kubernetes-bootcamp
Annotations:              <none>
Selector:                 app=kubernetes-bootcamp
Type:                     NodePort
IP Families:              <none>
IP:                       10.107.168.6
IPs:                      10.107.168.6
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30923/TCP
Endpoints:                172.18.0.2:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

- `kubernetes-bootcamp` 서비스가 생성되었다. PORT(S) 를 보면, `내부포트:외부포트` 를 확인할 수 있다.
- 내부포트는 8080이다.

```shell
$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
$ echo NODE_PORT=$NODE_PORT
NODE_PORT=30923

$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-fb5c67579-646sq | v=1
```

- `curl 10.0.0.8:30923` 을 실행한 것이다. 외부포트는 30923 임을 알 수 있다.

<br />

이제 label 을 사용해보자. 레이블은 `키=값` 형태의 스트링이다.

```shell
$ kubectl describe deployment
...
  Labels:  app=kubernetes-bootcamp
...
$ kubectl get pods -l app=kubernetes-bootcamp
NAME                                  READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-fb5c67579-646sq   1/1     Running   0          11m
$ kubectl get services -l app=kubernetes-bootcamp
NAME                  TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes-bootcamp   NodePort   10.107.168.6   <none>        8080:30923/TCP   10m
```

- `-l` 옵션으로 label 명을 가진 것만 쿼리할 수 있다.

레이블명을 가져와서 해당 레이블을 가진 오브젝트를 수정해보자.

```shell
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-fb5c67579-646sq

# 레이블을 추가한다.
$ kubectl label pods $POD_NAME version=v1
pod/kubernetes-bootcamp-fb5c67579-646sq labeled

# 레이블이 추가되어 총 3개가 됐다.
$ kubectl describe pods $POD_NAME
...
Labels:       app=kubernetes-bootcamp
              pod-template-hash=fb5c67579
              version=v1
...

# 추가한 레이블로 검색
$ kubectl get pods -l version=v1
NAME                                  READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-fb5c67579-646sq   1/1     Running   0          14m
```

<br />

```shell
# 레이블로 서비스 삭제
$ kubectl delete service -l app=kubernetes-bootcamp 
service "kubernetes-bootcamp" deleted

# 서비스 삭제 확인
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   17m

# 노드포트 타입인 서비스가 삭제되었으므로 외부에서 연결할 수 없음
$ curl $(minikube ip):$NODE_PORT
curl: (7) Failed to connect to 10.0.0.8 port 30923: Connection refused

# 파드에 직접 들어가서 확인
$ kubectl exec -it $POD_NAME -- curl localhost:8080
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-fb5c67579-646sq | v=1
```



# Scale your app

# 앱 변경





<br />

<br />

<br />

<br />

<br />
