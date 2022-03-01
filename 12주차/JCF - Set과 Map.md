## Set 인터페이스

Set은 중복된 요소를 저장하지 않고, 요소의 저장 순서가 유지되지 않는다는 특징이 있다. 아래 클래스로 구현이 가능하다.

- HashSet
- LinkedHashSet
- TreeSet

### HashSet 클래스

**특징**

- Set을 이용하기 위해 가장 많이 사용되는 구현 클래스다.
- HashSet은 중복된 요소를 저장하지 않고, 요소의 순서를 유지하지 않는다.
- HashSet은 해시 알고리즘을 사용해 삽입, 탐색이 List, Tree보다 빠르다. 시간복잡도는 O(1)이다.
- HashSet의 핵심음 해시 알고리즘인데, 자세한 내용은 따로 다루도록 하겠다.

**hashCode()와 equals()**

- HashSet은 요소를 저장하기 전에 `hashCode()` 메소드를 호출해 해시 코드를 얻어낸다.
- 저장되어 있는 객체들의 해시 코드와 비교한 뒤 같은 해시 코드가 있다면 다시 `equals()` 메소드로 두 객체를 비교한다.
- 이때, true가 나오면 동일한 객체로 판단하고 중복 저장을 하지 않는다.

### LinkedHashSet 클래스

- HashSet 클래스를 상속 받았으며, 중복된 요소는 저장하지 않는다는 점은 동일하다.
- HashSet과의 차이점은 입력된 순서대로 요소를 저장한다는 점이다.
- 요소의 순서를 유지하는 방법은 LinkedHashMap을 설명하면서 다루도록 하겠다.

## SortedSet 인터페이스

SortedSet 인터페이스는 특정 정렬 기준에 따라 요소의 순서를 정할 수 있다.

### TreeSet 클래스

- TreeSet 클래스는 SortedSet 인터페이스의 구현 클래스다.
- TreeSet은 레드-블랙 트리로 구현 되어있다.
- 따라서, 원하는 정렬 기준으로 요소의 순서를 저장할 수 있다.
- 레드-블랙 트리 자료구조는 따로 다루도록 하겠다.

## Map 인터페이스

- Key와 Value를 쌍으로 저장하는 자료구조다.
- Key는 중복될 수 없고 Value는 중복이 가능하다.
- Key를 통해 Value에 바로 접근이 가능하므로 상수 시간만에 탐색이 가능하다
- 데이터의 순서를 보장하지는 않는다.

### HashTable 클래스

해시 테이블에 대한 내용은 링크를 [참고](https://www.notion.so/HashTable-d646bc3039594d13918c591105c72572)하자.

### HashMap 클래스

**정의**

- Map 인터페이스의 구현 클래스이며, Hashtable을 보완한 클래스다.
- HashMap은 비동기로 작동하기 때문에 멀티 쓰레드 환경에서는 어울리지 않는다.
- 이를 보완하기 위해 ConcurrentHashMap이 등장하였다.

**HashTable과의 차이점**

- 동기화
    - HashTable은 동기화를 지원하지만, HashMap은 동기화를 지원하지 않는다.
- Null 여부
    - HashTable은 Null을 허용하지 않고, HashMap은 Null을 허용한다.
- 보조 해시함수
    - HashMap은 보조 해시함수를 사용하기 때문에 HashTable에 비해서 해시 충돌이 덜 발생한다.
    - 보조 해시함수는 Key의 해시값을 변경해 해시 충돌 가능성을 줄여주는 역할을 한다.
    

### LinkedHashMap

**정의**

- LinkedHashMap은 Map 인터페이스의 구현 클래스이며, HashMap을 확장하는 클래스다.
- 데이터가 입력된 순서를 기억하여 데이터의 순서를 보장한다.

**순서가 보장되는 이유**

- LinkedHashMap은 LinkedList처럼 동작한다.

![image](https://user-images.githubusercontent.com/55661631/156127319-c3a816ef-63f1-4b38-8ff6-c30b4dafb245.png)

- HEAD, TAIL을 설정해 두고 중간에 데이터인 Entry를 연결하여 구현한다.

## SortedMap 인터페이스

SortedMap 인터페이스는 특정 기준으로 요소를 정렬할 수 있도록 해준다.

### TreeMap 클래스

**정의**

- TreeMap 클래스는 SortedMap 인터페이스의 구현 클래스다.
- Key를 기준으로 요소를 원하는 대로 정렬할 수 있다.
- TreeSet과 마찬가지로 레드-블랙 트리 자료구조로 구현되어 있다.

### Map 인터페이스가 Collection 인터페이스를 상속받지 않은 이유

- List, Set, Queue는 Value로 요소가 구성되고, Map은 Key와 Value로 요소가 구성된다.
- 따라서, Map은 List, Set, Queue와 구조상 다르기 때문에 Collection 인터페이스로 묶을 수가 없다.
- 만약 Map이 Collections.remove(Object o)를 오버라이드한다면 Entry를 받아 Key와 Value를 제거해야 한다. 그러나 Map의 구조상 Key만 받아 Key와 Value를 제거하는 것이 가능하다.
- 정리하자면, Map은 List, Set, Queue와는 다른 구조를 갖고 있어 Collection 인터페이스를 상속받지 않았다.

## 참고
- [https://steady-coding.tistory.com/358](https://steady-coding.tistory.com/358)
- [https://crazykim2.tistory.com/583](https://crazykim2.tistory.com/583)
- [https://steady-coding.tistory.com/359](https://steady-coding.tistory.com/359)
- [https://crazykim2.tistory.com/590](https://crazykim2.tistory.com/590)
- [https://codingdog.tistory.com/entry/언급이-좀-되는-모-자료구조의-보조-해시-함수에-대한-이야기](https://codingdog.tistory.com/entry/%EC%96%B8%EA%B8%89%EC%9D%B4-%EC%A2%80-%EB%90%98%EB%8A%94-%EB%AA%A8-%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0%EC%9D%98-%EB%B3%B4%EC%A1%B0-%ED%95%B4%EC%8B%9C-%ED%95%A8%EC%88%98%EC%97%90-%EB%8C%80%ED%95%9C-%EC%9D%B4%EC%95%BC%EA%B8%B0)
