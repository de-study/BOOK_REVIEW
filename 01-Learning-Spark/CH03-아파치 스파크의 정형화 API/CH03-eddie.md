# INTRODUCTION
- 데이터 프레임이 나오게된 배경  
- 스파크 SQL엔진
<br><br>

# 01.RDD 
## (1)발생배경

### Hadoop-MapReduce의 한계 :
- Hadoop(하둡) : 분산 시스템 프레임워크, 2006년 Yahoo가 Google 논문 보고 만든 오픈소스
  - 구성요소
    - HDFS (Hadoop Distributed File System)
    - YARN (Yet Another Resource Negotiator) : 클러스터 리소스 관리
    - MapReduce 엔진
- MapReduce(맵리듀스): 데이터를 분산 처리하는 방식, 하둡에서 데이터 처리 엔진, 2004년 Google 논문에서 발표한 분산 처리 패러다임
  - Map 함수: 데이터를 key-value 쌍으로 변환
  - Reduce 함수: 같은 key의 value들을 집계
  - Spark에서도 map(), reduce() 쓰면 MapReduce 패러다임 사용함  
- MapReduce 예시
```
입력: 문서들

Map: (문서) → (단어, 1) 쌍들 생성
Shuffle: 같은 단어끼리 모음
Reduce: (단어, [1,1,1...]) → (단어, 합계)

출력: 단어별 빈도수
```
- 한계점(성능)
  - 매 작업마다 디스크 I/O 발생 (Map 결과를 HDFS에 쓰고, Reduce가 읽음)
  - 반복 작업(머신러닝, 그래프 처리)에서 엄청난 성능 저하
  - 메모리 사용안함 -> 중간 결과를 메모리에 캐싱할 방법이 없음
  - 중간에 뻑나면 끝. 다시 돌려야함  
<br><br>

### Spark-RDD-DataFrame까지 :   
> 하둡의 처리방식 엔진인 맵리듀스의 한계를 극복하기위해 spark가 나옴.  
> 하둡의 버전업이라고하기엔 처리 엔진이 완전히 달라졌으므로 새로운 프레임워크 명칭을 달고 나온듯   
> 하지만 하둡과의 호환성을 위해 기존 HDFS,Yarn을 그대로 사용가능하게 만듬

<details>
  <summary>Dataframe의 배경</summary>

  - Dataframe의 시작
  ```   
  R의 data.frame (1990년대) → Pandas DataFrame (2008) → Spark DataFrame (2014)
  ```
  - Dataframe을 사용하는 이유
  ```
  - 행렬(matrix)과 달리 컬럼마다 다른 타입 가능
  - 통계 분석에 최적화된 구조
  - SQL처럼 subset, merge, aggregate 가능
  ```
  끝~
  ---
  <br><br> 
  
</details>




- Spark의 해결책
  - HDFS 그대로 사용 (검증된 분산 파일시스템)
  - YARN으로 리소스 공유 (MapReduce와 Spark 동시 실행 가능)
  - MapReduce 엔진만 교체 (Spark 엔진으로) 
<br> 

**정리하자면,**
- Spark가 Hadoop에서 가져온 것
  - HDFS (분산 파일 저장)
  - YARN (리소스 관리, 선택사항)

- Spark가 버린 것
  - MapReduce 실행 엔진
  - 디스크 기반 shuffle
  - Job 단위 실행 모델

- Spark가 새로 만든 것
  - RDD 추상화 -> Dataframe으로 상위 API로 감싸서 업그레이드 됨   
  - DAG 기반 실행 엔진
  - 메모리 기반 처리
  - 통합 애플리케이션 모델
```
┌─────────────────┬──────────────────┐
│   Hadoop        │     Spark        │
├─────────────────┼──────────────────┤
│ HDFS            │ (HDFS 그대로)      │
│ YARN            │ (YARN 선택사용)    │
│ MapReduce 엔진   │ Spark Core 엔진   │
└─────────────────┴──────────────────┘
```
## (2) RDD의미
```
RDD
- Resilient (회복력) : 파티션이 손실되면 lineage(계보) 정보로 재계산
- Distributed (분산) : 데이터가 여러 파티션으로 나뉘어 클러스터 노드들에 분산
- Dataset (데이터셋) : 읽기 전용(immutable) 컬렉션
```
- Spark의 가장 기본적인 데이터 추상화 단위.
- 불변(immutable)한 분산 객체 컬렉션 
- 각 RDD는 여러 개의 파티션으로 나뉘며, 이 파티션들은 클러스터의 서로 다른 노드에서 계산될 수 있다.
- RDD는 Python, Java, Scala의 어떤 타입의 객체라도 담을 수 있다.

### SparkContext vs SparkSession
**2010년 초기 Spark**
- SparkContext만 존재
- RDD만 사용
- DataFrame 없음

**2014년 이후**
- SparkSession 추가 (DataFrame용)
- SparkContext는 SparkSession 내부에 포함
- 하위 호환성 유지
  
**RDD 사용 방법1:**
```
from pyspark import SparkContext, SparkConf

# SparkContext 직접 생성
conf = SparkConf().setAppName("RDD Only").setMaster("local[*]")
sc = SparkContext(conf=conf)

# RDD 작업
rdd = sc.parallelize([1, 2, 3, 4, 5])
result = rdd.map(lambda x: x * 2).collect()
print(result)

sc.stop()
```
주의
- SparkContext는 프로세스당 1개만 가능  
- 중복 생성하면 에러
<br>

**RDD 사용 방법2:**
```
from pyspark.sql import SparkSession

# SparkSession 생성
spark = SparkSession.builder \
    .master("local[*]") \
    .getOrCreate()

# SparkContext 추출
sc = spark.sparkContext

# RDD만 사용
rdd = sc.parallelize([1, 2, 3, 4, 5])
result = rdd.map(lambda x: x * 2).collect()
print(result)

spark.stop()
```
## (3) RDD한계
**RDD Lazy Evaluation**
```
- 최적화 아님
- 실행 지연으로 불필요한 계산 회피
- 조기 종료로 데이터 덜 읽기
- Lineage로 fault tolerance
```
**한계**
```
- 블랙박스 함수라 내부 모름
- 연산 순서 그대로 실행
- 사용자가 직접 최적화 필요
```
<br>

**DataFrame의 차이**
```
Lazy + Catalyst 최적화
쿼리 분석해서 자동 재배치
10~100배 빠름
```
## (4) RDD특징
- RDD : Spark 1.0부터 스파크에 도입된 가장 기초적인 데이터구조
- RDD의 세 가지 핵심 특징
    - 의존성(dependency) : 어떤 입력이 필요하고 생성되는 RDD가 어떻게 만들어지는지에 대한 정보
    - 파티션(partition)(지역성 정보 포함) : 작업을 나누어 이그제큐더들에 분산해 파티션별로 병렬 연산할 수 있는 능력 부여. 만약 파일을 읽는 경우 각 이그제큐터가 가까이 있는 데이터를 처리할 수 있는 이그제큐터에게 우선적으로 작업을 보냄.
    - 연산 함수(compute function) : 저장된 데이터를 Iterator[T] 형태로 만들어 줌.

## (5) RDD와 데이터프레임 실행
- RDD 파이썬예제(저수준)
```
sc = spark.sparkContext
# (name, age) 형태의 튜플로 된 RDD 생성
dataRDD = sc.parallelize([("Brooke", 20), ("Denny", 31), ("Jules", 30), ("TD", 35), ("Brooke", 25)])
# 집계와 평균을 위한 람다 표현식, map, reduceByKey transformation
ageRDD = (dataRDD
          .map(lambda x: (x[0], (x[1], 1)))
          .reduceByKey(lambda x, y: (x[0] + y[0], x[1] + y[1]))
          .map(lambda x: (x[0], x[1][0] / x[1][1]))
          )
print(ageRDD.collect())
```
<br>

- DataFrame 예제(고수준)
```
data_df = spark.createDataFrame([("Brooke", 20), ("Denny", 31), ("Jules", 30), ("TD", 35), ("Brooke", 25)], ['name', 'age'])
avg_df = data_df.groupBy('name').agg(avg('age'))
avg_df.show()
```
---
<br><br><br>

# 02. 데이터프레임 API

## (1) 데이터프레임
```
  R의 data.frame (1990년대) → Pandas DataFrame (2008) → Spark DataFrame (2014)
```
- 분산 인메모리 테이블처럼 동작  
- 컬럼별 데이터 타입 지정 가능 → RDD는 타입정보 없었음.  
- 불변성 - 원본유지 ex) replace=TRUE

## (2) 구조
### 구성요소
- 스키마  
- 데이터  
```
스키마:
- name: string
- age: int
- job: string

데이터:
Row(name='Alice', age=25, job='Engineer')
Row(name='Bob', age=30, job='Manager')
```
### 생성방법  
- 방법 1: 리스트에서 생성
```
data = [
    ("Brooke", 20),
    ("Denny", 31)
]

df = spark.createDataFrame(data, ["name", "age"])
```
- 방법 2: 파일에서 읽기
```
# CSV
df = spark.read.csv("data.csv", header=True, inferSchema=True)

# Parquet
df = spark.read.parquet("data.parquet")

# JSON
df = spark.read.json("data.json")
```
-  Pandas에서 변환
```
import pandas as pd

pandas_df = pd.DataFrame({
    'name': ['Alice', 'Bob'],
    'age': [25, 30]
})

spark_df = spark.createDataFrame(pandas_df)
```
### 주요 DataFrame API 구성
```
생성: createDataFrame, read.csv, read.parquet
조회: select, show, printSchema
필터: filter, where
변환: withColumn, withColumnRenamed, drop
집계: groupBy, agg, avg, sum, count
정렬: orderBy
결합: join
SQL: createOrReplaceTempView, sql
저장: write.csv, write.parquet
```
<br><br>

## (2)데이터 타입
- Spark는 자체 타입 시스템 보유
- Python/Java 타입을 Spark 타입으로 변환해서 처리  

### 1단계: type 시스템 개요

**문제상황**
> "각 언어마다 타입이 다르잖아. Spark는 이 모든 언어를 지원해야 하는데, 어떻게 통일하지?"
```
Python : int, str, list
Java   : Integer, String, ArrayList
Scala  : Int, String, List
```

**해결책**
> "아, Spark 자체 타입 시스템을 만들어서 중간에서 변환하는 거네."  
> "JVM에서 실행되니까 결국 Java 타입 기반이긴 한데, 각 언어에서 쉽게 쓸 수 있게 래핑한 거야."
- Spark 자체 타입 정의
- 모든 언어를 Spark 타입으로 통일
- JVM에서 실행되므로 Java 기반

**스키마선언 방법**
- 0) 스키마 선언
	- ```
	  from pyspark.sql.types import *

		schema = StructType([
		    StructField("user_id", IntegerType(), False),
		    StructField("is_active", BooleanType(), True),
		    StructField("created_at", DateType(), True)
		])

	  ```
- 1) DataFrame 생성 시 직접 지정
	- ```
	  df = spark.createDataFrame(data, schema)
	  ```
- 2) 파일 읽을 때 지정 (실무 핵심)
	- ```
	  df = spark.read \
			    .schema(schema) \
			    .csv("s3://bucket/users.csv")
	  ```
- 3) 테이블에서 상속 (가장 안정적)
	- ```
	df = spark.read.table("analytics.users")
	  ```
- 기본적으로 parquet에 타입이 설정되어있어서(강제) 지정안해도됨.  

### 2단계: 기본타입 및 설정방법

**1) 숫자타입**

```
from pyspark.sql.types import *

# 정수
ByteType()      # 8-bit 정수 (-128 ~ 127)
ShortType()     # 16-bit 정수 (-32768 ~ 32767)
✅IntegerType()   # 32-bit 정수 (일반적 사용)
LongType()      # 64-bit 정수 (큰 수)

# 실수
FloatType()     # 32-bit 부동소수점
DoubleType()    # 64-bit 부동소수점 (권장)

# 고정소수점
DecimalType(precision, scale)  # 금융 계산용
# DecimalType(10, 2) → 12345678.90
```

- 설정팁
```
# 정수
- ByteType, ShortType → 거의 안 씀. 메모리 극한 최적화할 때만.
- IntegerType → 일반적인 숫자 (나이, 개수)
- LongType → 큰 숫자 (사용자 ID, timestamp)

# 실수
"FloatType vs DoubleType는... 
- 과학 계산이면 DoubleType이 정밀도 높아서 이거 쓰고."

# 고정소수점
- 나쁜 예: 금액을 DoubleType으로
	amount = DoubleType()  # 999.99 + 0.01 = 1000.0000000001 (버그!)

- 좋은 예: 금액을 DecimalType으로  
	amount = DecimalType(12, 2)  # 정확히 1000.00
```

**2) 문자열/날짜**

```
StringType()    # 문자열, UTF-8 문자열 (무제한 길이)
BinaryType()    # 바이트 배열

BooleanType()   # True/False

DateType()      # 날짜만 (2025-01-30)
TimestampType() # 날짜+시간 (2025-01-30 15:30:00)
```

- 설정팁
```
# 날짜
"DateType vs TimestampType
- 생일 같은 건 Date
- 로그 분석할 때 시간까지 필요하면 Timestamp 
```

**3) 복합 타입**
- ArrayType (배열)
	- "오, array 안에 element 타입이 정의되네. Python list랑 비슷한데 타입이 명시되어 있어."
```
from pyspark.sql.types import ArrayType, IntegerType

# 정수 배열
ArrayType(IntegerType())

# 문자열 배열
ArrayType(StringType())

data = [
    (1, [101, 102, 103]),  # user_id: 1, 구매한 product_ids
    (2, [201])
]

df = spark.createDataFrame(data, ["user_id", "product_ids"])

df.printSchema()
# root
#  |-- user_id: long
#  |-- product_ids: array
#      |-- element: long

```
- MapType (딕셔너리)
	- "실무에서 언제 쓰나 생각해보면... 설정 값이나 메타데이터 저장할 때 유용하겠네."
```
from pyspark.sql.types import MapType

# 문자열 → 정수 맵
MapType(StringType(), IntegerType())



data = [
    (1, {"math": 90, "english": 85}),
    (2, {"math": 75, "english": 95})
]

df = spark.createDataFrame(data, ["id", "scores"])

df.printSchema()
# root
#  |-- id: long
#  |-- scores: map
#      |-- key: string
#      |-- value: long
```
- StructType (중첩 구조)
	- "address가 또 StructType이네. 중첩 구조. JSON이랑 완전 똑같은 구조야."
```
from pyspark.sql.types import StructType, StructField

# 스키마 정의
schema = StructType([
    StructField("name", StringType(), nullable=False),
    StructField("age", IntegerType(), nullable=True),
    StructField("address", StructType([
        StructField("city", StringType()),
        StructField("zip", StringType())
    ]))
])


data = [
    ("Alice", 25, {"city": "Seoul", "zip": "12345"}),
    ("Bob", 30, {"city": "Busan", "zip": "67890"})
]

df = spark.createDataFrame(data, schema)

df.printSchema()
# root
#  |-- name: string (nullable = false)
#  |-- age: integer (nullable = true)
#  |-- address: struct
#      |-- city: string
#      |-- zip: string
```

### 3단계 스키마 정의방법

**방법1.자동추론**
- 항상 정확하지 않음
- 성능 오버헤드 (데이터 스캔 필요)
```
data = [("Alice", 25), ("Bob", 30)]

# Spark가 자동 타입 추론
df = spark.createDataFrame(data, ["name", "age"])

df.printSchema()
# root
#  |-- name: string
#  |-- age: long (자동으로 long 선택)
```
**방법 2: DDL 문자열**
- 간단한 스키마
- "오, SQL 스타일이라 읽기 편하네."
- "근데 nullable 제어가 안 되네... 그럼 복잡한 건 못 쓰겠어."
```
# SQL DDL 스타일
schema_ddl = "name STRING, age INT, salary DOUBLE"

df = spark.createDataFrame(data, schema_ddl)
```
**방법 3: StructType 명시**
- "이게 제일 명확하고 안전해."
- 타입 완전 제어
- nullable 지정 가능
- 프로덕션 권장
- "user_id가 NULL이면 안 되잖아. 이런 거 명시해야 나중에 데이터 품질 문제 안 생겨."
```
from pyspark.sql.types import *

schema = StructType([
    StructField("name", StringType(), nullable=False), # NOT NULL
    StructField("age", IntegerType(), nullable=True),
    StructField("salary", DoubleType(), nullable=True)
])

df = spark.createDataFrame(data, schema)

```

**스키마선언 사용방법**
- 0) 스키마 선언 - 방법3
	- ```
	  from pyspark.sql.types import *

		schema = StructType([
		    StructField("user_id", IntegerType(), False),
		    StructField("is_active", BooleanType(), True),
		    StructField("created_at", DateType(), True)
		])

	  ```
- 1) DataFrame 생성 시 직접 지정
	- ```
	  df = spark.createDataFrame(data, schema)
	  ```
- 2) 파일 읽을 때 지정 (실무 핵심)
	- ```
	  df = spark.read \
			    .schema(schema) \
			    .csv("s3://bucket/users.csv")
	  ```
- 3) 테이블에서 상속 (가장 안정적)
	- ```
	df = spark.read.table("analytics.users")
	  ```
- 기본적으로 parquet에 타입이 설정되어있어서(강제) 지정안해도됨.  


### 4단계 타입변환 
> "CSV 읽으면 전부 문자열로 오는데, 어떻게 숫자로 바꾸지?"

- 캐스팅 예시
```
from pyspark.sql.functions import col

df = spark.createDataFrame([
    ("1", "100"),
    ("2", "200")
], ["id", "amount"])

# 문자열 → 정수
df = df.withColumn("id", col("id").cast(IntegerType()))
df = df.withColumn("amount", col("amount").cast(DoubleType()))

# 또는 문자열로
df = df.withColumn("id", col("id").cast("int"))
df = df.withColumn("amount", col("amount").cast("double"))

df.printSchema()
# root
#  |-- id: integer
#  |-- amount: double
```


- cast실패하면 null처리됨
	- "abc" → IntegerType 캐스팅 → NULL
	- "NULL로 변하네. 조용히 실패하니까 확인해야겠어."
```
# 변환 후 검증 
df.filter(col("age").cast("int").isNotNull())
```

- 관련 코드
```
# 구조 확인
df.printSchema()

# 스키마 객체 가져오기
schema = df.schema
print(schema)

# 특정 컬럼 타입
print(df.schema["age"].dataType)  # IntegerType


# 숫자형 컬럼만
numeric_cols = [f.name for f in df.schema.fields 
                if isinstance(f.dataType, (IntegerType, DoubleType, LongType))]

# 문자열 컬럼만
string_cols = [f.name for f in df.schema.fields 
               if isinstance(f.dataType, StringType)]
```

### 5단계 파이썬 매핑 주의사항

**Python vs Spark 타입 매핑**
- "Python 데이터를 createDataFrame하면 자동 변환되는데..."
```
Python          → Spark
------------------------------
None            → NULL
bool            → BooleanType
int             → LongType (주의: IntegerType 아님)
float           → DoubleType
str             → StringType
bytes           → BinaryType
datetime.date   → DateType
datetime.datetime → TimestampType
list            → ArrayType
dict            → MapType
```


**Python(int) → LongType**
- "어? int가 LongType이네? IntegerType 아니고?"
- "왜 long으로 추론하지? 아, Python int는 크기 제한 없어서 안전하게 LongType으로 매핑하는구나."
- "IntegerType 원하면 명시해야 해."
```
data = [(1, 2), (3, 4)]
df = spark.createDataFrame(data, ["a", "b"])

df.printSchema()
# root
#  |-- a: long (int 아님!)
#  |-- b: long

# 명시적으로 지정해주기
schema = StructType([
    StructField("a", IntegerType()),
    StructField("b", IntegerType())
])
```


### 6단계 null 처리 주의사항

**nullable False여도 에러안남**
- "nullable=False로 하면 NULL 들어오면 에러나나?"
- "아니, 경고만 하고 받아들여... Spark가 관대해. 그래서 더 위험해."
- "검증하는 출력문 작성해야해"

```
StructField("user_id", IntegerType(), nullable=False)


## 읽을 때 검증
df = spark.read.csv("data.csv", schema=schema)

# NULL 체크
null_count = df.filter(col("user_id").isNull()).count()
if null_count > 0:
    raise ValueError(f"user_id에 NULL {null_count}개 발견!")

```

**Null처리함수**
- "실무에선 fillna보다는 명시적으로 처리하는 게 나아."
```
# NULL 필터
df.filter(col("age").isNotNull())

# NULL 제거
df.dropna(subset=["age"])

# NULL 채우기
df.fillna({"age": 0, "name": "Unknown"})


# 명시적처리
from pyspark.sql.functions import when, col

df = df.withColumn("age_clean",
    when(col("age").isNull(), 0)
    .otherwise(col("age"))
)
```

### 7단계 - 복잡한 타입 다루기

**배열 컬럼 접근**
- "explode... 배열 한 개가 여러 행으로 펼쳐지네."
- "로그 분석할 때 유용하겠어. 한 사용자가 여러 이벤트 발생시킨 거 펼칠 때."
```
from pyspark.sql.functions import explode, col

data = [(1, ["A", "B", "C"])]
df = spark.createDataFrame(data, ["id", "items"])

# 배열 분해 (각 요소를 행으로)
df.select(col("id"), explode(col("items")).alias("item")).show()
# +---+----+
# | id|item|
# +---+----+
# |  1|   A|
# |  1|   B|
# |  1|   C|
# +---+----+

# 배열 인덱스 접근
df.select(col("items")[0].alias("first_item")).show()
```
**Map 컬럼 접근**
```
data = [(1, {"math": 90, "eng": 85})]
df = spark.createDataFrame(data, ["id", "scores"])

# 키로 접근
df.select(col("scores")["math"]).show()
```
**Struct 컬럼 접근**
- "JSON 경로 탐색하는 것처럼 점 찍어서 접근. 직관적이야."
```
# 점 표기법
df.select(col("address.city"), col("address.zip"))

# getField
df.select(col("address").getField("city"))
```

---

### 자주 쓰이는 데이터 프레임 작업들

**DataFrameReader와 DataFrameWriter**
- 다양한 데이터 소스(Json,csv,parquet,avro,orc) 읽기 → DataFrameReader로 통합
- 다양한 데이터 소스(Json,csv,parquet,avro,orc) 읽기 → DataFrameWriter로 통합

```python
# 데이터 불러오기 예제 - 샌프란시스코 소방서 데이터
fire_df = spark.read.csv('sf-fire-calls.csv', header=True)
fire_df.show(5)
```

**프로젝션과 필터**

- **프로젝션(projection)** : 특정 관계 상태와 매치되는 행들만 되돌려 주는 방법
    - .select() 뒤에 .where() or .filter() 를 이용하여 사용.

```python
# CallType이 "Medical Incident"가 아닌 데이터 필터
from pyspark.sql.functions import col
few_fire_df = (fire_df
               .select("IncidentNumber", "AvailableDtTM", "CallType")
               .where(col("CallType") != "Medical Incident"))
few_fire_df.show(5, truncate=False)
```

```python
# CallType의 종류가 몇 가지인지 세어주는 코드
(fire_df.select("CallType")
 .where(col("CallType").isNotNull())
 .agg(countDistinct("CallType").alias("DistinctCallTypes"))
 .show()
 )
```

```python
# 모든 행에서 null이 아닌 개별 CallType을 추출
(fire_df
 .select("CallType")
 .where(col("CallType").isNotNull())
 .distinct()
 .show(10, False)
 )
```

**칼럼의 이름 변경 및 추가 삭제**

- withColumnRenamed()

```python
new_fire_df = fire_df.withColumnRenamed("Delay", "changeDelay")
new_fire_df.select("changeDelay").show(5)
```

<aside>
💡 데이터 프레임 변형은 변경 불가 방식으로 동작하므로 withColumnsRenamed()로 칼럼 이름을 변경할 때는, 기존 칼럼 이름을 갖고 있는 원본을 유지한 채로 칼럼 이름이 변경된 새로운 데이터 프레임을 받아 오게 된다.

</aside>

**칼럼의 데이터 타입 변경**

- to_timestamp(), to_date()
    - 변경 코드
    
    ```python
    fire_ts_df = (new_fire_df
                  .withColumn("IncidentDate", to_timestamp(col("CallDate"), "MM/dd/yyyy"))
                  .drop("CallDate")
                  .withColumn("OnWatchDate", to_timestamp(col("WatchDate"), "MM/dd/yyyy"))
                  .drop("WatchDate")
                  .withColumn("AvailableDtTs", to_timestamp(col("AvailableDtTm"), "MM/dd/yyyy hh:mm:ss a"))
                  .drop("AvailableDtTm"))
    ```
    
    
    - 데이터 타입이 수정된 날짜/시간 칼럼을 가지게 되었으므로, 이후 데이터 탐색 과정에서 spark.sql.functions의 dayofmonth(), dayofyear(), dayofweek() 와 같은 함수를 이용하여 질의할 수 있다.

**집계연산**

- groupBy(), orderBy(), count()

```python
# CallType의 집계
(fire_ts_df
 .select("CallType")
 .where(col("CallType").isNotNull())
 .groupBy("CallType")
 .count()
 .orderBy("count", ascending = False)
 .show(n = 10, truncate = False))
```

### 정적타입, 동적타입 
- scala, java - 데이터프레임, 데이터세트 API 사용가능
- python - 데이터프레임 API 사용가능

Spark 코어 엔진:
100% Scala로 작성됨
JVM 위에서 실행됨
Catalyst optimizer, DAG scheduler 모두 Scala 코드

그런데 왜 Python으로 쓸 수 있나?
Python은 '클라이언트'일 뿐
실제 연산은 JVM에서 일어남
Python ↔ JVM 사이에 통신 레이어(Py4J) 존재


### 카탈리스트 옵티마이저

- 연산 쿼리를 받아 실행 계획으로 변환한다. 이는 다음의 4단계를 거친다.
    
1. **분석**
    - 추상 문법 트리(abstract syntax tree) 생성
2. **논리적 최적화**
    - 비용 기반 옵티마이저(cost based optimizer, CBO)를 사용하여 각 계획에 비용을 책정. 이는 연산 트리들로 배열됨.
    - 조건절 하부 배치, 칼럼 걸러내기, Boolean 연산 단순화 등을 포함
3. **물리 계획 수립**
    - 논리 계획을 바탕으로 대응되는 물리적 연산자를 사용하여 최적화된 물리 계획을 생성
4. **코드 생성**
    - 실제로 실행할 자바 바이트 코드를 생성
    - 실행 속도를 높이기 위한 코드 생성을 위해 최신 컴파일러 기술을 사용함. 다시 말하자면 스파크는 컴파일러처럼 동작한다고 할 수 있음.
    - 포괄 코드 생성을 가능하게 하는 텅스텐이 사용된다.
    
    <aside>
    💡 포괄 코드(whole-stage) 생성
    
    - 물리적 쿼리 최적화 단계로 전체 쿼리를 하나의 함수로 합치면서 가상 함수 호출이나 중간 데이터를 위한 CPU 레지스터 사용을 없애버림.
    </aside>
    




