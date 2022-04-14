## Stream이란?

스트림(Stream)은 자바 8부터 추가된 컬렉션의 저장 요소를 하나씩 참조해서 람다식으로 처리할 수 있는 반복자이다.

## Stream의 특징

### **람다식으로 요소 처리 코드를 제공한다**

Stream이 제공하는 대부분의 요소 처리 메소드는 함수적 인터페이스 매개 타입을 가지기 때문에 람다식 또는 메소드 참조를 이용해서 요소 처리 내용을 매개값으로 전달할 수 있다.

```java
Stream<Student> streaam = list.stream();
stream.forEach( s -> {
	  String name = s.getName();
	  int score = s.getScore();
	  System.out.println(name + "-" + score);
});
```

### 원본 데이터를 변경하지 않는다

Stream API는 원본의 데이터를 조회하여 원본의 데이터가 아닌 별도의 요소들로 Stream을 생성한다. 그렇기 때문에 원본의 데이터로부터 읽기만 할 뿐이며, 정렬이나 필터링 등의 작업은 별도의 Stream 요소들에서 처리가 된다.

### 내부 반복자를 사용하므로 병렬 처리가 쉽다

**외부 반복자**

개발자가 코드로 직접 컬렉션의 요소를 반복해서 가져오는 코드 패턴을 말한다. for문, Iterator를 사용하는 while문은 모두 외부 반복자를 이용하는 것이다.

**내부 반복자**

컬렉션 내부에서 요소들을 반복시키고, 개발자는 요소당 처리해야 할 코드만 제공하는 코드 패턴을 말한다.

![image](https://user-images.githubusercontent.com/55661631/163376942-630f77a2-5b39-44e2-ba29-96c549fee7b7.png)

내부 반복자를 사용하면 컬렉션 내부에서 어떻게 요소를 반복시킬 것인가는 컬렉션에게 맡겨두고, **개발자는 요소 처리 코드에만 집중**할 수 있게 된다.

내부 반복자는 요소들의 반복 순서를 변경하거나, 멀티 코어 CPU를 최대한 활용하기 위해 요소들을 분배시켜 병렬 작업을 할 수 있게 도와주기 때문에 하나씩 처리하는 순차적 외부 반복자보다는 효율적으로 요소를 반복시킬 수 있다.

![image](https://user-images.githubusercontent.com/55661631/163376976-9ae56e7a-f64d-4e12-8289-247ca3f1a364.png)

**병렬 처리란?**

병렬 처리란 한 가지 작업을 서브 작업으로 나누고, 서브 작업들을 분리된 스레드에서 병렬적으로 처리하는 것을 말한다.

병렬 처리 스트림을 이용하면 런타임 시 하나의 작업을 서브 작업으로 자동으로 나누고, 서브 작업의 결과를 자동으로 결합해서 최종 결과물을 생성한다.

예를 들어 컬렉션의 요소 총합을 구할 때 순차 처리 스트림은 하나의 스레드가 요소들을 순차적으로 읽어 합을 구하지만, 병렬 처리 스트림을 이용하면 여러 개의 스레드가 요소들을 부분적으로 합하고 이 부분 합을 최종 결합해서 전체 합을 생성한다.

```java
//순차 처리
Stream<String> stream = list.stream();
stream.forEach(ParallelExample::print);

//병렬 처리
Stream<String> parallelStream = list.parallelStream();
parallelStream.forEach(ParallelExample::print);
```

### 스트림은 중간 처리와 최종 처리를 할 수 있다

스트림은 컬렉션의 요소에 대해 중간 처리와 최종 처리를 수행할 수 있다.

- 중간 처리
    - 매핑, 필터링, 정렬 수행
- 최종 처리
    - 반복, 카운팅, 평균, 총합 등의 집계 처리를 수행

![image](https://user-images.githubusercontent.com/55661631/163377009-fcd21532-caa4-468d-91c7-aa0db82208fb.png)

## Stream의 파이프라인

대량의 데이터를 가공해서 축소하는 것을 일반적으로 리덕션(`Reduction`)이라고 하며, 데이터의 합계, 평균값, 카운팅, 최댓값, 최솟값 등이 대표적인 리덕션의 결과물이다.

하지만 컬렉션의 요소를 리덕션의 결과물로 바로 집계할 수 없을 경우에는 집계하기 좋도록 필터링, 매핑, 정렬, 그룹핑 등의 중간 처리가 필요하다.

### 중간 처리와 최종 처리

스트림은 데이터의 필터링, 매핑, 정렬, 그룹핑 등의 중간 처리와 합계, 카운팅, 최댓값, 최솟값 등의 최종 처리를 파이프라인으로 해결한다. 

파이프라인은 여러 개의 스트림이 연결되어 있는 구조를 말하며, 파이프라인에서 최종 처리를 제외하고는 모두 중간 처리 스트림이다.

![image](https://user-images.githubusercontent.com/55661631/163377041-fa6fe6af-1f48-4f1b-ae18-d79fb535f871.png)

중간 스트림이 생성될 때 요소들이 바로 중간 처리(필터링, 매핑, 정렬)되는 것이 아니라 **최종 처리가 시작되기 전까지 중간 처리는 지연된다.** 최종 처리가 시작되면 비로소 컬렉션의 요소가 하나씩 중간 스트림에서 처리되고 최종 처리까지 오게 된다.

Stream 인터페이스에는 필터링, 매핑, 정렬 등의 많은 중간 처리 메소드가 있는데, 이 메소드들은 중간 처리된 스트림을 리턴한다. 그리고 이 스트림에서 다시 중간 처리 메소드를 호출해서 파이프라인을 형성하게 된다.

```java
Stream<Member> maleFemaleStream = list.stream();
Stream<Member> maleStream = maleFemaleStream.filter(m -> m.getSex() == Member.MALE);
IntStream ageStream = maleStream.mapToInt(Member::getAge);
OptionalDouble optionalDouble = ageStream.average();
double ageAvg = optionalDouble.getAsDouble();
```

로컬 변수를 생략하고 연결하면 다음과 같은 형태의 파이프라인 코드만 남는다.

```java
double ageAvg = list.stream()				
        .filter(m -> m.getSex() == Member.MALE)	//중간 처리 스트림
        .mapToInt(Member :: getAge)			        //중간 처리 스트림
        .average()                              //최종 처리 스트림
        .getAsDouble();
```

### 중간 처리와 최종 처리 메소드

다음은 스트림이 제공하는 중간 처리와 최종 처리를 하는 메소드를 설명한 표이다.

**중간 처리 메소드**

![image](https://user-images.githubusercontent.com/55661631/163377115-533f4e27-6b79-413e-8000-4ae355ffb366.png)

**최종 처리 메소드**

![image](https://user-images.githubusercontent.com/55661631/163377141-8d80dd4f-0f55-4b2e-9e6e-0fd023f4db43.png)

## 중간 처리 메소드

### 필터링 - Filter, Distinct

**Filter**

Filter 메소드는 Stream에서 조건에 맞는 데이터만을 정제하여 더 작은 컬렉션을 만들어내는 연산이다. Filter 메소드는 인자로 함수형 인터페이스 Predicate를 받고 있기 때문에, boolean을 반환하는 람다식을 작성하여 구현할 수 있다.

```java
Stream<String> stream = list.stream()
								.filter(name -> name.contains("a"));
```

**Distinct**

Stream의 요소들에 중복된 데이터가 존재하는 경우, 중복을 제거하기 위해 distinct를 사용할 수 있다. distinct 메소드는 중복된 데이터를 검사하기 위해 Object의 equals() 메소드를 사용한다.

```java
Stream<String> stream = list.stream()
									.distinct()
```

### 매핑 - map, flatMap

**map**

![image](https://user-images.githubusercontent.com/55661631/163377174-6dfd2f7b-5792-4615-92c8-f51919915135.png)

map 메소드는 요소를 대체하는 요소로 구성된 새로운 스트림을 리턴한다.

```java
Stream<String> stream = names.stream()
									.map(s -> s.toUpperCase());
```

**flatMap**

![image](https://user-images.githubusercontent.com/55661631/163377202-b5bbe5fa-5f42-472a-abd3-313ed012677b.png)

flatMap 메소드는 요소를 대체하는 복수 개의 요소들로 구성된 새로운 스트림을 리턴한다.

```java
List<String> inputList = Arrays.asList("java8 lambda", "stream mapping");
inputList.stream()
        .flatMap(data -> Arrays.stream(data.split(" ")))
        .forEach(word -> System.out.println(word)); 
```

### 정렬 - Sorted

sorted 메소드는 Stream의 요소들을 정렬하기 위해 사용한다. 파라미터로 Comparator를 넘길 수도 있으며, Comparator 인자 없이 호출할 경우에는 오름차순으로 정렬된다.

```java
List<String> list = Arrays.asList("Java", "Scala", "Groovy", "Python", "Go", "Swift");

// [Go, Groovy, Java, Python, Scala, Swift]
Stream<String> stream = list.stream()
										.sorted() 
```

### 루핑 - Peek

peek 메소드는 Stream의 요소들을 대상으로 Stream에 영향을 주지 않고 특정 연산을 수행하기 위해 사용한다. peek 메소드는 파라미터로 함수형 인터페이스 Consumer를 인자로 받는다. 어떤 stream의 요소들을 중간에 출력하기를 원할 때 다음과 같이 활용할 수 있다.

```java
int sum = IntStream.of(1, 3, 5, 7, 9)
			.peek(System.out::println)
			.sum();
```

## 최종 처리 메소드

### 기본 집계 - Max, Min, Sum, Average, Count

Stream의 요소들을 대상으로 최솟값이나 최댓값 또는 총합을 구하기 위한 최종 처리 기능들이 존재한다.

min이나 max 또는 average는 Stream이 비어있는 경우에 값을 특정할 수 없다. 그렇기 때문에 다음과 같이 Optional로 값이 반환된다.

```java
OptionalInt min = IntStream.of(1, 3, 5, 7, 9).min();
int max = IntStream.of().max().orElse(0);
IntStream.of(1, 3, 5, 7, 9).average().ifPresent(System.out::println);
```

총합이나 갯수의 경우에는 값이 비어있는 경우 0으로 값을 특정할 수 있다. 그렇기 때문에 sum이나 count는 Optional이 아닌 원시 값을 반환한다.

```java
long count = IntStream.of(1, 3, 5, 7, 9).count();
long sum = LongStream.of(1, 3, 5, 7, 9).sum();
```

### 수집 - Collect

collect 메소드는 Stream의 요소들을 List나 Set, Map, 등 다른 종류의 결과로 수집할 때 사용하며 Stream의 요소들을 수집할 것인가를 정의한 Collector 타입을 인자로 받아서 처리한다.

Collectors 클래스에서 static 메소드로 제공하는 작업들을 살펴보자.

- Collectors.toList()
    - Stream에서 작업한 결과를 List로 반환받을 수 있다. 아래의 예제에서는 Stream의 요소들을 Product의 이름으로 변환하여, 그 결과를 List로 반환받고 있다.
        
        ```java
        List<String> nameList = productList.stream()
            .map(Product::getName)
            .collect(Collectors.toList());
        ```
        
- Collectors.joining()
    - 작업한 결과를 1개의 String으로 이어붙이기를 원하는 경우에 Collectors.joining()을 이용할 수 있다.
        
        ```java
        String listToString = productList.stream()
        						.map(Product::getName)
        						.collect(Collectors.joining());
        ```
        
- Collectors.groupingBy()
    - Stream에서 작업한 결과를 특정 그룹으로 묶기를 원할 때 Collectors.groupingBy()를 이용할 수  있다. 결과는 Map으로 반환받게 된다.
        
        ```java
        Map<Integer, List<Product>> collectorMapOfLists = productList.stream()
        								.collect(Collectors.groupingBy(Product::getAmount));
        
        /* {
        23=[Product{amount=23, name='potatoes'}, Product{amount=23, name='bread'}], 
        13=[Product{amount=13, name='lemon'}, Product{amount=13, name='sugar'}], 
        14=[Product{amount=14, name='orange'}]
        } */
        ```
        
- Collectors.partitioningBy()
    - Collectors.partitioningBy()는 함수형 인터페이스 Predicate를 받아 Boolean을 Key값으로 나눌 수 있다.
    - 예를 들어 제품의 갯수가 15보드 큰 경우와 그렇지 않은 경우를 나누고자 한다면 다음과 같이 코드를 작성할 수 있다.
        
        ```java
        Map<Boolean, List<Product>> mapPartitioned = productList.stream()
        						.collect(Collectors.partitioningBy(p -> p.getAmount() > 15)); 
        
        /* {
        false=[Product{amount=14, name='orange'}, Product{amount=13, name='lemon'}, Product{amount=13, name='sugar'}], 
        true=[Product{amount=23, name='potatoes'}, Product{amount=23, name='bread'}]
        } */
        ```
        

### 매칭 - allMatch, anyMatch, noneMatch

match 메소드는 Stream의 요소들이 특정한 조건을 충족하는지 검사할 때 사용할 수 있다. match 메소드는 함수형 인터페이스 Predicate를 받아서 해당 조건을 만족하는지 검사를 하게 되고, 검사 결과를 boolean으로 반환한다.

- anyMatch: 1개의 요소라도 해당 조건을 만족하는가
- allMatch: 모든 요소가 해당 조건을 만족하는가
- nonMatch: 모든 요소가 해당 조건을 만족하지 않는가

```java
List<String> names = Arrays.asList("Eric", "Elena", "Java"); 

//모두 True 반환
boolean anyMatch = names.stream().anyMatch(name -> name.contains("a")); 
boolean allMatch = names.stream().allMatch(name -> name.length() > 3);
boolean noneMatch = names.stream().noneMatch(name -> name.endsWith("s"));
```

### 루핑 - forEach

Stream의 요소들을 대상으로 어떤 특정한 연산을 수행하고 싶은 경우에는 forEach 메소드를 이용할 수 있다. 예를 들어 요소들을 출력하기를 원할 때 다음과 같이 forEach를 사용하면 된다.

```java
names.stream().forEach(System.out::println);
```

## Stream 주의할 점

### Stream 재사용

흔하게 접할 수 있는 실수 하나는 스트림을 재사용하는 것이다. 스트림은 오직 한 번만 사용할 수 있기 때문에 스트림을 컬렉션처럼 사용하면 `IllegalStateException`이 발생한다.

```java
// 문자열 스트림 생성
Stream<String> langNames = Stream.of("Java", "C++", "Python", "Ruby");

// 스트림 내 모든 요소 출력
langNames.forEach(System.out::println);

// "Java" 만 제외한 스트림을 다시 생성... Exception이 발생한다.
Stream<String> filtered = langNames.filter(lang -> !lang.equals("Java"));
filtered.forEach(System.out::println);
```

### ****Stream은 for-loop보다 느리다****

관련 참고자료를 인용하면, 단순 `for-loop`의 경우 오버헤드가 없는 단순한 인덱스 기반 메모리 접근이기 때문에 `Stream`을 사용했을 때보다 더 빠르다고 한다. 또한 JIT 컴파일러가 오랫동안 유지해온 `for-loop`에 대한 내부 최적화가 잘 되어 있는 반면에 스트림은 2015년 이후에 도입되었으므로 아직 컴파일러가 최적화를 제대로 하지 못했다고 한다.

Stream과 for-loop의 성능에 대해 자세히 살펴보고 싶으면 [링크](https://jypthemiracle.medium.com/java-stream-api%EB%8A%94-%EC%99%9C-for-loop%EB%B3%B4%EB%8B%A4-%EB%8A%90%EB%A6%B4%EA%B9%8C-50dec4b9974b)를 참고하자.

### 변수 접근

스트림을 이용하면서 람다(lambda) 또는 메서드 참조(method references)를 사용하는 경우에는 지역 변수(local variables)에 접근할 수 없다.

```java
int sumForLambda = 0;
for (int i = 0; i < 5; i++) {
    // it works!
    sumForLambda += i;
}

int sumForloop = 0;
IntStream.range(0, 5).forEach(i -> {
    // compile error
    sumForloop += i;
});
```

그리고 스트림의 파이프라인에서 연결된 각 단계의 값들에 접근할 수 없습니다. `peek`
 메서드로 연산 사이의 결과를 확인하고 싶지만 불가능하다.

```java
Arrays.stream(array)
        .filter(first -> first % 2 == 0)
        .filter(second -> second > 3)
        .peek(value -> {
            // compile error, can't access second filter's variable
            int printValue = value + second;
            System.out.println(printValue);
        })
        .sum();
```

### Stream 동작 순서

Stream의 동작 순서에 대해 알지 못하면 성능 이슈가 발생할 수 있다. 그렇기 때문에 우리가 작성한 Stream이 어떻게 동작할 것인지 정확히 이해하고 있는 것이 중요하다.

예를 들어 다음과 같은 코드가 있다고 할 때, 이를 실행한 결과를 한번 예측하고 확인해보자.

```java
Arrays.stream(new String[]{"c", "python", "java"})
        .filter(word -> {
            System.out.println("Call filter method: " + word);
            return word.length() > 3;
        })
        .map(word -> {
            System.out.println("Call map method: " + word);
            return word.substring(0, 3);
        }).findFirst();

/*
결과
Call filter method: c
Call filter method: python
Call map method: python
*/
```

3개의 요소에 대해서 메서드를 수행하므로 각각 3번의 호출이 이뤄질 것으로 예상했지만, 결과를 살펴보니 예상한 것과 다를 것이다.

왜냐하면 모든 요소에 대해 filter가 진행되고 map이 실행되는 것이 아니라, 각각 요소에 대해 filter와 map이 먼저 수행하는 수직적 구조로 순회하기 때문이다.

조금 더 자세히 살펴보면 아래와 같이 수직적 구조로 순회된다.

- 배열의 첫 번째 요소 `c`
    - filter 메서드가 실행되지만 length가 3보다 크지 않으므로 다음으로 진행되지는 않음
    - `Call filter method: c` 출력
- 배열의 두 번째 요소 `python`
    - filter 메서드가 실행되며 length가 3보다 크므로 다음으로 진행됨
    - `Call filter method: python` 출력
    - map 메서드에서 substring 메서드를 수행
    - `Call map method: python` 출력
    - 마지막 연산인 `findFirst`가 수행
- 조건에 맞는 하나의 결과를 찾았기 때문에 다음 요소인 `java`에 대해 연산을 수행하지 않는다.
    - 최종 결과는 `python`이다.

### 성능 개선

Stream의 동작 순서를 고려하면 성능을 개선할 수 있다. 잘못된 동작 순서는 연산 횟수를 불필요하게 증가시키기 때문이다.

```java
Stream.of("a", "b", "c", "d", "e")
        .map(s -> {
            System.out.println("map: " + s);
            return s.toUpperCase();
        })
        .filter(s -> {
            System.out.println("filter: " + s);
            return s.startsWith("A");
        })
        .forEach(s -> System.out.println("forEach: " + s));

/*
map: a
filter: A
forEach: A
map: b
filter: B
map: c
filter: C
map: d
filter: D
map: e
filter: E
*/
```

위의 코드는 map과 filter는 모든 문자열에 대해 각각 5번 불러졌고, forEach는 1번만 불러졌다. 

```java
Stream.of("a", "b", "c", "d", "e")
        .filter(s -> {
            System.out.println("filter: " + s);
            return s.startsWith("a");
        })
        .map(s -> {
            System.out.println("map: " + s);
            return s.toUpperCase();
        })
        .forEach(s -> System.out.println("forEach: " + s));

/*
filter: a
map: a
forEach: A
filter: b
filter: c
filter: d
filter: e
*/
```

위와 같이 수정된 코드는 filter가 5번, map과 forEach가 각각 1번씩 수행되었고, 동일한 입력과 결과에 대해 더 적은 연산으로 처리할 수 있게 되었다. 만약 처리해야 하는 데이터의 크기가 훨씬 많아지면 이는 성능의 저하의 원인이 될 것이다.

## 참고

- 이것이 자바다
- [https://madplay.github.io/post/mistakes-when-using-java-streams](https://madplay.github.io/post/mistakes-when-using-java-streams)
- [https://mangkyu.tistory.com/112](https://mangkyu.tistory.com/112)
- [https://codechacha.com/ko/java8-stream-limit-skip/](https://codechacha.com/ko/java8-stream-limit-skip/)
- [https://jypthemiracle.medium.com/java-stream-api는-왜-for-loop보다-느릴까-50dec4b9974b](https://jypthemiracle.medium.com/java-stream-api%EB%8A%94-%EC%99%9C-for-loop%EB%B3%B4%EB%8B%A4-%EB%8A%90%EB%A6%B4%EA%B9%8C-50dec4b9974b)

## 예상 면접 질문 및 답변

### Q. 스트림이란?

스트림(Stream)은 자바 8부터 추가된 컬렉션의 저장 요소를 하나씩 참조해서 람다식으로 처리할 수 있는 반복자입니다.

### Q. 스트림의 특징은?

- 스트림이 제공하는 대부분의 메소드는 함수형 인터페이스를 매개 타입으로 가지기 때문에 람다식으로 요소 처리 코드를 제공한다.
- 스트림은 원본 데이터를 조회하여 별도의 요소들로 스트림을 생성한다. 따라서 원본 데이터를 변경하지 않는다.
- 스트림은 내부 반복자를 사용하므로 병렬 처리가 쉽고, 개발자는 요소 처리 코드에만 집중할 수 있게 된다.
- 스트림은 컬렉션의 요소에 대해 중간 처리와 최종 처리를 수행할 수 있다.

### Q.스트림을 사용할 때 주의해야할 점은?

- 스트림을 재사용하지 않게 주의해야 한다.
- 스트림은 `for-loop`보다 성능이 느리다.
- 스트림을 이용하면서 람다 또는 메소드 참조를 사용하는 경우에는 지역 변수에 접근할 수 없다.
- 스트림의 동작 순서를 고려하지 않으면 불필요한 연산 횟수를 증가시킬 수 있다.
