## GC의 개념

GC는 메모리 관리 기법 중 하나로, 동적으로 할당했던 메모리 영역 중 필요 없게 된 영역을 해제하는 기능이다. 여기서 동적으로 할당했던 메모리 영역은 프로그램 런타임에 사용되는 Heap 영역 메모리를 뜻하고, 필요 없게 된 영역은 어떤 변수도 가리키지 않게 된 영역을 뜻한다.

```java
Person person = new Person(); 
person.setName("Jayon"); 
person = null;

person = new Person(); 
person.setName("Jerry");

```

person 변수는 기존의 Jayon 이름이 붙은 Person 객체가 존재하는 메모리 영역을 가리키고 있었으나, 추후 Jerry 이름이 붙은 Person 객체가 존재하는 메모리 영역을 가리키게 된다. 이때 Jayon 이름이 붙었던 Person 객체의 메모리 영역은 그 어떤 변수도 가리키고 있지 않다. 따라서 해당 영역을 어느 시점에서 자동으로 해제하는 것이 바로 GC의 역할이다.

C와 C++에서 Heap 영역의 메모리를 사용하기 위해서는 코드 레벨에서 직접 동적 메모리 영역을 할당 받고 해제하는 과정을 작성해야 했다. 하지만 이를 수동으로 직접 관리하는 것은 번거롭고 실수를 하기도 쉽다. 가령 메모리 영역을 할당 받고 해제하지 않으면 메모리 누수가 발생할 수 있고, 이미 해제한 메모리 영역을 또 해제하면 에러가 발생할 수 있다.반면, Java에서는 동적 메모리 영역을 GC가 알아서 수행해주므로 개발자 입장에서는 편하게 개발에 집중할 수 있다. 

참고로 GC를 의도적으로 `System.gc()` 를 사용하여 호출할 수 있지만, 오버헤드가 굉장히 크므로 GC에게 메모리 해제를 믿고 맡기는 것을 추천한다.

## GC의 장단점

### 장점

- 메모리를 수동으로 관리하던 것에서 비롯된 에러를 예방할 수 있다.
    - 개발자의 실수로 인한 메모리 누수,
    - 해제된 메모리를 또 해제하는 이중 해제
    - 해제된 메모리에 접근

### 단점

- GC의 메모리 해제 타이밍을 개발자가 정확히 알기 어렵다.
- 어떠한 메모리 영역이 해제의 대상이 될 지 검사하고, 실제로 해제하는 일이 모두 오버헤드다.

## GC에서 사용하는 알고리즘

### Reference Counting

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fd8fa2cdd-789b-41cc-b024-294e17e10d3b%2FUntitled.png?table=block&id=4c07c0af-289b-44a8-93a1-493ce157c859&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

그림 속 Root Space는 스택 변수, 전역 변수 등 heap 영역 참조를 가리키고 있는 공간이다.

Reference Counting은 힙 영역의 객체들이 각각 reference count라는 숫자를 가지고 있다고 생각하면 좋은데, 여기서 reference count는 몇 가지 방법으로 해당 객체에 접근할 수 있는 지를 뜻한다. 만약 reference count가 0에 다다르면 해당 객체에 접근할 수 있는 방법이 없다는 뜻이므로 메모리 해제의 대상이 되는 것이다. **Swift 언어가 Reference Count 방식**으로 메모리 관리를 한다.

하지만 Reference Counting은 순환 참조 문제가 발생할 수 있다. 그림 속 Root Space에서 모든 Heap Space의 참조를 끊는다고 가정하자. 그러면 노란색 고리 안의 객체는 서로가 서로를 참조하고 있기 때문에 reference count가 1로 유지된다. 결국 사용하지 않는 메모리 영역이 해제되지 못하고 메모리 누수가 발생하는 것이다.

### Mark And Sweep

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F0e985009-edd3-4418-992a-770520fa4974%2FUntitled.png?table=block&id=8ebae1e5-e1a7-4ccb-acc1-ed57eb69f924&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Mark And Sweep 알고리즘은 Reference Counting의 순환 참조 문제를 해결할 수 있다. 어떻게 해당 문제를 극복하는지 설명하기 전에 개념부터 살펴 보자.

Mark And Sweep 알고리즘은 Root Space부터 해당 객체에 접근 가능한지, 아닌지를 메모리 해제의 기준으로 삼는다. Root Space부터 그래프 순회를 통해 연결된 객체를 찾아내고(Mark) 연결이 끊어진 객체는 지운다.(Sweep) Root Space부터 연결된 객체는 Reachable, 연결되지 않은 객체는 Unreachable라고 부른다.

그림에서는 Sweep 이후에 분산되어 있던 던 메모리가 예쁘게 정리된 것을 볼 수 있는데, 이것은 메모리 파편화를 방지하는 Compaction 과정이다. 다만, Mark And Sweep 알고리즘에서 Compaction이 필수는 아니다.

이렇게 Mark And Sweep 방식을 사용하면, Root Space부터 연결이 끊긴 순환 참조되는 객체들도 지울 수 있다. **Java와 JavaScript가 Mark And Sweep 방식**으로 메모리 관리를 한다.

하지만 Mark And Sweep 방식도 단점이 있다. 객체의 reference count가 0이 되면 지워버리는 Reference Counting 방식과 달리, Mark And Sweep은 **의도적으로 특정 순간에 GC를 실행해야 한다.** 즉 어느 순간에는 실행 중인 애플리케이션이 GC에게 컴퓨터 리소스를 내어 주어야 한다는 뜻이다. 따라서 **애플리케이션의 사용성을 유지하면서 효율적으로 GC를 실행**하는 것이 중요하다.

**Key Point**

- 의도적으로 GC를 실행해야 한다.
- 애플리케이션과 GC 실행이 병행된다.

Mark And Sweep 알고리즘에서 위 두 가지 특징을 꼭 기억하자.

## GC의 동작 과정

### Root Space

위에서 Root Space는 Heap 영역 메모리에 대해 참조하고 있는 영역이라고 하였는데, 구체적으로 무엇이 있는지 살펴보려고 한다. JVM Memory 영역 중에서 Root Space는 다음 3가지가 해당된다.

- Stack의 로컬 변수
- Method Area의 Static 변수
- Native Method Stack의 JNI 참조

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F295fa47f-7cd0-4adc-8d01-7095b82009f7%2FUntitled.png?table=block&id=5298a974-f2c2-48c6-9919-3328c32c0152&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

노란색 박스 안의 영역이 바로 Root Space이다.

### GC 실행의 타이밍

Mark And Sweep 방식의 첫 번째 특징은 의도적으로 GC를 실행한다는 것이었다. 그렇다면, GC가 실행하는 순간은 언제일까? 이를 알기 위해서는 Heap 영역을 조금 더 디테일하게 살펴 보아야 한다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fe77a63a6-c2bb-432a-828c-232604a9bf63%2FUntitled.png?table=block&id=ef013174-36cc-48fc-a0ee-68dd017f1ad4&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

JVM의 Heap은 크게 Young Generation과 Old Generation으로 나뉜다. 전자에서 발생하는 GC는 minor gc, 후자에서 발하는 GC는 major gc(full gc)라고 부른다.

Young Generation은 또 다시 Eden, Survival 0, Survival 1 영역으로 나뉜다. Eden은 새롭게 생성된 객체들이 할당되는 영역이고, Survival 영역은 minor gc에서 살아남은 객체들이 존재하는 영역이다. 이때 Survival 영역에서 Survival  0과 Survival  1 중 하나는 꼭 비어 있어야 한다는 규칙이 있다.

minor gc의 실행 타이밍은 바로 Eden 영역이 꽉 찼을 때이다. 아래 그림을 살펴 보자.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F19b9587c-c8b1-42dd-a74f-c0d1bd29ac4e%2FUntitled.png?table=block&id=d1081cc3-116e-41f9-b0c6-166699423953&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

그림에서 회색 네모는 메모리에 할당된 객체를 생각하면 된다. minor gc가 발생하고 난 뒤 Reachable이라 판단된 객체는 Survival 0 영역으로 옮겨진다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F51f0e07c-ad1e-4add-b0d0-2854ea675df5%2FUntitled.png?table=block&id=23f886de-39d2-4023-b89d-ef5360bc5911&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

이때 살아 남은 객체들의 숫자들이 0에서 1로 변한 것을 알 수 있는데, 이는 age bit를 뜻한다. minor gc에서 살아남은 객체는 age bit가 1씩 증가하는데, 이것의 용도는 아래에서 자세히 설명할 것이다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fc9b6e6f1-97c0-4a59-8396-3c3dff3f8bb7%2FUntitled.png?table=block&id=6b1888cf-7603-4e28-9c68-2c171befdbe3&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

또 다시 Eden 영역이 꽉찼다. 

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F77a1439b-7a0a-4d6b-9c17-812ed453b6a7%2FUntitled.png?table=block&id=c2e75fb7-f180-41fe-b8cb-e22f090fb4ac&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

그러면 minor gc가 발생하여 Reachable이라 판단된 객체들은 Survival 1 영역으로 이동한다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2ac86894-36fe-4b3d-9af4-42ed4174af3e%2FUntitled.png?table=block&id=396f0432-01b5-4cdb-b803-ce5aecab2a96&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

이후 또 Eden 영역이 꽉찼다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F982b5ff2-f002-4e60-82ff-dd6a2e826081%2FUntitled.png?table=block&id=5c6dcab6-9cd2-4bb4-9d28-0caca4e6676b&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

그러면 minor gc가 발생하여 Reachable이라 판단된 객체들은 Survival 0 영역으로 이동한다. 어느덧 Survival 0 영역으로 넘어 온 객체 중 오래 살아 남아 age bit가 3이 된 객체가 보인다.

JVM GC에서는 일정 수준의 age bit를 넘어가면 오래도록 참조될 객체라고 판단하고, 해당 객체를 Old Generation에 넘겨 주는데, 이를 Promotion이라 부른다.

Java 8에서는 Parallel GC 방식 기준으로 age bit가 15가 되면 Promotion이 진행된다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fd1c7c90e-220c-423d-8880-d54648da56c7%2FUntitled.png?table=block&id=ed8c6652-87ef-4b6a-a956-69cf8b5bc2f7&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

이번 예제에서는 age bit가 3이 될 경우를 Promotion의 기준으로 잡았다. 그래서 Survival 0 영역의 age bit가 3인 객체가 Old Generation으로 Promotion되었다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F297bf91f-f811-427b-a848-12e7e53f2a34%2FUntitled.png?table=block&id=e2a125ed-b4a3-4cd7-96f5-933de64a433b&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

시간이 아주 많이 지나면 언젠간 old Generation도 다 채워지는 날이 올텐데, 이때 major gc가 발생하면서 Mark And Sweep 방식을 통해 필요 없는 메모리를 비우는데, minor gc에 비해 시간이 오래 걸린다.

Heap 영역을 굳이 Young Generation과 old Generation으로 나눈 이유는 무엇일까?

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F130dfe2a-4791-42e6-878b-be9a112275ac%2FUntitled.png?table=block&id=2179d4c1-e25a-4d2d-906d-e5120cd1c72b&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

그것은 바로 GC 설계자들이 애플리케이션을 분석해 보니 대부분의 객체가 수명이 짧다는 것을 깨달았기 때문이다. GC도 결국 비용이 드는 작업인데, 메모리의 전체 부분이 아닌 특정 부분만을 탐색하여 해제해야 효율적이다. 그래서 어차피 대다수의 객체가 금방 사라지니 Young Generation 안에서 최대한 메모리를 해제하도록 설계한 것이다.

### GC의 실행 방식

Mark And Sweep 방식의 두 번째 특징은 애플리케이션과 GC 실행이 병행된다는 것이었다. 즉, JVM에서는 애플리케이션과 GC를 병행하여 실행할 수 있는 여러 옵션들을 제공한다는 사실을 알 수 있다. 

GC가 어떤 방식으로 애플리케이션 실행과 병행되는지 살펴 보기 전에, Stop The World 개념을 알아야 한다. Stop The World란 GC를 실행하기 위해 JVM이 애플리케이션 실행을 멈추는 것을 말한다.

**Serial GC**

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fee944481-c360-4639-8eeb-271ba3a2ac8e%2FUntitled.png?table=block&id=739c2288-b44d-4af3-ac20-da319e33999f&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Serial GC는 하나의 스레드로 GC를 실행하는 방식인데, 하나의 스레드로 GC를 실행하다 보니 Stop The World 시간이 긴 것을 알 수 있다. 싱글 스레드 환경 및 Heap 영역이 매우 작을 때 사용되는 방식이다. 참고로 Mark And Sweep 이후 메모리 파편화를 막는 Compaction 과정도 진행된다.

**Parallel GC**

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F7b9d7734-0139-4eeb-9096-9a8b853bdc11%2FUntitled.png?table=block&id=fea9b2f8-b7a5-4903-92df-8dfd3dd02338&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Parallel GC의 기본적인 처리 과정은 Serial GC와 동일하다. 하지만 Parallel GC는 여러 개의 스레드로 GC를 실행하므로 앞선 Serial GC보다 Stop The World 시간이 짧아진 것을 알 수 있다. 멀티 코어 환경에서 애플리케이션 처리 속도를 향상시키기 위해 사용되며, **Java 8**에서 기본으로 쓰이는 GC 방식이다. Parallel GC가 GC의 오버헤드를 상딩히 줄여주었지만, 애플리케이션이 멈추는 것은 피할 수 없으므로 다른 알고리즘이 더 등장하게 되었다.

**CMS GC**

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F8b203a00-163a-4deb-a3dc-537ea613c38a%2FUntitled.png?table=block&id=b9764bfc-93c2-45cb-8fc6-c30d339eb8f5&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

CMS GC에서 CMS는 Concurrent-Mark-Sweep의 줄임말로 Stop The World 시간을 최소화하기 위해 고안되었다. 대부분의 가비지 수집 작업을 애플리케이션 스레드와 동시에 수행하여, Stop The World 시간을 최소화하고 있다. 하지만 메모리와 CPU를 많이 사용하고, Mark And Sweep 과정 이후 메모리 파편화를 해결하는 Compaction이 기본적으로 제공되지 않는다는 단점이 있다. 이 때문에 시스템이 장기적으로 운영되다가 조각난 메모리들이 많아 Compaction 단계를 수동으로 수행하면 오히려 Stop The World 시간이 길어지는 것을 알 수 있다.

CMS GC는 Java 9 버전부터 deprecated되었고 Java 14 버전부터는 사용이 중단되었다.

- Initial Mark: 클래스 로더에서 가장 가까운 객체 중 살아 있는 객체만 찾는다.
- Concurrent Mark: 위에서 살아 있다고 확인한 객체에서 참조되어 있는 객체를 확인한다.
- Remark: 위 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다.
- Concurrent Sweep: 쓰레기를 정리한다.

**G1 GC**

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Ff435c5c9-f5a3-4306-9fc3-0ce127778ed4%2FUntitled.png?table=block&id=ca35c7d9-cfaa-430e-a225-5baf3964c3b5&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

G1은 Garbage First의 줄임말로 Heap 영역을 위에서 설명한 방식과 다르게 사용한다. Heap을 일정 크기의 Region으로 잘게 나누어 어떤 영역은 Young Generation, 어떤 영역은 Old Generation으로 사용한다. 런타임에 따라 G1 GC가 필요에 따라 영역 별 Region 개수를 튜닝함으로써 Stop The World를 최소화할 수 있다. Java 9이상부터는 G1 GC를 기본 GC 실행 방식으로 사용한다.

자세한 내용은 다음 시간에 설명하려고 한다.

## 출처

- [https://papimon.tistory.com/93](https://papimon.tistory.com/93)
- [https://joel-dev.site/94](https://joel-dev.site/94)
- [https://mangkyu.tistory.com/118](https://mangkyu.tistory.com/118)
- [https://mangkyu.tistory.com/119](https://mangkyu.tistory.com/119)
- [https://huisam.tistory.com/entry/jvmgc](https://huisam.tistory.com/entry/jvmgc)

## 예상 면접 질문 및 답변

### GC란?

GC는 메모리 관리 기법 중 하나로, 동적으로 할당했던 메모리 영역 중 필요 없게 된 영역을 해제하는 기능이다. 여기서 동적으로 할당했던 메모리 영역은 프로그램 런타임에 사용되는 Heap 영역 메모리를 뜻하고, 필요 없게 된 영역은 어떤 변수도 가리키지 않게 된 영역을 뜻한다.

### GC의 장단점은?

### 장점

- 메모리를 수동으로 관리하던 것에서 비롯된 에러를 예방할 수 있다.
    - 개발자의 실수로 인한 메모리 누수,
    - 해제된 메모리를 또 해제하는 이중 해제
    - 해제된 메모리에 접근

### 단점

- GC의 메모리 해제 타이밍을 개발자가 정확히 알기 어렵다.
- 어떠한 메모리 영역이 해제의 대상이 될 지 검사하고, 실제로 해제하는 일이 모두 오버헤드다.

### GC에서 사용하는 알고리즘을 설명하라.

- Reference Counting
    - 힙 영역의 객체들이 각각 reference count를 가지고 있고, 이 count가 0이 되면 해당 객체를 해제한다. reference count는 몇 가지 방법으로 해당 객체에 접근할 수 있는 지 뜻한다.
- Mark And Sweep
    - 알고리즘은 Root Space부터 해당 객체에 접근 가능한지, 아닌지를 메모리 해제의 기준으로 삼는다. Root Space부터 그래프 순회를 통해 연결된 객체를 찾아내고(Mark) 연결이 끊어진 객체는 지운다.
    - Java에서 사용하는 GC 알고리즘이다.

### GC는 언제 실행되는가?

JVM의 Heap은 Young Generation, Old Generation으로 나뉘고, Young Generation은 다시 Eden, Servival 0, Servival 1 영역으로 나뉜다. Eden 영역이 꽉 차면 minor gc가 실행되어 Mark And Sweep 알고리즘을 통해 참조가 되고 있는 객체만 Servival 0 또는 Servival 1 영역으로 옮긴다.

만약 여러 번 minor gc를 실행하고도 참조된 객체가 있다면 이 객체를 Servival 영역에서 Old Generation 영역으로 옮긴다. 시간이 지나서 Old Generation 영역이 꽉 차면 major gc를 실행하여 Mark And Sweep 알고리즘을 통해 필요 없는 메모리를 비운다.

### 왜 Heap 영역은 Young Generation과 Old Generation으로 나누는가?

통계적으로 대부분의 객체가 수명이 짧으므로 특정 부분만 탐색함으로써 GC의 성능을 높이기 위해 나누었다.

### GC의 실행 방식을 설명하라.

- Serial GC
    - Serial GC는 하나의 스레드로 GC를 실행하는 방식인데, 하나의 스레드로 GC를 실행하다 보니 Stop The World 시간이 길다.
    - GC 실행 이후 메모리 파편화를 막는 Compaction 과정을 수행한다.
- Parallel GC
    - Serial GC를 멀티 스레딩으로 수행한다.
- CMS GC
    - 대부분의 가비지 수집 작업을 애플리케이션 스레드와 동시에 수행하여, Stop The World 시간을 최소화하고 있다. 하지만 메모리와 CPU를 많이 사용하고, Mark And Sweep 과정 이후 메모리 파편화를 해결하는 Compaction이 기본적으로 제공되지 않는다는 단점이 있다.

G1 GC는 다음 포스팅에서 자세히 설명할 예정.
