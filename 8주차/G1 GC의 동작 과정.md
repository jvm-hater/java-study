# G1 GC 동작 원리

## GC의 종류

이전 시간에 배웠던 GC의 종류를 간략하게 짚고 넘아가자.

- Serial GC
    - 하나의 CPU로 Young 영역과 Old 영역을 연속적으로 처리하는 방식이다. GC가 수행될 때 STW(Stop The World)가 발생한다.
- Parallel ****GC
    - Parallel GC의 목표는 다른 CPU가 GC의 진행시간 동안 대기 상태로 남아 있는 것을 최소화 하는 것이다. Serial GC의 Young 영역에서 진행하는 방식을 병렬로 처리하여 부하를 줄인다.
- Parallel Old GC
    - Parallel GC에서 사용하던 Mark Sweep Compation 대신 개선 버전인 Mark Summary Compaction 알고리즘을 사용한다.
    - Parallel GC는 Minor GC에 대해서 멀티 스레딩으로 수행하는 것에 반해, Parallel Old GC는 Major GC에 대해서도 멀티 스레딩으로 수행한다.
- Concurrent Mark-Sweep(CMS) GC
    - Application의 Thread와 GC Thread가 동시에 실행되어 STW를 최소화 하는 GC이다. Parallel GC와 가장 큰 차이점은 Compaction 작업 유무로 구분될 수 있다. Compaction은 메모리 공간에서 사용하지 않는 빈 공간이 없도록 옮겨서 메모리 분산을 제거하는 작업을 의미한다.
    - CMS GC는 Compacton을 지원하지 않는다.
- G1(Garbage First) GC

## G1 GC란?

G1 GC는 CMS GC를 대체하기 위해 새롭게 등장하였으며, 대용량의 메모리가 있는 멀티 프로세서 시스템을 위해 제작되었다. 빠른 처리 속도를 지원하면서 STW를 최소화한다. CMS GC보다 효율적으로 동시에 Application과 GC를 진행할 수 있고, 메모리 Compaction 과정까지 지원하고 있다. Java 9 버전부터 기본 GC 방식으로 채택되었다.

### 왜 이름이 G1일까?

G1은 Garbage First의 약어로 Garbage만 있는 Region을 먼저 회수한다고 해서 붙여진 이름이다. 빈 공간 확보를 더 빨리 한다는 것은 조기 승격이나 급격히 할당률이 늘어나는 것을 방지하여 Old Generation을 비교적 한가하게 만들 수 있다.

### 장점과 단점

- 장점
    - 별도의 STW 없이도 여유 메모리 공간을 압축하는 기능을 제공한다. 또한, 전체 Old Generation 혹은 Young Generation 통째로 Compaction을 할 필요 없고, 해당 Generation의 일부분 Region에 대해서만 Compaction을 하면 된다.
    - Heap 크기가 클수록 잘 동작한다.
    - CMS의 비해 개선된 알고리즘을 사용하고, 처리 속도가 더 빠르다.
    - Garbage로 가득찬 영역을 빠르게 회수하여 빈 공간을 확보하므로 GC 빈도가 줄어든다.
- 단점
    - 공간 부족 상태를 조심해야 한다. (Minor GC, Major GC 수행하고 나서도 여유 공간이 부족한 경우)
        - 이때는 Full GC가 발생하는데, 이 GC는 Single Thread로 동작한다.
        - Full GC는 heap 전반적으로 GC가 발생하는 것을 뜻한다.
    - 작은 Heap 공간을 가지는 Application에서는 제 성능을 발휘하지 못하고 Full GC가 발생한다.
    - Humonogous 영역은 제대로 최적화되지 않으므로 해당 영역이 많으면 성능이 떨어진다.

## G1 GC의 Heap 구조

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F575a0e57-411f-4f06-aa82-13a1c2d770b4%2FUntitled.png?table=block&id=bf513588-1c0f-4c13-9d3d-4a1a151df414&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

G1 GC는 기존 힙 구조와 완전히 다른 양상을 띈다. 전통적인 힙 구조는 Young, Old 영역을 명확하게 구분하였지만, G1 GC는 개념적으로 그들이 존재하나 일정 크기의 논리적 단위인 region으로 구분하고 있다.

- Humonogous: Region 크기의 50%를 초과하는 큰 객체를 저장하기 위한 공간
- Available/Unused: 아직 사용되지 않은 Region

## G1 GC의 Cycle

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F29ba4b49-599a-4282-a087-8b7513917e38%2FUntitled.png?table=block&id=9fb294e9-689f-47fa-a8ab-ab6bfff8be3b&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

G1 GC는 위 그림처럼 Young-only Phase와 Space Reclamation Phase를 반복한다. 사이클 중 모든 원은 STW가 발생한 것을 나타낸 것이고, 원의 크기에 따라 STW 소요 시간이 달라진다고 생각하면 된다.

파란 원은 Minor GC(= Young GC, Evacuation Pause)가 진행함에 따라 STW가 발생한 것이고, 주황 원은 Major GC(= Old GC, Concurrent Cycle)이 진행하면서 객체를 마킹하기 위해 STW가 발생한 것이고, 빨간 원은 Mixed GC를 진행함에 따라 STW가 발생한 것이다.

### Young Only Phase

Young Only (Garbage Collection) Phase는 Minor GC만 수행하다가 XX:InitiatingHeapOccupancyPercent(Old Generation 비율)에 지정된 값을 초과하는 순간 Major GC가 수행된다. 그림에서 `Old gen occupancy exceeds threshold` 부분을 말하는 것이다.

Major GC의 첫 단계는 Initial Mark이며 Minor GC와 동시에 수행되며 둘 다 STW를 수반하므로 다른 파란 원보다 크기가 크다. 그 이후에 애플리케이션 스레드, Minor GC, Concurrent Mark가 동시에 수행되는데 Remark가 수행되는 순간 다른 작업은 멈추게 된다. 그래서 Remark에 해당하는 주황색이 원이 큰 것을 알 수 있다. 그 이후에 자잘하게 Minor GC가 수행되다가 Major GC의 Cleanup이 발생한다.

### Space Reclamation Phase

Young Only Phase가 끝나고 Space Reclamation(공간 회수) Phase가 시작된다. 해당 Phase에서는 Mixed GC가 수행되는데 Mark 단계가 없어서 STW 빈도가 Young Only Phase에 비해 줄어든 것을 알 수 있다. Space Reclamation Phase가 끝나면 다시 Young Only Phase로 돌아가서 Minor GC를 수행한다.

## G1 GC의 동작 과정

위에서는 가볍게 G1 GC의 Cycle을 살펴 보았다. 중간 중간 언급했던 Minor GC, Major GC, Mixed GC를 디테일하게 알아 보자.

### Minor GC (Young Generation)

Minor GC 기존 GC와 원리가 비슷하나, 멀티 스레드에서 병렬로 작동한다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Feaa5e039-1af0-4daa-8adc-7fd1a6722ea9%2FUntitled.png?table=block&id=5b59fd58-d7c3-4726-befb-4acc8ff2baac&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

연속되지 않은 메모리 공간에 Young Generation이 Region 단위로 메모리에 할당되어 있다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fa2b8fbea-2956-44f3-be18-0b6b04c807a7%2FUntitled.png?table=block&id=f03a0825-6d35-4ce9-96eb-f5f7593f0080&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Young Generation에 있는 유효 객체(live object)를 Survivor Region이나 Old Generation으로 evacuation(copy or move)한다. 이 단계에서 STW가 발생하고, Eden Region과 Survivor Region의 크기는 다음 Minor GC를 위해 다시 계산된다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F283508fa-fbcd-4045-bcec-616b7a9ee219%2FUntitled.png?table=block&id=36081068-1823-4718-9bb0-6add35b3ca61&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Minor GC를 모두 마친 후의 모습이다. 청록색 영역은 Eden 영역에서 Survivor 영역으로 이동하거나 Survivor 영역에서 Survivor 영역으로 이동한 것을 뜻한다. Survivor 영역에서 Survivor 영역으로 이동했다는 뜻은 Survivor 0과 Survivor 1 사이의 이동을 생각하면 된다.

### Major GC(Old GC)

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Feb848c9f-58ea-4aaf-88fd-4506848f3d79%2FUntitled.png?table=block&id=2ca18d0e-43f6-47ad-84c2-bdb9784704b7&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Initial Mark 단계는 Old Region에 존재하는 객체들이 참조하는 Survivor Region이 있는지 파악해서 Survivor Region에 마킹하는 단계이다. Survivor Region에 의존적이기 때문에 Survivor Region은 깔끔한 상태여야 하고, Survivor Region이 깔끔하려면 Minor GC가 전부 끝난 상태여야 한다. 따라서 Initial Mark는 Minor GC에 의존적이며, STW를 발생한다.

**Root Region** **Scan**

Initial Mark 단계 다음으로 Root Region Scan 단계가 수행된다. Initial Mark 단계에서 마킹된 Survivor Region에서 Old Region에 대해 참조하고 있는 객체를 마킹한다. 멀티 스레드로 동작하며 다음 Minor GC가 발생하기 전에 동작을 완료한다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2cec3047-7df8-41d9-85c4-c60549ffca89%2FUntitled.png?table=block&id=90833afc-0678-42e7-a6d5-83d23b0dcde6&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Concurrent Marking 단계에서는 Old Generation 내에 생존해 있는 모든 객체를 마킹한다. STW가 발생하지 않으므로 애플리케이션 스레드와 동시에 동작하고, Minor GC와 같이 진행되므로 종종 Minor GC에 의해 중단될 수 있다. 위 사진에서 X 표시한 Region은 모든 객체가 Garbage 상태인 영역이다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F9c5eb0cd-9fd2-409a-b697-529e9c71ce63%2FUntitled.png?table=block&id=bd3f1c60-5a91-4fb0-9303-b7be8cfe442c&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Remark 단계는 Concurrent Mark에서 X 표시한 영역을 바로 회수하며, STW가 발생한다. 또한, Concurrent Mark 단계에서 작업하던 Mark를 이어서 작업하여 완전히 끝내버린다. 이때 SATB(Snapshot-At-The-Beginning) 기법을 사용하기 때문에 CMS GC보다 더 빠르다. 참고로 SATB는 STW 이후 살아있는 객체에만 마킹하는 알고리즘이다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F5a228503-3875-4134-aeb6-c241fd566f41%2FUntitled.png?table=block&id=8ba2c778-57c7-4103-bbdd-eba4c7f04f3a&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Copying/Cleanup 단계에서는 STW가 발생하며, live object의 비율이 낮은 영역 순으로 순차적으로 수거해 나간다.

먼저 해당 영역에서 live object를 다른 영역으로 evacuation(move or copy)한 후, Garbage를 수집한다. G1 GC는 이렇게 Garbage의 수집을 우선(First)해서 계속하여 여유 공간을 신속하게 확보해 둔다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F21fc8398-3265-4632-bc41-55a2ccd56c86%2FUntitled.png?table=block&id=4f15e2aa-664f-41e2-a5ee-63c119d8317e&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Major GC가 끝난 이후 live object가 새로운 Region으로 이동하고 메모리 Compaction이 일어나서 깔끔해진 것을 볼 수 있다.

### Mixed GC

Mixed GC는 Young 영역과 Old 영역의 Garbage를 수집한다. 한 번에 Old 영역의 Garbage를 수집하는 것은 비용이 크므로 Mixed GC는 기본적으로 8회 수행된다.

Mixed GC는 Minor GC에서 수행하는 단계와 동일하지만, 추가로 Old 영역의 Garbage를 수집한다. 즉, Mixed GC는 Minor GC와 Old 영역의 GC를 혼합한 과정이라고 할 수 있다.

## 출처

- [https://perfectacle.github.io/2019/05/11/jvm-gc-advanced/](https://perfectacle.github.io/2019/05/11/jvm-gc-advanced/)
- [https://huisam.tistory.com/entry/jvmgc](https://huisam.tistory.com/entry/jvmgc)
- [https://mirinae312.github.io/develop/2018/06/04/jvm_gc.html](https://mirinae312.github.io/develop/2018/06/04/jvm_gc.html)
- [https://initproc.tistory.com/entry/G1-Garbage-Collection](https://initproc.tistory.com/entry/G1-Garbage-Collection)
- [https://gona.tistory.com/61](https://gona.tistory.com/61)
- [https://thinkground.studio/일반적인-gc-내용과-g1gc-garbage-first-garbage-collector-내용/](https://thinkground.studio/%EC%9D%BC%EB%B0%98%EC%A0%81%EC%9D%B8-gc-%EB%82%B4%EC%9A%A9%EA%B3%BC-g1gc-garbage-first-garbage-collector-%EB%82%B4%EC%9A%A9/)

## 예상 면접 질문 및 답변

### G1 GC란?

G1은 Garbage First의 약어로 Garbage만 있는 Region을 먼저 회수한다고 해서 붙여진 이름이다. 빈 공간 확보를 더 빨리 한다는 것은 조기 승격이나 급격히 할당률이 늘어나는 것을 방지하여 Old Generation을 비교적 한가하게 만들 수 있다.

### G1 GC의 장단점은?

- 장점
    - 별도의 STW 없이도 여유 메모리 공간을 압축하는 기능을 제공한다. 또한, 전체 Old Generation 혹은 Young Generation 통째로 Compaction을 할 필요 없고, 해당 Generation의 일부분 Region에 대해서만 Compaction을 하면 된다.
    - Heap 크기가 클수록 잘 동작한다.
    - CMS의 비해 개선된 알고리즘을 사용하고, 처리 속도가 더 빠르다.
    - Garbage로 가득찬 영역을 빠르게 회수하여 빈 공간을 확보하므로 GC 빈도가 줄어든다.
- 단점
    - 공간 부족 상태를 조심해야 한다. (Minor GC, Major GC 수행하고 나서도 여유 공간이 부족한 경우)
        - 이때는 Full GC가 발생하는데, 이 GC는 Single Thread로 동작한다.
        - Full GC는 heap 전반적으로 GC가 발생하는 것을 뜻한다.
    - 작은 Heap 공간을 가지는 Application에서는 제 성능을 발휘하지 못하고 Full GC가 발생한다.
    - Humonogous 영역은 제대로 최적화되지 않으므로 해당 영역이 많으면 성능이 떨어진다.

### G1 GC의 Heap 구조를 설명하라.

G1 GC는 기존 힙 구조와 완전히 다른 양상을 띈다. 전통적인 힙 구조는 Young, Old 영역을 명확하게 구분하였지만, G1 GC는 개념적으로 그들이 존재하나 일정 크기의 논리적 단위인 region으로 구분하고 있다.

- Humonogous: Region 크기의 50%를 초과하는 큰 객체를 저장하기 위한 공간
- Available/Unused: 아직 사용되지 않은 Region

### G1 GC의 동작 과정을 설명하라.

기본적으로 G1 GC는 Young-Only 단계와 Space Reclamation 단계를 반복하면서 수행하는 Cycle 구조로 진행된다.

Young Only 단계는 Minor GC만 수행하다가 한정된 Old Generation 비율이 넘으면 Major GC가 수행된다. 그리고 Young Only 단계가 끝날 때까지 두 GC가 혼용된다.

Space Reclamation 단계는 Old 영역의 Garbage까지 수집하는 Minor GC 방식의 Mixed GC 방식이 수행된다.

### G1 GC에서 Minor GC는 어떻게 동작하는가?

연속되지 않은 Young Generation에 있는 live object를 Survivor 영역이나 Old Generation으로 이동시킨다. 이 단계에서 STW가 발생하며, 모든 과정은 멀티 스레드로 동작한다.

### G1 GC에서 Major GC는 어떻게 동작하는가?

Major GC는 initial mark, root region scan, concurrent mark, remark, copy/cleanup 단계로 구분되어 있으며, 중간 중간 마킹을 수행하면서 Garbage만 존재하는 영역 및 live object가 적은 순으로 영역을 정리하여 가용 공간을 만들어내는 것이 특징이다.

(디테일한 과정을 물어보면, 위 그림을 이해하여 답변할 것)

### G1 GC에서 Mixed GC는 어떻게 동작하는가?

Mixed GC는 Minor GC와 비슷한 방식으로 동작하지만, 추가로 Old 영역의 Garbage도 정리하는 방식으로 동작한다.
