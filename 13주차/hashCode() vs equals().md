## equals()

`equals()` 는 객체의 내용이 같은지 비교한다. 흔히 동등 비교라고 하며, `equals()` 를 재정의하지 않을 경우 내부적으로 `==`와 같으므로 동일 비교가 된다. 따라서 올바르게 객체를 동등 비교하고 싶다면 반드시 `equals()` 를 사용해야 한다.

```java
public boolean equals(Object obj) { 
    return (this == obj); 
}
```

## hashCode()

`hashCode()` 는 두 객체가 같은 객체인지 확인한다. `==` 과 같은 동일 비교 기능을 하지만, `hashCode()` 메소드는 반환 값으로 런타임 중 객체의 유일한 정수 값을 반환한다. 일반적으로 Heap에 저장된 객체의 메모리 주소를 반환한다.

```java
public native int hashCode();
```

해당 메소드는 native 키워드가 붙여져 있는데, 이것은 Java 외의 언어에서 개발된 언어를 Java에서 사용할 경우 사용하는 키워드이다. 

### 해시란?

해싱은 해시 함수를 사용하여 가변 크기의 입력 값에서 고정된 크기의 출력 값을 생성하는 과정을 의미한다. 해싱을 통해 얻어 온 값을 해시 코드라고 한다.

## equals()와 hashCode()

동일한 객체는 동일한 메모리 주소를 가져야 하므로, 동일한 객체는 동일한 해시 코드를 가져야 한다는 것은 자명하다. 따라서 `equals()` 말고 `hashCode()` 도 같이 재정의하여 동일한 해시 코드를 보장하도록 코드를 짜는 것이 바람직하다.

### 해시 자료 구조

```java
public class Main {

    public static void main(String[] args) throws IOException {
        Set<Person> people = new HashSet<>();

        people.add(new Person("제이온", 23));
        people.add(new Person("제이온", 23));
        System.out.println(people.size()); // 2
    }
}

class Person {

    private final String name;

    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {

        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }
}
```

위와 같이 HashSet 자료 구조에 동등한 객체 2개를 삽입해 보자. 해당 Set의 사이즈를 출력하면 2가 나올 것이다. 왜 그럴까?

바로 해시를 사용한 자료 구조는 Key를 결정할 때 `hashCode()` 를 사용하기 때문이다. 즉, 객체가 동일한지 비교하기 전에, 두 객체의 해시 코드가 같은지 비교하고 그 후 두 객체가 동등한지 판단한다. 이때, `hashCode()` 가 재정의되어 있지 않다면 Object의 `hashCode()`를 사용하므로 각 객체가 저장된 메모리 주소가 반환된다. 따라서 해시 자료 구조를 사용하는 경우를 위해 `equals()` 외에 `hashCode()` 도 재정의해 주는 것이 좋다.

```java
public static void main(String[] args) throws IOException {
        Set<Person> people = new HashSet<>();

        people.add(new Person("제이온", 23));
        people.add(new Person("제이온", 23));
        System.out.println(people.size()); // 1
    }
}

class Person {

    private final String name;

    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {

        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

## 출처

- [https://nesoy.github.io/articles/2018-06/Java-equals-hashcode](https://nesoy.github.io/articles/2018-06/Java-equals-hashcode)
- [https://mangkyu.tistory.com/101](https://mangkyu.tistory.com/101)

## 예상 면접 질문 및 답변

### equals()와 hashCode()는 왜 같이 사용하는가?

해시를 사용한 자료 구조는 Key를 결정할 때 `hashCode()` 를 사용하기 때문이다. 즉, 객체가 동일한지 비교하기 전에, 두 객체의 해시 코드가 같은지 비교하고 그 후 두 객체가 동등한지 판단한다. 이때, `hashCode()` 가 재정의되어 있지 않다면 Object의 `hashCode()`를 사용하므로 각 객체가 저장된 메모리 주소가 반환된다. 따라서 해시 자료 구조를 사용하는 경우를 위해 `equals()` 외에 `hashCode()` 도 재정의해 주는 것이 좋다.
