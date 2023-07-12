# Spark 3 - Structured APIs



# 스파크 RDD 

- RDD (Resilient Distributed Dataset)
- 특징
  - 의존성 - 현재 RDD가 만들어지는 과정을 스파크에게 알려주는 의존성이 필요하다.
  - 파티션 - 작업을 나눠 executor 들에 분산해 파티션별로 병렬 연산할 수 있다.
  - 연산 함수 ( `Partition => Iterator[T]` ) : RDD 에 저장되는 데이터를 이터레이터로 만들어주는 연산 함수를 가진다.
- 장점
  - 표현성, 단순성, 구성용이성, 통일성
  - 가독성좋고 명료한 API 제공. 원하면 자유도 넓은 하위 API 사용 가능
- 단점
  - 가독성, 투명성이 떨어진다.
    - 투명성 저하 : 로직을 새로 만들다보면 버그가 생기고 일관되지 못함 (잘 작성해놓은 고수준의 API 를 사용하는 것이 좋음)

<br />

# 스키마와 DataFrame API

- 스키마는 미리 정하는 것이 좋다. (그렇지 않으면 실행시 스키마를 추론하는데 자원이 든다.)
- 생성방법 - 프로그래밍 vs DDL (가독성 좋아 추천)
- pandas 에 영향을 받아 문법이 비슷해서 쉬움

## 컬럼

```scala
// 둘은 동일
df.sort(col("Id").desc).show()
df.sort($"Id".desc).show()
```

## 로우

## DataFrameReader, DataFrameWriter

- 스키마 미리 지정안하고 효율적으로 사용하려면, 샘플링을 통해 스키마를 추론하도록 한다.

```scala
val sampleDf = spark.read
	.option("sampllingRatio", 0.001)
	.option("header", true)
	.csv("/path/to/original.csv")
```

- 기본 포맷 : parquet (파케이, 인기있는 컬럼 지향 포맷)
  - 데이터 프레임이 파케이로 쓰여졌다면, 파케이 메타데이터 일부로 스키마가 보존될 수 있다. 이 경우 스키마를 수동으로 적용할 필요없다.
- 기본 데이터 압축 : snappy 압축

<br />

## 집계 연산

```scala
fireTsDF
	.select("CallType")
	.where(col("CallType"))
	.groupBy("CallType")
	.count()
	.orderBy(desc("count"))
	.show(10, false)
```

<br />

# DataSet API

- 데이터세트는 정적타입(typed) API 와 동적타입(untyped) API 두 특성을 모두 가진다.
  - 정적타입 : DataSet[T]
  - 동적타입 : DataFrame = DataSet[Row]
  - 데이터세트는 엄격하게 타입이 정해진 (정적타입, 자바에서는 클래스) JVM 객체 집합이다. 이와 동시에 데이터 프레임이라 불리는 동적 타입 뷰를 가진다.
    - 스칼라는 데이터셋, 데이터프레임 모두 가능 (정적, 동적타입)
    - 자바는 데이터셋 (정적타입)
  - 데이터프레임은 `Dataset[Row]` 이고 Row 타입의 데이터세트이다. Row 는 동적으로 여러 타입이 할당가능하다.
    - 파이썬과 R 은 데이터프레임만 사용 가능하다. (정적타입언어가 아니기 때문)

## 데이터세트 생성

- `case class` 를 만들어서 스키마를 미리 저장해놓으면 좋다.

  ```scala
  case class Person(id: Long, name: String, age: Int, createdAt: LocalDateTime)
  ```

  ```scala
  ds
  	.filter { d => d.temp > 25 }
  	.map { d => (d.temp, d.device_name, d.device_id) }
  	.toDf("temp", "device_name", "device_id")
  	.as[DeviceTempByCountry]
  ```

  - map() 은 select() 과 같은 일을 함
  - filter() 는 where() 과 같은 일을 함

- 데이터세트가 사용되는 동안 스파크 SQL 엔진은 JVM 객체의 생성, 변환, 직렬/역직렬화를 담당한다.
- 데이터세트 인코더의 도움을 받아 자바의 오프힙 메모리 관리한다. 성능이 빠르다.

<br />

## DataFrame vs. DataSet

- SQL 과 유사한 질의를 쓰는 관계형 변환 주로 한다면 DataFrame 사용

- 엄격한 타입체크를 원하고, 여러 케이스클래스(스키마) 만드는 부담이 없다면 DataSet

- **정형화 API 사용시 오류가 발견되는 시점**

  |           | SQL       | DataFrame   | DataSet     |
  | --------- | --------- | ----------- | ----------- |
  | 문법 오류 | 실행 시점 | 컴파일 시점 | 컴파일 시점 |
  | 분석 오류 | 실행 시점 | 실행 시점   | 컴파일 시점 |

<br />

## RDD

- 코드 최적화, 시간/공간복잡도 효율, 성능 튜닝할 때 사용
- 스파크가 어떻게 질의할지 정확하게 지정해주고 싶을 때 사용
- `df.rdd`
- DataFrame, DataSet 은 결국 RDD 로 분해되어 실행됨

<br />

# 스파크 SQL 과 하부의 엔진

- 언어 상관없이 최적 쿼리를 자바 바이트코드로 변환하여 각 물리머신에서 실행
- 카탈리스트 옵티마이저
  1. 분석
     - 스파크 SQL 엔진은 SQL, 데이터 프레임 쿼리를 위한 추상 문법 트리(AST, abstract syntax tree) 생성
  2. 논리 최적화
     - 내부 표준 규칙 기반으로 여러 계획 수립
     - 비용 기반 옵티마이저 (CBO, cost based optimizer) 로 비용 책정하여 선별
  3. 물리 계획 수립
     - 최적화 물리 계획 수립
  4. 코드 생성
     - 각 머신에서 실행할 효율적인 자바 바이트 코드 생성
     - 최신 컴파일러 기술 사용. 스파크는 컴파일러처럼 작동. 포괄(whole-stage)코드를 생성하는 텅스텐



