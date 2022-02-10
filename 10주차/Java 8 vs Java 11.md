# Java 8 vs Java 11

## Java란?

Java는 썬 마이크로시스템즈의 제임스 고슬링과 다른 연구원들이 개발한 객체지향적 프로그래밍 언어이다.

### Java 언어의 특징

- 객체지향 언어이다.
    - 유지 보수가 쉽고 확장성이 높다.
- 이식성이 높다.
    - JVM에서 작동하기 때문에 운영 체제의 종류에 상관없이 작동한다.
- 메모리를 자동으로 관리한다.
    - 프로그램이 C/C++에 비해 안정적이다.
- 멀티 스레드 구현이 쉽다.
    - 스레드 생성 및 제어와 관련된 라이브러리 API를 제공한다.
- 동적 로딩을 지원한다.
    - 실행 시에 모든 클래스가 로딩되는 것이 아니라 필요한 시점에 클래스를 로딩한다.
    - 애플리케이션의 변경 사항도 비교적 적은 작업으로 처리가 가능하다.
- 분산 환경을 지원한다.
    - 기본적으로 TCP/IP 라이브러리가 포함되어 있고, HTTP 프로토콜을 지원한다.
- 오픈 소스 라이브러리가 풍부하다.

### Java 언어의 단점

- JVM에 의해 기계어로 번역되고 실행되는 과정을 거쳐서, 컴파일되자마자 기계어로 변환되는 C,C++에 비하면 속도가 현저히 떨어진다
    - 다만 최근에는 JIT 컴파일 방식으로 자바 컴파일러를 개선하여 속도가 많이 빨라지긴 했다.
- 다른 언어에 비해 작성해야 하는 코드의 길이가 긴 편이다.
- 프로그램 발생 시 발생할 수 있는 예외를 개발자가 직접 처리해야 한다.

### Java를 왜 사용하는가? (개인적인 의견)

Java는 객체지향 프로그래밍을 가장 대중적이고 편하게 구현할 수 있는 언어라고 생각하며, 많은 엔터프라이즈급 회사에서 오랜 기간 사용된 만큼 안정성이 보장된 언어라고 판단했다. 또한 자바 진영은 하위 호환성을 중요하게 여기는 점과 레퍼런스와 커뮤니티가 활성화 되어있는 점이 매력적으로 다가왔다. 이러한 이유로 필자는 Java를 주 언어로 사용하여 개발하고 있다.

## Java 8의 특징

### ****Heap Permanent Generation 제거****

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fdbbe53c2-fdd7-446e-b0ec-692dc5468910%2FUntitled.png?table=block&id=22e810bf-af2a-4f74-a69c-64ca7a613f7e&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Java 8 이전에는 초기 설정할 때, PermSize와 MaxPermSize를 설정해 주어야 했으나, Java 8부터는 Permanent Generation이 Metaspace로 대체되었다. Metaspace는 런타임 시 메모리 요구 사항에 따라 자체 크기를 조정하며, 필요하다면 `MaxMetaspaceSize` 매개 변수를 설정하여 Metaspace의 양을 조절할 수 있다.

**Permanent Generation**

- Permanet Generation은 Class 혹은 Method Code가 저장되는 영역이다.
- PermGen은 Heap 영역에 속한다.
- Default로 제한된 크기를 갖고 있다.

**Metaspace**

- Metaspace는 Java의 클래스 로더가 현재까지 로드한 Class들의 메타데이터가 저장되는 공간이다.
- JVM에 의해 관리되는 Heap 영역이 아니라 OS 레벨에서 관리되는 Native 메모리 영역에 위치한다.
    - 따라서 개발자는 영역 확보의 상한을 크게 의식할 필요가 없다. 단지 PermGen이 빠진 Heap 영역만 잘 관리하면 된다.
- Default로 제한된 크기를 가지고 있지 않고, 필요한 만큼 늘어난다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2915919c-d2f1-496d-8355-131e53953ec4%2FUntitled.png?table=block&id=424ca8b8-fed1-4461-9f7f-77cd8d254167&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Java 8부터는 PermGen 영역이 제거되고 대신 Metaspace가 대체되어 위와 같은 구조를 띈다.

### Interface Default and Static Methods

인터페이스에 디폴트 메소드와 정적 메소드를 포함할 수 있게 되었다.

```java
interface TestInterface {

    default String logic() {
        return "test";
    }

    static String logic2() {
        return "test2";
    }
}
```

디폴트 메소드는 하위 구현 클래스에서 재정의가 가능하지만, 정적 메소드는 구현 클래스에서 재정의할 수 없다.

### 함수형 인터페이스

하나의 기능을 제공하는 단 하나의 추상 메소드를 정의하는 인터페이스가 추가되었다. 단, 추상 메소드 외에 디폴트 메소드나 정적 메소드는 원하는 만큼 사용할 수 있다. @FunctionalInterface와 함께 쓰이며, 만약 추상 메소드가 여러 개라면 컴파일 타임에서 에러를 잡아낼 수 있다.

주요 함수형 인터페이스는 Consumer, Supplier, Function 등이 있다.

### 람다 표현식

람다식은 메소드를 하나의 식으로 표현한 것을 말한다. 그리고 람다식은 함수의 이름이 없기 때문에 익명 함수라고 부르며, 메소드의 매개 변수로 전달되거나 메소드의 결과로 반환될 수 있는 특징이 있어서 함수를 변수로 다룰 수 있다는 장점이 있다. 

람다식은 메소드를 식으로 나타낸 것이긴 하지만, 엄밀히 말하면 이 메소드를 가진 객체를 생성해 낸다는 사실을 기억해야 한다. 그래서 주로 함수형 인터페이스의 익명 객체를 대체하기 위해 람다식을 사용하며, 간결하고 부수 효과가 없는 코드를 만들어 낼 수 있게 된다.

```java
// 외부 반복
for (String value: myCollection) { 
    System.out.println(value); 
}

// 내부 반복
myCollection.forEach(value -> System.out.println(value));
```

위 코드에서 `myCollection` 이 List라고 했을 때, `forEach()` 메소드의 인자는 Consumer가 된다. 람다식이 아닌 익명 객체를 구현해도 되지만, Java 8부터는 람다식을 사용하여 위와 같이 간결하게 표현할 수 있게 된다.

### 메소드 참조

메소드 참조는 람다식을 좀 더 짧고 읽기 쉽게 사용하는 것이다. 메소드 참조에는 4가지 방법이 있다.

**정적 메서드 참조**

정적 메서드에 대한 참조는 `클래스::메소드명`으로 사용된다. 먼저 람다식을 사용한 경우를 살펴보면 아래와 같다.

```java
boolean isReal = list.stream().anyMatch(u -> User.isRealUser(u));
```

이를 메서드 참조로 변경하면

```java
boolean isReal = list.stream().anyMatch(User::isRealUser);
```

위와 같이 수정할 수 있다.

**인스턴스 메서드 참조**

인스턴스 메서드에 대한 참조는 `인스턴스::메소드명`으로 사용된다. 아래 코드는 User의 `isLegalName(String string)`을 호출한 것을 뜻한다.

```java
User user = new User();
boolean isLegalName = list.stream().anyMatch(user::isLegalName);
```

**특정 유형의 객체의 인스턴스 메서드 참조**

특정 유형(String, BigDecimal 등)의 인스턴스 메소드에 대해 메소드 참조를 사용할 수 있다.

```java
long count = list.stream().filter(String::isEmpty).count();
```

**생성자 참조**

생성자 참조는 `클래스명::new`로 사용한다. 생성자는 특수한 메소드이기 대문에, 메소드 이름으로 new를 사용한다.

```java
Stream<User> stream = list.stream().map(User::new);
```

### Stream API

스트림은 컬렉션의 저장 요소를 하나씩 참조해서 람다식으로 처리할 수 있도록 해 주는 내부 반복자이다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F3f0a4cd7-41f0-4642-a29c-00a019334bfd%2FUntitled.png?table=block&id=5fdb4458-7fac-4da9-a90e-b2891b0bf1bf&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

스트림은 람다식으로 요소 처리 내용만 전달할 뿐, 반복은 컬렉션 내부에서 일어난다. 따라서 개발자 입장에서 작성할 코드가 간결해진다.

**스트림의 특징**

- 스트림 API의 특징에는 선언형, 조립할 수 있음, 병렬화가 있다.
- 스트림은 연속된 요소, 중간 연산, 최종 연산으로 이루어진다.
- 자바 컬렉션은 외부 반복, 스트림은 내부 반복을 사용한다.
- 중간 연산은 파이프라인으로 구성되어 최종 연산에서 한 번에 처리된다. 이를 지연 계산된다고 한다.
- 중간 연산은 Stream을 반환하고, 최종 연산은 스트림이 아닌 결과을 반환한다.
- 중간 연산의 예시로는 map, filter, flatMap 가 있고, 최종 연산의 예시로는 count, foreach, collect가 있다.

**데이터 컬렉션과 스트림의 차이는?**

컬렉션은 데이터를 어떻게 저장 및 관리하고 접근하는 지를 목표하고 있지만, 스트림은 데이터를 직접 접근하거나 조작하는 기능을 제공하지 않고 데이터를 어떻게 계산할 지를 목표로 하고 있다.

### Date and Time API 지원

Java 8부터 LocalDate, LocalTime, LocalDateTime 등의 라이브러리를 통해 기존보다 훨씬 쉽게 날짜 관련 로직을 작성할 수 있게 되었다.

```java
// Java 8 이전 - 오늘 날짜 구하기
Calendar cal = Calendar.getInstance();
String format = "yyyy-MM-dd";
SimpleDateFormat sdf = new SimpleDateFormat(format);
String date = sdf.format(cal.getTime());
System.out.println(date);

// Java 8 이후 - 오늘 날짜 구하기
LocalDate now = LocalDate.now();
System.out.println(now);
```

또한, 기존에 사용하던 날짜 관련 클래스인 Date, Calendar 등은 가변 객체이므로 Thread-Safe하지 않다.

### Optional 지원

Optional을 통해 null에 대한 참조를 안전하게 할 수 있다. 이전에는 null이 아닌지 검사하기 위한 조건문을 수두룩하게 사용했다면, Java 8부터 Optional을 통해 NPE를 얻을 상황을 손 쉽게 처리할 수 있게 되었다.

```java
// Java 8 이전
public String logic() {
    User user = getUser();
    if (user != null) {
        Address address = user.getAddress();
        if (address != null) {
            String street = address.getStreet();
            if (street != null) {
                return street;
            }
        }
    }
    return "not specified";
}

// Java 8 이후
public String logic() {
    Optional<User> user = Optional.ofNullable(getUser());
    String result = user
      .map(User::getAddress)
      .map(Address::getStreet)
      .orElse("not specified");
    return result;
}
```

### 배열 정렬의 병렬 처리

배열을 정리할 때는 `Arrays.sort()` 메소드를 사용했는데, 이 메소드는 정렬이 순차적으로 실행된다는 단점이 있었다. Java 8부터는 `Arrays.parallelSort()` 를 통해 병렬적으로 배열을 정렬할 수 있다.

### Garbage Collector

Java 8의 Default GC는 Paralle GC이다.

## Java 11

### String 클래스에 새로운 메소드 추가

다음 6가지 메소드가 추가 되었다.

- `strip()`: 문자열 앞, 뒤의 공백 제거.
- `stripLeading()`: 문자열 앞의 공백 제거.
- `stripTrailing()`: 문자열 뒤의 공백 제거.
- `isBlank()`: 문자열이 비어있거나, 공백만 포함되어 있을 경우 true를 반환한다.
    - `String.trim().isEmpty()` 와 결과가 동일함.
- `repeat(n)`: n개만큼 문자열을 반복하여 붙여서 반환함.

### java.nio.file.Files 클래스에 새로운 메소드 추가

다음 3가지 메소드가 추가 되었다.

- `Path writeString(Path, String, Charset, OpenOption)`: 파일에 문자열을 작성하고 Path로 반환한다. 파일 오픈 옵션에 따라 작동 방식을 달리하며, charset을 지정하지 않으면 UTF-8이 사용된다.
- `String readString(Path, Charset)`: 파일 전체 내용을 읽어서 String으로 반환하고, 파일 내용을 모두 읽거나 예외가 발생하면 알아서 close를 한다. charset을 지정하지 않으면 UTF-8이 사용된다.
- `boolean isSameFile(Path, Path)`: 두 Path가 같은 파일을 가리키며, true, 아니면 false를 반환한다.

### 컬렉션 인터페이스에 새로운 메소드 추가

컬렉션의 `toArray()` 메소드를 오버 로딩하는 메소드가 추가되었고, 원하는 타입의 배열을 선택하여 반환할 수 있게 되었다.

```java
List sampleList = Arrays.asList("Java", "Kotlin"); 
String[] sampleArray = sampleList.toArray(String[]::new); 
assertThat(sampleArray).containsExactly("Java", "Kotlin");
```

### Predicate 인터페이스에 새로운 메소드 추가

Predicate 인터페이스에 부정을 나타내는 `not()` 메소드가 추가되었다.

```java
List<String> sampleList = Arrays.asList("Java", "\n \n", "Kotlin", " "); 
List withoutBlanks = sampleList.stream() 
 .filter(Predicate.not(String::isBlank))
 .collect(Collectors.toList()); 
assertThat(withoutBlanks).containsExactly("Java", "Kotlin");
```

### 람다에서 로컬 변수 Var 사용

람다식에서 Var을 사용할 수 있게 되었다.

```java
List<String> sampleList = Arrays.asList("Java", "Kotlin"); 
String resultString = sampleList.stream() 
.map((@Nonnull var x) -> x.toUpperCase()) 
.collect(Collectors.joining(", ")); 
assertThat(resultString).isEqualTo("JAVA, KOTLIN");
```

### 자바 파일 실행

javac를 통해 컴파일 하지 않고도, 바로 java 파일을 실행할 수 있게 되었다.

```java
// Java 11 이전
$ javac HelloWorld.java
$ java Helloworld
Hello Java 8!

// Java 11 이후
$ java HelloWorld.java
Hello Java 11!
```

### Garbage Collector

Java 8의 Default GC는 G1 GC이다.

## Java 버전을 선택하는 기준 (개인적인 의견)

현재 기업을 보면 Java 8 혹은 Java 11을 주로 채택하여 사용하고 있는 것 같다. 개인적으로는 Java 8의 기능을 수용하면서 더 나은 라이브러리가 추가 되었고, 성능이 개선된 G1 GC를 사용하는 Java 11을 사용하는 편이다. 또한 Java 11은 LTS 버전이므로 장기적인 지원이 보장된 버전이므로 앞으로도 유익한 기능이 나올 것으로 기대한다.

## 출처

- [https://helloworld-88.tistory.com/3](https://helloworld-88.tistory.com/3)
- [https://goodgid.github.io/Java-8-JVM-Metaspace/](https://goodgid.github.io/Java-8-JVM-Metaspace/)
- [https://myhappyman.tistory.com/58](https://myhappyman.tistory.com/58)
- [https://livenow14.tistory.com/73](https://livenow14.tistory.com/73)
- [https://livenow14.tistory.com/81](https://livenow14.tistory.com/81)
- [https://parkcheolu.tistory.com/174](https://parkcheolu.tistory.com/174)

## 예상 면접 질문 및 답변

### Java 언어의 장단점을 설명하라.

- 장점
    - 객체지향 언어이다.
        - 유지 보수가 쉽고 확장성이 높다.
    - 이식성이 높다.
        - JVM에서 작동하기 때문에 운영 체제의 종류에 상관없이 작동한다.
    - 메모리를 자동으로 관리한다.
        - 프로그램이 C/C++에 비해 안정적이다.
    - 멀티 스레드 구현이 쉽다.
        - 스레드 생성 및 제어와 관련된 라이브러리 API를 제공한다.
    - 동적 로딩을 지원한다.
        - 실행 시에 모든 클래스가 로딩되는 것이 아니라 필요한 시점에 클래스를 로딩한다.
        - 애플리케이션의 변경 사항도 비교적 적은 작업으로 처리가 가능하다.
    - 분산 환경을 지원한다.
        - 기본적으로 TCP/IP 라이브러리가 포함되어 있고, HTTP 프로토콜을 지원한다.
    - 오픈 소스 라이브러리가 풍부하다.
- 단점
    - JVM에 의해 기계어로 번역되고 실행되는 과정을 거쳐서, 컴파일되자마자 기계어로 변환되는 C,C++에 비하면 속도가 현저히 떨어진다
        - 다만 최근에는 JIT 컴파일 방식으로 자바 컴파일러를 개선하여 속도가 많이 빨라지긴 했다.
    - 다른 언어에 비해 작성해야 하는 코드의 길이가 긴 편이다.
    - 프로그램 발생 시 발생할 수 있는 예외를 개발자가 직접 처리해야 한다.

### 왜 Java 언어를 사용하는가?

Java는 객체지향 프로그래밍을 가장 대중적이고 편하게 구현할 수 있는 언어라고 생각하며, 많은 엔터프라이즈급 회사에서 오랜 기간 사용된 만큼 안정성이 보장된 언어라고 판단했다. 또한 자바 진영은 하위 호환성을 중요하게 여기는 점과 레퍼런스와 커뮤니티가 활성화 되어있는 점이 매력적으로 다가왔다. 이러한 이유로 필자는 Java를 주 언어로 사용하여 개발하고 있다.

### Java 8에서 새로 추가된 기능을 설명하라.

- Heap Permanent Generation 제거
- 인터페이스에 디폴트 메소드와 정적 메소드 추가
- 함수형 인터페이스, 람다 표현식, 메소드 참조 기능 추가
- 스트림 API 도입
- 새로운 날짜 관련 라이브러리 추가
- Optional 지원
- 병렬 처리 지원

### Java 11에서 새로 추가된 기능을 설명하라.

- Spring에 새로운 메소드 추가
- Files 클래스에 새로운 메소드 추가
- 컬렉션 인터페이스에 새로운 메소드 추가
- Predicate 인터페이스에 새로운 메소드 추가
- 람다에서 로컬 변수 Var 사용
- 자바 파일 실행 방식 단순화

### Java 8과 Java 11 중 무엇을 선택할 것인가?

현재 기업을 보면 Java 8 혹은 Java 11을 주로 채택하여 사용하고 있는 것 같다. 개인적으로는 Java 8의 기능을 수용하면서 더 나은 라이브러리가 추가 되었고, 성능이 개선된 G1 GC를 사용하는 Java 11을 사용하는 편이다. 또한 Java 11은 LTS 버전이므로 장기적인 지원이 보장된 버전이므로 앞으로도 유익한 기능이 나올 것으로 기대한다.
