# Spark 7 - Spark Application Optimization and Tuning



## 스파크 설정

1. `$SPARK_HOME` 디렉토리에 여러 파일들
   - conf/spark-defaults.conf.template
   - conf/log4j.properties.template
   - conf/spark-env.sh.template
   - template 을 지우고 저장하여 새로운 설정 사용
2. spark-submit --conf
3. 스파크 애플리케이션 코드 : SparkSession.builder.conf

중복됐을 때 우선순위 3 > 2 > 1

<br />

## 대규모 워크로드를 위한 스파크 규모 확장

### 정적/동적 자원 할당

- 대규모 데이터 인입(컴퓨팅 자원 소모)에 따라 실시간으로 설정 변경

```
spark.dynamicAllocation.enabled true
spark.dynamicAllocation.minExecutors 2
spark.dynamicAllocation.schedulerBacklogTimeout 1m
spark.dynamicAllocation.maxExecutors 20
spark.dynamicAllocation.executorIdleTimeout 2min
```

- 태스크 큐가 늘어나면 1m 지날 때마다 이그제큐터가 실행하도록 요청
- 태스크를 완료하고 2m 동안 놀고 있으면 드라이버는 이그제큐터 중지

<br />

### 스파크 이그제큐터의 메모리와 셔플 서비스 설정

- executor 의 메모리, JVM GC 사용량 제어

  - 실행 메모리(60%), 저장 메모리(40%), 예비 메모리(300MB)

    ```properties
    spark.executor.memory=1g(=1GB)
    spark.memory.fraction=0.6
    
    spark.shuffle.file.buffer=32KB -> 1MB : 스파크 맵 결과를 디스크에 쓰기전에 버퍼링을 더 많이 할 수 있음
    spark.file.transferTo=true -> false : 디스크에 쓰기전에 파일버퍼를 사용하여 I/O 를 줄인다
    spark.shuffle.unsafe.file.output.buffer=32KB
    spark.io.compression.lz4.blockSize=32KB -> 512KB : 압축하는 블록 단위를 크게해서 셔플 파일의 크기를 줄인다
    spark.shuffle.service.index.cache.szie=100m
    spark.shuffle.registration.timeout=5000ms -> 120,000ms : 셔플 서비스 등록을 위한 최대 시간
    spark.shuffle.registration.maxAttempts=3 -> 5 (필요시)
    ```

- 맵, 셔플 작업에서 스파크는 로컬 디스크의 셔플 파일에 데이터를 쓰고 읽으므로 큰 I/O 발생. 

<br />

### 스파크 병렬성 최대화

- 파티션 : 병렬성의 기본 단위. 하나의 코어에서 돌아가는 하나의 스레드가 하나의 파티션을 처리
- 자원 사용을 최적화하고 병렬성을 최대로 끌어올리려면, 이그제큐터에 할당된 코어 개수만큼 파티션들이 최소한으로 할당되어야 한다.
- 이그제큐터 코어들보다 파티션이 더 많다면, 모든 코어가 더 바쁘게 돌아갈 것이다.

<br />

### 파티션

- 디스크 데이터는 조각이나 연속된 파일 블록으로 존재
  - 파일 블록은 64~128KB
  - HDFS, S3 에서 파일 블록 기본사이즈는 128KB
- 파티션 : 블록의 연속된 모음
  - `spark.sql.files.maxPartitionBytes=128KB`
  - 크기를 줄이면 '작은 파일 문제' : 작은 파티션 파일이 많아지면서 디스크 I/O 급증. 디렉토리 여닫기, 목록 조회 등으로 성능 저하
  - `repartition` 으로 파티션 개수 명시 가능
  - `spark.sql.shuffle.partitions=50` : 셔플 단계에서 만들어지는 셔플 파티션의 개수 지정. 너무 작은 파티션들이 이그제큐터들에게 할당되지 않도록 조정

<br />

## 데이터 캐싱과 영속화

- 스파크에서는 데이터 캐싱과 영속화가 같음. `cache()`, `persist()`
  - 영속화는 storage level 을 정의 가능

<br />

## 스파크 조인

- 트렌스포메이션 연산 형태. 일치하는 키를 기준으로 병합하는 연산
- 셔플. (shuffle): spark executor 사이에 방대한 데이터 이동을 일으킴. 데이터 생성, 특정 키와 관련된 데이터를 디스크에 쓰고, 특정 키와 데이터를 groupBy(), join(), sortBy(), redcueByKey() 같은 작업들의 일부가 되는 노드들에 옮김
- 이그제큐터 간 데이터를 교환, 이동, 정렬, 그룹화, 병ㄹ합하는 다섯 종류의 조인 전략을 가짐.
  - Broadcast Hash Join
  - Shuffle Hash Join
  - Shuffle Sort Merge Join
  - Broadcast Nested Loop Join
  - Cartesian Product Join

### 브로드캐스트 해시 조인, BHJ

- 맵사이드 조인
- 데이터 이동이 거의 필요 없도록 한쪽은 작고, 다른 쪽은 큰 두 종류의 데이터를 사용하여 조인
- 기본적으로 스파크는 작은 쪽 데이터가 10MB 이하일 때 브로드캐스트 조인을 사용한다. : `spark.sql.autoBroadcastJoinThreshold`
- 셔플이 일어나지 않기 때문에 스파크가 제공하는 가장 쉽고 빠른 조인
  - 브로드캐스팅 후 이그제큐터에 필요한 모든 데이터는 로컬 메모리에서 접근 가능해진다.
- 언제 사용?
  - 양쪽 데이터세트의 각 키가 스파크에서 동일한 파티션 안에 해시될 때
  - 한 데이터가 다른 쪽보다 많이 작은 경우. 10MB 기본 설정, 메모리에 데이터가 충분히 들어갈 수 있을 때
  - 정렬되지 않은 키들 기준으로 두 데이터를 결합하면서 동등 조인을 수행할 때
  - 더 작은 쪽의 데이터가 모든 스파크 이그제큐터에 브로드캐스트될 때 발생하는 과도한 네트워크 대역폭이나 OOM 오류에 걱정할 필요가 없는 경우

### 셔플 소트 머지 조인

- `spark.sql.autoBroadcastJoinThreshold=-1` 일 경우
- spark.sql.join.preferSortMergeJoin 설정으로 활성화. spark 2.3 부터 기본값
- exchange = shuffle (이그제큐터들 간 네트워크상으로 파티션들이 셔플되어야 한다)
- 공통 파티션에 저장 가능한 공통 키를 기반으로, 큰 두 종류의 데이터세트를 합칠 수 있는 효과적인 방법
  - 동일 키를 가진 데이터세트의 모든 레코드가 동일 이그제큐터의 동일 파티션에 존재한다.
- 버켓팅(`bucketBy`)을 사용하면, 미리 저장해두므로 셔플 과정을 생략할 수 있다.
  - 특정 정렬 컬럼을 저장할 명시적 개수의 버킷을 만들 수 있다. (버킷당 키 하나)
  - Exchange 를 생략하고 바로 WholeStageCodegen 으로 넘어가므로 성능을 올릴 수 있다.
- 언제 사용?
  - 두 큰 데이터세트의 각 키가 정렬 및 해시되어 스파크에서 동일 파티션에 위치할 수 있을 때
  - 동일 조건 조인만 수행하고, 정렬된 동일 키 기반으로 두 데이터세트를 조합하기 원할 때
  - bucket 사용 : 네트워크 간 규모가 큰 셔플을 일으키는 Exchange, Sort 연산을 피하고 싶을 때



- `spark.sql.Shuffle=300`
- 스파크 셔플 파틴 최적화 : 쿼리 > 파티션 수 > 코어당 메모리 수

<br />

## 스파크 UI

- Jobs
- Stage
- Executors
- Storage
- SQL
- Enviornment
- 스파크 애플리케이션 디버깅



- 왠만한 경우 spark 내부적으로 알아서 최적화해서 성능을 고려할 요소가 크게 있지 않았다.
- 셔플도 꼭 필요한 셔플이라면 감안한다.
- 셔플 비용보다 데이터 skew(치우침) 관련한 부분이 있다면 해결하는 것이 좋다.
- conf 설정을 바꾸는 것 보다 내부 쿼리 최적화, 데이터 처리구조를 고민하는 것이 좋다.