이전 글([링크](https://www.notion.so/cd3366ae5c3c45a48934ab13e46e47d0))에서 말했듯이 가시성과 원자성을 보장해서 동시성 프로그래밍에서 생길 수 있는 문제를 해결할 수 있다.

가시성은 `volatile` 키워드를 사용해 보장할 수 있다. 원자성은 `synchronized` 키워드와 `atomic 변수`를 통해 보장할 수 있다. `synchronized` 키워드에 대한 내용은 [링크](https://www.notion.so/synchronized-35f1ce8a74404ce2843d7b40586f0a55)에서 확인하면 된다.

이번 글에서는 `atomic` 변수를 이용한 non-blocking 동기화에 대해 알아보려 한다.

# synchronized의 문제점

`synchronized`는 Blocking을 사용해 멀티 스레드 환경에서 공유 객체를 동기화하는 키워드이다. 그러나 Blocking에는 여러가지 단점이 존재한다. 그 중에서 손꼽는 문제는 성능 이슈이다. 

특정 스레드가 해당 블락 전체에 lock을 걸면 해당 lock에 접근하는 스레드들은 Blocking 상태에 들어가기 때문에 아무 작업도 하지 못해 자원이 낭비되고, 상태가 변경되는 동안 시스템의 자원을 사용하게 된다. 결국 이 문제는 성능 저하로 이어진다.

예를 들어 자동차 운전을 한다고 가정해보자. 운전자가 방향 전환을 하려고 하는데, 마침 앞에 다른 자동차가 대기하고 있다. 운전자는 앞의 차가 먼저 지나가기를 기다리기 위해 정차를 하고, 앞의 차가 지나가면 다시 출발을 해야 한다. 이처럼 자동차를 정차하고 다시 출발하고, 앞의 차를 기다리며 많은 에너지가 소비되는 것과 비슷한 이치이다.

이러한 문제점 때문에 non-blocking을 하며 원자성을 보장하기 위한 방법이 `atomic 변수`이다.

# 원자성과 Atomic Type

`atomic 변수`는 멀티 스레드 환경에서 원자성을 보장하기 위해 나온 개념이다. `synchronized`와는 다르게 blocking이 아닌 non-blocking 하면서 원자성을 보장해 동기화 문제를 해결한다. 

`**atomic`의 동작 핵심 원리는 CAS(Compared And Swap) 알고리즘이다.**

## CAS (Compare And Swap) 알고리즘

![image](https://user-images.githubusercontent.com/55661631/148049941-668a7947-fdd2-41d7-84f8-1b458a923fee.png)

CAS 알고리즘의 동작 원리는 다음과 같다.

- 인자로 기존값과 변경할 값을 전달한다.
- 기존값이 현재 메모리가 가지고 있는 값과 같다면 변경할 값을 반영하며, true를 반환한다.
- 반대로 기존값이 현재 메모리가 가지고 있는 값과 다르다면 값을 반영하지 않고 false를 반환한다.

여기서 ‘기존값으로 던진 값이 현재 메모리가 가지고 있는 값과 다른 경우는 뭐지?’라는 의문이 생길 수 도 있다. 

이 말의 의미는 스레드 A가 공유 변수를 계산을 하고 메모리에 반영하기 직전에 다른 스레드 B가 공유 변수를 변경해 메모리에 반영한 경우를 의미한다. 이런 경우에는 당연히 스레드 A의 변경할 값을 메모리에 반영하면 안된다.

따라서 false를 반환한 경우에는 무한 루프를 구성해 다시 변경된 값(다른 스레드에 의해 변경된 메모리 값)을 읽고 같은 시도를 반복하거나, 다른 더 중요한 작업이 있으면 다른 작업을 해도 된다. 개발자가 결정하면 된다.

정리하자면, `synchronized`를 사용해 blocking이 발생하는 것에 비해 훨씬 효율적인 방법이라고 할 수 있다. 무한 루프를 돌면서 값을 반영할 수 있는지 물어보는 경우에도 성능 더 우수하다(스레드 상태를 변경, 바꾸는 작업보다 자원 낭비가 덜하기 때문이다).

### 자바에서의 CAS 동작 예시

이해를 돕기 위해 CAS 동작 원리를 자바에 적용시켜 보겠다.

![image](https://user-images.githubusercontent.com/55661631/148049965-6536af75-5f47-4494-a3f8-7c641f818209.png)

위 그림은 JVM 내의 스레드 스케줄러에 의해 각각의 core에 스레드-1과 스레드-2가 선점된 상태이고 두 스레드는 각각 for 문 안에서 count를 증가시킨다고 가정하겠다.

1. 각 스레드는 힙 내에 있는 count 변수를 읽어 CPU Cache Memory에 저장한다.
2. 각 스레드는 번갈아가며 for문을 돌며 count 값을 1씩 증가시킨다.
3. 스레드-1 또는 스레드-2는 변경한 count 값을 힙에 반영하기 위해 변경하기 전의 count 값과 힙에 저장된 count 값을 비교한다. 여기서 이후 상황이 두 가지로 나뉜다.
    1. 변경하기 전의 count 값과 힙에 저장된 count 값이 다를 경우 false를 반환하며, 힙에 저장된 값을 읽어 2번 과정으로 돌아간다.
    2. 변경하기 전의 count 값과 힙에 저장된 count 값이 같은 경우 힙에 변경한 값을 저장한다.
4.  힙에 변경한 값을 저장한 스레드-1 또는 스레드-2는 1번 과정으로 돌아간다(for문이 종료할 때까지).

## 정리

`Atomic 변수`의 핵심 원리인 CAS 알고리즘은 원자성뿐만 아니라 가시성 문제도 해결해주는 것을 볼 수 있다. 그리고 non-blocking이 가능하기 때문에 blockning 방식인 `synchronized`보다 성능상 이점이 있다는 것도 알 수 있었다.

참고로 `synchronized` 키워드의 경우 `synchronized` 블록에 진입 전에 `CPU Cache Memory`와 `Main Memory` 값을 동기화하여 가시성을 해결한다.

# AtomicInteger 살펴보기

atomic type인 AtomicInteger 클래스가 동기화 문제를 해결해주는지 살펴보자.

```java
public class AtomicIntegerTest {

    private static int count;

    public static void main(String[] args) throws InterruptedException {
        AtomicInteger atomicCount = new AtomicInteger(0);
        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 100000; i++) {
                count++;
                atomicCount.incrementAndGet();
            }
        });

        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < 100000; i++) {
                count++;
                atomicCount.incrementAndGet();
            }
        });

        thread1.start();
        thread2.start();

        Thread.sleep(5000);
        System.out.println("atomic 결과 : " + atomicCount.get());
        System.out.println("int 결과 : " + count);
    }
}
```

AtomicInteger와 int 타입 count 변수를 생성한 다음 두 개의 스레드에서 `count++` 연산을 하는 예제이다.

결과를 살펴보자.

```
//결과
atomic 결과 : 200000
int 결과 : 152298
```

AtomicInteger 타입인 atomicCount는 의도대로 200000이 출력되는 것을 볼 수 있고, int 타입인 count는 동기화가 지켜지지 않아 잘못된 값을 출력하는 것을 볼 수 있다.

동기화가 어떻게 지켜지는지 AtomicInteger 클래스의 `incrementAndGet()` 메소드를 살펴보자.

```java
public class AtomicInteger extends Number implements java.io.Serializable {

		private static final Unsafe U = Unsafe.getUnsafe();
		private static final long VALUE = U.objectFieldOffset(AtomicInteger.class, "value");
    private volatile int value;
		
		public final int incrementAndGet() {
				return U.getAndAddInt(this, VALUE, 1) + 1;
		}
}

public final class Unsafe {
		@HotSpotIntrinsicCandidate
		public final int getAndAddInt(Object o, long offset, int delta) {
				int v;
				do {
						v = getIntVolatile(o, offset);
				} while (!weakCompareAndSetInt(o, offset, v, v + delta));
						return v;
		}
}
```

`incrementAndGet()` 메소드 내부에서 CAS 알고리즘의 로직을 구현하고 있다. `getAndAddInt()` 내부에서는 `weakCompareAndSetInt()` 메소드를 호출하여 메모리에 저장된 값과 현재 값을 비교해서 동일한 경우에 메모리에 변경한 값을 저장하고 true를 반환해 while문을 빠져나온다.

추가로 눈여겨볼 점은 value 변수에 volatile 키워드가 붙은 것이다. volatile 키워드는 가시성 문제를 해결하기 위해서 사용한 것인데(AtomicInteger의 get(), set() 등의 메소드의 가시성 해결), 모든 쓰레드가 쓰기 작업을 하면 원자성을 보장할 수 없기 때문에 CAS 알고리즘(가시성 , 원자성 보장)을 추가로 사용해 더욱 안전하게 설계한 것이다. 

# 참고

- [https://rightnowdo.tistory.com/entry/JAVA-concurrent-programming-Atomic원자성](https://rightnowdo.tistory.com/entry/JAVA-concurrent-programming-Atomic%EC%9B%90%EC%9E%90%EC%84%B1)
- [https://n1tjrgns.tistory.com/244](https://n1tjrgns.tistory.com/244)
- [https://javaplant.tistory.com/23](https://javaplant.tistory.com/23)
- [https://beomseok95.tistory.com/225](https://beomseok95.tistory.com/225)
- [https://velog.io/@syleemk/Java-Concurrent-Programming-가시성과-원자성](https://velog.io/@syleemk/Java-Concurrent-Programming-%EA%B0%80%EC%8B%9C%EC%84%B1%EA%B3%BC-%EC%9B%90%EC%9E%90%EC%84%B1)
- [https://didrlgus.github.io/java/05-post/#atomic과-cas](https://didrlgus.github.io/java/05-post/#atomic%EA%B3%BC-cas)

# 예상 면접 질문 및 답변

## Q. synchronized의 문제점은?

`synchronized`는 스레드가 해당 블록에 lock을 걸면 lock에 접근하는 스레드들은 Blocking되기 때문에 성능 저하로 이어집니다. 스레드가 Blocking 상태에 들어가면 아무 작업도 하지 못해 자원이 낭비되고, 상태가 변경되는 동안에도 시스템의 자원을 사용하기 때문이다.

## Q. atomic type에 대해 설명

`atomic type`은 멀티 스레드 환경에서 원자성을 보장하기 위한 개념이다. CAS 알고리즘을 통해 non-blocking하면서 가시성과 원자성을 보장해 동기화 문제를 해결한다.

## Q. CAS 알고리즘에 대해 설명

CAS 알고리즘은 현재 스레드가 가지고 있는 기존값과 메모리가 가지고 있는 값을 비교해 같은 경우 변경할 값을 메모리에 반영하고 true를 반환한다. 다른 경우에는 변경값이 반영되지 않고 false를 반환한 다음 재시도를 하는 방식으로 동작한다. CAS 알고리즘을 통해 가시성과 원자성 문제를 해결할 수 있다.
