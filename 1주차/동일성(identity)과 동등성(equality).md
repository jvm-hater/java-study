## 동일성

동일성은 동일하다는 뜻으로 두 개의 객체가 완전히 같은 경우를 의미한다. 여기서 완전히 같다는 뜻은 두 객체가 사실상 하나의 객체로 봐도 무방하며, 주소 값이 같기 때문에 두 변수가 같은 객체를 가리키게 된다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F3ccdcfbe-6eb0-477f-9422-eeb5881b5644%2FUntitled.png?table=block&id=0ef66661-53c6-48c7-8bbe-fe18865adfec&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

위 예제에서 refVar1은 객체1을 가리키고 있고, refVar2와 refVar3는 객체2를 가리키고 있다. refvar2와 refVar3는 동일한 객체를 가리키고 있으므로, 두 변수는 동일하다고 이야기할 수 있다. 그리고 해당 변수가 동일한지 `==` 연산자를 통해 판별할 수 있다.

참고로 Primitive 타입은 객체가 아니라 주소가 없으므로 `==` 연산자를 사용하였을 때 내용이 같으면 동일하다고 말한다.

## 동등성

동등성은 동등하다는 뜻으로 두 개의 객체가 같은 정보를 갖고 있는 경우를 의미한다. 동등성은 변수가 참조하고 있는 객체의 주소가 서로 다르더라도 내용만 같으면 두 변수는 동등하다고 이야기할 수 있다. 동일하면 동등하지만, 동등하다고 동일한 것은 아니다. 그리고 해당 변수가 동등한지 `equals` 연산자를 통해 판별할 수 있다.

```java
String str1 = new String("aaa");
String str2 = new String("aaa");

System.out.println(str1 == str2);
System.out.println(str1.equals(str2));
```

new 키워드를 통해 다른 String 객체를 메모리에 할당하였으므로 str1과 str2가 가리키는 객체의 주소 값은 다르므로 동일하지 않다. 하지만 String이 재정의한 `equals()` 연산자에 의해 두 객체의 내용이 같으므로 동등하다고 할 수 있다.

## `equals` 연산자

어떠한 객체든 `equals` 연산자만 사용하면 두 객체에 대한 동등성 판단이 가능한 것은 아니다. 먼저 해당 연산자가 최초로 정의된 형태를 보자.

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

위 메소드는 모든 객체의 조상인 `Object` 객체에서 정의하고 있는 `equals()` 메소드이다. 반한 형태를 보면 알 수 있듯이, 단순히 동일성 비교를 하고 있다. 즉, 해당 메소드를 자식 객체에서 재정의하지 않는 이상 `equals()` 연산자는 `==` 연산자와 다르지 않다. 대표적으로 `equals()` 메소드를 재정의한 객체 몇 가지를 살펴 보자.

### String의 `equals` 연산자

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

String 클래스는 위와 같이 `equals()` 를 재정의하여 인자로 전달된 String의 문자열을 비교하고 있다. 코드를 보면 `==` 키워드를 통해 두 객체의 동일성 여부를 판단하고, 두 객체가 동일하지 않다면 String인지 여부를 판단한 뒤, 문자 하나 하나가 같은지 비교한다. 만약 모든 문자가 같다면 두 객체의 내용이 같은 것이므로 동등하다고 판별할 수 있다.

### Integer의 `equals` 연산자

```java
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
```

Integer 클래스는 위와 같이 `equals()` 를 재정의하여 인자로 전달된 Integer의 값을 비교하고 있다. 코드를 보면 먼저 인자가 Integer인지 확인하고, 내용에 해당하는 정수를  `==` 키워드를 통해 같은지 판단하고 있다. 만약 정수가 같다면 두 객체의 내용이 같은 것이므로 동등하다고 판별할 수 있다.

### Objects의 `equals` 연산자

```java
public static boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}
```

Objects 클래스는 위와 같이 `equals()` 를 재정의하여 두 객체가 동등한지 비교하는 기능을 수행하고 있다. `==` 연산자를 통해 두 객체가 동일하다면 동등하므로 true를 반환하고, 그렇지 않다면 첫 번째 매개 변수로 들어온 객체의 `equals()` 메소드를 호출하여 동등성을 판단한다. 해당 메소드는 전적으로 첫 번째 매개 변수로 들어온 객체에 의존하는 것을 알 수 있다. 만약 첫 번째 매개 변수로 들어온 객체가 `equals()` 메소드를 재정의하지 않았다면 동일성 비교를 하게 될 것이다. 즉 개발자가 커스텀 객체를 만들었을 때, 해당 객체에 동등성 비교가 필요하다면 반드시 `equals()` 메소드를 재정의해 주어야 한다.

### ArrayList의 `equals` 연산자

```java
private boolean equalsArrayList(ArrayList<?> other) {
    final int otherModCount = other.modCount;
    final int s = size;
    boolean equal;
    if (equal = (s == other.size)) {
        final Object[] otherEs = other.elementData;
        final Object[] es = elementData;
        if (s > es.length || s > otherEs.length) {
            throw new ConcurrentModificationException();
        }
        for (int i = 0; i < s; i++) {
            if (!Objects.equals(es[i], otherEs[i])) {
                equal = false;
                break;
            }
        }
    }
    other.checkForComodification(otherModCount);
    return equal;
}
```

코드는 더 복잡하지만 핵심이 되는 로직만 가져왔다. 전부 이해할 필요 없이 흐름만 보자면, 자신의 element (객체의 내용)와 매개 변수로 들어온 ArrayList의 element (객체의 내용)이 서로 같은지 비교하고 있다. 그리고 중간 중간 두 element가 같은지 판단하기 위해 `Objects.equals()` 메소드를 사용하고 있다. 즉 ArrayList의 `equals()` 메소드는 서로의 요소 하나 하나에 대해 동등성 비교를 수행하여 모든 요소가 동등하면 두 ArrayList는 동등하다고 판별한다.

## 참고

- [https://joont.tistory.com/143](https://joont.tistory.com/143)
- [https://codechacha.com/ko/java-string-compare/](https://codechacha.com/ko/java-string-compare/)

## 예상 면접 질문 및 답변

### 동일성과 동등성의 차이는 무엇인가?

두 객체가 할당된 메모리 주소가 같으면 동일하고, 두 객체의 내용이 같으면 동등하다고 말한다. 동일성은 `==` 연산자를 통해 판별할 수 있고, 동등성은 `equals` 연산자를 통해 판별할 수 있다.

### `==` 연산자와 `equals` 연산자의 차이는 무엇인가?

`==` 연산자는 객체의 동일성을 판별하기 위해 사용하며, `equals` 연산자는 두 객체의 동등성을 판별하기 위해 사용한다.

`equals` 연산자는 재정의하지 않으면 내부적으로  `==` 연산자와 같은 로직을 수행하므로 차이가 없다. 따라서 `equals` 연산자는 각 객체의 특성에 맞게 재정의를 해야 동등성의 기능을 수행한다.
