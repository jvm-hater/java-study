## JCF란?

JCF는 Java Collections Framework의 약어로, Java에서 데이터를 저장하는 기본적인 자료구조들을 한 곳에 모아 관리하고 편하게 사용하기 위해서 제공하는 프레임워크다.

💡 **프레임워크(Framework) vs 라이브러리(Library)**   
프레임워크는 소프트웨어의 특정 문제를 해결하기 위해서 상호 협력하는 클래스와 인터페이스의 집합이며, 라이브러리는 단순 활용가능한 도구들의 집합을 말한다.

이 둘의 가장 큰 차이는 **흐름에 대한 제어 권한이 어디에 있는냐의 차이**다. 프레임워크는 전체적인 흐름을 자체적으로 가지고 있으며, 개발자가 그 안에 필요한 코드를 작성하는 반면에 라이브러리는 사용자가 직접 흐름에 대해 제어를 하며 필요한 상황에 가져다 쓰는 것이다.

## JCF의 도입 배경

JCF가 도입되기 전, 자바 객체를 그룹핑(Collection)하는 표준화된 방법은 Arrays, Vectors, Hashtables였으며, 이 Collection들에는 어떠한 공통 인터페이스가 존재하지 않았다.

따라서 이 Collection들의 사용 목적이 동일하더라도, 각자 따로 정의해야하고 각각의 Collection마다 사용하는 메소드, 문법, 생성자가 달랐기에 개발자가 이들을 사용하면서 혼동하기가 쉬웠다.

이러한 문제를 해결하기 위해 JDK 1.2 버전부터 JCF가 도입되었다.

## JCF의 계층 구조

JCF는 크게 Iterable를 상속받은 **Collection 인터페이스**와 Iterable를 상속받지 않은 **Map 인터페이스**로 나눌 수 있다. Map 인터페이스가 Iterable 인터페이스를 상속받지 않은 이유는 Map을 설명할 때 자세히 다루겠다.

**Collection 인터페이스**

![image](https://user-images.githubusercontent.com/55661631/154045732-47e0d067-6924-4174-afdb-bf9fe2016b56.png)

**Map 인터페이스**

![image](https://user-images.githubusercontent.com/55661631/154045761-3a3426af-1882-4a97-9582-5f832b7c0de9.png)

## List 인터페이스

List는 순서가 있는 데이터의 집합이며, 데이터의 중복을 허용한다. List 인터페이스의 구현 클래스는 다음과 같다.

- ArrayList
- LinkedList
- Vector

### ArrayList 클래스

ArrayList는 List 인터페이스의 구현 클래스이며, ArrayList에 객체를 추가하면 객체가 인덱스로 관리된다. 일반 배열과 ArrayList는 인덱스로 객체를 관리한다는 점은 유사하지만, 큰 차이점을 가지고 있다. 배열은 생성할 때 크기가 고정되고 사용 중에 크기를 변경할 수 없지만, ArrayList는 크기가 가변적인 선형 리스트로 용량을 넘어서면 자동으로 용량이 증가해 객체를 삽입할 수 있다.

![image](https://user-images.githubusercontent.com/55661631/154045802-8211c5d5-752b-434a-a23e-5e16f9b421a7.png)

**ArrayList의 용량**

아래 코드는 ArrayList 클래스에 객체를 삽입하는 메소드다.

```java
private static final int DEFAULT_CAPACITY = 10;

private void add(E e, Object[] elementData, int s) {
      if (s == elementData.length)
          elementData = grow();
      elementData[s] = e;
      size = s + 1;
}

private Object[] grow() {
      return grow(size + 1);
}

private Object[] grow(int minCapacity) {
	    return elementData = Arrays.copyOf(elementData, newCapacity(minCapacity));
}

private int newCapacity(int minCapacity) {
      // overflow-conscious code
      int oldCapacity = elementData.length;
      int newCapacity = oldCapacity + (oldCapacity >> 1);      //1번
      if (newCapacity - minCapacity <= 0) {
          if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
              return Math.max(DEFAULT_CAPACITY, minCapacity);  //2번
          if (minCapacity < 0) // overflow
              throw new OutOfMemoryError();
          return minCapacity;
      }
      return (newCapacity - MAX_ARRAY_SIZE <= 0)
          ? newCapacity
          : hugeCapacity(minCapacity);
}
```

- `DEFAULT_CAPACITY` 변수와 `newCapacity()` 메소드의 k번을 보면 처음 삽입할 때 ArrayList의 용량을 기본용량(10)으로 초기화하는 것을 확인할 수 있다.
- 새로운 객체가 삽입될 때 `add()` 메소드를 쭉 따라가다보면 1번 코드에서 ArrayList의 크기는 기존 용량 + (기존 용량 >> 1)만큼 추가되는 것을 확인할 수 있다.

**삽입**

- `add(object)`를 사용해서 맨 뒤에 삽입하거나 `add(index, object)`를 사용해서 특정 위치에 삽입할 수 있다.
- 맨 뒤에 삽입할 경우 시간복잡도는 O(1)이다.
- 특정 위치에 삽입할 경우 뒤의 요소들을 밀어야하기 때문에 시간복잡도가 O(N)이 된다.

**삭제**

- `remove()`를 사용한다.
- 특정 위치를 삭제할 경우 뒤의 요소들을 당겨야하기 때문에 시간복잡도가 O(N)이 된다.

**탐색**

- `contains()` 를 사용해 탐색할 수 있다.
- 리스트의 처음부터 끝까지 탐색하므로 시간복잡도는 O(N)이다.

**반환**

- `get()`을 사용해 특정 요소를 반환할 수 있다. 인덱스를 사용하기 때문에 시간복잡도는 O(1)이다.

### LinkedList 클래스

LinkedList는 각 노드가 데이터와 포인터를 가지고 한 줄로 연결되어있는 자료구조다. 데이터를 담고 있는 노드들이 연결되어 있고, 노드의 포인터가 이전 노드와 다음 노드와의 연결을 담당한다.

![image](https://user-images.githubusercontent.com/55661631/154045961-27e65aca-9431-4c25-bb3d-5aac0297bba4.png)

**삽입**

- `add(object)`와 `add(index, object)`를 사용해서 요소를 삽입할 수 있다.
- 맨 앞과 맨 뒤에 요소를 삽입할 경우에 포인터만 변경하면 되기 때문에 시간복잡도는 O(1)이다.
- **중간에 요소를 삽입할 경우에 탐색을 해야하기 때문에 시간복잡도는 O(N)이다.** 그러나 ArrayList와 다르게 요소들을 뒤로 밀지 않아도 되기 때문에 처리 속도가 더 빠르다.

**삭제**

- ArrayList와 비슷하고 시간복잡도는 O(N)이다.

**탐색**

- ArrayList와 동일하다.

**반환**

- LinkedList는 인덱스가 없어 맨 앞부터 탐색해야 한다. 따라서 시간복잡도는 O(N)이다.

**ArrayList vs LinkedList**

- LinkedList는 중간에 데이터를 추가나 삭제하더라도 전체의 인덱스가 한 칸씩 뒤로 밀리거나 당겨지는 일이 없기에 ArrayList에 비해서 데이터의 추가나 삭제 처리 속도가 빠르다.
- 그러나 인덱스가 없기에 특정 요소에 접근하기 위해서는 순차 탐색이 필요로 하여 탐색 속도가 떨어진다는 단점이 있다.
- 그러므로 탐색 또는 정렬을 자주 하는 경우엔 ArrayList를 사용하고 데이터의 추가/삭제가 많은 경우 LinkedList를 사용하는 것이 좋다.

### Stack과 Vector

- Vector
    - ArrayList와 거의 유사하다.
    - JDK 1.0부터 존재하는 자료구조이다. 호환성을 위해 남겨둔 레거시 자료구조이므로 사용하지 않는 것이 좋다.
    - **단점**
        - synchronized 메소드로 동기화 처리해서 모든 메소드가 하나의 Lock을 공유한다. 이 때문에 멀티 스레드 환경에서 싱글 스레드처럼 동작하여 비효율적이다.
        - 동기화 처리를 해두어 멀티 스레드에서 안전할 것 같지만, 여러 연산 묶어서 사용한다면 멀티 스레드에 안전하지 않다.
- Stack
    - Vector를 상속하여 만든 LIFO 방식의 클래스이다. Vector와 마찬가지로 호환성을 위해 남겨둔 레거시 자료구조이므로 사용하지 않는 것이 좋다.
    - Stack을 사용하려면 Deque를 사용하는 것이 좋다.

## 참고

- [https://steady-coding.tistory.com/356](https://steady-coding.tistory.com/356)
- [https://dodo-factory.tistory.com/6](https://dodo-factory.tistory.com/6)
- [https://mungto.tistory.com/465](https://www.geeksforgeeks.org/internal-working-of-arraylist-in-java/)
- [https://www.geeksforgeeks.org/internal-working-of-arraylist-in-java/](https://www.geeksforgeeks.org/internal-working-of-arraylist-in-java/)
- 이것이 자바다

## 예상 면접 질문 및 답변

### Q. JCF란?

JCF는 데이터를 저장하는 기본적인 자료구조들을 한 곳에 모아 관리하고 편하게 사용하기 위해서 제공하는 프레임워크다.

### Q. 프레임워크와 라이브러리의 차이

프레임워크는 소프트웨어의 특정 문제를 해결하기 위한 클래스와 인터페이스의 집합이며, 라이브러리는 단순 활용가능한 도구들의 집합을 말한다.

이 둘의 가장 큰 차이는 **흐름에 대한 제어 권한이 어디에 있는냐의 차이**다. 프레임워크는 전체적인 흐름을 자체적으로 가지고 있으며, 개발자가 그 안에 필요한 코드를 작성하는 반면에 라이브러리는 사용자가 직접 흐름에 대해 제어를 하며 필요한 상황에 가져다 쓰는 것이다.

### Q. List 인터페이스의 특징은?

List는 순서가 있는 데이터의 집합이며, 데이터의 중복을 허용한다. 구현 클래스로는 ArrayList, LinkedList, Vector이 있다.

### Q. ArrayList의 특징은?

ArrayList는 List 인터페이스의 구현 클래스이며, ArrayList 내의 객체가 인덱스로 관리되고 크기가 가변적인 선형 리스트라는 특징이 있다.

삽입 - 중간 O(N), 맨 뒤 O(1)

탐색 - O(N)

삭제 - 중간 O(N), 맨 뒤 O(1)

반환 - O(1) 

### Q. ArrayList의 용량이 증가하는 시점은?

빈 ArrayList에 처음 객체가 삽입될 때 기본 용량인 10으로 초기화된다. 이후로 새로운 객체가 삽입될 때마다 ArrayList의 용량은 기존 용량 + (기존 용량 >> 1)만큼 증가한다.

### Q. 일반 배열과 ArrayList의 차이점은?

일반 배열은 생성할 때 크기가 고정되어 사용 중에는 크기를 변경할 수 없다. 반면에 ArrayList는 객체를 추가하면 크기가 변경된다.

### Q. LinkedList의 특징은?

LinkedList는 각 노드가 데이터와 포인터를 가지고 한 줄로 연결되어있는 자료구조다. 데이터를 담고 있는 노드들이 연결되어 있고, 노드의 포인터가 이전 노드와 다음 노드와의 연결을 담당한다.

삽입 - 중간 O(N), 맨 뒤 또는 맨 앞 O(1)

탐색 - O(N)

삭제 - 중간 O(N), 맨 뒤 또는 맨 앞 O(1)

반환 - O(N) 

### Q. ArrayList와 LinkedList의 성능 차이

LinkedList는 중간에 데이터를 추가나 삭제하더라도 전체의 인덱스가 한 칸씩 뒤로 밀리거나 당겨지는 일이 없기에 ArrayList에 비해서 데이터의 추가나 삭제 처리 속도가 빠르다. 그러나 인덱스가 없어 특정 요소에 접근하기 위해서는 순차 탐색이 필요로 하여 탐색 속도가 떨어진다는 단점이 있다.

### Q. Vector란 무엇이고 사용하지 않는 이유는?

ArrayList와 거의 비슷한 기능을 한다. 그러나 synchronized 메소드로 동기화 처리해서 모든 메소드가 하나의 Lock을 공유해 멀티 스레드 환경에서 싱글 스레드처럼 동작한다는 점과 모든 메소드에 동기화 처리를 해두어 멀티 스레드에서 안전할 것 같지만, 여러 연산 묶어서 사용한다면 멀티 스레드에 안전하지 않다는 단점이 있어 사용하지 않는다.
