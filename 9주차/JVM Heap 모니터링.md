
# JVM Heap 모니터링하기

## Heap 영역 모니터링의 필요성

Spring 개발한 웹 애플리케이션을 운영하다 보면, 어떠한 이유로 사용하지 않는 메모리가 쌓이면서 메모리 누수 현상이 발생할 수 있다. 이것은 모니터링을 하지 않는 이상 정확한 이유를 알기 매우 어려우며, 메모리 누수를 방치하면 어느 순간 OOM(Out Of Memory) 에러가 발생할 수 있다. 따라서 주기적으로 Heap 영역을 모니터링하고, 의심되는 상황이 있다면 Heap Dump를 떠서 어떤 객체가 원인이 분석해야 한다.

## Heap 모니터링

### VisualVM

[**VisualVM**](https://visualvm.github.io/download.html)은 OracleJDK에서 제공하는 GUI 모니터링 툴이다. ****링크를 들어가서 압축 파일을 다운로드한 후, bin 경로에 있는 exe 파일을 실행하자. 그리고 VisualVM 상단에 [Tools] - [Plugin]을 접속하여 VisualGC 플러그인을 다운로드 받는다. 해당 플러그인은 GC를 좀 더 자세하고 이쁘게(?) 볼 수 있도록 도와준다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F7e9042c2-7805-4466-8c06-3e8c244897ae%2FUntitled.png?table=block&id=1067beeb-d644-4f62-bb09-57fdf7cf4f4a&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

플러그인 다운이 끝났다면, 창을 종료하고 좌측 메뉴를 살펴 보자.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F88585ead-7cec-42e4-82b4-84b96e8d9797%2FUntitled.png?table=block&id=61a25d07-009a-450a-9672-d1c09760a078&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

다양한 프로세스 목록을 확인할 수 있다. 필자는 위 예제의 애플리케이션 이름을 HeapDumpApplication로 지었으므로 pid가 26532번인 프로세스를 사용하려고 한다. 해당 프로세스를 더블 클릭하면 오른쪽 창에 여러 가지 탭이 보인다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F87879ebb-9fb4-4729-abac-396067c51a1f%2FUntitled.png?table=block&id=180ef4fd-a7fd-4649-a656-6011839729d3&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

여기서 Monitor를 클릭하면 전반적인 CPU 사용량, GC 현황 등을 확인할 수 있다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F0a86cd33-585b-4f53-b833-de70b4c15c69%2FUntitled.png?table=block&id=d418bde8-93fb-47bd-85f4-ea72bbff13e9&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

그 외의 Visual GC를 통해 JVM Heap 메모리 영역을 중점적으로 살펴볼 수 있다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fdcdb9a7c-470d-419d-b589-abb32239197c%2FUntitled.png?table=block&id=51914312-a7b8-4e7c-b24b-d5b7da921af4&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

필자는 아래에서 이야기할 GC를 괴롭히는 API를 호출해 보았다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F96f8089b-ae11-4321-9456-816b0a2e92de%2FUntitled.png?table=block&id=86732d9d-c88e-4c6c-80cb-f0eb8036931a&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

짧은 시간에 엄청나게 많은 객체가 생성되어 Old Generation이 꽉 차기 시작하고, 조금 더 지나면 거의 100% 꽉 차는 것을 확인할 수 있다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F0073fe9b-66b2-427b-b5a0-2dbe96168f0d%2FUntitled.png?table=block&id=a62f4051-d381-4808-b0d4-c0f88acd43f7&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Monitor에서도 Heap 메모리 범위를 초과할랑 말랑하고 있는 그래프를 확인할 수 있다. 이제 여기서 시간이 좀 더 지나면 애플리케이션은 더 이상 버티지 못하고 OOM 에러를 내뱉고 죽게 된다.

### jstat

jstat 명령어를 통해 간단히 콘솔 환경에서 모니터링을 수행할 수 있다. 많은 명령어가 있지만, gc와 관련된 `-gc` 옵션과 `-gcutil` 만 설정하여 살펴 보자. 그 전에 jps 명령어에 대해 알아야 한다.

```java
$ jps
26532 HeapDumpApplication
... 중략
```

jps 명령어를 입력하게 되면 현재 JVM에서 실행되고 있는 애플리케이션의 PID 목록을 얻어올 수 있다. 이 PID를 알아야 jstat 명령어를 비로소 써 먹을 수 있게 된다.

```java
$ jstat -gc 26532
S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
0.0    0.0    0.0    0.0   28672.0  10240.0   40960.0     9744.5   36784.0 34801.9 4864.0 4223.8     52    3.302  16     43.246   46.549
```

gc 옵션은 GC 통계 정보를 출력하며, 각 데이터의 의미는 아래를 참고하자.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2adbffd9-2652-43b1-8687-c0026d603a70%2FUntitled.png?table=block&id=fc05222d-0e30-493d-96e3-1fd89cc86ba8&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

```java
$ jstat -gcutil 26532
S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
0.00   0.00  35.71  23.79  94.61  86.84     52    3.302    16   43.246   46.549
```

gcutil 옵션은 각 영역의 사용량을 나타내기 때문에 한 눈에 Heap 메모리 현황을 보기 좋다. 각 데이터의 의미는 아래를 참고하자.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F8e285d1e-e9e9-461e-93c7-b44a08fddeb0%2FUntitled.png?table=block&id=205cbc70-ebaa-4c94-b5c1-45ce484be44c&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

개인적으로는 다음과 같이 연속적으로 결과를 볼 수 있는 명령어 조합을 선호한다.

```java
$ jstat -gcutil -h5 26532 1000 10
S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
0.00   0.00  35.71  23.79  94.61  86.84     52    3.302    16   43.246   46.549
0.00   0.00  35.71  23.79  94.61  86.84     52    3.302    16   43.246   46.549
0.00   0.00  35.71  23.79  94.61  86.84     52    3.302    16   43.246   46.549
0.00   0.00  35.71  23.79  94.61  86.84     52    3.302    16   43.246   46.549
0.00   0.00  35.71  23.79  94.61  86.84     52    3.302    16   43.246   46.549
S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
0.00   0.00  35.71  23.79  94.61  86.84     52    3.302    16   43.246   46.549
0.00   0.00  35.71  23.79  94.61  86.84     52    3.302    16   43.246   46.549
0.00   0.00  35.71  23.79  94.61  86.84     52    3.302    16   43.246   46.549
0.00   0.00  35.71  23.79  94.61  86.84     52    3.302    16   43.246   46.549
0.00   0.00  35.71  23.79  94.61  86.84     52    3.302    16   43.246   46.549
```

위 명령어는 -gcutil 옵션을 주고, 5개씩 출력할 때마다 헤더를 출력하고, 1초마다 출력하되 10개의 결과를 보여준다는 뜻이다.

## Heap Dump 분석의 필요성

Spring으로 개발한 웹 애플리케이션을 운영하다 보면, 위에서 언급한 메모리 누수도 발생할 수 있지만 많은 트래픽이 몰리거나 구현 상의 버그로 인해 Heap의 사용량이 순간적으로 크게 증가할 수 있다. 이 경우 GC가 과도하게 일어나면서 애플리케이션의 성능을 저해할 수 있고, 심한 경우에는 OOM(Out Of Memory) 에러가 발생하여 애플리케이션이 다운될 수 있다.

해당 에러가 발생하면 코드 상으로 어떠한 객체가 원인인지 찾아내기 상당히 어렵다. 그래서 OOM이 발생한 시점 혹은 그 근방에 시점에 대해 Heap Dump를 떠서 분석해야 한다.

### OOM이 발생하는 예제

폭발적으로 Java 객체가 생성되는 상황을 상상해 보자. 우리에게 익숙한 대학교 수강 신청이 하나의 예시가 될 것이다. 이번 예제는 수많은 대학생들이 원하는 강의를 얻기 위해 눈물 겨운 수강 신청을 하는 상황을 재현해 볼 것이다.

```java
@RequiredArgsConstructor
public class Student {

    private final Integer id;

    private final String name;
}

@RestController
public class RegisterController {

    @GetMapping("/register")
    public Integer registerCourse() {
        List<Student> students = new ArrayList<>();
        for (int i = 0; i < 100_000_000; i++) {
            students.add(new Student(i, "student" + i));
        }
        return students.size();
    }
}
```

코드는 매우 간단하다. Student 객체를 만들어 주고, 수강 신청 버튼을 클릭한 학생이 1억 명이라고 가정해 보겠다. 이 상황에서 /register api를 호출한다면 어떻게 될까?

```java
java.lang.OutOfMemoryError: Java heap space
	at java.base/jdk.internal.misc.Unsafe.allocateUninitializedArray0(Unsafe.java:1278) ~[na:na]
	at java.base/jdk.internal.misc.Unsafe.allocateUninitializedArray(Unsafe.java:1271) ~[na:na]
	at java.base/java.lang.invoke.StringConcatFactory$MethodHandleInlineCopyStrategy.newArray(StringConcatFactory.java:1633) ~[na:na]
	at java.base/java.lang.invoke.DirectMethodHandle$Holder.invokeStatic(DirectMethodHandle$Holder) ~[na:na]
	at java.base/java.lang.invoke.LambdaForm$MH/0x00000008000eac40.invoke(LambdaForm$MH) ~[na:na]
	at java.base/java.lang.invoke.Invokers$Holder.linkToTargetMethod(Invokers$Holder) ~[na:na]
	at com.example.heapdump.controller.RegisterController.registerCourse(RegisterController.java:17) ~[classes/:na]
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:na]
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
	at java.base/java.lang.reflect.Method.invoke(Method.java:566) ~[na:na]
	... 중략
```

예상할 수 있듯이, OOM 에러가 발생하여 더이상 Java Heap에 객체를 할당할 수 없다는 메시지를 확인할 수 있다.

## Heap Dump 뜨는 방법

Heap Dump는 크게 OOM 에러 혹은 메모리 누수 문제를 분석하기 위해 사용된다. 먼저, VisualVM을 사용하여 Heap Dump를 떠 보자.

### Visual VM

수강신청 API 요청을 보낸 후 우측 상단에 있는 Heap Dump 버튼을 클릭하면 된다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F46eef020-6a0e-41de-9e20-9073ba9538b6%2FUntitled.png?table=block&id=202c3fd4-f4d9-47d5-90df-4fbac73fbe0e&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

가볍게 Heap 메모리 정보를 분석할 수 있다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F8692cfaf-fb18-4559-bb78-95ae4a3800ad%2FUntitled.png?table=block&id=8721bba4-a89a-4cf9-8db9-241ef7d316a8&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

하지만 VisualVM은 아무래도 모니터링 위주의 툴이라 그런지 Heap Dump를 분석하기 적합하지 않은 부분이 있고, 갑작스럽게 메모리 사용량이 매우 늘었을 경우 Heap Dump를 정상적으로 뜰 수가 없다. 그래서 Heap Dump는 주로 jmap 명령어를 이용하여 뜬다.

### jmap

```java
$ jps
26532 HeapDumpApplication
... 중략
```

jstat과 마찬가지로 jps를 통해 얻어 온 PID를 알아야 jmap 명령어를 사용하여 Heap Dump를 뜰 수 있다. jamp 명령어는 테스트해 본 결과 갑작스럽게 메모리 사용량이 크게 증가해도 성공적으로 Dump가 가능했다.

```java
$ jmap -dump:format=b,file=heapdump.hprof 26532
```

`heapdump` 는 파일의 이름으로 자유롭게 지정하면 된다. GC를 최대한 괴롭힌 다음에 위 명령어를 실행해서 Dump 파일을 얻어 내자.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/88985c66-e7d2-46ed-a4ae-5f6569658dbe/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220204%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220204T075226Z&X-Amz-Expires=86400&X-Amz-Signature=4d39e694a1d7e1141a46d271a32ac8f6827a6535e4fd1c068669093600d7dfaa&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

heapdump.hrof 파일이 성공적으로 생성된 것을 확인할 수 있다.

### OOM 발생 시 자동 Heap Dump 생성

VM Options에 두 가지 명령어를 추가하면 OOM이 발생했을 때 자동으로 Heap Dump를 얻을 수 있다.

```java
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=c:/dump/heapdump.hprof
```

참고로 2번째 줄 명령어는 생략해도 되는데, 생략하면 애플리케이션을 실행한 루트 디렉토리에 Dump 파일이 생성된다. 아래는 OOM이 발생하여 Dump 파일을 만들 때 발생하는 로그이다.

```java
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid33720.hprof ...
Heap dump file created [12806244846 bytes in 83.737 secs]
```

### Heap Dump 분석하기

생성된 Heap Dump를 [Eclipse Memory Analyzer(MAT)](https://www.eclipse.org/mat/downloads.php) 툴을 이용하여 분석할 수 있다. 해당 링크에서 다운 받아서 압축 해제를 한 후 MAT exe 파일을 실행하자.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F3eaf15ce-ab99-43d4-8f2f-381715311997%2FUntitled.png?table=block&id=399c8ade-b31d-44d3-ad8b-6810a8a4df69&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

위 사진에서 Open a Heap Dump 버튼을 클릭하여 진행하면 된다. jmap으로 얻은 Heap Dump 파일 혹은 OOM이 발생하여 자동 생성된 Heap Dump 파일을 지정한다. 필자는 OOM이 발생하여 자동 생성된 Heap Dump를 분석하려고 한다.

이때 주의할 점이 MAT 툴이 할당할 수 있는 메모리의 크기가 분석하려는 Heap Dump 크기보다 커야 한다. 따라서 위에서 압축 해제한 루트 디렉토리에서 `MemoryAnalyzer.ini` 파일을 실행하여 Xmx 크기를 적절하게 조절한다. 아래와 같이 세팅하면 된다.

```java
-vmargs
-Xmx15G
-XX:-UseGCOverheadLimit
```

성공적으로 Heap Dump 분석이 끝나면 다음과 같이 원 그래프 및 다양한 기능이 나타난다. 

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Face805ce-7bcf-408b-8687-0e10056b91d3%2FUntitled.png?table=block&id=efefbd8e-0483-4db8-a481-a966c55e9249&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

하단에 있는 Dominator Tree를 클릭하면 Heap Dump를 뜰 당시에 만들어진 Java 객체를 한 눈에 확인할 수 있다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F5a1aac0e-7831-470d-bdc1-4f4ad972445d%2FUntitled.png?table=block&id=4d341b30-8ee6-412c-b99b-d7b9a482fcdd&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

여기서 사용량이 매우 높은 객체를 유심히 분석해 보니, Student 객체가 무려 8400만 개나 만들어진 것을 알 수 있다. 이제 우리는 수강 신청 애플리케이션에서 OOM이 발생한 이유를 단 기간에 빠르게 생성된 Student 객체가 원인임을 추측이 아닌 객관적인 자료로 판단할 수 있다.

## 출처

- [https://jupiny.com/2019/07/15/java-heap-dump-analysis/](https://jupiny.com/2019/07/15/java-heap-dump-analysis/)
- [https://waspro.tistory.com/145](https://waspro.tistory.com/145)
- [https://d2.naver.com/helloworld/6043](https://d2.naver.com/helloworld/6043)
- [https://stackoverflow.com/questions/9819905/eclipse-memory-analyser-but-always-shows-an-internal-error-occurred](https://stackoverflow.com/questions/9819905/eclipse-memory-analyser-but-always-shows-an-internal-error-occurred)
- [https://stackoverflow.com/questions/7254017/tool-for-analyzing-large-java-heap-dumps](https://stackoverflow.com/questions/7254017/tool-for-analyzing-large-java-heap-dumps)
- [https://5dol.tistory.com/182](https://5dol.tistory.com/182)

## 예상 면접 질문 및 답변

### Heap을 왜 모니터링 해야 하는가?

Spring 개발한 웹 애플리케이션을 운영하다 보면, 어떠한 이유로 사용하지 않는 메모리가 쌓이면서 메모리 누수 현상이 발생할 수 있다. 이것은 모니터링을 하지 않는 이상 정확한 이유를 알기 매우 어려우며, 메모리 누수를 방치하면 어느 순간 OOM(Out Of Memory) 에러가 발생할 수 있다. 따라서 주기적으로 Heap 영역을 모니터링하고, 의심되는 상황이 있다면 Heap Dump를 떠서 어떤 객체가 원인이 분석해야 한다.

### Heap Dump는 언제 사용하는가?

Heap Dump는 메모리 누수 현상 혹은 OOM 에러가 발생했을 때, 어떠한 객체가 원인인지 분석하기 위해 사용된다.

### Heap Dump를 분석하여 OOM을 해결한 사례가 있는가?

개인의 경험을 이야기할 것. 필자는 엑셀 라이브러리인 XSSF 라이브러리를 사용하다가 OOM 에러가 발생하였음.
