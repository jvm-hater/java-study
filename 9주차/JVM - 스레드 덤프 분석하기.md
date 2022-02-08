웹 애플리케이션에 장애가 발생하거나 느리게 동작할 때, 스레드 덤프를 분석해 봐야 한다. 이번 글에서는 스레드 덤프를 획득하는 방법부터 어떻게 스레드 덤프를 해석해야 하는지, 어떤 점을 눈여겨 봐야 하는지 설명하려 한다.

## 스레드 덤프가 필요한 이유

웹 서버에서는 많은 수의 동시 사용자를 처리하기 위해 수십~수백 개 정도의 스레드를 사용한다. 두 개 이상의 스레드가 같은 자원을 이용할 때는 필연적으로 스레드 간에 경합(Contention)이 발생하고 경우에 따라서는 데드락(Deadlock)이 발생할 수도 있다.

스레드 경합 때문에 다양한 문제가 발생할 수 있으며, 이런 문제를 분석하기 위해서는 **스레드 덤프(Thread Dump)**를 이용한다. 각 스레드의 상태를 정확히 알 수 있기 때문이다.

💡 **경합이란?**   
경합은 어떤 스레드가 다른 스레드가 획득하고 있는 락(lock)이 해제되기를 기다리는 상태를 말한다. 웹 애플리케이션에서 여러 스레드가 공유 자원에 접근하는 일은 매우 빈번하다.
대표적으로 로그를 기록하는 것도 로그를 기록하려는 스레드가 락을 획득하고 공유 자원에 접근하는 것이다.


💡 **데드락이란?**   
데드락은 스레드 경합의 특별한 경우인데, 두 개 이상의 스레드에서 작업을 완료하기 위해서 상대의 작업이 끝나야 하는 상황을 말한다.


## Java 스레드 배경 지식

### 스레드 동기화

- 스레드는 다른 스레드와 동시에 실행할 수 있다. 여러 스레드가 공유 자원을 사용할 때 정합성을 보장하려면 스레드 동기화로 한 번에 하나의 스레드만 공유 자원에 접근할 수 있게 해야 한다.
- Java에서는 Monitor를 이용해 스레드를 동기화한다. 모든 Java 객체는 하나의 Monitor를 가지고 있다. 그리고 Monitor는 하나의 스레드만 소유할 수 있다.
- 어떠한 스레드가 소유한 Monitor를 다른 스레드가 획득하려면 해당 Monitor를 소유하고 있는 스레드가 Monitor를 해제할 때까지 Wait Queue에서 대기하고 있어야 한다.

### 스레드 상태

스레드 덤프를 분석하려면 스레드의 상태를 알아야 한다. 스레드의 상태는 java.lang.Thread 클래스 내부에 State라는 이름을 가진 Enumerated Types(열거형)으로 선언되어 있다.

![image](https://user-images.githubusercontent.com/55661631/152982968-d67c90c5-32b8-4dc0-8ef4-609d59183d94.png)

- **NEW**
    - 스레드가 생성되었지만 아직 실행되지 않은 상태
- **RUNNABLE**
    - 현재 CPU를 점유하고 작업을 수행 중인 상태. 운영체제의 자원 분배로 인해 WAITING 상태가 될 수도 있다.
- **BLOCKED**
    - Monitor를 획득하기 위해 다른 스레드가 락을 해제하기를 기다리는 상태
- **WAITING**
    - `wait()` 메서드, `join()` 메서드, `park()` 메서드 등를 이용해 대기하고 있는 상태
- **TIMED_WAITING**
    - `sleep()` 메서드, `wait()` 메서드, `join()` 메서드, `park()` 메서드 등을 이용해 대기하고 있는 상태.
    - WAITING 상태와의 차이점은 메서드의 인수로 최대 대기 시간을 명시할 수 있어 외부적인 변화뿐만 아니라 시간에 의해서도 WAITING 상태가 해제될 수 있다는 것이다.

### 스레드 종류

Java 스레드는 **데몬 스레드(Daemon Thread)**와 **비데몬 스레드(Non-daemon Thread)**로 나눌 수 있다.

- 데몬 스레드는 다른 비데몬 스레드가 없다면 동작을 중지한다.
    - 대표적인 예로, `static void main(String[] args)` 메서드가 실행되는 스레드는 비데몬 스레드로 생성되며, 이 스레드가 동작을 중지하면 다른 데몬 스레드도 같이 동작을 중지하게 된다.

## 스레드 덤프 획득

### 1. jstack 사용

JDK 1.6 이상을 사용하고 있다면 jstack을 이용해 스레드 덤프를 획득할 수 있다. 

**Java 애플리케이션 프로세스 PID 확인**

```java
jps -v
```

`jstack`으로 스레드 덤프를 획득하려면, 먼저 수행 중인 Java 애플리케이션 프로세스 PID를 확인해야 한다.

**스레드 덤프 획득**

```java
jstack [PID]
```

`jps` 명령어로 획득한 PID를 인수로 넣어 jstack을 실행하면 스레드 덤프가 출력된다.

### 2. Java VisualVM 사용

Java VisualVM과 같은 GUI 프로그램으로도 스레드 덤프를 획득할 수 있다.

![image](https://user-images.githubusercontent.com/55661631/152983000-720e8215-1492-4ffc-a34e-f1b3f439b6ee.png)

- 왼쪽 패널는 실행 중인 프로세스 목록이다. 원하는 프로세스를 선택하면 된다.
- 다음 오른쪽 패널에서 Threads 탭을 선택하면 스레드의 정보를 실시간으로 확인할 수 있다.
- 스레드 덤프를 파일로 저장하려면 Thread Dump 버튼을 클릭하면 된다.

### 3. kill 사용

Linux, Unix 계열에서는 `kill` 명령어를 사용하여 확인 가능하다.

```
kill -3 [PID]
```

자바 프로그램을 실행한 터미널에서 `stdout`으로 출력될 것이고, `stdout`이 리다이렉트 된 경우엔 해당 파일을 확인하면 된다.

## 스레드 덤프 정보

획득한 스레드 덤프에는 다음과 같은 정보가 들어 있다.

![image](https://user-images.githubusercontent.com/55661631/152983039-b40c9a54-b5b3-4ee5-946a-85518212d446.png)

- **Thread Name**
    - 스레드 이름이며, 이름을 변경하여 사용하는 경우 스레드 덤프에도 반영된다. 일반적으로 스레드 덤프를 해석하기 쉽게 의미있는 이름으로 설정한다.
- **ID**
    - JVM 내 의 각 스레드에 할당된 고유 ID이다. 1부터 시작한다.
- **Thread Priority**
    - Java 스레드의 우선순위이다.
- **OS Thread Priority**
    - 자바의 스레드는 운영체제(OS)의 스레드와 매핑이 되는데, 매핑된 운영체제 스레드의 우선순위이다.
- **Java-Level Thread ID**
    - JVM 내부(JNI 코드)에서 관리하는 Native Thread 구조체의 포인터 주소이다.
- **Native Thread ID**
    - 자바 스레드에 매핑된 OS 스레드의 ID이다.
    - Windows에서는 OS Level의 스레드 ID이며, Linux에서는 LWP(Light Weight Process)의 ID를 의미한다.
- **Thread State**
    - 스레드의 상태이다.
- **Last Known Java Stack Pointer**
    - 스레드의 현재 Stack Pointer(SP)의 주소를 의미한다.
- **Call Stack**
    - 현재 스레드가 수행되는 함수들의 호출 관계(콜 스택)를 표현한다.

## 스레드 덤프 유형별 패턴

### 1. 락을 획득하지 못하는 경우(BLOCKED)

한 스레드가 락을 소유하고 있어 다른 스레드가 락을 획득하지 못해 애플리케이션의 전체적인 성능이 느려지는 경우이다.

다음 스레드 덤프 예에서는 **BLOCKED_TEST pool-1-thread-1** 스레드가 **<0x0000000780a000b0>** 락을 소유한 상태에서 동작하고 있어, **BLOCKED_TEST pool-1-thread-2** 스레드와 **BLOCKED_TEST pool-1-thread-3** 스레드는 **<0x0000000780a000b0>** 락을 획득하기 위해 대기하고 있는 상태이다.

**Java VisualVM**

![image](https://user-images.githubusercontent.com/55661631/152983110-aa73375e-f67b-4baa-bc4b-6b241bbadb3f.png)

**스레드 덤프**

```java
"BLOCKED_TEST pool-1-thread-1" prio=6 tid=0x0000000006904800 nid=0x28f4 runnable [0x000000000785f000]
	java.lang.Thread.State: RUNNABLE
			at java.io.FileOutputStream.writeBytes(Native Method)
			at java.io.FileOutputStream.write(FileOutputStream.java:282)
			at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:65)
			at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:123)
			- locked <0x0000000780a31778> (a java.io.BufferedOutputStream)
			at java.io.PrintStream.write(PrintStream.java:432)
			- locked <0x0000000780a04118> (a java.io.PrintStream)
			at sun.nio.cs.StreamEncoder.writeBytes(StreamEncoder.java:202)
			at sun.nio.cs.StreamEncoder.implFlushBuffer(StreamEncoder.java:272)
			at sun.nio.cs.StreamEncoder.flushBuffer(StreamEncoder.java:85)
			- locked <0x0000000780a040c0> (a java.io.OutputStreamWriter)
			at java.io.OutputStreamWriter.flushBuffer(OutputStreamWriter.java:168)
			at java.io.PrintStream.newLine(PrintStream.java:496)
			- locked <0x0000000780a04118> (a java.io.PrintStream)
			at java.io.PrintStream.println(PrintStream.java:687)
			- locked <0x0000000780a04118> (a java.io.PrintStream)
			at com.nbp.theplatform.threaddump.ThreadBlockedState.monitorLock(ThreadBlockedState.java:44)
			- locked <0x0000000780a000b0> (a com.nbp.theplatform.threaddump.ThreadBlockedState)
			at com.nbp.theplatform.threaddump.ThreadBlockedState$1.run(ThreadBlockedState.java:17)
			at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886)
			at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
			at java.lang.Thread.run(Thread.java:662)
			
	Locked ownable synchronizers:
			- <0x0000000780a31758> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)

 

"BLOCKED_TEST pool-1-thread-2" prio=6 tid=0x0000000007673800 nid=0x260c waiting for monitor entry [0x0000000008abf000]
	java.lang.Thread.State: BLOCKED (on object monitor)
			at com.nbp.theplatform.threaddump.ThreadBlockedState.monitorLock(ThreadBlockedState.java:43)
			- waiting to lock <0x0000000780a000b0> (a com.nbp.theplatform.threaddump.ThreadBlockedState)
			at com.nbp.theplatform.threaddump.ThreadBlockedState$2.run(ThreadBlockedState.java:26)
			at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886)
			at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
			at java.lang.Thread.run(Thread.java:662)

	Locked ownable synchronizers:
			- <0x0000000780b0c6a0> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)

 

"BLOCKED_TEST pool-1-thread-3" prio=6 tid=0x00000000074f5800 nid=0x1994 waiting for monitor entry [0x0000000008bbf000]
	java.lang.Thread.State: BLOCKED (on object monitor)
			at com.nbp.theplatform.threaddump.ThreadBlockedState.monitorLock(ThreadBlockedState.java:42)
			- waiting to lock <0x0000000780a000b0> (a com.nbp.theplatform.threaddump.ThreadBlockedState)
			at com.nbp.theplatform.threaddump.ThreadBlockedState$3.run(ThreadBlockedState.java:34)
			at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886)
			at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
			at java.lang.Thread.run(Thread.java:662)

	Locked ownable synchronizers:
			- <0x0000000780b0e1b8> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)
```

### 2. **데드락 상태인 경우**

스레드 A가 작업을 계속하려면 스레드 B가 소유한 락을 획득해야 하고, 스레드 B가 작업을 계속하려면 스레드 A가 소유한 락을 획득해야 해서 데드락 상태에 있는 경우이다.

다음 스레드 덤프 예에서 **DEADLOCK_TEST-1** 스레드는 **<0x00000007d58f5e48>** 락을 소유하고 있으며, **DEADLOCK_TEST-2** 스레드가 소유한 **<0x00000007d58f5e60>** 락을 획득하려 한다. 한편 **DEADLOCK_TEST-2** 스레드는 **<0x00000007d58f5e60>** 락을 소유하고 있으며, **DEADLOCK_TEST-3** 스레드가 소유한 **<0x00000007d58f5e78>** 락을 획득하려 한다. 그리고 **DEADLOCK_TEST-3** 스레드는 **<0x00000007d58f5e78>** 락을 수유하고 있으며, **DEADLOCK_TEST-1** 스레드가 소유한 **<0x00000007d58f5e48>** 락을 획득하려 한다.

**Java VisualVM**

![image](https://user-images.githubusercontent.com/55661631/152983153-30c608cb-7b00-4bee-ba5a-f52e1d893e34.png)

**스레드 덤프**

```java
"DEADLOCK_TEST-1" daemon prio=6 tid=0x000000000690f800 nid=0x1820 waiting for monitor entry [0x000000000805f000]
		java.lang.Thread.State: BLOCKED (on object monitor)
				at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.goMonitorDeadlock(ThreadDeadLockState.java:197)
				- waiting to lock <0x00000007d58f5e60> (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
				at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.monitorOurLock(ThreadDeadLockState.java:182)
				- locked <0x00000007d58f5e48> (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
				at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.run(ThreadDeadLockState.java:135)
	
		Locked ownable synchronizers:
				- None

 

"DEADLOCK_TEST-2" daemon prio=6 tid=0x0000000006858800 nid=0x17b8 waiting for monitor entry [0x000000000815f000]
		java.lang.Thread.State: BLOCKED (on object monitor)
				at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.goMonitorDeadlock(ThreadDeadLockState.java:197)
				- waiting to lock <0x00000007d58f5e78> (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
				at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.monitorOurLock(ThreadDeadLockState.java:182)
				- locked <0x00000007d58f5e60> (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
				at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.run(ThreadDeadLockState.java:135)
				
		Locked ownable synchronizers:
				- None

 

"DEADLOCK_TEST-3" daemon prio=6 tid=0x0000000006859000 nid=0x25dc waiting for monitor entry [0x000000000825f000]
		java.lang.Thread.State: BLOCKED (on object monitor)
				at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.goMonitorDeadlock(ThreadDeadLockState.java:197)
				- waiting to lock <0x00000007d58f5e48> (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
				at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.monitorOurLock(ThreadDeadLockState.java:182)
				- locked <0x00000007d58f5e78> (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
				at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.run(ThreadDeadLockState.java:135)
				
		Locked ownable synchronizers:
				- None
```

### 3. **스레드 리소스를 정상적으로 정리하지 못하는 경우**

불필요한 스레드가 계속해서 늘어나는 경우이다. 스레드 리소스를 정상적으로 정리 못하고 있는 경우이기 각 스레드를 정리하는 모습 혹은 스레드가 종료되는 조건을 확인하는 것이 좋다.

**Java VisualVM**

![image](https://user-images.githubusercontent.com/55661631/152983230-d656b142-565b-485b-8f02-3f46d6451c73.png)

## 참고

- [https://d2.naver.com/helloworld/10963](https://d2.naver.com/helloworld/10963)
- [https://bestugi.tistory.com/38](https://bestugi.tistory.com/38)
- [https://sjh836.tistory.com/151](https://sjh836.tistory.com/151)
- [https://blog.voidmainvoid.net/365](https://blog.voidmainvoid.net/365)

## 예상 면접 질문 및 답변

### Q. 스레드 모니터링과 스레드 덤프가 왜 필요한가?

웹 서버에서는 많은 수의 동시 사용자를 처리하기 위해 수많은 스레드를 사용한다. 두 개 이상의 스레드가 같은 자원을 이용할 때는 필연적으로 스레드 간에 경합(Contention)이 발생한다. 스레드 경합 때문에 데드락과 같은 다양한 문제가 발생할 수 있으며, 이런 문제들을 분석하기 위해서는 스레드 모니터링과 스레드 덤프가 필수이다.
