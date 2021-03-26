# Spark
**인메모리 기반의 대용량 데이터 병렬 처리 오픈소스 엔진으로, 범용 분산 클러스터 컴퓨팅 프레임워크**

<br>

#### 하둡과 스파크   

하둡과 스파크 모두 빅데이터 처리 프레임워크라는 공통점이 있지만, 하둡은 대량의 데이터를 서버 클러스터 내 복수의 노드들에 분산시키는 역할을 한다. 스파크는 저장소 시스템의 데이터를 연산하는 역할만 수행할 뿐 영구 저장소 역할은 수행하지 않는다.

하둡과 아파치는 상호 독립적, 상호 보완적이다. 하둡은 HDFS와 맵리듀스를 핵심 구성요소로 제공하므로 스파크를 필수적으로 필요로 하지 않는다. 반대로 스파크도 하둡을 필수적으로 필요로 하지 않는다. Amazon S3, Apache Cassandra 등을 저장소로 지원하기 때문이다. 그러나 이 둘은 함께 할 때 좋은 궁합을 보인다.

데이터 운영 및 리포팅 요구 대부분이 정적인 것이고, 배치 모드의 프로세싱을 기다릴 수 있다면, 굳이 스파크를 쓰지 않고 맵리듀스 프로세싱 방식을 채택해도 무방하다. 스파크가 필수적인 비즈니스는 실시간으로 수집되는 스트리밍 데이터를 처리하거나, 머신러닝 알고리즘과 같이 애플리케이션이 복합적인 운영을 필요로 하는 경우이다. 또 스파크의 속도는 맵리듀스의 속도보다 월등히 빠르다.

<br>

## 스파크에 대해 간단하게 시작하기
### Spark Application
스파크 애플리케이션은 **driver** 프로세스와 다수의 **executor** 프로세스로 구성된다.  
**driver** : 스파크 애플리케이션 정보의 유지 관리, 사용자 프로그램이나 입력에 대한 응답, 전반적인 executor process 작업과 관련된 분석, 배포 그리고 스케줄링 (사용자의 스파크 어플리케이션을 여러 노드로 분산하여 처리 할 수 있게 변환).  
**executor** : driver process가 할당한 작업을 수행. 즉, driver가 할당한 코드를 실행하고 진행 상황을 다시 driver 노드에 보고.

<br>

The architecture of a Spark Application   

<img width="400" src="https://user-images.githubusercontent.com/55703132/111318820-bee15c00-86a8-11eb-9a1a-84d021c4be3f.png" />

<br>

`스파크 애플리케이션은 'SparkSession'이라 불리는 driver process로 제어한다. 하나의 SparkSession은 하나의 스파크 애플리케이션에 대응한다.`

핵심 !!
- 스파크는 사용 가능한 자원을 파악하기 위해 클러스터 매니저(spark standlone, hadoop yarn, mesos)를 사용한다.
- driver process는 주어진 작업을 완료하기 위해 드라이버 프로그램의 명령을 executor에서 실행할 책임이 있다.

<br>

### Transformation and Action
DataFrame or RDD를 '변경'하려면 **transformation**을 실행한다. (이해를 돕기 위한 transformation, action 메서드는 [여기](https://www.tutorialspoint.com/apache_spark/apache_spark_core_programming.htm)를 참고하라)   
transformation에는 두가지 유형이 있다.

![narrow transformation and wide transformation](https://user-images.githubusercontent.com/55703132/111321494-5182fa80-86ab-11eb-9e6a-39f627e3c33d.JPG)

**transformation**을 사용해 **논리적 실행 계획**을 세우고 **실제 연산을 수행하려면 action 명령을 내려야 한다.** - **lazy evaluation** (지연 연산)

<br>

## 구조적 API
구조적(고수준) API - DataFrame, SQL, Dataset  
- 사용자가 정의한 다수의 transformation은 DAG(directed acyclic graph)로 표현되는 명령을 만들어낸다.
- action은 하나의 잡을 클러스터에서 실행하기 위해 stage와 task로 나누고 DAG 처리 프로세스를 실행한다.
- transformation과 action을 다루는 논리적 구조가 DataFrame과 Dataset이다.

### DataFrame과 Dataset
DataFrame과 Dataset은 다양한 데이터 타입의 테이블형 데이터를 보관할 수 있는 Row 타입의 객체로 구성된 분산 컬렉션(수천 대의 컴퓨터에 분산되어 있다)이다.

- DataFrame과 Dataset 비교
   - df는 스키마에 명시된 데이터 타입의 일치여부를 런타임에 확인하고, ds는 컴파일 타임에 확인한다.
   - ds는 scala, java에서만 지원하며 python, R에서는 지원하지 않는다.
   - DataFrame은 Row 타입으로 구성된 Dataset이다.

- Schema
   - 스키마는 DataFrame의 컬럼명과 데이터 타입을 정의한다.
   - 스키마는 데이터소스에서 얻거나 직접 정의할 수 있다.

### 구조적 API 실행 과정
1. 코드 작성
2. 스파크가 코드를 **논리적 실행 계획**으로 변환
3. 스파크가 **논리적 실행 계획을 물리적 실행 계획으로** 변환하며 그 과정에서 추가적인 **최적화**를 할 수 있는지 확인
4. 스파크는 클러스터에서 **물리적 실행 계획(RDD 처리)** 을 실행

<br>

카탈리스트 옵티마이저가 코드를 넘겨 받고 실제 실행 계획을 생성한다.  

<img width="400" src="https://user-images.githubusercontent.com/55703132/111328741-db35c680-86b1-11eb-85ca-285ee6ad68ea.png" />
<br>

- **논리적 실행 계획**   
<img width="600" alt="논리적 실행계획" src="https://user-images.githubusercontent.com/55703132/111329127-39fb4000-86b2-11eb-8896-b8e8073af05a.png">

논리적 실행 계획 단계에서는 추상적인 transformation만 표현된다. 필요한 컬럼과 테이블이 카탈로그에 있는지 검증하고 이 결과를 catalyst optimizer로 전달된다. catalyst optimizer는 논리적 실행 계획을 최적화하는 규칙의 모음이다.

- **물리적 실행 계획 (스파크 실행 계획)**   
<img width="600" alt="물리적 실행계획" src="https://user-images.githubusercontent.com/55703132/111329967-e1787280-86b2-11eb-85c0-d6c249768515.png">

논리적 실행 계획을 클러스터 환경에서 실행하는 방법을 정의한다. 다양한 물리적 실행 전략을 생성하고 비용 모델을 이용해서 비교한 후 최적의 전략을 선택한다.   
물리적 실행 계획은 일련의 RDD와 transformation으로 변환된다. 스파크는 DataFrame, Dataset, SQL로 정의된 쿼리를 RDD transformation으로 컴파일한다.

<br>

## 저수준 API
저수준 API - RDD, SparkContext, accumulator, broadcast variable   

**RDD** (Resilient Distributed Datasets 탄력적 분산 데이터셋) : immutable distributed collection of objects. spark의 기본 데이터 구조.   

<img width="546" alt="rdd" src="https://user-images.githubusercontent.com/55703132/111334697-21d9ef80-86b7-11eb-8e81-5c261725dda6.png">

RDD에 있는 각각 데이터셋은 클러스터의 다른 노드에서 계산 될 수 있는 logical partitions로 나뉜다.   
즉, 하나의 작업을 처리할 때 파티션 단위로 나눠서 병렬로 처리한다.   
`스파크는 모든 executor가 병렬로 작업을 수행할 수 있도록 '파티션'이라 불리는 chunk 단위로 데이터를 분할한다. 파티션은 클러스터의 물리적 머신에 존재하는 row의 집합을 의미한다.`

<details>
<summary>Partition</summary>
<div markdown="1">

하둡에 있는 파일을 읽어서 RDD형태로 가지고 있는게 파티션인데  
파티션은 블럭을 읽어서 메모리에 올려져있다고 생각하는거라 (스파크가 인메모리 컴퓨팅이잖아요)  
여러 머신 메모리에 올라가 있는 상태가 RDD이고, 그 조각이 파티션.  

</div>
</details>
<br>


구조적 API와 다르게 내부 구조를 스파크에서 파악할 수 없으므로 최적화를 하려면 훨씬 많은 수작업 필요하다. RDD는 DataFrame API에서 최적화된 물리적 실행 계획을 만드는 데 대부분 사용된다. 사용자가 실행한 모든 Dataframe나 Dataset 코드는 RDD로 컴파일된다.

<br>

## RDD vs DataFrame vs Dataset
RDD, DataFrame, Dataset 공통점 : 불변성, 인메모리, 탄력성, 분산컴퓨팅       

차이점   
- RDD   
RDD는 위 Low level API(Transform, Action)를 모든 파티션에 병렬로 제공할 수 있다.   
**RDD 언제사용하는가?**   
   - Low-level transformation, actions 처리할 때
   - Unstructured data
   - Functional Programming으로 처리하고 싶을 때
   - 스키마 신경 쓰고 싶지 않을 때
   - Structured, Semi-Structured 데이터 처리 시, DataFrame, Dataset으로 얻을 수 있는 이점을 포기해도 될 때(Optimization, Performance)

- Dataframe, Dataset    
**Datasets vs DataFrames, 언제 무엇을 사용해야 할까?**  
   - 높은 추상화의 API를 사용하고 싶은 경우 —> DataFrame, Dataset
   - map, filters 등 다양한 Spark API를 사용하고 싶다면 —> DataFrame, Dataset
   - type-safety를 컴파일 단계에서 확인하고 싶다면 —> Dataset

<br>

! 아래 표 내용에서 `Serialization`, `Efficiency/Memory use` 특히 잘 모르겠다ㅠㅠ ! 

||RDD|DataFrame|Dataset|
|--------|--------|--------|--------|
|Data Formats|structured(RDB, csv),<br>unstructured(영상,이미지,음성) data<br>& 스키마 추론 불가능|structured,<br>semi-structured(json, html, xml) data|structured, unstructured data|
|Compile-time type safety|O|X (run-time)|O|
|Serialization|Java serialization<br>(개별 java, scala objects를 직렬화하는 overhead는 비용이 많이들고 노드간에 data와 structure를 모두 전송해야 한다)|데이터를 binary형식의 off-heap storage (in memory)로 직렬화 한 다음, spark가 schema를 이해하므로 off heap memory에서 많은 transformation을 수행할 수 있다.||
|Garbage Collection|O<br>개별 object를 만들고 삭제함으로 인해 GC overhead가 있다.|X<br>데이터셋 각 row에 대해 개별 object를 구성할 때 GC 비용을 방지한다.|X|
|Efficiency/Memory use|많은 시간동안 java, scala 객체에서 개별적으로 직렬화를 수행하면 효율성이 저하된다.|직렬화를 위해 off heap memory를 사용하면 overhead가 줄어든다. It generates byte code dynamically so that many operations can be performed on that serialized data. No need for deserialization for small operations.||
|Performance optimization|X|O<br>DataFrame, Dataset은 Spark SQL Engine을 바탕으로 만들어졌다. 이들은 Catalyst를 사용하기 때문에 논리적/물리적으로 최적화 쿼리를 사용할 수 있다.|O|
|Lazy Evalution|O|O|O|

+'직렬화' 행에 대한 추가설명  
Spark 가 바이너리 형식의 오프힙저장소로 데이터를 직렬화 한다음 오프 힙메모리에서 직접 많은 변환을 수행하여 개별 개체를 구성하는 것과 관련된 가비지 수집 비용을 피할 수 있으므로 단일 프로세스에서 계산을 수행할 때 장점이 있다.(데이터를 인코딩하기 위해 자바 직렬화를 사용할 필요가 없음)

<details>
<summary>참고할 여러 개념들</summary>
<div markdown="1">

### 컴파일 타임과 런타임
Compile time : 개발자가 소스코드를 작성하고 컴파일이라는 과정을 통해 기계어코드로 변환 되어 실행 가능한 프로그램이 되며, 이러한 편집 과정을 컴파일타임(Compiletime) 이라고 부른다.   
Run time : 컴파일 과정을 마친 응용 프로그램이 사용자에 의해서 실행되어 지는 때(time)를 의미한다.

### 직렬화와 역직렬화
- 직렬화(serialization)   
메모리를 디스크에 저장하거나 네트워크 통신에 사용하기 위한 형식으로 변환하는 것을 말한다.
- 역직렬화(deserialization)   
직렬화의 반대로 디스크에 저장한 데이터를 읽거나, 네트워크 통신으로 받은 데이터를 메모리에 쓸 수 있도록 다시 변환하는 것이다. 

앞서 얘기한대로 직렬화는 데이터를 저장 혹은  통신에 사용하기 위함인데 **데이터를 그냥 사용하면 안되고 왜 직렬화라는 과정을 거쳐야 할까?**   

**직렬화는 왜 필요한가? 사용이유**  

우선 메모리(힙, 스택 영역 등)에 대한 기본적인 지식이 있어야 이해가 가능하다.  

개발 언어로 무엇을 사용하던(C++, C, JAVA etc..) 사용하는 데이터들의 메모리 구조는 크게 2가지로 나뉜다.  
**1. 값 형식 데이터(Value Type)** : 우리가 흔히 선언해서 사용하는 int, float, char 등 값 형식 데이터들은 스택에 메모리가 쌓이고 직접 접근이 가능하다.  
**2. 참조 형식 데이터(Reference Type)** : C#에서 Object 타입 혹은 C++에서 포인터 변수들이 여기에 해당된다. 해당 형식의 변수를 선언하면 힙에 메모리가 할당되고 스택에서는 이 힙 메모리를 참조하는(힙의 메모리 번지 주소를 가지고 있음) 구조로 되어있다.  

이 두가지 데이터 중에서 **디스크에 저장하거나 통신에는 값 형식 데이터(Value Type)만 가능**하다.  
참조 형식 데이터(Reference Type)는 실제 데이터 값이 아닌 힙에 할당되어 있는 메모리 번지 주소를 가지고 있기 때문에 저장, 통신에 사용할 수 없다.  

**왜 참조 형식 데이터는 사용할 수 없을까?**  
예를 들자면  
포인터 변수를 선언하여 그 주소값이 0x000f라고 가정했을 때  
프로그램을 종료하고 다시 실행해서 주소값 0x000f을 가져오더라도 기존 데이터를 가져올 수 없다.(프로그램이 종료되면 기존에 할당되었던 메모리는 해제되고 없어진다)  

네트워크 통신 또한 마찬가지이다.  
각 PC마다 사용하고 있는 메모리 공간 주소는 전혀 다르다.  
그렇기 때문에 내가 다른 PC로 전송한 데이터(0x000f)는 무의미하다.  
이 데이터를 받은 PC에서의 메모리 주소 0x000f에는 전혀 다른 값이 존재하기 때문이다.  

직렬화를 하게 되면 각 주소값이 가지는 데이터들을 전부 끌어모아서 값 형식(Value Type)데이터로 변환해준다.  
이러한 이유때문에 데이터를 저장, 통신 전에 **'데이터 직렬화(Serialization)'** 작업이 필요한 것이다!!  
직렬화가 된 데이터들은 언어에 따라서 텍스트 혹은 바이너리 등의 형태가 되는데, 이러한 형태가 되었을 때 저장하거나 통신 시 파싱이 가능한 유의미한 데이터가 되는 것이다.  

한 가지 예시를 더 말하자면,  
String이 포인터로 구현되어 있는 경우 int, double 등 과는 다르게 내부적으로 메모리가 연속적으로 되어있지 않다 (*int*는 *4byte*씩 *double*는 *8byte*씩 메모리가 연속적으로 배치되어 있음). 이 String 데이터를 무사히 저장 혹은 전송하기 위해서는 이 메모리 데이터들을 연속적으로 배치, 값 타입 변조 즉 직렬화를 해줘야 하는 것이다.  

마지막으로 요약해보면  
**직렬화를 쓰는 이유는 사용하고 있는 데이터들을 파일 저장 혹은 데이터 통신에서 파싱할 수 있는 유의미한 데이터를 만들기 위함이다.**  

<br>

java에서 serialize란 (아래 설명의 반대가 deserialize)
- 자바 시스템 내부에서 사용되는 Object 또는 data를 외부의 자바 시스템에서도 사용할 수 있도록 byte 형태로 데이터를 변환하는 기술
- JVM의 메모리에 상주(heap or stack)되어 있는 객체 데이터를 byte 형태로 변환하는 기술

### [Java] On-heap, Off-heap

```
The on-heap store refers to objects that will be present in the Java heap (and also subject to GC).
On the other hand, the off-heap store refers to (serialized) objects that are managed by EHCache, 
but stored outside the heap (and also not subject to GC). As the off-heap store continues to be managed in memory, 
it is slightly slower than the on-heap store, but still faster than the disk store.
```

- **On-heap** stroe   
Java 프로그램에서 new 연산 등을 이용해 객체를 생성하면 JVM은 힙(Heap) 메모리 영역에 객체를 생성하여 저장, 관리한다. C/C++과 다르게 Java에서는 명시적으로 할당받은 메모리 영역을 해제하는 등의 조치는 취하지 않아도 된다. GC(Garbage Collection) 알고리즘이 귀찮고 어렵고 문제를 발생시킬 수 있는 메모리 관리를 대신 해주기 때문이다.  
Java에서 생성된 객체가 저장되는 힙 영역을 좀 더 자세하게 On-heap store라고 한다.   
일반적인 경우라면 On-heap 영역만 인지하고 사용해도 애플리케이션을 작성하는데 문제는 없지만, On-heap store에 객체를 저장하다보면 GC 오버헤드(특히 Full GC)때문에 애플리케이션의 성능이 저하되는 경험을 할 수 있다. 

- **Off-heap** store   
EHCache에 에 의해 관리되지만 힙 외부에 저장되는 (직렬화 된) 객체를 참조한다. 즉 Off-heap store는 '힙 밖에 저장한다'라는 의미를 가지고 있고, 힙 밖에 저장한다는 의미는 GC 대상으로 삼지 않겠다는 의미다.  
EHCache의 off-heap store는 일반 object를 heap에서 가져와서 직렬화하여 EHCache가 관리하는 메모리 덩어리에 바이트로 저장한다. 이 상태에서는 객체를 직접 사용할 수 없으므로 먼저 deserialization 해야한다.

</div>
</details>
<br>

## 클러스터에서 스파크 실행하기
물리적 실행 계획은 클러스터의 머신에서 실행되는 단위인 RDD 작업으로 구성된다. 스파크에서 코드를 실행할 때 어떤 일이 발생하는지 알아보도록 하자.  

### 스파크 애플리케이션의 아키텍처
- driver  
**물리적 머신의 프로세스(process)**  
**스파크 클러스터(executor의 상태와 task)에서 실행 중인 애플리케이션의 상태를 유지**한다. 또한 물리적 컴퓨팅 자원 확보와 executor 실행을 위해 clust manager와 통신할 수 있어야 한다.  

- executor  
**driver가 할당한 task를 수행하는 프로세스**  
driver가 할당한 task를 받아 실행하고 task의 상태와 결과를 driver에 보고한다. 모든 스파크 애플리케이션은 개별 executor process를 사용한다.  

- cluster manager  
**스파크 애플리케이션을 실행한 클러스터 머신을 유지(관리)**  
클러스터 매니저는 '마스터'(드라이브라 부르기도 함)와 '워커'라는 개념을 가지고 있다. 가장 큰 차이점은 **프로세스가 아닌 물리적 머신에 연결되는 개념**이라는 것이다.

스파크 애플리케이션을 실행할 때가 되면 **cluster manager에 자원할당을 요청**해야 한다. 스파크는 `spark standlone, hadoop yarn, apache mesos` 클러스터 매니저를 지원한다.  

실행 중인 스파크 애플리케이션이 없는 클러스터 마스터(드라이버)와 워커   
<img width="600" src="https://user-images.githubusercontent.com/55703132/111569291-b04c8f00-87e5-11eb-8b55-f41ea583f8cd.JPG" />

- 실행모드  
애플리케이션을 실행할 때 요청한 자원의 물리적인 위치를 결정한다.

1. **cluster mode**  

<img width="600" src="https://user-images.githubusercontent.com/55703132/111570028-4634e980-87e7-11eb-99fd-fbfe8febad31.JPG" />

컴파일된 JAR파일이나 파이썬 스크립트 또는 R 스크립트를 클러스터 매니저에 전달해야 한다.   
클러스터 매니저는 파일을 받은 다음 **워커노드에 driver, executor process를 실행**한다. 즉, **클러스터 매니저는 모든 스파크 애플리케이션과 관련된 프로세스를 유지하는 역할**을 한다.

<br>

2. **client mode**   

<img width="700" src="https://user-images.githubusercontent.com/55703132/111570633-9496b800-87e8-11eb-971e-4b7a2f08a975.JPG" />

애플리케이션을 제출한 클라이언트 머신에 spark driver가 위치한다는 것을 제외하면 클러스터 모드와 비슷하다.  
즉, 클라이언트 머신은 driver process를 유지하며 클러스터 매니저는 executor process를 유지한다.

<br>

3. **local mode**  
앞선 두 모드와 상당히 다르다. 단일 머신에서 실행된다. 로컬 모드는 애플리케이션의 병렬 처리를 위해 단일 머신의 thread를 활용한다.  
(스파크를 학습하거나, 애플리케이션 테스트 그리고 개발 중인 애플리케이션을 반복적으로 실행하는 용도로 주로 사용된다.)


### 스파크 애플리케이션의 생애주기
- 스파크 외부   
클러스터 관점(스파크를 지원하는 인프라 관점).

1. 클라이언트 요청   
<img width="500" src="https://user-images.githubusercontent.com/55703132/111571482-2e129980-87ea-11eb-959e-26b185901a79.JPG" />

2. 스파크 애플리케이션 시작   
<img width="500" src="https://user-images.githubusercontent.com/55703132/111571979-1851a400-87eb-11eb-97f0-c676853a83c5.JPG" />

코드에 반드시 스파크 클러스터(ex. driver, executor)를 초기화하는 **SparkSession**이 포함되어야 한다. 

3. 실행  
<img width="400" src="https://user-images.githubusercontent.com/55703132/111572116-623a8a00-87eb-11eb-9a3a-ca35ffef6e27.JPG" />

드라이버는 각 워커에 태스크를 할당한다. 태스크를 할당받은 워커는 태스크의 상태와 성공/실패 여부를 드라이버에 전송한다.

4. 완료  
스파크 애플리케이션의 실행이 완료되면 클러스터 매니저는 드라이버가 속한 스파크 클러스터의 모든 익스큐터를 정료시킨다.

<br>

- 스파크 내부   
   - **Job**   
   보통 **action 하나당 하나의 스파크 job이 생성되며 action은 항상 결과를 반환**한다. Job은 일련의 stage로 나뉘며 stage 수는 shuffle 작업이 얼마나 많이 발생하는지에 따라 달라진다.
   - **Stage**   
   다수의 머신에서 동일한 연산을 수행하는 task의 그룹이다. 즉, Stage란 DAGSchedualr에 의해서 생성된 물리적 실행 계획의 단계(Step)들이다. 하나의 Stage를 처리하기 위해서는 하나 이상의 Task가 필요하다.  
   Stage가 나뉘어지는 기준은 shuffle dependencies에 의해서 정해진다. 즉, Shuffle을 발생시키는 join, repartition과 같은 연산이 Stage가 나뉘어지는 기준이 된다.
   - **Task**   
   **단일 executor에서 실행할 데이터의 블록과 다수의 transformation 조합**으로 볼 수 있다. 즉, task는 데이터 단위(파티션)에 적용되는 연산 단위를 의미한다.
