## String

String 클래스는 불변 객체이다. String 클래스의 문자열을 저장하는 char[]을 보면 final로 선언되어 있고, 해당 배열을 재할당하는 코드는 존재하지 않는다.

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. **/
    private final char value[];
    ...
}
```

따라서 한 번 할당한 문자열을 변경하는 것은 불가능하며, 더하기 연산을 하여 문자를 이어 붙일 때는 새로운 객체가 생성되어 재할당된다.

```java
String s = "hello";
System.out.println(s.hashCode());  // 99162322
s += " world";
System.out.println(s.hashCode());  // 1776255224
```

위 예제에서 알 수 있듯이 hashCode가 달라지므로 두 객체는 다른 객체가 된다. 그런데, 반복적으로 문자열을 이어 붙이다 보면 Heap 영역에서 참조를 잃은 문자열 객체가 계속해서 쌓이게 된다. 물론, 나중에 GC에 의해 수거가 되지만 메모리 관리 측면에서 이러한 코드는 결코 좋은 코드라고 할 수 없다. 또한 계속해서 객체를 생성하므로 연산 속도 측면에서도 성능이 떨어질 수 밖에 없다.

### String Pool

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Facc89544-c288-4196-898a-bf6893b495ba%2FUntitled.png?table=block&id=7d548a89-acfa-49b4-99ce-962f4d03f273&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

`=` 연산자를 통해 값을 String에 대입하면 Heap 영역 내에 있는 String Pool이라는 공간에 문자열이 저장되고, `new` 연산자를 통해 String을 만들면 String Pool이 아닌 일반 Heap 영역 어딘가에 저장된다. 둘다 Heap 영역에 저장되는 것은 동일한데, String Pool에 값이 저장되면 어떠한 이점이 있는 것일까?

전자의 방식을 String literal이라고 하는데, String literal로 생성한 객체는 String Pool의 메모리 주소를 가리키게 된다. 그래서 똑같은 String literal 객체가 생성될 경우 같은 값의 주소를 가리키게 되므로 하나의 메모리를 재사용할 수 있다.

반면 후자는 일반적인 `new` 연산자를 통해 객체를 생성하는 방식이므로 String Pool의 해당 값이 있더라도 Heap 영역 내 별도의 메모리를 할당하여 주소를 가리키게 된다.

```java
String a = "Cat";
String b = "Cat";
String c = new String("Cat");
System.out.println(a == b);  // true
System.out.println(a == c);  // false
```

위 개념을 이해한 상태로 해당 예제 코드를 보면 이해가 갈 것이다. a와 b는 String Pool의 동일한 주소를 가리키고 있고, c는 별도로 생성한 메모리의 주소를 가리키므로 서로 참조하는 주소가 다르다는 사실을 기억하자.

**String의 불변성**

앞서 String은 불변 객체라고 이야기하였다. 만약 String이 가변 객체라면 String Pool을 사용할 수 없을 것이다. 왜 그럴까?

위 예제 코드에서 String Pool의 “Cat”이 위치한 주소를 a와 b가 가리키고 있는데, String이 가변 객체라면 `a = "Test"` 로 바꿔버리는 순간 b가 가리키는 값은 더 이상 “Cat”아닌 “Test”가 되어 버린다. 따라서 String Pool의 재활용성을 이용하기 위해 String은 불변 객체로 설계해야 한다. (물론 String을 불변 객체로 하였을 때, 이 외에도 여러 장점이 있는데 [해당 링크](https://steady-coding.tistory.com/559)를 참고하자.)

## StringBuilder

String Pool의 장점은 알겠지만, 어쨌든 문자열의 변화가 상당히 많아서 String 객체를 많이 생성하면 성능 이슈가 발생한다. 이럴 때는 String 객체를 하나로 두고, 내부 상태를 변경하는 가변 객체로 만드는 것이 좋은데, Java에서는 StringBuilder와 StringBuffer를 지원한다.

StringBuilder는 AbstractStringBuilder 클래스의 상속을 받는데, 아래와 같이 내부 상태를 변경할 수 있도록 설계가 되어 있다.

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {

    /** The value is used for character storage **/
    char[] value;
    ...
}
```

그리고 문자를 이어 붙이기 위해서는 `append()` 메소드를 호출하는데, `char[]` 배열의 길이를 늘리고 해당 배열에 문자열을 더하는 방식으로 구현되어 있다. 정확히 이야기하면, `char[]` 배열인 value에 사용되지 않고 남아 있는 공간에 새로운 문자열이 들어갈 정도의 크기가 있을  때는그대로 문자열을 삽입한다. 그렇지 않다면 value의 크기를 약 2배로 증가하여 기존의 문자열을 복사하고 새로운 문자열을 삽입한다. 아래 코드를 참고하자.

```java
    public AbstractStringBuilder append(String str) {
        if (str == null) str = "null";
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
		
    // (기존 문자열 길이 + 새로 들어 갈 문자열 길이)와 현재 value 길이를 비교
    private void ensureCapacityInternal(int minimumCapacity) {
        // overflow-conscious code
        if (minimumCapacity - value.length > 0)
            expandCapacity(minimumCapacity);
    }

    void expandCapacity(int minimumCapacity) {
        int newCapacity = value.length * 2 + 2;
        if (newCapacity - minimumCapacity < 0)
            newCapacity = minimumCapacity;
        if (newCapacity < 0) {
            if (minimumCapacity < 0) // overflow
                throw new OutOfMemoryError();
            newCapacity = Integer.MAX_VALUE;
        }
        value = Arrays.copyOf(value, newCapacity);
    }
```

그리고 String과 달리 StringBuilder 객체 자체를 새롭게 만드는 것은 아니므로 StringBuilder 객체의 주소는 유지가 된다.

```java
StringBuilder s = new StringBuilder("hello");
System.out.println(s.hashCode());  // 859417998
s.append("world");
System.out.println(s.hashCode());  // 859417998
```

위 예제처럼 hashCode의 값이 동일한 것을 알 수 있다. 문자열에 이어 붙이는 작업이 많다면 적극적으로 StringBuilder를 고려해 보자.

## StringBuffer

StringBuffer는 대부분의 메소드에 synchronized가 적용되어 일반적으로 멀티 스레드 환경에서 스레드 안전하게 동작한다. 한마디로 동기화를 지원하는 StringBuilder라고 생각하면 된다.

```java
@Override
public synchronized StringBuffer append(CharSequence s) {
    toStringCache = null;
    super.append(s);
    return this;
}
```

다만 무식하게 모든 메소드에 대해 synchronized를 통해 blocking을 거는 것은 성능 상으로 좋지 않다. 또한 synchronized 메소드 하나를 여러 스레드가 호출하는 것은 스레드 안전한 것이 맞지만, 여러 synchronized 메소드로 이루어진 하나의 메소드를 여러 스레드가 호출할 때는 스레드 안전하지 않을 수 있다. 아래 예시를 보자.

```java
public class StringBufferTest {

    private static final StringBuffer sb = new StringBuffer();

    public static void main(String[] args) throws InterruptedException {
        String[] names = {"A", "B", "C", "D", "E", "F", "G", "H", "I", "J"};
        String[] values = {"a", "b", "c", "d", "e", "f", "g", "h", "i", "j"};

        for (int i = 0; i < 10; i++) {
            final int j = i;
            Thread thread = new Thread(() -> addProperty(names[j], values[j]));
            thread.start();
        }
        new Thread(() -> System.out.println(sb.toString())).start();
    }

    public static void addProperty(String name, String value) {
        if (value != null && value.length() > 0) {
            if (sb.length() > 0) {
                sb.append(',');
            }
            sb.append(name).append('=').append(value);
        }
    }
}
```

위 코드에서 `addProperty()` 는 StringBuffer의 synchronized 처리 된 `length()` 메소드와 `append()` 메소드를 혼합하여 사용하고 있고, 총 10개의 스레드가 `addProperty()` 메소드를 호출하고 있다. 개발자가 의도한 방식은 사전 순으로 “A=a, B=b, ...”는 아니더라도, 적어도 “A=a, C=c, ...” 처럼 알파벳 간을 콤마(,)로 나누고 싶어할 것이다. (사용자의 컴퓨터 환경에 따라 동기화 문제가 발생하지 않을 수 있다는 점을 주의하라.)

```java
C=c,J=j,A=a,H=h,G=g,,E=Be=b,F=f,D=d,I=i
```

하지만 중간에 콤마(,)가 두 번 들어간 것을 확인할 수 있다. 이것이 발생하는 예상 시나리오는 아래와 같다.

1. 1번 스레드가 `sb.length()` 를 호출하였고, 나머지 스레드는 모두 blocked 상태가 되었다.
2. 그런데, 이미 StringBuffer에는 특정 문자가 들어가 있어서 1번 스레드는 조건문을 만족하여 `sb.append()` 를 호출하려고 하였으나, 그 찰나에 순간에 2번 스레드가 조건문 아래에 있는 `sb.append()` 를 호출하였다.
3. 그러면 StringBuffer에는 콤마가 들어가기도 전에 다른 문자가 들어가게 된다.
4. 2번 스레드의 작업을 마치면 1번 스레드가 `sb.append()` 를 호출하여 콤마를 StringBuffer에 집어 넣게 된다.

위 시나리오는 해당 예제의 결과 값에서 `E=Be=b` 를 나타낸 것이다. 이 결과는 심지어 대문자와 소문자 쌍까지도 맞지 않는 심각한 동기화 문제가 발생하였다.

따라서 해당 `addProperty()` 를 스레드 안전하게 사용하려면 아래와 같이 synchronized 블럭을 사용해야 한다.

```java
public class Main {

    private static final StringBuffer sb = new StringBuffer();

    public static void main(String[] args) {
        String[] names = {"A", "B", "C", "D", "E", "F", "G", "H", "I", "J"};
        String[] values = {"a", "b", "c", "d", "e", "f", "g", "h", "i", "j"};

        for (int i = 0; i < 10; i++) {
            final int j = i;
            Thread thread = new Thread(() -> addProperty(names[j], values[j]));
            thread.start();
        }
        new Thread(() -> System.out.println(sb.toString())).start();
    }

    public static void addProperty(String name, String value) {
        synchronized (sb) {
            if (value != null && value.length() > 0) {
                if (sb.length() > 0) {
                    sb.append(',');
                }
                sb.append(name).append('=').append(value);
            }
        }
    }
}
```

위와 같이 synchronized 블럭을 사용하면 블럭 내에 있는 연산에 대해 원자성을 보장해 줄 수 있으므로 스레드 안전하게 작업을 진행할 수 있다.

정리하자면, StringBuffer는 단일 synchronized 메소드를 여러 스레드가 사용하는 것은 스레드 안전하지만, 단일 synchronized 메소드 여러 개로 구성된 일반 메소드에서 사용할 때는 스레드 안전하지 않으므로 주의하여 사용해야 한다.

이러한 이유로 Vector는 이미 레거시 클래스가 되어 Concurrent 라이브러리를 사용하는 컬렉션으로 대체가 되었지만, StringBuffer의 대체재는 찾지 못하였다. 

## 출처

- [https://velog.io/@dnjscksdn98/Java-String-vs-StringBuilder-vs-StringBuffer](https://velog.io/@dnjscksdn98/Java-String-vs-StringBuilder-vs-StringBuffer)
- [https://starkying.tistory.com/entry/what-is-java-string-pool](https://starkying.tistory.com/entry/what-is-java-string-pool)
- [https://cjh5414.github.io/why-StringBuffer-and-StringBuilder-are-better-than-String/](https://cjh5414.github.io/why-StringBuffer-and-StringBuilder-are-better-than-String/)

## 예상 면접 질문 및 답변

### String Pool에 대해 설명하라.

String Pool은 Java Heap 영역에 존재하는 String 저장소로, String literal을 보관한다.

### String Pool이 있으면 무엇이 좋은가?

`=` 연산자로 String을 생성할 때, 같은 값이 String Pool에 있다면, 새로운 메모리 할당 없이 주소를 공유하여 사용할 수 있다. 또한, String은 불변 객체이므로 동일한 주소를 여러 객체가 참조하여도 동기화 문제가 발생하지 않으므로 안전하다.

### String과 StringBuilder의 차이는?

String은 불변 객체이므로 `+` 연산으로 문자열을 이어 붙이려고 해도 매번 새로운 객체를 할당해야 한다. 하지만, StringBuilder는 가변 객체이고 내부 `char[]` 배열의 사이즈를 조절하여 문자열을 이어 붙이기 때문에 새로운 객체를 최소로 할당할 수 있다.

### StringBuilder와 StringBuffer의 차이는?

StringBuilder는 동기화 처리가 되어 있지 않아 스레드 불안전하지만, StringBuffer는 대부분의 메소드에 대해 synchronized 처리가 되어 있으므로 일반적으로 스레드 안전하다.

### StringBuffer가 일반적으로 스레드 안전하다는 것은 무슨 뜻인가?

StringBuffer는 단일 synchronized 메소드를 여러 스레드가 사용하는 것은 스레드 안전하지만, 단일 synchronized 메소드 여러 개로 구성된 일반 메소드에서 사용할 때는 스레드 안전하지 않다. (자세한 예시는 본문 참조)
