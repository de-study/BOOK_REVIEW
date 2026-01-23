# Spark 시작하기: 설치 및 핵심 개념 요약
## 1. 설치 및 환경 설정
지원 언어: Python, Java, Scala (Spark 자체는 Scala로 작성됨)

필수 요건: Java 6 이상 설치 필수. Python API 사용 시 Python 2.6 이상 필요(제공된 텍스트 기준이며, 최신 버전은 Python 3 지원)

다운로드: spark.apache.org에서 pre-built 버전을 받아 압축 해제

디렉토리 구조: * bin: Spark와 상호작용하는 실행 파일(shell 등) 포함

examples: API 학습을 위한 예제 코드

core, streaming 등: 주요 컴포넌트 소스 코드

## 2. Spark 실행 모드
로컬 모드(Local Mode): 단일 머신에서 비분산 모드로 실행. 학습 및 테스트용

클러스터 모드: Mesos, YARN, 또는 Spark Standalone 스케줄러를 사용하여 여러 노드에서 분산 실행

## 3. 대화형 쉘 (Python & Scala)
Spark는 즉각적인 데이터 분석을 위해 대화형 쉘을 제공함

실행: bin/pyspark (Python), bin/spark-shell (Scala)

특징: 일반적인 쉘과 달리 여러 장비에 분산된 메모리와 디스크의 데이터를 제어할 수 있음

로그 조절: conf/log4j.properties 설정을 통해 화면에 출력되는 로그 수준(INFO, WARN 등) 변경 가능

## 4. 핵심 프로그래밍 개념
RDD (Resilient Distributed Dataset): Spark의 기본 추상화 개념. 클러스터 전체에 분산된 데이터 컬렉션

드라이버 프로그램(Driver Program): 어플리케이션의 main 함수를 실행하고 클러스터에 연산을 지시하는 프로그램

SparkContext (sc): 클러스터 연결을 나타내는 객체. RDD를 생성할 때 사용하며, 쉘 실행 시 sc 변수로 자동 생성됨

익스큐터(Executors): 드라이버가 지시한 작업을 실제로 수행하는 노드 프로세스

### 4.1 핵심 프로그래밍 개념 상세 정리
1. RDD (Resilient Distributed Dataset)
Spark의 가장 기본적인 데이터 추상화 모델로, 다음과 같은 특징을 가짐

불변성 (Immutable): 한 번 생성된 RDD는 수정할 수 없으며, 연산을 통해 새로운 RDD를 생성함

복구력 (Resilient): 노드 장애로 데이터가 손실되어도 계보(Lineage)를 기억해 자동으로 복구함

분산 처리 (Distributed): 대규모 데이터를 파티션 단위로 나누어 클러스터 전체에 분산 저장함

2. 연산 구조: Transformation과 Action
Spark은 연산을 효율적으로 관리하기 위해 두 단계로 나눔

Transformation (변환): 기존 RDD로 새로운 RDD를 만드는 과정 (예: filter, map). 지연 평가(Lazy Evaluation) 방식을 채택하여 실제 결과가 필요하기 전까지는 연산을 수행하지 않고 논리적 계획만 세움

Action (실행): 실제 계산을 수행해 결과를 드라이버에 반환하거나 저장하는 과정 (예: count, collect, saveAsTextFile). Action이 호출되는 시점에 모든 Transformation이 실행됨

3. 실행 구성 요소: Driver와 Executor
Driver Program: main 함수를 실행하고 SparkContext를 통해 전체 작업을 지휘함. 연산 로직을 Task로 쪼개 클러스터에 배분함

Executors: 드라이버가 보낸 Task를 실제로 실행하고 데이터를 저장하는 일꾼 프로세스임

4. 함수 전달 (Passing Functions)
Spark API의 핵심은 사용자가 정의한 로직(함수)을 클러스터로 보내는 것임

Python/Scala: lambda나 => 같은 간결한 구문을 사용해 인라인 함수를 정의하고 전달함

직렬화: 드라이버에서 작성한 코드는 직렬화되어 네트워크를 통해 익스큐터로 복제된 후 데이터 위에서 실행됨

## 5. 독립형 어플리케이션 (Standalone Apps)
쉘이 아닌 실제 프로그램 작성 시의 흐름

SparkContext 초기화: SparkConf를 생성(앱 이름, 마스터 URL 설정)한 뒤 이를 통해 SparkContext 객체를 직접 생성해야 함

빌드 도구: Java/Scala는 Maven이나 sbt를 사용하여 의존성(spark-core)을 관리

배포: bin/spark-submit 스크립트를 사용하여 작성한 스크립트나 JAR 파일을 Spark 환경에서 실행
