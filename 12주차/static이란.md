## Static의 개념

Static(정적)은 고정된이라는 의미를 가지고 있으며, Static 키워드를 통해 정적 변수와 정적 메소드를 만들 수 있다. 이렇게 만들어진 정적 변수나 정적 메소드는 프로그램이 종료되기 전까지 사용할 수 있고, GC에 의해 수집되지 않는다. 참고로, 정적 객체는 GC에 의해 수집될 수 있다.

동적은 객체를 런타임 도중에 힙 영역에 할당하는 행위를 말한다. 반대로, 정적은 프로그램이 시작되는 시점에 클래스 로더가 클래스를 해석하여 메소드 영역 혹은 힙 영역에 클래스 메타 데이터 및 정적 변수를 적재한다.

## Static이 저장되는 위치

static이 저장되는 위치를 알기 전에 permanent 영역과 metaspace 영역을 이해해야 한다.

### Permanent vs Metaspace

- Permanent
    - Java 8 이전까지 존재했던 메소드 영역
    - 클래스 내부의 메타 데이터를 저장하는 영역
    - Heap 영역에 속하며 클래스, 메소드 메타 데이터, 정적 객체, 정적 변수, 상수 풀 등을 관리했다.
    - Java 8 이후부터는 Metaspace 영역으로 대체되었다.
    - 특이하게 Heap 영역 안에 존재했던 메소드 영역임을 기억하자.
- Metaspace
    - Java 8부터 생긴 영역으로 permanent 영역이 관리하던 일부 정보를 저장한다.
    - 메소드 영역이 아니라 네이티브 메모리 영역으로서 JVM이 아닌 OS에서 관리한다.
    - 메소드 영역이 네이티브 메모리 영역에 포함된다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F60524bcb-ec9d-4d84-8fb0-a0c0565981d9%2FUntitled.png?table=block&id=8467d77f-8dbb-4b8f-bc06-e5f1ab2c8052&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

### Java 8 이전

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F93e6f736-2f7f-4256-ac1d-037f2bc63f74%2FUntitled.png?table=block&id=68b20cef-6337-47ba-885c-55883b3dc8ef&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Java 8 이전의 힙을 보면, Permanent 영역이 존재하고 이 안에 클래스 메타 데이터, 정적 변수 등이 저장된다. 이때, Permanent 영역은 메소드 영역에 해당하므로 Java 8 이전 static 변수는 메소드 영역에 저장되는 것이 맞다.

### Java 8 이후

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fbb28b74a-13b8-4336-80fa-34a03fe7d74f%2FUntitled.png?table=block&id=ac13369b-47ab-41fa-8230-80462e538795&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Java 8 이후의 힙을 보면, Permanent 영역이 사라졌다. 대신 힙 외부에서 Metaspace라고 하는 네이티브 메모리에서 Permanent가 저장하던 정보를 들고 있도록 바뀌었다. 이때 주의할 점은 permanent가 저장하던 정보를 전부 Metaspace에서 관리하는 것은 아니다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F5988b6cb-e899-4816-b5df-a1e28229b38e%2FUntitled.png?table=block&id=9974aae4-9120-4de7-84a9-9d82e137cf07&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

JDK 8의 레퍼런스를 살펴 보자면, Permanent 영역에서 관리하던 클래스 메타 데이터는 Metaspace로 옮겨지고 interned String과 클래스 정적 변수는 힙 영역에 옮겨진다는 사실을 알 수 있다. 즉, java 8 이후부터 static을 힙 영역에서 관리하게 된다.

### 왜 Permanent 영역이 사라졌을까?

이유는 단순하다. 각종 클래스 메타 데이터를 힙 영역에서 관리하다 보니, 힙 영역에 들어간 데이터의 사이즈를 계산하기 어려워서 OOM이 자주 발생하곤 했다. 그래서 클래스 메타 데이터는 네이티브 메모리에서 관리하고, 정적 변수만 힙 영역에서 관리하게 된 것이다.

### Garbage Collection

Java 8 이전이든 이후든, 정적 객체는 GC 수집에 대상이 된다. 그래서 프로그램 전체 동안 정적 변수가 사라지지 않는다는 것은 일부만 맞는 말이고, 참조를 잃은 정적 객체는 언제나 GC 수집에 대상이 될 수 있다. 당연한 이야기지만 primitive 타입의 정적 변수는 프로그램 종료 전까지 살아남게 된다.

## Static을 지양해야 하는 이유

- 메모리 문제
    - static은 프로그램 실행 시점에 메모리에 할당을 하며, 웬만하면 프로그램 종료 시점까지 메모리에서 해제되지 않는다.
- 동시성 이슈 문제
    - static은 전역에서 접근이 가능하므로 별도의 동기화 전략을 수립해야 한다.
- 런타임 다형성 불가
    - static으로만 이루어진 메소드를 사용하는 객체의 경우, 해당 객체를 메모리로 할당하여 사용하는 것이 아니고 `객체.메소드`로 바로 접근하여 호출한다.
- 객체의 상태를 이용할 수 없다.
    - 정적 메소드를 사용하기 위해서는 필요로 하는 인자를 모두 외부에서 주입해야 한다. 정적 메소드 안에는 클래스의 인스턴스 필드를 사용할 수 없기 때문이다.
        - static은 프로그램 시점에 메모리에 올라가는데, 정적 메소드 안에 객체의 인스턴스 필드가 초기화되지 않았다면 문제가 생길 수 있다. 그래서 정적 메소드 안에는 정적 변수만 사용할 수 있다.
    - 일반 메소드라면 객체 내의 있는 상태를 통해 해당 메소드를 구현해 줄 수 있으므로 변화하는 상태에 따라 다채로운 기능 구현이 가능하다.
    - 결국 객체 내의 정적 메소드가 많아지면, 외부 값에 의존하는 수동적인 객체가 되어 버린다.
- 테스트하기 어려움
    - 정적 필드는 전역으로 관리되기 때문에 프로그램 전체에서 이 필드에 접근하고 수정할 수 있다. 따라서 해당 필드를 추론하기 어려워 테스트하기 까다롭다.

## Static은 언제 사용할까?

위 단점만 보면 static은 절대 쓰면 안될 것처럼 보인다. 필자의 개인적인 생각으로는 static은 되도록 지양하는 것이 좋으나, 사용하면 좋은 상황들이 몇몇 있다.

**상수 정의**

절대 변하지 않는 변수를 상수라고 하는데, 해당 상수는 객체 내에서 매번 일반 변수로 정의하기 보다는 한 번 정적 변수로 정의하면 메모리를 아낄 수 있다.

```java
private static final String NAME = "제이온";
```

**유틸리티 클래스 정의**

유틸리티  클래스는 인스턴스 메소드와 인스턴스 변수를 제공하지 않고, 데이터 처리를 위한 정적 메소드만 존재하는 클래스를 말한다. java에서 Math 클래스를 보면, 상수 외에 인스턴스 변수가 하나도 없고 계산을 위한 정적 메소드만 제공한다.

이렇게 애초부터 객체의 상태를 이용할 생각이 없고, 여러 객체들의 필요에 의해 데이터를 처리하는 공통 로직이 필요할 때는 static을 사용하여 설계한다.

## 출처

- [https://mangkyu.tistory.com/47](https://mangkyu.tistory.com/47)
- [https://jgrammer.tistory.com/144](https://jgrammer.tistory.com/144)
- [https://www.scaler.com/topics/static-variable-in-java/](https://www.scaler.com/topics/static-variable-in-java/)
- [https://goodgid.github.io/Java-8-JVM-Metaspace/](https://goodgid.github.io/Java-8-JVM-Metaspace/)
- [https://vaert.tistory.com/101](https://vaert.tistory.com/101)
- [https://kellis.tistory.com/127](https://kellis.tistory.com/127)

## 예상 면접 질문 및 답변

### static이란?

Static(정적)은 고정된이라는 의미를 가지고 있으며, 프로그램이 시작되는 시점에 클래스 로더가 클래스를 해석하여 메소드 영역 혹은 힙 영역에 클래스 메타 데이터 및 정적 변수 등을 적재한다.

### static은 어디에 저장되는가?

java 8 이전에는 메소드 영역인 Permanent 영역, Java 8 이후에는 힙 영역에 저장된다.

### static 사용을 왜 지양해야 하는가?

- 메모리 문제
- 동시성 이슈 문제
- 런타임 다형성 불가
- 객체의 상태를 이용할 수 없다.
- 테스트하기 어려움

### static은 언제 사용하는가?

- 상수 정의
- 유틸리티 클래스 정의
