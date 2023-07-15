# Spark 5 - Spark SQL and Dataframe with External Data



## Apache Hive

- 사용자 정의 함수 (UDF, User Defined Function) 

  - UDF 는 세션별로 작동하고 메타 스토어에는 유지되지 않는다.

    ```scala
    spark.udf.register("cubed", (s: Long) -> s * s * s)
    spark.range(1, 9).createOrReplaceTempView("usf_test")
    spark.sql("SELECT id, cubed(id) AS id_cubed FROM udf_test").show()
    ```

- 스파크에서 SQL 평가 순서는 코드에 적힌 순서를 보장하지 않는다. 때문에 널처리 주의 (null 분기, null 처리)

## Spark SQL Shell, Beeline, Tableau

`$SPARK_HOME`

- Spark SQL shell : `./bin/spark-sql`

- Beeline :

  ```shell
  # 스파크 드라이버, 워커 시작
  ./sbin/start-all.sh
  
  # 쓰리프트서버(JDBC/ODBC) 시작
  ./sbin/start-thriftserver.sh
  
  # beeline
  ./bin/beeline
  !connect jdbc:hive2://localhost:10000
  ```

- Tableau : 스파크 쓰리프트서버 실행 필요

## 외부 데이터 소스

- JDBC, SQL Database
  - `./bin/spark-shell --driver-class-path $database.jar --jars $databse.jar`
  - 모든 데이터가 하나의 드라이버 연결을 통해 처리되므로 성능저하 발생 가능. 따라서 파티셔닝 중요
  - 데이터 스큐 방지위해 균일하게 분산될 수 있는 partitonColumn 선택
- PostgreSQL
  - `./bin/spark-shell --jars postgresql-42.2.6.jar`
- MySQL
  - `./bin/spark-shell --jars mysql-connector-java_8.0.16-bin.jar`
- Azure CosmosDB, MS SQL, 아파치 카산드라, 스노우플레이크, 몽고DB



<br />

## 데이터 프레임, 스파크 SQL 고차 함수

- explod(values)
- udf
- 내장 함수
- 고차 함수
  - `transform(values, value -> lambda expression)`
  - `filter`
  - `exists`
  - `reduce`
- union
- join
- 윈도우 함수 - 랭킹 함수, 분석 함수 등
- 열추가 `withColumn()`
- 열삭제 `drop()`
- 컬럼명바꾸기 `withColumnRenamed()`
- 피벗 : 로우와 컬럼을 바꿈. `PIVOT`

