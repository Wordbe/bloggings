# Spark - 2 Getting Started

# 1 다운로드, 로컬모드 실행

## 다운로드

https://spark.apache.org/downloads.html

```bash
tar -xf spark-3.4.0-bin-hadoop3.tgz
```

- bin/ : 스파크와 상호작용 하기 위한 스크립트
- sbin/ : 배포시 클러스터의 스파크 컴포넌트들을 관리 (시작, 중지 등)
- kubernetes/ : spark 2.4 부터 k8s 클러스터에서 사용하기 스파크를 사용하기 위한 Dockerfile
- data/ : MLlib, GraphX 등 입력으로 사용되는 txt 파일들



## 로컬 모드 실행

```bash
# 실행
./bin/spark-shell
scala> spark.version
res0: String = 3.4.0

# 종료
ctrl-D 
```

./bin/spark-shell 실행 오류시

```bash
sudo sh -c "echo '127.0.0.1 $(hostname)' >> /etc/hosts"
```

Web UI 주소 : http://localhost:4040 = http://192.168.35.129:4040

<br />

<br />

# 2 스파크 애플리케이션 개념, spark-shell

- 스파크 애플리케이션, 스파크 드라이버, 스파크 세션
- 잡, 스테이지, 태스크

<br />

## Spark Application and SparkSession

![](https://file.notion.so/f/s/de7aaac9-fdef-446f-8356-1f9bb53a9ee6/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-05-27_17.58.16.png?id=6ae9cf3e-5384-4a94-a0ae-7b27aa01394c&table=block&spaceId=73952c66-a666-433c-9327-81c0165106da&expirationTimestamp=1685955760843&signature=hPmRTiHqcNhQrrBAFL7oKZwf5oRE1Eg-enSEUgDAd7w&downloadName=%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2023-05-27+17.58.16.png)

- Spark Application : 스파크 사용자 프로그램
- Spark Driver : 스파크 애플리케이션 핵심 프로그램. SparkSession 객체를 미리 생성하여 제공.
- SparkSession : 스파크 코어 기능을 사용할 수 있는 진입점, API 제공. `spark` 변수로 접근 가능

<br />

## Spark Job, Stage, and Tasks

![](https://file.notion.so/f/s/598c5f56-0846-42da-8aaf-5030a2b7c01c/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-05-27_17.57.26.png?id=892fa680-0252-4e42-b98f-04111e7c0a15&table=block&spaceId=73952c66-a666-433c-9327-81c0165106da&expirationTimestamp=1685955823108&signature=nIqBLMcAK3o6InT8SKMkJo-HNrQgYZ3vbEMoJMTbGmM&downloadName=%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2023-05-27+17.57.26.png)

- Job : 드라이버가 스파크 애플리케이션을 N개의 잡으로 변환 (DAG,  Directed Acyclic Graph)

- Stage : 여러 스테이지(직렬 또는 병렬)로 나누어 스파크 연산 실행

- Task : 스파크 최소 실행 단위. 스파크 이큐제큐터들에서 연합하여 실행됨. 각 태스크는 개별 CPU 코어에 할당되어 데이터의 개별 파티션을 갖고 작업한다. (병렬 처리)

- 예제

  ```bash
  import org.apache.spark.sql.functions._
  val strings = spark.read.text("../README.md")
  val filtered = strings.filter(col("value").contains("Spark"))
  filtered.count()
  ```

<br />

## Transformation, Action, and Lazy Evaluation

- 트랜스포메이션 : 원본 데이터프레임을 수정하지 않고, 새로운 데이터 프레임으로 변형. (불변 → 데이터 내구성 향상)
  - `orderBy()`, `groupBy()`, `filter()`, `select()`, `join()`, …
  - 모든 트렌스포메이션은 지연 평가 된다. 즉시 계산되지 않고 계보(lineage) 형태로 기록된다. (장애 유연성 확보)
  - 기록된 리니지는 트렌스포메이션끼리 재배열 또는 합침으로 연산을 최적화한다. (쿼리 최적화)
  - 병렬처리 중 여러 파티션을 사용하는지 여부에 따라
    - narrow 트렌스포메이션 : filter, contains, …
    - wide 트렌스포메이션 : groupBy, orderBy, …
- 액션 : 트렌스포메이션의 지연 연산 발동시킨다. (지연 평가)
  - `show()`, `take()`, `count()`, `collect()`, `save()`, …

<br />

## Spark UI

- Web UI, 기본포트 4040

<br />

<br />

# 3 CLI 예제 실행

- gradle 처럼 scala 빌드 도구 sbt 설치

```bash
git clone <https://github.com/databricks/LearningSparkV2.git>
cd LearningSparkV2/chapter2/scala

brew install sbt
sbt clean package # build.sbt 있는 위치

/Users/jacob/workspace/spark/spark-3.4.0-bin-hadoop3/bin/spark-submit \\
--class main.scala.chapter2.MnMcount \\
target/scala-2.12/main-scala-chapter2_2.12-1.0.jar \\
data/mnm_dataset.csv

...
+-----+------+----------+
|State| Color|sum(Count)|
+-----+------+----------+
|   CA|Yellow|    100956|
|   CA| Brown|     95762|
|   CA| Green|     93505|
|   CA|   Red|     91527|
|   CA|Orange|     90311|
|   CA|  Blue|     89123|
+-----+------+----------+
```





