## 가변 객체 (Mutable Object)

가변 객체는 Java에서 Class의 인스턴스가 생성된 이후에 내부 상태가 변경 가능한 객체이다. 가변 객체는 멀티 스레드 환경에서 사용하려면 별도의 동기화 처리가 필요하며, 대표적인 가변 객체로 ArrayList, HashMap, StringBuilder, StringBuffer 등이 존재한다. 이외에도 프로그래머가 커스텀 객체를 생성하여 내부 상태를 변경할 수 있게 만든다면, 그것도 가변 객체가 된다.

## 불변 객체 (Immutable Object)

불변 객체는 가변 객체와 반대로 Java에서 Class의 인스턴스가 생성된 이후에 내부 상태를 변경할 수 없는 객체이다. 불변 객체는 멀티 스레드 환경에서도 안전하게 사용할 수 있다는 신뢰성을 보장하며, 대표적인 불변 객체로 String 등이 존재한다. 이외에도 프로그래머가 커스텀 객체를 생성하여 내부 상태가 변경되지 않게 만들면, 그것도 불변 객체가 된다.

### 불변 객체의 장점

- Thread-safe하여 병렬 프로그래밍에 유용하며, 동기화를 고려하지 않아도 된다.
    - 멀티 스레드 환경에서 동기화 문제가 발생하는 이유는 공유 자원에 동시 쓰기 연산 때문이다. 하지만 공유 자원 불변 객체라면, 항상 동일한 값만 반환하므로 동기화를 고려할 필요가 없다. 이는 안정성을 보장할 뿐만 아니라 동기화를 하지 않음으로써 성능 상의 이점도 가져다 준다.
- 실패 원자적인(Failure Atomic) 메소드를 만들 수 있다.
    - 가변 객체를 통해 어떠한 작업을 하는 도중 예외가 발생하면 해당 객체가 불안정한 상태에 빠질 수 있다. 그리고 불안정한 상태를 갖는 객체는 변경된 상태로 인해 새로운 에러를 유발할 수 있다. 하지만 불변 객체라면 어떠한 예외가 발생하여도 메소드 호출 전의 상태를 유지할 수 있으므로 예외가 발생하여도 변경된 상태로 인한 추가 에러를 막을 수 있다.
- Cache, Map, Set 등의 요소로 활용하기에 적합하다.
    - 만약 캐시나 Map 또는 Set 등으로 사용되는 객체가 변경되었다면 이를 갱신하는 등의 작업이 필요하다. 하지만 객체가 불변이라면 한 번 데이터가 저장된 이후에 다른 부가 작업을 고려하지 않아도 될 것이고, 이는 캐시나 다른 자료 구조를 사용하는데 용이하게 작용한다.
- 부수 효과(Side Effect)를 피해 오류 가능성을 최소화할 수 있다.
    - 부수 효과란 변수의 값이 변경되거나, 필드 값이 설정되는 등의 변화가 발생하는 효과를 의미한다. 만약 객체의 수정자(Setter)가 구현되어 있고, 여러 메소드에서 객체의 값이 변경된다면 객체를 예측하기 어려워진다. 그래서 이러한 부수 효과가 없는 순수 함수를 만드는 것이 중요한데, 객체가 불변이라면 어떨까?
    불변 객체는 기본적으로 값의 수정이 불가능하므로 변경 가능성이 적으며 객체의 생성과 사용이 제한된다. 그러므로, 메소드들은 자연스럽게 순수 함수로 구성될 것이고 다른 메소드가 호출되어도 객체의 상태가 유지되기 때문에 객체를 안전하게 재사용할 수 있다. 이러한 불변 객체는 오류를 줄여 유지 보수성이 높은 코드를 작성하게 도와준다.
- 다른 사람이 작성한 함수를 예측 가능하며 안전하게 사용할 수 있다.
    - 불변 객체는 값이 변하지 않음을 보장하므로 우리가 다른 사람의 코드를 변경에 대한 불안 없이 이용할 수 있다.
- 가비지 컬렉션의 성능을 높일 수 있다. (자세한 내용은 GC 파트에서 다룰 예정)

## 불변 객체를 만드는 방법

불변 객체는 read-only 메소드만 제공하며, 객체의 내부 상태를 알려주는 메소드를 제공하지 않거나 제공할 경우 방어적 복사 혹은 Unmodified 라이브러리를 통해 제공한다.

또한, 객체의 필드는 모두 final을 사용하여 처음 할당된 이후 상태가 바뀌지 않도록 설정해야 한다. 다만 무조건 final만 필드에 붙인다고 해당 객체를 불변 객체라고 부를 수 있는 것은 아니다. 객체가 가질 수 있는 상태의 종류에 따라 불변 객체를 만들어 보자.

### 필드가 모두 primitive type인 경우

```java
public class Car {

    private final String name;

    private final int position;

    public Car(String name, int position) {
        this.name = name;
        this.position = position;
    }

    // 필요하다면 getter만 사용. setter는 금지
}
```

원시 타입은 참조 값이 존재하지 않기 때문에 값을 그대로 외부로 내보내는 경우에도 내부 객체는 불변이므로 setter가 없고, 원시 타입 필드에 대해 final을 설정하였다면 해당 객체인 Car는 불변 객체가 된다.

### 필드에 reference type이 있는 경우

```java
public class Car {

    private final String name;

    private final Position position;

    public Car(String name, Position position) {
        this.name = name;
        this.position = position;
    }

    // 필요하다면 getter만 사용. setter는 금지
}
```

위와 같이 Car 객체의 position 필드를 포장하여 참조 객체로 수정하였다. 이때 Position 필드에 단순히 final만 붙이면 Car 객체는 불변 객체가 되는 것처럼 보인다.

```java
public class Position {

    private int value;

    public Position(int value) {
        this.value = value;
    }

    public void setValue(int value) {
        this.value = value;
    }
}
```

하지만 Position이 불변 객체가 아니라면, Car 객체의 `getPosition()` 메소드를 통해 Position을 가져 오고 Position 객체의 `setValue()` 메소드를 사용하면 Position의 상태를 바꿀 수 있다. 그리고 Position의 상태를 바꿀 수 있다는 말은 즉, Car 객체의 상태를 바꿀 수 있다는 의미가 된다. 따라서 필드에 참조 타입이 있는 경우, 해당 참조 타입 객체도 불변 객체로 만들어야 한다.

### 필드에 primitive type 컬렉션이 있는 경우

```java
public class Car {

    private final String name;

    private final int position;

    private final List<Integer> monthlyMileages;

    public Car(String name, int position, List<Integer> monthlyMileages) {
        this.name = name;
        this.position = position;
        this.monthlyMileages = monthlyMileages;
    }

    // 필요하다면 getter만 사용. setter는 금지
}
```

이번에는 첫 번째 Car 예제 코드에서 월별 주행량 필드를 추가하였다. 해당 차가 매달 얼마나 주행했는지 나타내는 지표라고 생각하면 된다. 이것을 위와 같이 필드에 final을 추가하고, setter를 사용하지 않으니까 불변 객체라고 생각할 수 있다.

```java
public class Main {

    public static void main(String[] args) {
        List<Integer> monthlyMileages = new ArrayList<>(Arrays.asList(10, 15, 20, 10, 15));
        Car car = new Car("jayon", 10, monthlyMileages);
        System.out.println(car);

        monthlyMileages.add(1000);
        System.out.println(car);
    }
}
```

위와 같이 monthlyMileages를 사용하여 Car 객체를 만들고, 처음에 주입한 monthlyMileages를 조작한다면 어떻게 될까? 편의상 Car 객체에 `toString()` 을 추가하여 출력해 보았다.

```java
Car{name='jayon', position=10, monthlyMileages=[10, 15, 20, 10, 15]}
Car{name='jayon', position=10, monthlyMileages=[10, 15, 20, 10, 15, 1000]}
```

결과에서 알 수 있듯이, Car 객체의 monthlyMileages 상태가 변경된 것을 알 수 있다. 이것은 처음에 Car 객체의 생성자로 monthlyMileages를 넘길 때 주소가 공유되기 때문에 발생한 것이다. 따라서 생성자로 monthlyMileages를 그냥 넘겨 주면 안 되고 방어적 복사를 거치고 넘겨야 한다.

```java
public class Car {

    private final String name;

    private final int position;

    private final List<Integer> monthlyMileages;

    public Car(String name, int position, List<Integer> monthlyMileages) {
        this.name = name;
        this.position = position;
        this.monthlyMileages = new ArrayList<>(monthlyMileages);
    }

    // 필요하다면 getter만 사용. setter는 금지
}
```

방어적 복사는 `new ArrayList<>()` 와 같이 메모리를 새로 할당하여 기존 List와의 참조 주소를 끊어 내는 복사를 말한다. 이렇게 하고 아까 실행한 Main 메소드 예제 코드를 실행해 보자.

```java
Car{name='jayon', position=10, monthlyMileages=[10, 15, 20, 10, 15]}
Car{name='jayon', position=10, monthlyMileages=[10, 15, 20, 10, 15]}
```

처음 주입한 monthlyMileages와 Car 내의 존재하는 monthlyMileages은 서로 다른 List이므로 영향을 받지 않는 것을 알 수 있다.

그런데, 우리는 상황에 따라 monthlyMileages을 getter로 반환해 주어야 할 때가 있다.

```java
public class Car {

    private final String name;

    private final int position;

    private final List<Integer> monthlyMileages;

    public Car(String name, int position, List<Integer> monthlyMileages) {
        this.name = name;
        this.position = position;
        this.monthlyMileages = new ArrayList<>(monthlyMileages);
    }

    public List<Integer> getMonthlyMileages() {
        return monthlyMileages;
    }
}
```

생성자에서 방어적 복사를 취했으니, 위 getter 메소드는 안전할까?

```java
public class Main {

    public static void main(String[] args) {
        List<Integer> monthlyMileages = new ArrayList<>(Arrays.asList(10, 15, 20, 10, 15));
        Car car = new Car("jayon", 10, monthlyMileages);
        System.out.println("====처음 상태====");
        System.out.println(car);
        System.out.println();

        System.out.println("====초기 주입한 monthlyMileages add() 수행====");
        monthlyMileages.add(1000);
        System.out.println(car);
        System.out.println();

        System.out.println("====getter로 받은 monthlyMileagesOfCar add() 수행====");
        List<Integer> monthlyMileagesOfCar = car.getMonthlyMileages();
        monthlyMileagesOfCar.add(10000);
        System.out.println(car);
    }
}
```

이번에는 car에서 처음에 주입한 monthlyMileages에 값을 add해 보고, car의 getter로 받은 monthlyMileagesOfCar에 값을 add해 보겠다.

```java
====처음 상태====
Car{name='jayon', position=10, monthlyMileages=[10, 15, 20, 10, 15]}

====초기 주입한 monthlyMileages add() 수행====
Car{name='jayon', position=10, monthlyMileages=[10, 15, 20, 10, 15]}

====getter로 받은 monthlyMileagesOfCar add() 수행====
Car{name='jayon', position=10, monthlyMileages=[10, 15, 20, 10, 15, 10000]}
```

예상했듯이, 생성자에서 방어적 복사를 수행하였으므로 초기 주입한 monthlyMileages와 car의 monthlyMileages는 다른 객체가 맞다. 하지만, getter로 반환한 monthlyMileages는 여전히 주소를 공유하고 있으므로 monthlyMileagesOfCar의 상태가 변경되면 monthlyMileages도 상태가 변경된다. 따라서 car의 getter에 대해서도 방어적 복사를 취해야 한다.

```java
public class Car {

    private final String name;

    private final int position;

    private final List<Integer> monthlyMileages;

    public Car(String name, int position, List<Integer> monthlyMileages) {
        this.name = name;
        this.position = position;
        this.monthlyMileages = new ArrayList<>(monthlyMileages);
    }

    public List<Integer> getMonthlyMileages() {
        return new ArrayList<>(monthlyMileages);
    }
}
```

이렇게 설정하고 아까 Main 코드를 다시 실행해 보자.

```java
====처음 상태====
Car{name='jayon', position=10, monthlyMileages=[10, 15, 20, 10, 15]}

====초기 주입한 monthlyMileages add() 수행====
Car{name='jayon', position=10, monthlyMileages=[10, 15, 20, 10, 15]}

====getter로 받은 monthlyMileagesOfCar add() 수행====
Car{name='jayon', position=10, monthlyMileages=[10, 15, 20, 10, 15]}
```

정상적으로 Car의 상태가 바뀌지 않는 것을 알 수 있다.

참고로 getter에서는 방어적 복사 외에 `Collections.unmodifiableList()` 를 사용하는 방법도 있다. 이 메소드는 요소의 수정이 발생하면 예외를 던짐으로써 상태의 변화를 막는다.

```java
public class Car {

    private final String name;

    private final int position;

    private final List<Integer> monthlyMileages;

    public Car(String name, int position, List<Integer> monthlyMileages) {
        this.name = name;
        this.position = position;
        this.monthlyMileages = new ArrayList<>(monthlyMileages);
    }

    public List<Integer> getMonthlyMileages() {
        return Collections.unmodifiableList(monthlyMileages);
    }
}
```

이 상태에서 Main 코드를 실행해 보자.

```java
====처음 상태====
Car{name='jayon', position=10, monthlyMileages=[10, 15, 20, 10, 15]}

====초기 주입한 monthlyMileages add() 수행====
Car{name='jayon', position=10, monthlyMileages=[10, 15, 20, 10, 15]}

====getter로 받은 monthlyMileagesOfCar add() 수행====
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.Collections$UnmodifiableCollection.add(Collections.java:1055)
	at Main.main(Main.java:23)
```

getter로 받은 mothlyMileagesOfCar는 car의 monthlyMilages와 주소를 공유하고 있으므로 mothlyMileagesOfCar가 변경되면 monthlyMilages가 변경되지만, `Collections.unmodifiableList()` 의 특성으로 인해 예외를 던짐으로써 상태의 변경을 막아준다. 하지만, 해당 라이브러리를 불변을 보장하는 것은 아니다.

```java
public class Car {

    private final String name;

    private final int position;

    private final List<Integer> monthlyMileages;

    public Car(String name, int position, List<Integer> monthlyMileages) {
        this.name = name;
        this.position = position;
        this.monthlyMileages = monthlyMileages;
    }

    public List<Integer> getMonthlyMileages() {
        return Collections.unmodifiableList(monthlyMileages);
    }
}
```

가령, 위와 같이 생성자에서 방어적 복사를 취하지 않으면 초기에 주입한 monthlyMileages의 상태가 변경되면 car의 monthlyMileages 상태도 같이 변경된다.

그리고 개인적으로 `Collections.unmodifiableList()` 는 예외 핸들링을 추가로 해 주어야 하므로 getter에서도 방어적 복사를 수행하는 것을 추천한다.

### 필드에 reference type 컬렉션이 있는 경우

```java
public class Cars {

    private final List<Car> cars;

    public Cars(List<Car> cars) {
        this.cars = new ArrayList<>(cars);
    }

    public List<Car> getCars() {
        return new ArrayList<>(cars);
    }
}

public class Car {

    private final String name;

    public int position;

    public Car(String name, int position) {
        this.name = name;
        this.position = position;
    }
}
```

이번에는 간단한 Car 형태의 일급 컬렉션인 Cars를 만들어 보았다. 위에서 설명한 대로, cars에 대해 final도 붙이고, 생성자 및 getter에서 방어적 복사를 수행하였다. 그런데, 위 Car 객체는 position 필드가 public으로 열려 있고 final이 아니므로 수정이 가능하다. 하지만 Cars를 보면, 생성자에서 방어적 복사를 통해 원본 Car 컬렉션과 참조를 끊었으며, getter에서도 Cars의 Car 컬렉션 필드와 참조를 끊었기 때문에 불변성을 지켰다고 생각할 수 있다.

```java
public class Main {

    public static void main(String[] args) {
        List<Car> carList = new ArrayList<>(Arrays.asList(new Car("car1", 1), new Car("car2", 2)));
        Cars cars = new Cars(carList);
        System.out.println("====처음 상태====");
        System.out.println(cars);
        System.out.println();

        System.out.println("====초기 주입한 carList 내의 요소의 상태 변경====");
        carList.get(0).position = 10000;
        System.out.println(cars);
    }
}
```

Main 코드에서 이번에는 초기 주입한 carList 내의 요소를 바꾸어 보았다. 결과를 확인해 보자.

```java
====처음 상태====
Cars{cars=[Car{name='car1', position=1}, Car{name='car2', position=2}]}

====초기 주입한 carList 내의 요소의 상태 변경====
Cars{cars=[Car{name='car1', position=10000}, Car{name='car2', position=2}]}
```

초기 주입한 carList 자체와의 참조를 끊은 것은 맞지만, 내부 요소와의 참조는 유지되는 사실을 알 수 있다. 그렇다. 방어적 복사는 만능이 아니다. 왜 내부 요소와의 참조는 유지되는지 `new ArrayList<>()` 내부 코드를 뜯어 보자.

```java
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

elementData는 ArrayList의 내부 배열이며 `c.toArray()` 아래 코드는 일반적인 상황에서 잘 발생하지 않으므로 크게 고려하지 않아도 된다. 그러면 `toArray()` 를 다시 한 번 뜯어 보자. 참고로 c는 ArrayList이다.

```java
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }
```

그러면 또 다시 `Arrays.copyOf()` 를 까 보자.

```java
    public static <T> T[] copyOf(T[] original, int newLength) {
        return (T[]) copyOf(original, newLength, original.getClass());
    }
```

또 다른 메소드인 `copyOf()` 를 호출한다. 이것은 Arrays의 내부 메소드이다.

```java
    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```

자, 이제 다 왔다. 위 코드를 보면 copy 배열은 삼항 연산자에서 전자를 선택하나, 어쨌든 두 경우 모두 새로운 배열을 새로운 메모리에 할당하는 것을 알 수 있다. 그리고 다음 `System.arraycopy()` 메소드를 살펴 보자.

```java
public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

위 메소드는 native 키워드가 붙은 것을 알 수 있다. native 키워드는 자바로 구현하기 까다로운 것을 다른 언어로 구현해서 자바로 사용하는 방법이다. 구현할 때 JNI(Java Native Interface)를 활용한다. 다만, 이 메소드의 C++ 버전을 굳이 분석하지는 않고 결론만 말하겠다.

`System.arraycopy()` 는 요소에 대해 얕은 복사를 수행한다. 즉, 위 메소드를 사용하면 A 배열의 요소를 B 배열로 복사하되, A와 B 배열의 요소는 같은 주소를 가리키게 된다.

이를 토대로 `new ArrayList<>()` 는 ArrayList 객체를 할당하기 위한 메모리를 새롭게 할당하고, ArrayList 객체가 사용하는 elementData 배열을 위한 메모리를 새로 할당한다. 다만, elementData 배열의 요소는 얕은 복사를 수행하므로 인자로 들어온 ArrayList의 요소와 같은 주소를 참조한다. 따라서 인자로 들어온 ArrayList 요소의 상태가 바뀌면 Car의 ArrayList 요소의 상태가 바뀌게 되므로 Car는 가변 객체가 된다.

정리하자면, 참조 타입 컬렉션을 필드로 사용할 때는 방어적 복사를 맹신하지 말고, 참조 타입 자체를 불변 객체로 보장해 주어야 한다.

## 출처

- [https://mangkyu.tistory.com/131](https://mangkyu.tistory.com/131)
- [https://sup2is.github.io/2020/01/29/java-immutable-object-with-string.html](https://sup2is.github.io/2020/01/29/java-immutable-object-with-string.html)

## 예상 면접 질문 및 답변

### 가변 객체와 불변 객체를 설명하라

**가변 객체**

가변 객체는 Java에서 Class의 인스턴스가 생성된 이후에 내부 상태가 변경 가능한 객체이다. 가변 객체는 멀티 스레드 환경에서 사용하려면 별도의 동기화 처리가 필요하며, 대표적인 가변 객체로 ArrayList, HashMap, StringBuilder, StringBuffer 등이 존재한다. 이외에도 프로그래머가 커스텀 객체를 생성하여 내부 상태를 변경할 수 있게 만든다면, 그것도 가변 객체가 된다.

**불변 객체**

불변 객체는 가변 객체와 반대로 Java에서 Class의 인스턴스가 생성된 이후에 내부 상태를 변경할 수 없는 객체이다. 불변 객체는 멀티 스레드 환경에서도 안전하게 사용할 수 있다는 신뢰성을 보장하며, 대표적인 불변 객체로 String 등이 존재한다. 이외에도 프로그래머가 커스텀 객체를 생성하여 내부 상태가 변경되지 않게 만들면, 그것도 불변 객체가 된다.

### 불변 객체의 장점을 설명하라

- Thread-safe하여 병렬 프로그래밍에 유용하며, 동기화를 고려하지 않아도 된다.
- 실패 원자적인(Failure Atomic) 메소드를 만들 수 있다.
- Cache, Map, Set 등의 요소로 활용하기에 적합하다.
- 부수 효과(Side Effect)를 피해 오류 가능성을 최소화할 수 있다.
- 다른 사람이 작성한 함수를 예측 가능하며 안전하게 사용할 수 있다.

### 불변 객체를 만드는 방법을 설명하라

- 모든 필드에 대해 final을 설정한다.
- 필드에 참조 타입이 있을 경우, 해당 객체도 불변성을 보장해야 한다.
- 필드에 컬렉션이 존재할 경우, 생성자 및 getter에 대해 방어적 복사를 수행해야 한다.

### 방어적 복사와 Unmodifiable의 차이점은?

방어적 복사는 A 리스트와 B 리스트 사이의 참조를 끊는 행위이지만, Unmodifiable은 참조를 끊지 않고 단순히 특정 리스트에서 요소의 변경이 일어날 경우 예외를 던진다. 그래서 생성자 단계에서 방어적 복사를 취하지 않고, getter에서 Unmodifiable만 취할 경우 초기 생성자로 주입한 컬렉션의 변화가 생기면 불변성이 깨진다.

### 방어적 복사를 사용하면 항상 불변성을 보장하는가?

그렇지 않다. 방어적 복사는 컬렉션의 요소에 대해 얕은 복사를 수행하므로 컬렉션의 참조 타입이 가변 객체라면, 복사하려는 컬렉션의 요소가 변경될 경우 불변성이 깨진다.
