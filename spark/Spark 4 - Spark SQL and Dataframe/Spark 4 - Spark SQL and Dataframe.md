# Spark 4 - Spark SQL and Dataframe



```scala
spark.sql("CREATE DATABASE mydb")
val tableDF = spark.sql("SELECT * FROM table")
```



## SQL 테이블과 뷰

- 스파크는 아파치 하이브 메타스토어를 사용해서 테이블의 메타데이터를 관리한다. `/user/hive/warehouse`

- 관리형 테이블 : 메타데이터와 데이터를 모두 관리. 파일 저장소(HDFS, S3 등)

  ```sql
  CREATE TABLE managed my_table (
  	date STRING,
  	delay INT, ...
  )
  ```

  

- 비관리형 테이블 : 메타데이터만 관리. 외부 데이터 소스(카산드라 등)에서 데이터 직접 관리

  ```sql
  CREATE TABLE my_table(
  	date STRING,
  	delay INT, ...
  )
  USING csv OPTIONS (PATH '/csv/path.csv')
  ```

- 뷰 : 전역(클러스터 전체) 또는 세션(SparkSession) 안에서 일시적으로 뷰 생성. 스파크 애플리케이션 종료되면 사라짐

  - 전역 임시 데이터베이스 : `global_temp`



### 메타데이터

- 스파크 SQL 의 상위 추상화 모듈인 카탈로그에 저장된다.

  ```
  spark.catalog.listDatabases()
  spark.catalog.listTables()
  spark.catalog.listColumns("my_table")
  ```

### SQL 테이블 캐싱하기

- SQL 테이블, 뷰는 캐시, 언캐싱이 가능하다.

  ```
  CACHE [LAZY] TABLE <table-name>
  UNCACHE TABLE <table-name>
  ```

<br />

## 데이터 프레임과 SQL 테이블

- DataFrameReader : 데이터 읽기

  ```
  DataFrameReader.format(args).option("key", "value").schema(args).load()
  ```

  ```scala
  // 인스턴스를 얻으려면 spark.read
  
  val file = "/path/example.parquet"
  val df = spark.read.foramt("parquet").load(file)
  ```

- DataFrameWriter

  ```
  DataFrameWriter.foramt(args).option(args).bucketBy(args).partitionBy(args).save(path)
  ```

  ```scala
  df.write
    .format("json")
    .mode("overwrite")
  	.option("mode", "FAILFAST")
    .option("compression", "snappy")
    .save(location)
  ```

  ```
  _SUCCESS
  part-00000-<...>-c000.json
  ```

  

### Parquet

- 파케이 파일형식은 I/O 최적화 (저장공간 절약, 컬럼에 대한 빠른 접근이 가능한 압축) 를 제공한다.
- 데이터 파일, 메타데이터, 여러 압축 파일, 상태 파일 포함
- 스파크에서 선호되는 기본 내장 데이터 소스 파일 형식

### JSON

- JavaScript Object Notation

### CSV

### Avro

- 에이브로는 특히 아파치 카프카에서 메세지를 직/역직렬화 할 때 사용된다.
- JSON 에 대한 직접 매핑, 속도와 효율성을 제공한다.

### ORC

- 벡터화된 ORC 리더 : 한 번에 한 행이 아닌 행 블록(블록당 1,024개 행)을 읽어 작업을 간소화한다. 검색, 필터, 집계, 조인 등 작업에 대한 CPU 사용량을 줄인다.

  ```
  spark.sql.orc.imple: native
  spark.sql.orc.enableVectorizedReader: true
  ```

### Image

- 텐서플로, 파이토치 같은 DL 프레임워크 지원. 이 중 이미지 로드, 쓰기

### Binary file

- DataFrameReader 는 각 이진 파일의 원본 내용과 메타데이터를 포함하는 단일 데이터 프레임 행으로 변환한다.
  - "binaryFile"
  - 경로, 수정시간, 길이, 내용

