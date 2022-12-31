# Stateless Applications



Deployment 를 작성하고 배포해보자. 한 컨테이너에 대해 복사본 5개를 만든다.

ReplicaSet 은 5개의 파드를 가진다.

**load-balancer-example.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	lables:
		app.kubernetes.io/name: load-balancer-example
	name: hello-world
spec:
	replicas: 5
	selector:
		matchLabels:
			app.kunbernetes.io/name: load-balancer-example
	template:
		metadata:
			labels:
				app.kubernetes.io/name: load-balancer-example
		spec:
			containers:
			- image: gcr.io/google-samples/node-hello:1.0
				name: hello-world
				ports:
				- containerPort: 8080
```

```shell
$ kubectl apply -f https://k8s.io/examples/service/load-balancer-example.yaml

$ kubectl get deployments hello-world
$ kubectl describe deployments hello-world

$ kubectl get rs
$ kubectl describe rs
```

<br />

서비스를 만들어 디플로이먼트를 노출(expose) 시켜보자.

**my-service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: load-balancer-example
  name: my-service
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app.kubernetes.io/name: load-balancer-example
  type: LoadBalancer
```

```shell
$ kubectl apply -f my-service.yaml

$ kubectl get services my-service
```

- `EXTERNAL-IP` 를 보면 `<pending>` 으로 나오게 되는데, `LoadBalancer` 타입을 사용하려면 외부 프로바이더(AWS EKS, GCP GKE 등)가 제공해주는 로드밸런서가 필요하다.



---



## PHP Guesetbook application with Redis

**redis-leader-deployment.yaml**

```shell
$ kubectl apply -f https://k8s.io/examples/application/guestbook/redis-leader-deployment.yaml
$ kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
redis-leader-5596fc7b68-tfmph   1/1     Running   0          22s
```

<br />

**redis-leader-service.yaml**

guestbook 애플리케이션이 데이터 쓰기를 하기 위해 leader 와 연결이 필요하므로, 서비스를 설정한다.

```shell
$ kubectl apply -f https://k8s.io/examples/application/guestbook/redis-leader-service.yaml
$ kubectl get service
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
redis-leader   ClusterIP   10.103.53.109   <none>        6379/TCP   11s
```

<br />

**redis-follower-deployment.yaml**

```shell
$ kubectl apply -f https://k8s.io/examples/application/guestbook/redis-follower-deployment.yaml
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
redis-follower-74d9c98c76-fxjjs   1/1     Running   0          26s
redis-follower-74d9c98c76-h4n28   1/1     Running   0          26s
redis-leader-5596fc7b68-tfmph     1/1     Running   0          5m6s
```

<br />

**redis-follower-service.yaml**

guestbook 애플리케이션이 데이터 읽기를 하기 위해 leader 와 연결이 필요하므로, 서비스를 설정한다.

```shell
$ kubectl apply -f https://k8s.io/examples/application/guestbook/redis-follower-service.yaml
$ kubectl get service
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
redis-follower   ClusterIP   10.101.16.251   <none>        6379/TCP   11s
redis-leader     ClusterIP   10.103.53.109   <none>        6379/TCP   5m17s
```

<br />

**fronted-deployment.yaml**

프론트 앱에 대한 디플로이먼트를 생성한다.

```shell
$ kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-deployment.yaml
$ kubectl get pods -l app=guestbook -l tier=frontend
NAME                        READY   STATUS              RESTARTS   AGE
frontend-5b46649f74-d5vmj   0/1     ContainerCreating   0          43s
frontend-5b46649f74-jtqw8   1/1     Running             0          43s
frontend-5b46649f74-njbv7   1/1     Running             0          43s
```

<br />

**fronted-service.yaml**

프론트 앱은 쿠버네티스 클러스터 외부에서 접근할 수 있어야 하기 때문에 `NodePort` 등 외부 IP 를 만들어 주어야한다. 또는 AWS EKS, GCP GKE 같은 프로바이더가 제공해주는 로드밸런서가 있다면 서비스에 `type: LoadBalancer` 를 추가해서 외부 IP 를 설정할 수 도 있다.

여기 실습에서는 클라이언트가 proxy 를 통해 서비스에 접근한다고 가정하고, 기본값인 `type: ClusterIP` 로 설정한다.

```shell
$ kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-service.yaml
$ kubectl get services
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
frontend         ClusterIP   10.110.43.124   <none>        80/TCP     7s
redis-follower   ClusterIP   10.101.16.251   <none>        6379/TCP   8m44s
redis-leader     ClusterIP   10.103.53.109   <none>        6379/TCP   13m
```

<br />

```shell
$ kubectl port-forward svc/frontend 8080:80
```

로컬 머신 8080 포트에 서비스 80 포트가 대응되도록 프록시를 설정한다.

그리고 localhost:8080 에 접속해서 브라우저로 프론트서버를 접속해볼 수 있다. 메시지를 전송하고 읽는 것을 확인해 볼 수 있다.

<br />

<br />

<br />

<br />

<br />









