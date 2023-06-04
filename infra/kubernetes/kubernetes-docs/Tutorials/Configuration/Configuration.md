# Configuration

- 여러 컨테이너에 재사용 될 수 있는 장점이 있다.
- `ConfigMaps` 는 API 오브젝트이다. 민감하지 않은 키-값 쌍을 저장한다.
- `Secrets` 는 기밀, 민감 정보를 Base64 인코딩을 사용하여 저장한다.



## 코드 외부로 설정을 분리 (externalizing)

- 설정 정보는 환경에 따라 바뀔 수 있으므로 외부 설정은 유용할 수 있다.
- `ConfigMap`  과 `Secret` 을 만들고,   `MicroProfile Config` 를 사용해서 마이크로서비스 설정을 주입해보자.

<br />

## Configuration 실습

```shell
mvn package -pl system
mvn package -pl inventory

kubectl apply -f kubernetes.yaml
$ kubectl get deployments
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
inventory-deployment   1/1     1            1           66s
kubernetes-bootcamp    1/1     1            1           10m
system-deployment      1/1     1            1           66s
$ kubectl get services
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
inventory-service   NodePort    10.98.250.226   <none>        9080:32000/TCP   25s
kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP          9m56s
system-service      NodePort    10.102.95.248   <none>        9080:31000/TCP   25s
```

<br />

**kubernetes.yaml** 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: system-deployment
  labels:
    app: system
spec:
  selector:
    matchLabels:
      app: system
  template:
    metadata:
      labels:
        app: system
    spec:
      containers:
      - name: system-container
        image: system:1.0-SNAPSHOT
        ports:
        - containerPort: 9080
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 9080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 3
          failureThreshold: 1
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: inventory-deployment
  labels:
    app: inventory
spec:
  selector:
    matchLabels:
      app: inventory
  template:
    metadata:
      labels:
        app: inventory
    spec:
      containers:
      - name: inventory-container
        image: inventory:1.0-SNAPSHOT
        ports:
        - containerPort: 9080
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 9080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 3
          failureThreshold: 1
---
apiVersion: v1
kind: Service
metadata:
  name: system-service
spec:
  type: NodePort
  selector:
    app: system
  ports:
  - protocol: TCP
    port: 9080
    targetPort: 9080
    nodePort: 31000
---
apiVersion: v1
kind: Service
metadata:
  name: inventory-service
spec:
  type: NodePort
  selector:
    app: inventory
  ports:
  - protocol: TCP
    port: 9080
    targetPort: 9080
    nodePort: 32000
```

<br />

아래 명령어로 파드의 상태를 확인하고, 준비 상태(condition met 이면 준비상태)인지 확인한다.

```shell
$ kubectl wait --for=condition=ready pod -l app=inventory
pod/inventory-deployment-67ffcfc8f6-fqxrq condition met
$ kubectl wait --for=condition=ready pod -l app=system
pod/system-deployment-85b9978c95-hn6cf condition met

$ curl -u bob:bobpwd http://$( minikube ip ):31000/system/properties
$ curl http://$( minikube ip ):32000/inventory/systems/system-service

$ curl -# -I -u bob:bobpwd -D - http://$( minikube ip ):31000/system/properties | grep -i ^X-App-Name:

X-App-Name: system
X-App-Name: system
```

**curl**

- `-#` : `--progress-bar`
- `-u` : Basic Auth `userid:password`
- `-I` : 헤더만 가져오기
- `-D` : `--dump-header` , `-D <filename>` 으로 헤더 정보를 파일로 보낼 수 있다. `filename` 을 `-` 로 하면 헤더를 표준출력할 수 있다.

**grep**

- `-i` : `--ignore-case`, 대소문자 구분없이 검색

<br />

**ConfigMap, Secret 생성**

```shell
$ kubectl create configmap sys-app-name --from-literal name=my-system
configmap/sys-app-name created

$ kubectl create secret generic sys-app-credentials --from-literal username=bob --from-literal password=bobpwd
secret/sys-app-credentials created
```

<br />

이제 `kubernetes.yaml` 을 수정해서 각 deployment 가 바라보는 설정정보가 우리가 만든 ConfigMap, Secret 의 특정 키를 바라보도록 하자.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: system-deployment
  labels:
    app: system
spec:
  selector:
    matchLabels:
      app: system
  template:
    metadata:
      labels:
        app: system
    spec:
      containers:
      - name: system-container
        image: system:1.0-SNAPSHOT
        ports:
        - containerPort: 9080
        # Set the APP_NAME environment variable
        env:
        - name: APP_NAME
          valueFrom:
            configMapKeyRef:
              name: sys-app-name
              key: name
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: inventory-deployment
  labels:
    app: inventory
spec:
  selector:
    matchLabels:
      app: inventory
  template:
    metadata:
      labels:
        app: inventory
    spec:
      containers:
      - name: inventory-container
        image: inventory:1.0-SNAPSHOT
        ports:
        - containerPort: 9080
        # Set the SYSTEM_APP_USERNAME and SYSTEM_APP_PASSWORD environment variables
        env:
        - name: SYSTEM_APP_USERNAME
          valueFrom:
            secretKeyRef:
              name: sys-app-credentials
              key: username
        - name: SYSTEM_APP_PASSWORD
          valueFrom:
            secretKeyRef:
              name: sys-app-credentials
              key: password
...

```

<br />

자바 파일을 수정했으니, 다시 메이븐을 패키징한다.

- system 프로젝트에서는 환경변수 `APP_NAME` 을 변수 `appName` 에 주입해주었다.

  ```java
  @Inject
  @ConfigProperty(name = "APP_NAME")
  private String appName;
  ```

- inventory 프로젝트에서는 `username`, `password` 에 각각 환경변수가 세팅되도록 주입해주었다.

  ```java
  // Basic Auth Credentials
  @Inject
  @ConfigProperty(name = "SYSTEM_APP_USERNAME")
  private String username;
  
  @Inject
  @ConfigProperty(name = "SYSTEM_APP_PASSWORD")
  private String password;
  ```

자바 프로젝트 다시 패키징

```shell
$ mvn package -pl system
$ mvn package -pl inventory
```

<br />

이제 쿠버네티스를 다시 배포해보자.

아래 명령어로 실행중인 `kubernetes.yaml` (디플로이먼트, 서비스) 를 삭제하고, 다시 생성할 수 있다.

```shell
$ kubectl replace --force -f kubernetes.yaml
deployment.apps "system-deployment" deleted
deployment.apps "inventory-deployment" deleted
service "system-service" deleted
service "inventory-service" deleted
deployment.apps/system-deployment replaced
deployment.apps/inventory-deployment replaced
service/system-service replaced
service/inventory-service replaced

# 이제 파드가 종료되고 다시 생성되는 것을 계속 지켜보자. (다 Running 으로 바뀌면 Ctrl-C 로 탈출)
$ kubectl get --watch pods
NAME                                   READY   STATUS    RESTARTS   AGE
inventory-deployment-8b65548b9-zhb6j   1/1     Running   0          4m13s
kubernetes-bootcamp-fb5c67579-484x8    1/1     Running   0          48m
system-deployment-5f46847c5-szf9f      1/1     Running   0          4m13s
```

결과 확인

```shell
$ curl -# -I -u bob:bobpwd -D - http://$( minikube ip ):31000/system/properties | grep -i ^X-App-Name:

X-App-Name: my-system
X-App-Name: my-system
# 정상적으로 ConfigMap 에 세팅한 name 의 값 my-system 이 설정되었다.

$ curl http://$( minikube ip ):32000/inventory/systems/system-service
# 정상 응답이 온다. (Secret 환경변수가 잘 등록되었다.)
```

<br />

<br />

<br />

<br />

<br />