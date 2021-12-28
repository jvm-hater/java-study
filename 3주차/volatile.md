이전 글에서 가시성 문제를 해결하기 위해 `volatile` 키워드를 사용한다고 했다. [링크](https://www.notion.so/ff49e1fdd0d5405f92dde801bb2b0394) 

이번 글에서는 `votaile` 키워드를 사용해 가시성을 보장하는 방법을 간단한 예와 함께 설명하려고 한다.

아래 코드를 살펴보자.

```java
public class Volatile {

    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested) {
                i++;
            }
        });
        backgroundThread.start();

        Thread.sleep(1000);
        stopRequested = true;
    }
}
```

메인 스레드가 1초 후 `stopRequested` 변수를 `true`로 설정하기 때문에 `backgroupdThread`는 1초 후 반복문을 빠져나올 것처럼 보일 것이다. 그러나 실제로 실행시키면 위 코드는 아래처럼 반복문을 빠져나오지 못해 영원히 실행될 수도 있다.

![image](https://user-images.githubusercontent.com/55661631/147546241-18384066-e524-460a-933b-c10143db25c7.png)

다음 그림을 살펴보자.

![image](https://user-images.githubusercontent.com/55661631/147546262-52e8a4a5-606c-4c8a-b50d-66f613ec1a8b.png)

CPU1에서 수행된 스레드를 `backgroundThread`, CPU2에서 수행된 스레드를 `mainThread`라고 하자. `mainThread`는 `CPU Cache Memory 2`와 `RAM`에 공유 변인 `stopRequested`를 `true`로 쓰기 작업을 완료했으나, `backgroundThread`는 `CPU Cache Memory 1`에서 읽은 업데이트 되지 않은 `stopRequested`값을 사용한다.  이 값은 `false`이므로 무한 루프를 수행하게 된다. 즉, `mainThread`가 수정한 값을 `backgroundThread`가 언제쯤에나 보게 될지 보증할 수 없다. 이러한 문제점을 **가시성 문제**라고 한다.

이 문제를 해결하기 위해서는 `stopRequested` 변수를 `volatile`로 선언하면 된다. 그럼 다음 그림과 같이 `CPU Cache Memory`를 거치지 않고 `RAM`으로 직접 읽고 쓰는 작업을 수행하게 된다.

![image](https://user-images.githubusercontent.com/55661631/147546304-17654e0f-50e6-4093-b763-6f58539d3303.png)

변경된 코드도 살펴보자.

```java
public class Volatile {

    private static volatile boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested) {
                i++;
            }
        });
        backgroundThread.start();

        Thread.sleep(1000);
        stopRequested = true;
    }
}
```

코드를 실행시키면 1초 후 프로그램이 종료되는 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/55661631/147546326-bd25a8ca-aa0f-4bf8-ad1d-0f644807fc84.png)

# 참고

- [https://badcandy.github.io/2019/01/14/concurrency-02/](https://badcandy.github.io/2019/01/14/concurrency-02/)
- [https://ecsimsw.tistory.com/entry/자바의-동기화-방식-메모리-가시성이란-synchronized-volatile-atomic](https://ecsimsw.tistory.com/entry/%EC%9E%90%EB%B0%94%EC%9D%98-%EB%8F%99%EA%B8%B0%ED%99%94-%EB%B0%A9%EC%8B%9D-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EA%B0%80%EC%8B%9C%EC%84%B1%EC%9D%B4%EB%9E%80-synchronized-volatile-atomic)

# 예상 면접 질문 및 답변

### Q. `volatile` 키워드에 대해 설명

동시성 프로그래밍에서 발생할 수 있는 문제 중 하나인 가시성 문제를 해결하기 위해 사용되는 키워드 이다.  가시성 문제는 여러 개의 스레드가 사용됨에 따라, `CPU Cache Memory`와 `RAM`의 데이터가 서로 일치하지 않아 생기는 문제를 의미한다. `volatile` 키워드를 붙인 공유 자원은 `RAM`에 직접 읽고 쓰는 작업을 수행할 수 있도록 해준다.
