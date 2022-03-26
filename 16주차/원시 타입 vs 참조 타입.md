## 원시 타입

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2cdfbdc7-a14c-452a-9cb1-b747b0dd8e19%2FUntitled.png?table=block&id=c9288a45-8d4b-4829-99d9-1180fd4a8e59&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

원시 타입은 정수, 실수, 문자, 논리 리터럴 등 실제 데이터 값을 저장하는 타입이다. `int a = 10;` 와 같이 코드를 작성했다면 정수 값이 할당될 수 있는 a라는 이름의 메모리 공간이 JVM 스택 영역에 생성되고, 10이라는 값이 들어간다. 즉, 원시 타입은 메모리 공간의 실제 데이터 값이 저장되어 있다.

### char 연산 시 주의할 점

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2848be39-05ae-41b3-98bb-b139ef7811ae%2FUntitled.png?table=block&id=85b30912-ed12-447a-8cc2-747795f9413c&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

JVM은 연산할 때 피연산자 스택을 활용한다. 예를 들어 `13 + 20 + 7` 을 계산한다면, `20 + 7` 을 계산한 결과를 스택에 집어 넣고, 다시 `27 + 13` 을 계산한 결과를 스택에 집어 넣는 방식을 반복한다. 이때 주의할 점은 JVM의 피연산자 스택은 피연산자를 4 Bytes 단위로 저장한다는 것이다. 즉, char나 short와 같이 int보다 작은 자료형의 값을 계산하면 int형으로 자동 형 변환되어 연산이 수행된다.

## 참조 타입

참조 타입은 기본 타입을 제외한 타입으로, 객체의 주소를 저장하는 타입이다. 문자열, 배열, 열거형 상수, 클래스, 인터페이스 등이 있다. Java에서 실제 객체는 JVM 힙 영역에 저장되며, 참조 타입 변수는 실제 객체의 주소를 JVM 스택 영역에 저장한다. 그리고 객체를 사용할 때마다 참조 변수에 저장된 객체의 주소를 불러와 사용하게 된다.

`Person p = new Person();` 이라는 코드를 작성했다면 p라는 이름의 메모리 공간이 스택 영역에 생성되고, 생성된 p의 인스턴스는 힙 영역에 생성된다, 즉, 스택 영역에 생성된 참조 변수 p는 힙 영역에 생성된 p의 인스턴스 주소 값을 가지게 된다.

## 원시 타입 vs 참조 타입

### 성능 관점

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F84c0a7a7-3b02-4b02-a08a-9fd006833435%2FUntitled.png?table=block&id=29a2c753-afd7-4296-a0be-8a11b21e4df2&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

원시 타입은 스택 영역에 존재한다. 반면 참조 타입은 스택 영역에는 참조 값만 있고, 실제 값은 힙 영역에 존재한다. 참조 타입은 최소 2번 메모리 접근을 해야 하고, 일부 타입의 경우 값을 필요로 할 때 언박싱 과정(ex. Double → double, Integer → int)을 거쳐야 하므로 원시 타입과 비교해서 접근 속도가 느린 편이다.

### 메모리 관점

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F10c82cc4-8f25-4266-870f-d79fa2fdaef3%2FUntitled.png?table=block&id=849fae95-8225-44be-92d9-ad0c1fc6e7f8&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

원시 타입보다 참조 타입이 사용하는 메모리 양이 압도적으로 높다. 이외의 참조 타입은 최근 들어 64 비트의 JVM을 많이 사용하므로 일반적으로 64 bits를 차지한다고 한다.

### NULL 관점

원시 타입은 null을 담을 수 없지만, 참조 타입은 null을 담을 수 있다. 이것은 원시 타입의 경우, 값이 없으면 디폴트 값을 반환하기 때문이다. (ex. int은 0, boolean은 false)

### 제네릭 관점

원시 타입은 제네릭 타입에서 사용할 수 없지만, 참조 타입은 가능하다.

## 출처

- [https://velog.io/@gillog/원시타입-참조타입Primitive-Type-Reference-Type](https://velog.io/@gillog/%EC%9B%90%EC%8B%9C%ED%83%80%EC%9E%85-%EC%B0%B8%EC%A1%B0%ED%83%80%EC%9E%85Primitive-Type-Reference-Type)
- [https://week-year.tistory.com/141](https://week-year.tistory.com/141)
- [https://www.korecmblog.com/jvm-stack-and-register/](https://www.korecmblog.com/jvm-stack-and-register/)
- [https://kjw1313.tistory.com/24](https://kjw1313.tistory.com/24)
- [https://stackoverflow.com/questions/5350950/reference-type-size-in-java](https://stackoverflow.com/questions/5350950/reference-type-size-in-java)

## 예상 면접 질문 및 답변

### 원시 타입의 크기를 설명하라.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fce7d4153-972f-4ee9-b430-ad907a4e1bb0%2FUntitled.png?table=block&id=60c51e20-41c7-4fee-bc2f-82ba2ddb534b&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

### 원시 타입과 참조 타입의 차이를 설명하라.

- 성능 관점
    - 원시 타입은 스택 영역에 존재한다. 반면 참조 타입은 스택 영역에는 참조 값만 있고, 실제 값은 힙 영역에 존재한다. 참조 타입은 최소 2번 메모리 접근을 해야 하고, 일부 타입의 경우 값을 필요로 할 때 언박싱 과정(ex. Double → double, Integer → int)을 거쳐야 하므로 원시 타입과 비교해서 접근 속도가 느린 편이다.
- 메모리 관점
    - 원시 타입보다 참조 타입이 사용하는 메모리 양이 압도적으로 높다. 이외의 참조 타입은 최근 들어 64 비트의 JVM을 많이 사용하므로 일반적으로 64 bits를 차지한다고 한다.
- NULL 관점
    - 원시 타입은 null을 담을 수 없지만, 참조 타입은 null을 담을 수 있다. 이것은 원시 타입의 경우, 값이 없으면 디폴트 값을 반환하기 때문이다. (ex. int은 0, boolean은 false)
- 제네릭 관점
    - 원시 타입은 제네릭 타입에서 사용할 수 없지만, 참조 타입은 가능하다.
