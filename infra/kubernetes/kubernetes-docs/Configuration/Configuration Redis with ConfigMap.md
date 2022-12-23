# Configuration Redis with ConfigMap



[Killercoda](https://killercoda.com/playgrounds/scenario/kubernetes) 에 접속해서 쿠버네티스 클러스터를 활용한다.



```shell
cat <<EOF >./example-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: ""
EOF

kubectl apply -f example-redis-config.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml

$ kubectl get configmaps 
NAME                   DATA   AGE
example-redis-config   1      5m22s
kube-root-ca.crt       1      38d

$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
redis   1/1     Running   0          43s

# 아래처럼 하나의 특정 오브젝트만 확인가능
$ kubectl get pod/redis configmap/example-redis-config
NAME        READY   STATUS    RESTARTS   AGE
pod/redis   1/1     Running   0          6m43s

NAME                             DATA   AGE
configmap/example-redis-config   1      6m49s
```

[`redis-pod.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml) 의 내용을 조금 보면,

- config 라는 volumn 이 생성된다. 이 볼륨에서 `ConfigMap` 으로부터 받아온 설정 정보를  `redis.conf` 라는 파일에 `redis-config` 라는 변수로 매핑한다.

<br />

```shell
# configmap 안의 key-value 까지 모두 본다.
$ kubectl describe configmap/example-redis-config
Name:         example-redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
----


BinaryData
====

Events:  <none>
```

레디스에 접속해보자.

```shell
$ kubectl exec -it redis -- redis-cli

# redis 설정 정보 확인
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "0"

127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "noeviction"
```

이 값들을 설정해보기 위해 configmap yaml 파일을 수정해보자.

**example-redis-config.yaml**

```yaml
...
data:
  redis-config: |
    maxmemory 2mb
    maxmemory-policy allkeys-lru
```

```shell
$ kubectl apply -f example-redis-config.yaml 
configmap/example-redis-config configured

$ kubectl describe configmap/example-redis-config
...
Data
====
redis-config:
----
maxmemory 2mb
maxmemory-policy allkeys-lru
...
```

이제 레디스 파드를 삭제하고 다시 실행한다.

```shell
$ kubectl replace --force -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
pod "redis" deleted
pod/redis replaced

# 아래 동작을 위 replace --force 로 한 번에 할 수 있다.
$ kubectl delete pod redis
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml

# 레디스 설정 값 변경 확인
$ kubectl exec -it redis -- redis-cli
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "2097152"
127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "allkeys-lru"

# delete 후 실습 마무리
$ kubectl delete pod/redis configmap/example-redis-config
```

