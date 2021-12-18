## 메인 스레드

모든 자바 애플리케이션은 메인 스레드가 `main()` 메소드를 실행하면서 시작되며, `main()` 메소드에서 마지막 코드를 실행하거나 return문을 만나면 종료된다. 

```java
public static void main(String[] args) { // 코드의 실행 흐름 => 스레드
    String data = null;

    if (...) {
    }

    while(...) {
    }
}
```

메인 스레드는 필요에 따라 작업 스레드를 만들어서 병렬로 코드를 실행할 수 있다. 

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F5089670e-3bc8-4691-ab6e-08dbcd834b9b%2FUntitled.png?table=block&id=7c56c8a4-4543-40e4-984e-99081120d01c&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

싱글 스레드 애플리케이션에서는 메인 스레드가 종료하면 프로세스도 종료되지만, 멀티 스레드 애플리케이션에서는 실행 중인 스레드가 하나라도 있으면 프로세스는 종료되지 않는다. 특히 메인 스레드가 작업 스레드보다 먼저 종료되어도 작업 스레드가 실행 중이라면 프로세스는 종료되지 않는 점을 주의해야 한다.

## 작업 스레드 생성과 실행

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F43e4f78e-3b63-4344-9369-f29fb688ae0e%2FUntitled.png?table=block&id=25765def-f832-4baf-8126-a864cee9936c&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

자바에서는 작업 스레드도 객체로 생성되므로 클래스가 필요하다. 크게 Thread 클래스를 직접 객체화하는 방법과 Thread를 상속한 클래스를 생성하는 방법이 있다.

### Thread 클래스에서 직접 생성

Thread 클래스로부터 작업 스레드 객체를 직접 생성하려면 다음과 같이 `Runnable` 을 매개값으로 갖는 생성자를 호출하면 된다.

```java
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        
    }
});
```

Runnable은 작업 스레드가 실행할 수 있는 코드를 가지고 있는 객체이다. 다만, Runnable은 인터페이스이므로 구현 클래스를 정의해 주어야 한다. `run()` 메소드를 재정의해 주면 되고, 위 코드 대신 람다식으로 간단하게 표현해 주어도 된다.

```java
Thread thread = new Thread(() -> System.out.println("작업 스레드입니다."));
```

작업 스레드는 생성되자마자 바로 실행되지는 않고, 별도로 `start()` 메소드를 호출해야한다. 해당 메소드가 호출되면 작업 스레드가 매개값으로 받은 Runnable의 `run()` 메소드를 호출하여 자신의 작업을 처리한다.

이제 스레드를 이해하기 위하여 예제를 하나 살펴 보자. 이번에 볼 예제는 0.5초 주기로 비프(beep)음을 발생시키면서 동시에 프린팅하는 작업이다. 

**메인 스레드만 사용한 예제**

```java
    public static void main(String[] args) {
        Toolkit toolkit = Toolkit.getDefaultToolkit();
        for (int i = 0; i < 5; i++) {
            toolkit.beep();
            try {
                Thread.sleep(500);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        for (int i = 0; i < 5; i++) {
            System.out.println("띵");
            try {
                Thread.sleep(500);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
```

메인 스레드만 있다면 비프음을 내는 작업이 다 끝나야만 프린팅이 가능하다. 실제로 실행해보면 비프음이 5번 발생이 발생한 이후에 출력이 되는 것을 알 수 있다.

**메인 스레드와 작업 스레드를 동시에 사용한 예제**

```java
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            Toolkit toolkit = Toolkit.getDefaultToolkit();
            for (int i = 0; i < 5; i++) {
                toolkit.beep();
                try {
                    Thread.sleep(500);
                } catch (Exception e) {
                    e.printStackTrace();;
                }
            }
        });
        thread.start();

        for (int i = 0; i < 5; i++) {
            System.out.println("띵");
            try {
                Thread.sleep(500);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
```

메인 스레드에서 비프음을 발생하던 일을 작업 스레드에게 위임하였다. `thread.start()` 를 호출하는 순간 새로운 작업 스레드가 비프음을 발생하는 일을 시작하고, 메인 스레드는 그 아래 비프음을 출력하는 작업을 시작한다. 거의 동시에 비프음이 발생하면 출력이 되는 것을 알 수 있다.

### Thread 하위 클래스에서 작성

말 그대로 Thread 클래스를 상속하여 `run()` 메소드를 재정의하는 방법이다.

```java
public class BeepThread extends Thread {

    @Override
    public void run() {
        Toolkit toolkit = Toolkit.getDefaultToolkit();
        for (int i = 0; i < 5; i++) {
            toolkit.beep();
            try {
                Thread.sleep(500);
            } catch (Exception e) {
                e.printStackTrace();;
            }
        }
    }

    // 추가 로직 작성 가능
}

public class Main {

    public static void main(String[] args) {
        Thread thread = new BeepThread();
        thread.start();

        for (int i = 0; i < 5; i++) {
            System.out.println("띵");
            try {
                Thread.sleep(500);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

람다식을 사용한 예제와 거의 유사하지만, Thread 클래스를 상속하여 만든 하위 클래스로 작업 스레드를 구현하면 `run()` 메소드 외에 프로그래머가 원하는 메소드를 더 정의할 수 있다는 장점이 있다.

## 스레드의 이름

스레드의 이름은 기본적으로 `Thread-n` 형식으로 붙여진다. n은 0부터 시작하며, 원한다면 프로그래머가 임의의 이름을 정해줄 수 있다.

만약 스레드의 이름은 모르지만 현재 코드를 실행하는 스레드가 누구인지 궁금할 수 있다. 이때는 Thread의 정적 메소드인 `currentThread()` 메소드를 호출하면 된다. 자세한 예제 코드를 아래를 살펴보자.

```java
public class Main {

    public static void main(String[] args) {
        Thread mainThread = Thread.currentThread();
        System.out.println("프로그램 시작 스레드 이름: " + mainThread.getName());

        Thread threadA = new ThreadA();
        System.out.println("작업 스레드 이름: " + threadA.getName());
        threadA.start();

        Thread threadB = new ThreadB();
        System.out.println("작업 스레드 이름: " + threadB.getName());
        threadB.start();
    }
}

public class ThreadA extends Thread{

    public ThreadA() {
        setName("ThreadA");
    }

    @Override
    public void run() {
        for (int i = 0; i < 2; i++) {
            System.out.println(getName() + "가 출력한 내용");
        }
    }
}

public class ThreadB extends Thread{

    @Override
    public void run() {
        for (int i = 0; i < 2; i++) {
            System.out.println(getName() + "가 출력한 내용");
        }
    }
}
```

ThreadA는 이름을 정해 주었고, ThreadB는 디폴트 명명 규칙을 따라 간다. 위 코드를 실행하면 다음가 같이 출력된다.

```java
프로그램 시작 스레드 이름: main
작업 스레드 이름: ThreadA
ThreadA가 출력한 내용
ThreadA가 출력한 내용
작업 스레드 이름: Thread-1
Thread-1가 출력한 내용
Thread-1가 출력한 내용
```

출력 순서는 환경에 따라 조금 다를 수 있다.

## 스레드 우선 순위

### 동시성 (Concurrency) vs 병렬성 (Parallelism)

동시성과 병렬성은 두 단어 모두 말 그대로 동시에 하는 것이 아닌가 생각이 들어서 혼동하기 쉽다.

- 동시성 (병행성)
    - 멀티 작업을 위해 하나의 코어에서 멀티 스레드가 번걸아가며 실행하는 성질이다.
    - CPU 하나가 Time Sharing 기법을 통해 실제로 동시에 스레드가 실행되는 것은 아니지만, CPU 제어권을 매우 빠르게 스레드에게 줬다가 뺏으면서 사람이 보기에 마치 동시에 실행되는 것처럼 보이는 것을 뜻한다.
- 병렬성
    - 멀티 작업을 위해 멀티 코어에서 개별 스레드를 할당 받아 동시에 실행하는 성질이다.
    - 정말 말 그대로 CPU 각자가 나뉘어서 각자의 일을 하여 실질적인 동시 작업을 수행하는 것을 뜻한다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F8565deef-43f8-4ec5-8ddd-f8120204c7e8%2FUntitled.png?table=block&id=b52ef7c6-e7ce-4044-9738-61da9c5c824f&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

동시성과 병렬성은 스레드 외에 프로세스에도 해당되지만, 이번 포스팅은 스레드에 관한 이야기이므로 프로세스는 논외로 한다.

### 스레드 스케줄링

스레드의 개수가 코어의 수보다 많을 경우 어떤 스레드에게 CPU 제어권을 주어야 하는지 결정해야 하는데, 이를 스레드 스케줄링이라고 한다. 스레드 스케줄링에 의해 스레드들은 아주 짧은 시간에 번갈아가면서 자신이 `run()` 메소드를 조금씩 실행한다.

자바의 스레드 스케줄링은 주로 우선 순위 방식과 라운드 로빈 방식을 사용하며, 전자는 프로그래머가 특정 스레드에게 우선 순위를 코드로 제어할 수 있지만 후자는 JVM에 의해 정해지므로 코드로 제어할 수 없다. 그래서 우선 순위 방식으로 스레드 스케줄링 하는 방법만 이야기하고자 한다.

### 우선 순위 방식

우선 순위는 1에서부터 10까지 주어지는데, 숫자가 클수록 우선 순위가 높다. Thread의 `setPriority()` 메소드를 통해 구현할 수 있다.

```java
public class Main {

    public static void main(String[] args) {
        for (int i = 1; i <= 10; i++) {
            Thread thread = new CalcThread("thread" + i);
            if (i != 10) {
                thread.setPriority(Thread.MIN_PRIORITY);
            } else {
                thread.setPriority(Thread.MAX_PRIORITY);
            }
            thread.start();
        }
    }
}

public class CalcThread extends Thread {

    public CalcThread(String name) {
        setName(name);
    }

    @Override
    public void run() {
        for (int i = 0; i < 2_000_000_000; i++) {

        }
        System.out.println(getName());
    }
}
```

가독성을 위해 Thread의 우선 순위 상수를 사용했다. MIN은 1, NORM은 5, MAX는 10을 뜻한다. 위와 같이 코드를 작성하여 실행하면 다음과 같은 결과를 얻는다.

```java
thread10
thread8
thread5
thread9
thread6
thread1
thread7
thread3
thread4
thread2
```

thread10이 우선 순위가 가장 높으므로 먼저 수행되고, 나머지는 우선 순위가 같으므로 정확한 순서를 판단할 수는 없다. 실제로 실행해 보면 매번 다르게 나온다.

## 동기화 메소드와 동기화 블록

### 공유 객체를 사용할 때의 주의할 점

멀티 스레드 환경에서 객체를 공유해서 사용하면 문제가 생길 수 있다. 예제 코드를 바로 살펴 보자.

```java
public class Main {

    public static void main(String[] args) {
        Calculator calculator = new Calculator();

        User1 user1 = new User1();
        user1.setCalculator(calculator);
        user1.start();

        User2 user2 = new User2();
        user2.setCalculator(calculator);
        user2.start();
    }
}

public class Calculator {

    private int memory;

    public int getMemory() {
        return memory;
    }

    public void setMemory(int memory) {
        this.memory = memory;
        try {
            Thread.sleep(2000);
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + ": " + this.memory);
    }

}

public class User1 extends Thread {

    private Calculator calculator;

    public void setCalculator(Calculator calculator) {
        this.setName("User1");
        this.calculator = calculator;
    }

    @Override
    public void run() {
        calculator.setMemory(100);
    }
}

public class User2 extends Thread {

    private Calculator calculator;

    public void setCalculator(Calculator calculator) {
        this.setName("User2");
        this.calculator = calculator;
    }

    @Override
    public void run() {
        calculator.setMemory(50);
    }
}
```

User1이 계산기에 100을 입력하고 잠시 화장실을 다녀 올 동안 User2가 계산기에 50을 입력하게 되면, User1이 보는 계산기는 50이므로 의도한 결과와 다르게 연출이 된다.

```java
User2: 50
User1: 50
```

### 동기화 메소드 및 동기화 블록

**임계 영역 (Critical Section)**

멀티 스레드 프로그램에서 단 하나의 스레드만 실행할 수 있는 코드 영역을 임계 영역이라고 부른다. 

그리고 자바는 이러한 임계 영역을 지정하기 위해 동기화(synchronized) 메소드와 동기화 블록을 제공한다. 스레드가 객체 내부의 동기화 메소드 또는 블록에 들어가면 즉시 해당 객체에 lock을 걸어 다른 스레드가 임계 영역 코드를 실행하지 못하도록 한다.

**동기화 메소드**

```java
public synchronized void method() {
    임계 영역; // 단 하나의 스레드만 실행
}
```

동기화 메소드는 메소드 어디든 붙일 수 있으며, 스레드가 동기화 메소드를 실행하는 즉시 객체에 lock이 걸리고 동기화 메소드 실행을 종료해야 lock이 풀린다.

**동기화 블록**

일부 내용만 임계 영역을 만들고 싶다면 다음과 같이 동기화 블록을 만들 수 있다.

```java
public void method() {
    // 여러 스레드가 실행 가능한 영역
    ...
    synchronized(공유 객체) {
        임계 영역 // 단 하나의 스레드만 실행
    }
    // 여러 스레드가 실행 가능한 영역
    ...
}
```

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Faba2ca9a-41f0-4908-b11c-495f26f9b920%2FUntitled.png?table=block&id=15535d8f-ca21-4c52-babe-822fb7842395&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

### 계산기 예제에 동기화 처리

**동기화 메소드**

```java
public class Calculator {

    private int memory;

    public int getMemory() {
        return memory;
    }

    public synchronized void setMemory(int memory) {
        this.memory = memory;
        try {
            Thread.sleep(2000);
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + ": " + this.memory);
    }

}
```

공유 자원을 사용하는 메소드에 동기화 처리를 해 주면 끝난다. 구체적인 실행 과정은 다음과 같다.

1. User1 스레드는 Calculator 객체의 동기화 메소드인 `setMemory()` 를 실행하는 순간 Calculator 객체에 lock을 건다.
2. 메인 스레드가 User2 스레드를 실행하지만, 동기화 메소드인 `setMemory()` 를 실행하지는 못하고 User1이 `setMemory()` 를 모두 실행할 때까지 기다린다.
3. User2가 `setMemory()` 를 실행한다.

**동기화 블록**

```java
public class Calculator {

    private int memory;

    public int getMemory() {
        return memory;
    }

    public void setMemory(int memory) {
        synchronized (this) {
            this.memory = memory;
            try {
                Thread.sleep(2000);
            } catch (Exception e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + ": " + this.memory);
        }
    }

}
```

동기화 블록을 사용하면, 해당 메소드를 실행할 수는 있지만 동기화 블록에 한 스레드가 진입하는 순간 동기화 블록의 매개 변수에 해당하는 객체에 lock을 건다. 그 외의 과정은 동기화 메소드와 같다.

## 스레드 상태

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F7c170f40-25f9-428e-b68f-9eb3a3110490%2FUntitled.png?table=block&id=aa3eef74-ccd4-4984-934b-7e7393f17d43&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

`getState()` 메소드를 통해 스레드의 상태를 확인할 수 있다. 위 그림과 같이 스레드 객체가 만들어져서 실행 대기하다가 실행도 하고 일시 정지 상태가 되다가 최종적으로 종료 상태가 된다.

### 스레드 상태에 따른 열거 상수 정리

- 객체 생성
    - NEW
        - 스레드 객체가 생성되었고, 아직 `start()` 메소드가 호출되지 않은 상태
- 실행 대기
    - RUNNABLE
        - 실행 상태로 언제든지 갈 수 있는 상태
- 일시 정지
    - WAITING
        - 다른 스레드가 통지할 때까지 기다리는 상태
    - TIMED_WAITING
        - 주어진 시간 동안 기다리는 상태
    - BLOCKED
        - 사용하고자 하는 객체의 lock이 풀릴 때까지 기다리는 상태
- 종료
    - TERMINATED
        - 실행을 마친 상태

### 스레드 상태를 확인하는 예제

```java
public class Main {

    public static void main(String[] args) {
        StatePrintThread statePrintThread = new StatePrintThread(new TargetThread());
        statePrintThread.start();
    }
}

public class StatePrintThread extends Thread{

    private Thread targetThread;

    public StatePrintThread(Thread targetThread) {
        this.targetThread = targetThread;
    }

    @Override
    public void run() {
        while(true) {
            Thread.State state = targetThread.getState();
            System.out.println("타켓 스레드 상태: " + state);

            if (state == Thread.State.NEW) {
                targetThread.start();
            }

            if (state == Thread.State.TERMINATED) {
                break;
            }

            try {
                Thread.sleep(300);
            } catch (Exception e) {
                e.printStackTrace();;
            }
        }
    }
}

public class TargetThread extends Thread {

    @Override
    public void run() {
        for (long i = 0; i < 1_000_000_000; i++) {
        }

        try {
            Thread.sleep(1000);
        } catch (Exception e) {
            e.printStackTrace();
        }

        for (long i = 0; i < 1_000_000_000; i++) {
        }
    }
}
```

**StatePrintThread**

- 0.3초마다 현재 스레드의 상태를 출력한다.
- 스레드가 NEW 상태면 RUNNABLE 상태로 만들어 주고, 스레드가 TERMINATED 상태면 무한 루프를 종료한다.

**TargetThread** 

- 10억 번 루프를 돌고, 1초 동안 스레드를 TIMED_WATING 상태에 빠지게 하고, 다시 10억 번 루프를 돈다.

**출력 결과**

```java
타켓 스레드 상태: NEW
타켓 스레드 상태: RUNNABLE
타켓 스레드 상태: TIMED_WAITING
타켓 스레드 상태: TIMED_WAITING
타켓 스레드 상태: TIMED_WAITING
타켓 스레드 상태: RUNNABLE
타켓 스레드 상태: TERMINATED
```

환경에 따라 출력 결과는 다르게 나올 수 있다.

## 스레드 상태 제어

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fc4693664-ee46-4743-a412-291e21561d38%2FUntitled.png?table=block&id=790dddf1-ed78-4ec8-8717-0a6cce87deb9&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

위에서는 스레드 상태를 단순히 알아보았지만, 이번에는 스레드 상태를 제어할 수 있는 메소드를 소개하려고 한다.

| 메소드 | 설명 |
| --- | --- |
| interrupt() | 일시 정지 상태의 스레드에서 InterruptedException 예외를 발생시켜, 예외 처리 코드(catch)에서 실행 대기 상태로 가거나 종료 상태로 갈 수 있도록 한다. |
| notify(), notifyAll() | 동기화 블록 내에서 wait() 메소드에 의해 일시 정지 상태에 있는 스레드를 실행 대기 상태로 만든다. |
| resume() | suspend() 메소드에 의해 일시 정지 상태에 있는 스레드를 실행 대기 상태로 만든다. 다만, 이 메소드는 Deprecated 되었으므로 notify()나 notifyAll()을 사용한다. |
| sleep() | 주어진 시간동안 스레드를 일시 정지 상태로 만든다. 주어진 시간이 지나면 자동으로 실행 대기 상태가 된다. |
| join() | join() 메소드를 호출한 스레드는 일시 정지 상태가 된다. 실행 대기 상태로 가려면, join() 메소드를 멤버로 가지는 스레드가 종료되거나, 매개 값으로 주어진 시간이 지나야 한다. |
| wait() | 동기화 블록 내에서 스레드를 일시 정지 상태로 만든다. 매개 값으로 주어진 시간이 지나면 자동으로 실행 대기 상태가 된다. 시간이 주어지지 않으면 notify()나 notifyAll()을 호출해야만 실행 대기 상태로 갈 수 있다. |
| suspend() | 스레드를 일시 정지 상태로 만든다. resume() 메소드를 호출하면 다시 실행 대기 상태가 된다. 다만, 이 메소드는 Deprecated 되었으므로 wait()을 사용한다. |
| yield() | 실행 중에 우선 순위가 동일한 다른 스레드에게 실행을 양보하고 실행 대기 상태가 된다. |
| stop() | 스레드를 즉시 종료한다. 다만, 이 메소드는 Deprecated 되었으므로 사용하지 않는 것이 좋다. |

위 표에서 wait(), notify(), notifyAll()은 Object 클래스의 메소드이고 나머지는 Thread 클래스의 메소드이다.

### 주어진 시간동안 일시 정지 (`sleep()`)

```java
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    // interrupt() 메소드가 호출되면 실행
}
```

말 그대로 매개 변수로 들어온 시간만큼 스레드를 일시 정지 상태로 만들고, 시간이 다 지나면 실행 대기 상태로 변경한다. 일시 정지 시간이 다 지나기 전에 스레드를 실행 대기 상태로 만들고 싶다면 `interrupt()` 메소드를 외부에서 호출하면 된다. `interrupt()` 메소드는 나중에 자세히 살펴 본다.

### 다른 스레드에게 실행 양보 (`yield()`)

`yield()` 메소드를 호출한 스레드는 실행 대기 상태로 돌아가고 동일한 우선 순위 또는 높은 우선 순위를 갖는 다른 스레드가 실행 기회를 가질 수 있도록 한다.

```java
public class Main {

    public static void main(String[] args) {
        ThreadA threadA = new ThreadA();
        ThreadB threadB = new ThreadB();

        threadA.start();
        threadB.start();

        try {
            Thread.sleep(30);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        threadA.work = false;

        try {
            Thread.sleep(30);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        threadA.work = true;

        try {
            Thread.sleep(30);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        threadA.stop = true;
        threadB.stop = true;
    }
}

public class ThreadA extends Thread {

    public boolean stop = false;
    public boolean work = true;

    @Override
    public void run() {
        while (!stop) {
            if (work) {
                System.out.println("ThreadA 작업 내용");
            } else {
                Thread.yield();
            }
        }
        System.out.println("ThreadA 작업 종료");
    }
}

public class ThreadB extends Thread {

    public boolean stop = false;
    public boolean work = true;

    @Override
    public void run() {
        while (!stop) {
            if (work) {
                System.out.println("ThreadB 작업 내용");
            } else {
                Thread.yield();
            }
        }
        System.out.println("ThreadB 작업 종료");
    }
}
```

ThreadA와 ThreadB 모두 각각 플래그를 가지고 있고, 이 플래그에 따라 출력 내용이 추가되거나 무한 루프를 탈출한다. `run()` 메소드를 보면 work가 false일 때 `yield()` 를 호출하여 다른 스레드에게 CPU 제어권을 넘기는 것을 알 수 있다.

그래서 `main()` 메소드에서 초반에는 두 스레드가 번갈아가면서 CPU를 얻다가 ThreadA의 work가 false가 되면 `yield()` 메소드를 호출하여 ThreadB가 CPU 제어권을 많이 얻도록 한다. 이런 식으로 무의미하게 CPU를 잡고 시간을 보내는 일을 줄일 수 있다.

### 다른 스레드의 종료를 기다림 (`join()`)

A 스레드가 B 스레드의 일이 종료할 때까지 기다렸다가 일을 수행해야 할 수 있다. 이때 사용하는 메소드가 바로 `join()` 이다.

```java
public class Main {

    public static void main(String[] args) {
        SumThread sumThread = new SumThread();
        sumThread.start();

        try {
            sumThread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("1~100 합: " + sumThread.getSum());
    }
}

public class SumThread extends Thread {

    private long sum;

    public long getSum() {
        return sum;
    }

    public void setSum(long sum) {
        this.sum = sum;
    }

    @Override
    public void run() {
        for (int i = 1; i <= 100; i++) {
            sum += i;
        }
    }
}
```

Main 함수에서 `sumThread.join()` 을 통해 sumThread의 작업이 끝날 때까지 메인 스레드는 대기한다. 만약 `join()` 메소드가 없다면 바로 출력문을 실행하므로 1에서 100까지의 합이 0으로 출력될 것이다.

### 스레드 간 협업 (`wait()` , `notify()` , `notifyAll()`)

정확히 교대 작업이 필요한 상황이 있다. 자신의 작업이 끝나면 상대방 스레드를 일시 정지 상태에서 풀어 주고, 자신은 일시 정지 상태로 만드는 것이다. 

이 방법은 공유 객체를 사용하는데, 먼저 공유 객체에 대해 두 스레드가 작업할 내용을 각각 동기화 메소드 또는 동기화 블록 처리를 한다. 그리고 한 스레드가 작업을 완료하면 `nofity()` 메소드를 호출해서 일시 정지 상태에 있는 다른 스레드를 실행 대기 상태로 만들고 자신은 `wait()` 메소드를 호출하여 일시 정지 상태로 만든다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F3f68e678-2c89-43e7-9466-60ce9edbf14b%2FUntitled.png?table=block&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&id=f5e9e0ed-903d-40d9-b3ed-2fa6c59742e6&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

만약 `wait()` 대신 `wait(long timeout)` 같이 매개 변수가 있는 wait을 사용하면 `notify()` 메소드 없이도 주어진 시간이 지나면 자동으로 스레드가 실행 대기 상태가 된다. 참고로 `notify()` 는 `wait()` 에 의해 일시 정지된 스레드 중 하나를 실행 대기 상태로 만들고,  `notifyAll()` 은 `wait()` 에 의해 일시 정지된 모든 스레드를 실행 대기 상태로 만든다.

위에서 언급한 3가지 메소드는 반드시 동기화 메소드 또는 동기화 블록 내에서 사용해야 한다.

**예제1**

```java
public class Main {

    public static void main(String[] args) {
        WorkObject sharedObject = new WorkObject();

        ThreadA threadA = new ThreadA(sharedObject);
        ThreadB threadB = new ThreadB(sharedObject);

        threadA.start();
        threadB.start();
    }
}

public class WorkObject {

    public synchronized void methodA() {
        System.out.println("ThreadA의 methodA() 작업 실행");
        notify(); // 일시 정지 상태에 있는 ThreadB를 실행 대기 상태로 만듦
        try {
            wait(); // ThreadA를 일시 정지 상태로 만듦
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public synchronized void methodB() {
        System.out.println("ThreadB의 methodB() 작업 실행");
        notify(); // 일시 정지 상태에 있는 ThreadA를 실행 대기 상태로 만듦
        try {
            wait(); // ThreadB를 일시 정지 상태로 만듦
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}

public class ThreadA extends Thread {

    private WorkObject workObject;

    public ThreadA(WorkObject workObject) {
        this.workObject = workObject;
    }

    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            workObject.methodA();
        }
    }
}

public class ThreadB extends Thread {

    private WorkObject workObject;

    public ThreadB(WorkObject workObject) {
        this.workObject = workObject;
    }

    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            workObject.methodB();
        }
    }
}
```

주석에 써 놓은 대로 서로의 스레드를 일시 정지 상태로 만들었다가 실행 대기 상태로 만드는 과정을 반복하는 것을 알 수 있다.

```java
ThreadA의 methodA() 작업 실행
ThreadB의 methodB() 작업 실행
ThreadA의 methodA() 작업 실행
ThreadB의 methodB() 작업 실행
ThreadA의 methodA() 작업 실행
ThreadB의 methodB() 작업 실행
```

**예제2**

이번 예제는 데이터를 저장하는 생산자 스레드와 데이터를 소비하는 소비자 스레드가 각자의 역할을 교대로 수행하는 내용이다. 생산자 스레드는 소비자 스레드가 읽기 전에 새로운 데이터를 두 번 생성하면 안 되고, 소비자 스레드는 생산자 스레드가 새로운 데이터를 생성하기 전에 이전 데이터를 두 번 읽으면 안 된다.

```java
public class Main {

    public static void main(String[] args) {
        DataBox dataBox = new DataBox();

        ProducerThread producerThread = new ProducerThread(dataBox);
        ConsumerThread consumerThread = new ConsumerThread(dataBox);

        producerThread.start();
        consumerThread.start();
    }
}

public class DataBox {

    private String data;
		
    // 소비자 스레드의 임계 영역
    public synchronized String getData() {
        if (this.data == null) { // 데이터가 있어야 읽을 수 있다.
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        String returnValue = data;
        System.out.println("ConsumerThread가 읽은 데이터: " + returnValue);
        data = null;
        notify();
        return returnValue;
    }

    // 생산자 스레드의 임계 영역
    public synchronized void setData(String data) {
        if (this.data != null) { // 데이터가 없어야 생산이 가능하다.
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        this.data = data;
        System.out.println("ProducerThread가 생산한 데이터: " + data);
        notify();
    }

}

public class ProducerThread extends Thread {

    private DataBox dataBox;

    public ProducerThread(DataBox dataBox) {
        this.dataBox = dataBox;
    }

    @Override
    public void run() {
        for (int i = 1; i <= 3; i++) {
            String data = "Data-" + i;
            dataBox.setData(data);
        }
    }
}

public class ConsumerThread extends Thread {

    private DataBox dataBox;

    public ConsumerThread(DataBox dataBox) {
        this.dataBox = dataBox;
    }

    @Override
    public void run() {
        for (int i = 1; i <= 3; i++) {
            String data = dataBox.getData();
        }
    }
}
```

## 스레드의 안전한 종료 (stop 플래그, `interrupt()`)

가장 쉬운 방법은 `stop()` 메소드를 사용하여 실행 중인 스레드를 강제 종료할 수 있다. 다만, 해당 메소드는 사용 중이었던 자원을 회수하지 않고 종료해 버린다는 단점으로 인해 현재 Deprecated된 상태이므로 이 메소드는 사용을 지양하자. 그 대신 아래에서 2가지 대체 방법을 소개하고자 한다.

### stop 플래그를 이용하는 방법

```java
public class XXXThread extends Thread {

    private boolean stop;

    @Override
    public void run() {
        while (!stop) {
            // 스레드가 반복 실행하는 코드;
        }
        // 스레드가 사용한 자원 정리
    }
}
```

### `interrupt()` 메소드를 이용하는 방법

`interrupt()` 메소드는 스레드가 일시 정지 상태에 있을 때 InterruptedException 예외를 발생시키는 역할을 한다. 이를 이용하여 프로그래머는 catch 블록에서 스레드를 정상 종료하거나, `Thread.interrupted()` 메소드를 통해 현재 스레드가 interrupt 요청을 받았는지 확인하여 스레드를 종료할 수 있다.

**InterruptedException 예외를 이용한 예제**

```java
public class Main {

    public static void main(String[] args) {
        Thread thread = new PrintThread();
        thread.start();

        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        thread.interrupt();
    }
}

public class PrintThread extends Thread {

    @Override
    public void run() {
        try {
            while (true) {
                System.out.println("실행 중");
                Thread.sleep(1);
            }
        } catch (InterruptedException e) {
        }

        System.out.println("자원 정리");
        System.out.println("실행 종료");
    }
}
```

주의할 점은 스레드가 실행 대기 상태거나 실행 중인 상태라면 interrupt 요청이 들어와도 스레드를 종료하지 않는다. 대신 interrupt 요청이 들어온 상태에서 일시 정지 상태가 되는 즉시 InterruptedException 예외가 발생한다. 이를 의도하기 위해 PrintThread의 `run()` 메소드 내부에 `Thread.sleep(1)` 을 넣어 놓은 것이다.

`Thread.interrupted()` 메소드를 이용한 예제

```java
public class Main {

    public static void main(String[] args) {
        Thread thread = new PrintThread();
        thread.start();

        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        thread.interrupt();
    }
}

public class PrintThread extends Thread {

    @Override
    public void run() {
        while (true) {
            System.out.println("실행 중");
            if (Thread.interrupted()) {
                break;
            }
        }

        System.out.println("자원 정리");
        System.out.println("실행 종료");
    }
}
```

현재 스레드에 대해 `interrupt()` 메소드가 실행 되었는지 확인하고, 실행 된 상태라면 true를 반환하여 무한 루프를 탈출하는 로직이다.

## 데몬 스레드

데몬 스레드는 주 스레드의 작업을 돕는 보조 스레드로, 스레드가 종료되면 데몬 스레드는 같이 종료되는 특징이 있다. 데몬 스레드의 적용 예로는 워드프로세서의 자동 저장, 미디어 플레이어의 동영상 및 음악 재생, 가비지 컬렉터 등이 있다. 이 기능들은 주 스레드인 워드 프로세서, 미디어 플레이어, JVM이 종료되면 같이 종료된다.

스레드를 데몬 스레드를 만들어 주려면 thread 내부의 `setDaemon()` 메소드의 매개 변수를 true로 하여 실행하면 된다.

```java
public class Main {

    public static void main(String[] args) {
        AutoSaveThread autoSaveThread = new AutoSaveThread();
        autoSaveThread.setDaemon(true);
        autoSaveThread.start();
        
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e){
            e.printStackTrace();
        }

        System.out.println("메인 스레드 종료");
    }
}

public class AutoSaveThread extends Thread {

    public void save() {
        System.out.println("작업 내용을 저장함.");
    }

    @Override
    public void run() {
        while (true) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                break;
            }
            save();
        }
    }
}
```

AutoSaveThread는 메인 스레드의 데몬 스레드가 되어 보조 역할을 수행한다. AutoSaveThread는 1초 주기로 작업 내역을 저장하다가 메인 스레드가 종료되면 같이 종료된다.

## 스레드 그룹

스레드 그룹은 관련된 스레드를 묶어서 관리할 목적으로 이용된다. JVM이 실행되면 system 스레드 그룹을 만들고, JVM 운영에 필요한 스레드를 생성하여 system 스레드 그룹에 포함한다. 그리고 system의 하위 스레드 그룹으로 main을 만들고 메인 스레드를 main 스레드 그룹에 포함한다.

스레드는 반드시 하나의 스레드 그룹에 포함되는데, 스레드 그룹을 명시적으로 정해주지 않으면 자신을 생성한 스레드와 같은 스레드 그룹에 속한다. 프로그래머가 생성하는 작업 스레드는 대부분 메인 스레드가 생성하므로 main 스레드 그룹에 속한다 생각하면 된다.

### 스레드 그룹 이름 얻기

`Thread.currentThread().getThreadGroup()` 를 이용하여 특정 스레드의 그룹 이름을 가져오거나 `Thread.getAllStackTraces()` 를 이용하여 현재 프로세스 내에서 실행하는 모든 스레드에 대한 정보를 얻어와서 그룹 이름을 얻을 수 있다.

```java
public class Main {

    public static void main(String[] args) {
        AutoSaveThread autoSaveThread = new AutoSaveThread();
        autoSaveThread.setName("AutoSaveThread");
        autoSaveThread.setDaemon(true);
        autoSaveThread.start();

        Map<Thread, StackTraceElement[]> threadInfos = Thread.getAllStackTraces();
        threadInfos.keySet().forEach(thread -> {
            System.out.println("Name: " + thread.getName() + ((thread.isDaemon()) ? "(데몬)" : "(주)"));
            System.out.println("\t" + "소속 그룹: " + thread.getThreadGroup().getName());
            System.out.println();
        });
    }
}
```

이번 예제에서는 현재 프로세스 내의 모든 스레드 정보를 가져와 보았다. 출력 결과는 다음과 같다.

```java
Name: Finalizer(데몬)
	소속 그룹: system

Name: Monitor Ctrl-Break(데몬)
	소속 그룹: main

Name: Reference Handler(데몬)
	소속 그룹: system

Name: Common-Cleaner(데몬)
	소속 그룹: InnocuousThreadGroup

Name: AutoSaveThread(데몬)
	소속 그룹: main

Name: Attach Listener(데몬)
	소속 그룹: system

Name: Signal Dispatcher(데몬)
	소속 그룹: system

Name: main(주)
	소속 그룹: main
```

참고로 Finalizer 스레드는 가비지 컬렉션을 담당한다.

### 스레드 그룹 생성

스레드 그룹을 만드는 방법은 2가지가 있다.

**부모 스레드 그룹을 명시하지 않음**

```java
ThreadGroup tg = new ThreadGroup(String name);
```

위 경우에는 현재 스레드가 속한 그룹의 하위 그룹으로 생성된다.

**부모 스레드 그룹을 명시함**

```java
ThreadGroup tg = new ThreadGroup(ThreadGroup parent, String name);
```

매개 변수로 넘겨 준 parent에 해당하는 스레드가 속한 그룹의 하위 그룹으로 생성된다.

위의 2가지 중에 한 가지 방법으로 스레드 그룹을 생성하면, 이 그룹에 스레드를 포함할 수 있다. 다음 4가지 생성자 중 하나를 택하면 된다.

```java
Thread t = new Thread(ThreadGroup group, Runnable target);
Thread t = new Thread(ThreadGroup group, Runnable target, String name);
Thread t = new Thread(ThreadGroup group, Runnable target, String name, long stackSize);
Thread t = new THread(ThreadGroup group, String name);
```

Runnable 타입의 target은 Runnable의 구현 객체, stackSize는 JVM이 이 스레드에 할당할 stack의 크기이다.

### 스레드 그룹의 일괄 `interrupt()`

스레드 그룹의 `interrupt()` 메소드를 호출하면 해당 그룹에 속한 모든 스레드에 interrupt 요청을 보낼 수 있다.

```java
public class Main {

    public static void main(String[] args) {
        ThreadGroup myGroup = new ThreadGroup("myGroup");
        WorkThread workThreadA = new WorkThread(myGroup, "workThreadA");
        WorkThread workThreadB = new WorkThread(myGroup, "workThreadB");

        workThreadA.start();
        workThreadB.start();

        System.out.println("[main 스레드 그룹의 list() 메소드 출력 내용]");
        ThreadGroup mainGroup = Thread.currentThread().getThreadGroup();
        mainGroup.list();
        System.out.println();

        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("[myGroup 스레드 그룹의 interrupt() 메소드 호출");
        myGroup.interrupt();
    }
}

public class WorkThread extends Thread {

    public WorkThread(ThreadGroup threadGroup, String threadName) {
        super(threadGroup, threadName);
    }

    @Override
    public void run() {
        while (true) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                System.out.println(getName() + " interrupted");
                break;
            }
        }
        System.out.println(getName() + " 종료됨");
    }
}
```

출력 결과는 다음과 같다.

```java
[main 스레드 그룹의 list() 메소드 출력 내용]
java.lang.ThreadGroup[name=main,maxpri=10]
    Thread[main,5,main]
    Thread[Monitor Ctrl-Break,5,main]
    java.lang.ThreadGroup[name=myGroup,maxpri=10]
        Thread[workThreadA,5,myGroup]
        Thread[workThreadB,5,myGroup]

[myGroup 스레드 그룹의 interrupt() 메소드 호출
workThreadB interrupted
workThreadA interrupted
workThreadA 종료됨
workThreadB 종료됨
```

현재 main 스레드 그룹의 스레드 목록을 보여 준 뒤, myGroup 스레드 그룹의 스레드에 대해 모두 interrupt 요청을 보내서 해당 스레드를 종료하였다.
