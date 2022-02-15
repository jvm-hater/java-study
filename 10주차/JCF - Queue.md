## Queue 인터페이스

Queue 인터페이스는 먼저 들어온 요소가 먼저 나가는 방식인 FIFO 구조를 가진 자료구조이다.

![image](https://user-images.githubusercontent.com/55661631/154046191-c5ced146-0420-4db0-9d19-5cdb21d140f5.png)

### Queue 생성

Queue를 생성할 때는 아래와 같이 LinkedList를 사용해 구체화한다.

```java
Queue<Integer> queue = new LinkedList<>();
```

ArrayList가 아닌 LinkedList를 사용하는 이유는 LinkedList의 삽입, 삭제 연산의 시간복잡도가 O(1)이기 때문이다.

삽입, 삭제할 위치를 모르면 탐색까지 해야해서 연산의 시간복잡도가 O(N)이지만, Queue는 요소를 맨 앞과 맨 뒤에만 삽입, 삭제하므로 탐색하는 시간이 들지 않아 O(1)이다.

### Queue 메소드

Queue 인터페이스는 6개의 메소드를 제공한다.

- add(E e), offer(E e)
    - e 객체를 맨 뒤에 삽입한다.
- element()
    - 맨 앞의 요소를 반환한다. 큐가 비어있을 경우 NoSuchElementException을 반환한다.
- peek()
    - 맨 앞의 요소를 반환한다.
- poll()
    - 맨 앞의 요소를 반환한 뒤 삭제한다.
- remove()
    - 맨 앞의 요소를 반환한 뒤 삭제한다. 큐가 비어있을 경우 NoSuchElementException을 반환한다.
    

## Priority Queue 클래스

- Priority Queue는 우선순위 큐라고 부른다.
- Queue 인터페이스를 상속받았으며, 일반적인 Queue와 다르게 **우선순위가 높은 요소가 먼저 나가는 자료구조**다.
- 우선순위 큐는 힙을 이용하여 구현하는 것이 일반적이다. 따라서 요소를 삽입할 떄 우선순위를 기준으로 최대 힙 또는 최소 힙을 구성한다.
- 우선순위를 위한 대소비교가 필요하기 때문에 Comparator를 정의하거나 Comparable을 상속한 객체를 이용해야 한다.
- 우선순위 큐에 들어갈 요소가 Integer, Double, String, Character와 같은 경우, 따로 Comparator을 정의하지 않으면 정렬 기준이 오름차순이 된다.
- 우선순위 큐는 null을 삽입할 수 없다.

## Deque 인터페이스

Deque는 Double-Ended Queue의 약어로, Queue의 양쪽 끝에서 추가와 삭제가 일어날 수 있는 자료구조이다.

![image](https://user-images.githubusercontent.com/55661631/154046230-eca2ac31-b212-4004-aade-f1e5f8d0e2ac.png)

Deque는 Stack과 Queue의 기능을 모두 갖고 있기 때문에 아래와 같이 Deque만의 메소드를 갖고 있다.

![image](https://user-images.githubusercontent.com/55661631/154046271-1e4d47c3-32dc-49d6-b7c4-0114f80204a6.png)

참고로 모든 메소드의 시간복잡도는 O(1)이다.

### ArrayDeque 클래스

- ArrayDeque 클래스는 Deque 인터페이스를 상속받아서 사용하는 구현 클래스다.
- ArrayDeque는 사이즈 제한이 없는 가변 배열이다.
    - 초기 사이즈 16에서 2배씩 늘려나간다.
- null 요소는 저장할 수 없다.
- Deque는 LinkedList, ArrayDeque 등으로 구현할 수 있으며, 주로 ArrayDeque를 사용한다.
- ArrayDeque는 Stack과 LinkedList보다 빠르다.

### **ArrayDeque가 LinkedList보다 빠른 이유**

- LinkedList로 구현한 Queue의 경우, 맨 마지막 요소를 추가할 때 새로운 노드에 대한 참조를 해야한다.
- ArrayDeque로 구현한 Queue의 경우, 마지막 인덱스에 요소를 할당해 주면 된다.
- 이러한 차이로 여러 장점이 있다.
    - 수행속도
        - ArrayDeque는 LinkedList보다 cache-locality에 더 친숙(메모리 상에 연속해서 위치할 확률이 높음)해 수행속도가 더 빠르다.
    - 메모리
        - ArrayDeque는 LinkedList와는 다르게 다음 노드에 대한 추가 참조를 유지할 필요가 없어 메모리를 효율적으로 사용할 수 있다.

### ArrayDeque 주의점

- ArrayDequq는 사이즈가 2배씩 늘어나는 가변 배열이다. 따라서 저장해야할 데이터가 매우 방대하다면 사용을 피하는 것이 좋다고 한다.

## 참고

- [https://steady-coding.tistory.com/357](https://steady-coding.tistory.com/357)
- [https://stackoverflow.com/questions/6163166/why-is-arraydeque-better-than-linkedlist](https://stackoverflow.com/questions/6163166/why-is-arraydeque-better-than-linkedlist)
- [https://mungto.tistory.com/465](https://mungto.tistory.com/465)
- 이것이 자바다

## 예상 면접 질문 및 답변

### Q. Queue 인터페이스의 특징은?

Queue 인터페이스는 먼저 들어온 요소가 먼저 나가는 방식인 FIFO 구조를 가진 자료구조이다. Queue를 구현할 때 LinkedList를 사용해 구체화를 하는데 그 이유는 맨 뒤에 삽입, 맨 앞에 삭제 연산의 시간복잡도가 O(1)이기 때문이다.

### Q. Priority Queue의 특징은?

Priority Queue는 우선순위 큐라고 부른다. Queue 인터페이스를 상속받았으며, 일반적인 Queue와 다르게 우선순위가 높은 요소가 먼저 나가는 자료구조다. 우선순위 큐는 힙을 이용하여 구현하는 것이 일반적이다. 따라서 요소를 삽입할 떄 우선순위를 기준으로 최대 힙 또는 최소 힙을 구성한다.

### Q. Deque의 특징은?

Deque는 Double-Ended Queue의 약어로, Queue의 양쪽 끝에서 추가와 삭제가 일어날 수 있는 자료구조이다. ArrayDeque와 LinkedList로 구체화할 수 있으며, 모든 메소드의 시간 복잡도는 O(1)이다.

### Q. ArrayDeque의 특징은?

- ArrayDeque 클래스는 Deque 인터페이스를 상속받아서 사용하는 구현 클래스다.
- ArrayDeque는 사이즈 제한이 없는 가변 배열이며 초기 사이즈 16에서 2배씩 늘려나간다.
- ArrayDeque는 Stack과 LinkedList보다 빠르다.

### Q. ArrayDeque 주의점은?

ArrayDequq는 사이즈가 2배씩 늘어나는 가변 배열이다. 따라서 저장해야할 데이터가 매우 방대하다면 LinkedList를 사용해야 한다.

### Q. ArrayDeque와 LinkedList의 차이는?

- LinkedList로 구현한 Queue의 경우, 맨 마지막 요소를 추가할 때 새로운 노드에 대한 참조를 해야한다.
- ArrayDeque로 구현한 Queue의 경우, 마지막 인덱스에 요소를 할당해 주면 된다.

### Q. ArrayDeque가 LinkedList보다 성능이 좋은 이유는?

- 수행속도
    - ArrayDeque는 LinkedList보다 cache-locality에 더 친숙(메모리 상에 연속해서 위치할 확률이 높음)해 수행속도가 더 빠르다.
- 메모리
    - ArrayDeque는 LinkedList와는 다르게 다음 노드에 대한 추가 참조를 유지할 필요가 없어 메모리를 효율적으로 사용할 수 있다.
