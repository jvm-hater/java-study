# synchronized

`Java` 는 크게 3가지 메모리 영역을 가지고 있다.

- `static` 영역
- `heap` 영역
- `stack` 영역

자바 멀티 스레드 환경에서는 스레드들끼리 `static` 영역과 `heap` 영역을 공유하기 때문에, 공유 자원에 대한 동기화 문제를 신경써야 한다. 

이번 글에서는 동기화 문제에 대한 해결방법 중 하나인 `Synchronized` 키워드에 대해 설명하려 한다.

`synchronized` 는 `lock` 을 이용해 동기화를 시키며 4가지의 사용법이 있다.

- synchronized method
- static synchronized method
- synchronized block
- static synchronized block

## synchronized method

```java
public class Method {

    public static void main(String[] args) {

        Method sync = new Method();
        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작 " + LocalDateTime.now());
            sync.syncMethod1("스레드1");
            System.out.println("스레드1 종료 " + LocalDateTime.now());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작 " + LocalDateTime.now());
            sync.syncMethod2("스레드2");
            System.out.println("스레드2 종료 " + LocalDateTime.now());
        });

        thread1.start();
        thread2.start();
    }

    private synchronized void syncMethod1(String msg) {
        System.out.println(msg + "의 syncMethod1 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private synchronized void syncMethod2(String msg) {
        System.out.println(msg + "의 syncMethod2 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

`Method` 인스턴스를 한 개 생성하고, 두 개의 스레드를 만들어 각각 `synchronized` 키워드가 붙은 `syncMethod1` , `syncMethod2`를 호출했다.

결과를 확인해보자.

```
스레드1 시작 2021-12-20T17:32:09.983815800
스레드2 시작 2021-12-20T17:32:09.983815800
스레드1의 syncMethod1 실행중2021-12-20T17:32:10.003817100
스레드1 종료 2021-12-20T17:32:15.013816900
스레드2의 syncMethod2 실행중2021-12-20T17:32:15.013816900
스레드2 종료 2021-12-20T17:32:20.014763700
```

스레드1이 `syncMethod1` 호출한 후 종료된 다음 스레드2가 `syncMethod2` 를 호출한 것을 확인할 수 있다.

위 예시는 **하나의 인스턴스를 서로 다른 스레드가 실행**한 경우이다. 그럼 **각각의 인스턴스를 만들고 스레드들이 메소드를 호출**하도록 해보자.

```java
public class Method {

    public static void main(String[] args) {

        Method method1 = new Method();
        Method method2 = new Method();

        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작 " + LocalDateTime.now());
            method1.syncMethod1("스레드1");
            System.out.println("스레드1 종료 " + LocalDateTime.now());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작 " + LocalDateTime.now());
            method2.syncMethod2("스레드2");
            System.out.println("스레드2 종료 " + LocalDateTime.now());
        });

        thread1.start();
        thread2.start();
    }

    private synchronized void syncMethod1(String msg) {
        System.out.println(msg + "의 syncMethod1 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private synchronized void syncMethod2(String msg) {
        System.out.println(msg + "의 syncMethod2 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

결과를 확인해보자.

```
스레드1 시작 2021-12-20T17:39:12.626511900
스레드2 시작 2021-12-20T17:39:12.626511900
스레드1의 syncMethod1 실행중2021-12-20T17:39:12.644481900
스레드2의 syncMethod2 실행중2021-12-20T17:39:12.644481900
스레드1 종료 2021-12-20T17:39:17.653776400
스레드2 종료 2021-12-20T17:39:17.653776400
```

이 상황에서는 `lock` 을 공유하지 않기 때문에 스레드간의 동기화가 발생하지 않는다.

결과를 보면 알 수 있듯이 `synchronized method` 는 **인스턴스**에 `lock` 을 건다. 인스턴스에 `lock` 을 건다고 표현해서 인스턴스 접근 자체에 `lock` 이 걸리는 것인지 혼동이 올 수도 있는데 그렇지 않다.

다음 예시를 보자.

```java
public class Method {

    public static void main(String[] args) {

        Method method = new Method();
        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작 " + LocalDateTime.now());
            method.syncMethod1("스레드1");
            System.out.println("스레드1 종료 " + LocalDateTime.now());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작 " + LocalDateTime.now());
            method.syncMethod2("스레드2");
            System.out.println("스레드2 종료 " + LocalDateTime.now());
        });

        Thread thread3 = new Thread(() -> {
            System.out.println("스레드3 시작 " + LocalDateTime.now());
            method.method3("스레드3");
            System.out.println("스레드3 종료 " + LocalDateTime.now());
        });

        thread1.start();
        thread2.start();
        thread3.start();
    }

    private synchronized void syncMethod1(String msg) {
        System.out.println(msg + "의 syncMethod1 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private synchronized void syncMethod2(String msg) {
        System.out.println(msg + "의 syncMethod2 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private void method3(String msg) {
        System.out.println(msg + "의 method3 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

스레드3을 추가하고 `synchronized` 키워드가 붙지 않은 `method3` 를 호출했다.

결과를 확인해보자.

```
스레드2 시작 2021-12-20T17:49:44.512644400
스레드3 시작 2021-12-20T17:49:44.512644400
스레드1 시작 2021-12-20T17:49:44.512644400
스레드3의 method3 실행중2021-12-20T17:49:44.529689700
스레드2의 syncMethod2 실행중2021-12-20T17:49:44.529689700
스레드2 종료 2021-12-20T17:49:49.539983200
스레드3 종료 2021-12-20T17:49:49.539983200
스레드1의 syncMethod1 실행중2021-12-20T17:49:49.539983200
스레드1 종료 2021-12-20T17:49:54.540973700
```

이 상황에서 스레드3에는 동기화가 발생하지 않은 것을 확인할 수 있다.

결과를 보면 알 수 있듯이 **`synchronized` 메소드는 인스턴스 단위로 `lock` 을 걸지만, `synchronized` 키워드가 붙은 메소드에 대해서만 `lock` 을 공유한다.**

## static synchronized method

`static` 키워드가 포함된 `**synchronized` 메소드**는 인스턴스가 아닌 클래스 단위로 `lock` 을 공유한다.

다음 예시를 보자.

```java
public class StaticMethod {

    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작 " + LocalDateTime.now());
            syncStaticMethod1("스레드1");
            System.out.println("스레드1 종료 " + LocalDateTime.now());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작 " + LocalDateTime.now());
            syncStaticMethod2("스레드2");
            System.out.println("스레드2 종료 " + LocalDateTime.now());
        });

        thread1.start();
        thread2.start();
    }

    public static synchronized void syncStaticMethod1(String msg) {
        System.out.println(msg + "의 syncStaticMethod1 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static synchronized void syncStaticMethod2(String msg) {
        System.out.println(msg + "의 syncStaticMethod2 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

결과를 확인해보자.

```
스레드1 시작 2021-12-20T18:07:09.872182200
스레드2 시작 2021-12-20T18:07:09.872182200
스레드1의 syncStaticMethod1 실행중2021-12-20T18:07:09.887184500
스레드1 종료 2021-12-20T18:07:14.895190300
스레드2의 syncStaticMethod2 실행중2021-12-20T18:07:14.895190300
스레드2 종료 2021-12-20T18:07:19.895435900
```

`static` 함수라도 일반 메소드와 동일하게 함수 간의 동기화가 발생하는 것을 확인할 수 있다. 중요한 점은 인스턴스 단위로 `lock` 을 공유하는 것이 아닌 **클래스 단위로 `lock` 을 공유한다는 점이다.**

여기에 일반 `synchronized` 메소드를 추가한다면 어떻게 될까?

```java
public class StaticMethod {

    public static void main(String[] args) {
        StaticMethod staticMethod = new StaticMethod();

        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작 " + LocalDateTime.now());
            syncStaticMethod1("스레드1");
            System.out.println("스레드1 종료 " + LocalDateTime.now());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작 " + LocalDateTime.now());
            syncStaticMethod2("스레드2");
            System.out.println("스레드2 종료 " + LocalDateTime.now());
        });

        Thread thread3 = new Thread(() -> {
            System.out.println("스레드3 시작 " + LocalDateTime.now());
            staticMethod.syncMethod3("스레드3");
            System.out.println("스레드3 종료 " + LocalDateTime.now());
        });

        thread1.start();
        thread2.start();
        thread3.start();
    }

    public static synchronized void syncStaticMethod1(String msg) {
        System.out.println(msg + "의 syncStaticMethod1 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static synchronized void syncStaticMethod2(String msg) {
        System.out.println(msg + "의 syncStaticMethod2 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private synchronized void syncMethod3(String msg) {
        System.out.println(msg + "의 syncMethod3 실행중" + LocalDateTime.now());
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

위 예시에서 `synchronized` 메소드를 추가했다. 

결과를 확인해보자.

```
스레드2 시작 2021-12-20T18:16:47.589799300
스레드1 시작 2021-12-20T18:16:47.589799300
스레드3 시작 2021-12-20T18:16:47.589799300
스레드2의 syncStaticMethod2 실행중2021-12-20T18:16:47.605801200
스레드3의 syncMethod3 실행중2021-12-20T18:16:47.605801200
스레드2 종료 2021-12-20T18:16:52.615917600
스레드3 종료 2021-12-20T18:16:52.615917600
스레드1의 syncStaticMethod1 실행중2021-12-20T18:16:52.615917600
스레드1 종료 2021-12-20T18:16:57.616291400
```

 `static sychronized` 메소드를 사용하는 스레드1과 스레드2 간에는 동기화가 잘 지켜지는 것을 확인할 수 있다. 그러나  `sychronized` 메소드를 사용한 스레드3은 개발자가 의도한 동기화가 지켜지지 않았다. 

정리하자면, 클래스 단위에 거는 `lock` 과 인스턴스 단위에 거는 `lock` 은 공유가 안 되기 때문에 혼용해서 쓰게 된다면 동기화 이슈가 발생하게 된다.

## synchronized block

`synchronized block` 은 인스턴스의 `block` 단위로 `lock` 을 걸며, 2가지의 사용 방법이 있다.

- synchornized(this)
- synchornized(Object)

### synchronized(this)

```java
public class Block1 {

    public static void main(String[] args) {

        Block1 block = new Block1();

        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작 " + LocalDateTime.now());
            block.syncBlockMethod1("스레드1");
            System.out.println("스레드1 종료 " + LocalDateTime.now());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작 " + LocalDateTime.now());
            block.syncBlockMethod2("스레드2");
            System.out.println("스레드2 종료 " + LocalDateTime.now());
        });
        thread1.start();
        thread2.start();
    }

    private void syncBlockMethod1(String msg) {
        synchronized (this) {
            System.out.println(msg + "의 syncBlockMethod1 실행중" + LocalDateTime.now());
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void syncBlockMethod2(String msg) {
        synchronized (this) {
            System.out.println(msg + "의 syncBlockMethod2 실행중" + LocalDateTime.now());
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

위와 같이 `synchronized` 인자값으로 `this` 를 사용하면 모든 `synchronized block` 에 `lock` 이 걸린다. 쉽게 말하자면, 여러 스레드가 들어와서 서로 다른 `synchronized block` 을 호출해도 `this` 를 사용해 자기 자신에 `lock` 을 걸었기 때문에 기다려야 한다.

결과를 확인해보자.

```
스레드2 시작 2021-12-20T18:40:11.652664400
스레드1 시작 2021-12-20T18:40:11.652664400
스레드2의 syncBlockMethod2 실행중2021-12-20T18:40:11.668626700
스레드2 종료 2021-12-20T18:40:16.675778600
스레드1의 syncBlockMethod1 실행중2021-12-20T18:40:16.675778600
스레드1 종료 2021-12-20T18:40:21.676119600
```

`synchronized(this)` 블럭으로 감싸진 부분끼리 동기화가 잘 지켜지는 것을 확인할 수 있다.

### synchronized(Object)

그러나 위 방식은 모든 블럭에 `lock` 이 걸리기 때문에 상황에 따라 비효율적일 수도 있다. 따라서 `synchronized(Object)` 방식으로 블록마다 다른 `lock` 이 걸리게해 훨씬 효율적인 코드를 작성할 수 있다.

```java
public class Block2 {

    private final Object o1 = new Object();
    private final Object o2 = new Object();

    public static void main(String[] args) {

        Block2 block = new Block2();

        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작 " + LocalDateTime.now());
            block.syncBlockMethod1("스레드1");
            System.out.println("스레드1 종료 " + LocalDateTime.now());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작 " + LocalDateTime.now());
            block.syncBlockMethod2("스레드2");
            System.out.println("스레드2 종료 " + LocalDateTime.now());
        });
        thread1.start();
        thread2.start();
    }

    private void syncBlockMethod1(String msg) {
        synchronized (o1) {
            System.out.println(msg + "의 syncBlockMethod1 실행중" + LocalDateTime.now());
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void syncBlockMethod2(String msg) {
        synchronized (o2) {
            System.out.println(msg + "의 syncBlockMethod2 실행중" + LocalDateTime.now());
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

결과를 확인해보자.

```
스레드2 시작 2021-12-20T18:44:09.476825500
스레드1 시작 2021-12-20T18:44:09.476825500
스레드2의 syncBlockMethod2 실행중2021-12-20T18:44:09.494827100
스레드1의 syncBlockMethod1 실행중2021-12-20T18:44:09.494827100
스레드2 종료 2021-12-20T18:44:14.504974100
스레드1 종료 2021-12-20T18:44:14.504974100
```

스레드1과 스레드2 간의 동기화가 지켜지지 않은 것을 확인할 수 있다. 따라서 `this` 가 아닌 `o1` 과 `o2` 객체를 만들어 인자로 넘겨주면 동시에 `lock` 이 걸려야 하는 부분을 따로 지정해 줄 수 있다.

## static synchronized block

`static method` 안에 `synchronized block`을 지정할 수 있다. `static`의 특성상 `this`같이 현재 객체를 가르키는 표현을 사용할 수 없다. 

`static synchroinzed method`방식과 차이는 `lock`객체를 지정하고 `block`으로 범위를 한정지을 수 있다는 점이다. 이외에 클래스 단위로 `lock`을 공유한다는 점은 같다.

# Singleton 객체에서의 동기화

 `Singleton` 객체를 생성하는 가장 쉬운 방법은 다음과 같다.

```java
public class BasicSingleton {

    private static BasicSingleton sBasicSingleton;

    public static BasicSingleton getInstance() {
        if (Objects.isNull(sBasicSingleton)) {
            sBasicSingleton = new BasicSingleton();
        }
        return sBasicSingleton;
    }
}
```

하지만 이 방식은 싱글 스레드 환경에서는 문제가 없지만, 멀티 스레드 환경일 경우 `getInstance()` 가 동시에 불릴 수 있어 동기화 이슈가 발생한다.

단순히 `getInstance()` 메소드에 `synchronized` 키워드를 붙여 동기화 이슈를 해결할 수 있다.

```java
public class BasicSingleton {

    private static BasicSingleton sBasicSingleton;

    public static synchronized BasicSingleton getInstance() {
        if (Objects.isNull(sBasicSingleton)) {
            sBasicSingleton = new BasicSingleton();
        }
        return sBasicSingleton;
    }
}
```

그러나 사실 이 방법은 좋은 방법이 아니다. `Singleton` 에 `synchronized` 메소드가 많을수록 멀티 스레드는 병목현상을 겪게 되기 때문이다. 쉽게 말하자면, 기껏 멀티 스레드를 사용하는데 `Singleton` 을 사용할 때는 싱글 스레드처럼 동작하는 문제가 발생한다는 뜻이다.

## Double Checked Locking

`DCL` 이라고 불리는 이 방식은 현재 사용하지 않는 기법이다.  코드는 다음과 같다.

```java
public class LazySingleton {

    private volatile static LazySingleton sLazySingleton;

    private LazySingleton() {
    }

    public static LazySingleton getInstance() {
        if (sLazySingleton == null) {
            synchronized (LazySingleton.class) {
                if (sLazySingleton == null) {
                    sLazySingleton = new LazySingleton();
                }
            }

        }
        return sLazySingleton;
    }
}
```

메소드에서 `synchronized` 를 빼면서 동기화 오버헤드를 줄여보고자 하는 의도로 설계된 방식이다. 최초 인스턴스가 생성된 이후로는 동기화 블럭에 진입하지 않기 때문에 효율적인 방식이라고 생각할 수 있지만, 특정 상황에서는 정상 동작하지 않을 수 있다.

예를 들어, `Thread A`와 `Thread B`가 있다고 하자. `Thread A`가 `instance`의 생성을 완료하기 전에 메모리 공간에 할당이 가능하기 때문에 `Thread B`가 할당된 것을 보고 `instance`를 사용하려고 하나 생성과정이 모두 끝난 상태가 아니기 때문에 오동작할 수 있다는 것이다. 물론 이러할 확률은 적겠지만 혹시 모를 문제를 생각하여 쓰지 않는 것이 좋다.

## LazyHolder

현재 사용되고 있는 방식이다. 코드는 다음과 같다.

```java
public class Singleton {

    private Singleton() {
    }

    public static Singleton getInstance() {
        return Holder.instance;
    }
    
    private static class Holder {
        public static final Singleton instance = new Singleton();
    }
}
```

개발자가 직접 동기화 문제에 대해 코드를 작성하고 문제를 회피하려 한다면 프로그램 구조가 그 만큼 복잡해지고 비용 문제가 생길 수 있고 특히 정확하지 못한 경우가 많다.

이 방법은 JVM의 클래스 초기화 과정에서 보장되는 원자적 특성을 이용해 싱글톤의 초기화 책임을 JVM에게 넘긴다.

`Singleton` 클래스에는 `LazyHolder` 클래스의 변수가 없기 때문에 `Singleton` 클래스 로딩시 `Holder` 클래스를 초기화하지 않는다. `Holder` 클래스는 `Singleton` 클래스의 `getInstance()` 메서드에서 `Holder.instance`를 참조하는 순간 클래스가 로딩되며 초기화가 진행된다. 

클래스를 로딩하고 초기화하는 시점은 동기화를 보장하기 때문에 `volatile`이나 `synchronized` 같은 키워드가 없어도 동기화를 보장 하면서 성능도 좋은 방식이다.

# 참고

- [https://tourspace.tistory.com/55?category=788398](https://tourspace.tistory.com/55?category=788398)
- [https://brunch.co.kr/@kd4/156](https://brunch.co.kr/@kd4/156)
- [https://jgrammer.tistory.com/entry/Java-혼동되는-synchronized-동기화-정리](https://jgrammer.tistory.com/entry/Java-%ED%98%BC%EB%8F%99%EB%90%98%EB%8A%94-synchronized-%EB%8F%99%EA%B8%B0%ED%99%94-%EC%A0%95%EB%A6%AC)
- [https://medium.com/@joongwon/multi-thread-환경에서의-올바른-singleton-578d9511fd42](https://medium.com/@joongwon/multi-thread-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-%EC%98%AC%EB%B0%94%EB%A5%B8-singleton-578d9511fd42)

## 예상 면접 질문 및 답변

### Q. 자바에서 동기화 문제를 신경써야 하는 이유는?

자바 멀티 스레드 환경에서는 스레드들끼리 `static` 영역과 `heap` 영역을 공유하기 때문에, 공유 자원에 대한 동기화 문제를 신경써야 한다.

### Q. synchronized 키워드에 대해 설명

`synchronized` 는 `lock` 을 이용해 동기화를 시키는데 총 4가지 사용법이 있다.

- synchronized method
    - 동기화를 시키고 싶은 클래스의 메소드에 `synchronized` 키워드를 붙이면 되고, `synchronized method` 는 인스턴스 단위의 `synchronized` 키워드가 붙은 메소드에 대해서만 `lock` 공유한다.
- static synchronized method
    - `static synchronized method` 는 인스턴스가 아닌 클래스 단위로 `lock` 을 공유하며, `synchronized method` 와 동일하게 함수 간의 동기화가 발생한다. 만약 `synchronized method` 과 함께 사용하게 되면 인스턴스 락과 클래스 락은 공유가 안 되기 때문에 동기화 이슈가 발생할 수 있다는 점을 주의해야 한다.
- synchronized block
    - `synchronized block` 은 `block` 단위로 `lock` 을 걸며, 2가지 사용 방법이 있다.
    - 첫번 째는 `synchronized` 의 인자 값에 `this` 를 사용하는 방식이다. 이 방식은 여러 스레드가 들어와 서로 다른 `block` 을 호출해도 `this` 를 사용해 자기 자신에 `lock` 을 걸었기 때문에 기다려야 한다.
    - 두번 쨰는 `synchronized` 의 인자 값에 `Object` 를 사용하는 방식이다. 이 방식은 `block` 마다 서로 다른 `lock` 을 걸리게해 훨씬 효율적인 코드를 작성할 수 있다.
- static synchronized block
    - `static synchroinzed method`방식과 차이는 `lock`객체를 지정하고 `block`으로 범위를 한정지을 수 있다는 점이다. 클래스 단위로 `lock`을 공유한다는 점은 같다.
    

### Q. Singleton 객체에 synchronized 키워드를 사용하면 생기는 문제점은?

`Singleton` 에 `synchronized` 메소드가 많을수록 멀티 스레드는 병목현상을 겪게 된다. 쉽게 말하자면 멀티 스레드를 사용하지만 `Singleton` 을 사용할 때는 싱글 스레드처럼 동작하는 문제가 발생한다.

### Q. 위 문제의 해결 방법은?

`LazyHolder` 방식을 사용하면 된다. 이 방법은 `JVM` 의 클래스 초기화 과정에 보장되는 `thread-safe` 특성을 이용해 `Singleton` 의 초기화 책임을 `JVM` 에게 넘겨 개발자가 직접 동기화 코드를 작성할 때 생기는 문제를 회피할 수 있다. 또한 내부 `static class` 를 참조하기 전까지 클래스 로딩이 발생하지 않기 때문에 메모리를 효율적으로 사용할 수 있다.
