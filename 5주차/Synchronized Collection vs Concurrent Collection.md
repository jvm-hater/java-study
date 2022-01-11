# Synchronized Collection(동기화된 컬렉션)

동기화된 컬렉션에는 대표적으로 다음과 같은 클래스가 있다.

- Vector
- HashTable
- Collections.synchronizedXxx

이 클래스 모두 public으로 선언된 모든 메소드에 synchronized 키워드를 사용해 내부의 값을 한 스레드만 사용할 수 있도록 제어하면서 동시성을 보장하고 있다.

위 클래스들이 어떻게 구현되어 있는지 직접 코드를 살펴보자.

## Vector

다음 코드는 Vector 클래스의 일부를 가져온 코드이다.

```java
public class Vector<E> extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    ...
    public synchronized boolean add(E e) {
				modCount++;
				add(e, elementData, elementCount);
				return true;
    }
    ...
}
```

Vector 클래스에서 요소를 추가하는 `add()` 메소드를 살펴보면 synchronized 키워드가 보인다. 즉, 내부적으로 Vector에서 요소 삽입 연산이 진행될 때 동기화가 보장된다.

## HashTable

다음 코드는 HashTable 클래스의 일부를 가져온 코드이다.

```java
public class Hashtable<K,V> extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {
		...
		public synchronized boolean contains(Object value) {
        if (value == null) {
            throw new NullPointerException();
        }

        Entry<?,?> tab[] = table;
        for (int i = tab.length ; i-- > 0 ;) {
            for (Entry<?,?> e = tab[i] ; e != null ; e = e.next) {
                if (e.value.equals(value)) {
                    return true;
                }
            }
        }
        return false;
    }
		...
}
```

HashTable 클래스에서 동일한 value가 존재하는지 확인하는 `contains()` 메소드를 살펴보면 Vector 클래스와 동일하게 synchronized 키워드가 사용된 것을 확인할 수 있다.

## Collections.synchronizedXxx

`Collections.synchronizedList()` 메소드를 사용해 생성되는 SynchronizedList<E> 클래스의 일부를 가져온 코드이다.

```java
static class SynchronizedList<E> extends SynchronizedCollection<E>
		implements List<E> {

		final Object mutex;

		...
		public E get(int index) {
				synchronized (mutex) {return list.get(index);}
		}

		public E set(int index, E element) {
				synchronized (mutex) {return list.set(index, element);}
		}
		
		public void add(int index, E element) {
				synchronized (mutex) {list.add(index, element);}
		}
		
		public E remove(int index) {
				synchronized (mutex) {return list.remove(index);}
		}
		...
}
```

SynchronizedList<E> 클래스의 메소드들을 살펴보면 모두 synchronized 키워드가 사용된 것을 확인할 수 있다. 다만 위 클래스와 다른 점은 synchronized 메소드가 아닌 블록이라는 점이다.

`Collections.synchronizedXxx()` 메소드를 사용해 생성할 수 있는 동기화된 컬렉션은 더 많지만, 비슷하기 때문에 생략하겠다.

## Synchronized Collection(동기화된 컬렉션)의 문제점

멀티 스레드 환경에서 동기화된 컬렉션을 사용해야 하는 경우도 있지만, 왠만해서는 다른 동기화 방식을 사용하는 것이 좋다. 그 이유는 다음과 같다.

- 동기화된 컬렉션 클래스는 멀티 스레드 환경에서 동시성을 보장한다. 하지만 여러 연산을 묶어 하나의 단일 연산처럼 활용해야 하는 경우 문제가 발생한다. 
이러한 상황에서 동기화된 컬렉션을 사용해도 올바르게 동작하지 않을 수도 있다. 두 개 이상의 연산을 묶어 사용해야 하는 예는 반복(iteration), 이동(navigation), 없는 경우에만 추가(add if absent) 등이 있다.
- 동기화된 컬렉션은 Collections.synchronizedXxx 예시를 보면 알 수 있듯이, 모든 메소드가 mutex 객체에 걸리는 잠금(Lock)을 공유하고 있다. 따라서, 컬렉션에 하나의 스레드가 잠금(Lock)을 획득하면 다른 스레드들은 모든 메소드를 사용하지 못하고 Blocking 하게 된다. 스레드가 Blocking 되는 것은 애플리케이션 성능 저하의 원인이 될 수 있다.

# Concurrent Collection(병렬 컬렉션)

위에서 설명했듯이 동기화된 컬렉션을 여러 스레드가 사용하려고 하면 많은 자원을 낭비하게 되고, 이는 성능 저하의 원인이 된다. 지금부터 설명할 Concurrent Collection(병렬 컬렉션)은 여러 스레드에서 동시에 사용할 수 있도록 설계되어 있다. 

`java.util.concurrent` 패키지에서 제공하는 병렬 컬렉션의 종류는 다음과 같다. 이번 글에서는 이 중 일부만 다룰 예정이다.

- **CopyOnWriteArrayList**
    - List 클래스의 하위 클래스이며 객체 목록을 반복시키며 열람하는 연산의 성능을 최우선으로한 병렬 컬렉션이다.
- **ConcurrentMap**
    - 병렬 컬렉션이며, 인터페이스를 보면 추가하려는 항목이 기존에 없는 경우에만 새로 추가하는 put-if-absent, replace, conditional remove 연산 등이 정의되어 있다.
- **ConcurrentHashMap**
    - ConcurrentMap의 하위 클래스이며, HashMap을 대치하면서 병렬성을 확보한 병렬 컬렉션이다.
- **ConcurrentLinkedQueue**
    - FIFO 방식 Queue이며, 병렬성을 확보한 병렬 컬렉션이다. 큐에서 꺼낼 원소가 없다면 즉시 리턴하고 다른 일을 수행하러 간다.
- **LinkedBlockingQueue**
    - ConcurrentLinkedQueue와 비슷하다. 그러나 큐가 비어 있다면 큐에서 항목을 봅아내는 연산은 새로운 항목이 추가될 때까지 대기하고, 반대로 큐에 크기가 지정되어 있는 경우에 큐가 지정한 크기만큼 가득 차 있다면, 큐에 새로운 항목을 추가하는 연산은 큐에 빈 자리가 생길 때까지 대기한다는 차이점이 있다.
- **ConcurrentSkipListMap, ConcurrentSkipListSet**
    - 각각 SortedMap 과 SortedSet 클래스의 병렬성을 높이도록 발전된 형태이다

기존에 사용하던 동기화 컬렉션 클래스를 병렬 컬렉션으로 교체하는 것만으로도 별다른 위험 요소 없이 전체적인 성능을 상당히 끌어 올릴 수 있다.

Concurrent Collection(병렬 컬렉션)을 대치되는 Synchronized Collection(동기화된 컬렉션)과 비교하며 자세히 알아보자.

## CopyOnWriteArrayList

동기화된 ArrayList를 생성하는 방법은 아래 두 가지가 있다.

- Collections.synchronizedList()
- CopyOnWriteArrayList(COWL로 축약해서 부르겠다.)

synchronizedList는 자바 1.2 버전에 추가되었다. 하지만 모든 읽기와 쓰기 동작은 이름에 걸맞게 synchronized 키워드로 동기화가 되어 있는데, 이는 상황에 따라 융통성이 없는 설계라고 할 수 있다. 따라서 비슷한 컨셉의 COWL이 등장하게 되었다.

### 읽기 동작

SynchronizedList는 읽기와 쓰기 동작시 인스턴스 자체에 잠금이 걸린다. 그러나 COWL은 모든 쓰기 동작시 원본 배열에 있는 요소를 복사하여 새로운 임시 배열을 만들고 이 임시 배열에 쓰기 동작을 수행한 후 원본 배열을 갱신한다. 이 덕분에 읽기 동작은 잠금이 걸리지 않으며, 이는 읽기 동작을 수행할 시 COWL가 SynchronizedList보다 성능이 좋은 이유이다.

다음 코드는 COWL클래스에서 읽기 수행을 하는 `get()` 메소드이다. 

```java
public class CopyOnWriteArrayList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {

    final transient Object lock = new Object();
    private transient volatile Object[] array;

		...
		public E get(int index) {
        return elementAt(getArray(), index);
    }
		...
}
```

`get()` 메소드를 수행할 때 잠금이 걸리지 않는 것을 확인할 수 있다.

### 쓰기 동작

COWL은 쓰기 동작 수행시 **명시적 락**을 사용한다. 결국 두 종류의 Collection 모두 이 동작에서 잠금이 걸리는 것이다. 하지만 COWL비용이 상대적으로 높은 배열 복사 작업을 하기 때문에 성능의 측면에서 손해를 볼 수 있다. 따라서, 변경 작업보다 반복문으로 읽어내는 일이 훨씬 빈번할 경우 효과적이다.

다음 코드는 배열에 새로운 원소를 삽입하는 `add()` 메소드의 일부이다.

```java
public class CopyOnWriteArrayList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {

    final transient Object lock = new Object();
    private transient volatile Object[] array;

		public void add(int index, E element) {
        synchronized (lock) {
            ...
            int numMoved = len - index;
            if (numMoved == 0)
                newElements = Arrays.copyOf(es, len + 1);
            else {
                newElements = new Object[len + 1];
                System.arraycopy(es, 0, newElements, 0, index);
                System.arraycopy(es, index, newElements, index + 1,
                                 numMoved);
            }
						...
        }
    }
}
```

위에서 말했듯이, 쓰기 동작시 잠금을 걸고 `System.arraycopy()` 메소드를 사용해 배열 복사 작업을 한다.

### Iterater

COWL에서 iterator을 뽑아내 사용한다면 iterator을 뽑아내는 시점의 컬렉션 데이터를 기준으로 반복하며, 반복하는 동안 컬렉션에 추가되거나 삭제되는 내용은 반복문과 상관 없는 복사본을 대상으로 반영하기 때문에 동시 사용성에 문제가 없다.

## CopyOnWriteArraySet

동기화된 Set을 생성하는 방법은 아래 두 가지가 있다.

- Collections.synchronizedSet()
- CopyOnWriteArraySet(COWS로 축약해서 부르겠다.)

메소드 네이밍을 보면 알 수 있듯이, CopyOnWriteArrayList와 동작 방식이 자료구조 특징을 제외하면 거의 동일하다.

### 읽기 동작

코드를 직접 살펴보자. 다음 코드는 COWS 클래스의 전달한 원소가 존재하는지 확인하는 `contains()` 메소드이다.

```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E> implements java.io.Serializable {

    private final CopyOnWriteArrayList<E> al;
		
		public boolean contains(Object o) {
        return al.contains(o);
    }
}
```

COWS 클래스의 상단을 보면 `CopyOnWriteArrayList<E>` 타입의 인스턴스 변수가 선언된 것을 볼 수 있다. 그리고 `contains()` 메소드 내부를 살펴보면 COWL의 `contains()` 메소드를 호출하는 것을 확인할 수 있다.

`al.contains()` 메소드를 살펴보자.

```java
public boolean contains(Object o) {
	    return indexOf(o) >= 0;
}

public int indexOf(Object o) {
	    Object[] es = getArray();
	    return indexOfRange(o, es, 0, es.length);
}

	private static int indexOfRange(Object o, Object[] es, int from, int to) {
	    if (o == null) {
	        for (int i = from; i < to; i++)
	            if (es[i] == null)
	                return i;
	    } else {
	        for (int i = from; i < to; i++)
	            if (o.equals(es[i]))
	                return i;
	    }
	    return -1;
}
```

SynchronizedSet과 다르게 COWS의 읽기 동작에는 잠금이 걸리지 않는 것을 확인할 수 있다.

### 쓰기 동작

쓰기 동작을 하는 COWS의 `add()` 메소드도 살펴보자.

```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E> implements java.io.Serializable {

		private final CopyOnWriteArrayList<E> al;

		public boolean add(E e) {
        return al.addIfAbsent(e);
    }
}
```

`add()` 메소드도 마찬가지로 COWL의 `addIfAbsent()` 메소드를 호출하는 것을 볼 수 있다.

`addIfAbsent()` 메소드를 살펴보자.

```java
public boolean addIfAbsent(E e) {
    Object[] snapshot = getArray();
    return indexOfRange(e, snapshot, 0, snapshot.length) < 0
        && addIfAbsent(e, snapshot);
}

private boolean addIfAbsent(E e, Object[] snapshot) {
	    synchronized (lock) {
	        Object[] current = getArray();
	        int len = current.length;
	        if (snapshot != current) {
	            // Optimize for lost race to another addXXX operation
	            int common = Math.min(snapshot.length, len);
	            for (int i = 0; i < common; i++)
	                if (current[i] != snapshot[i]
	                    && Objects.equals(e, current[i]))
	                    return false;
	            if (indexOfRange(e, current, common, len) >= 0)
	                    return false;
	        }
	        Object[] newElements = Arrays.copyOf(current, len + 1);
	        newElements[len] = e;
	        setArray(newElements);
	        return true;
	    }
  }
```

`addIfAbsent()` 메소드를 살펴보면 COWS는 쓰기 동작시 잠금을 사용하며, 원본 배열에 있는 요소를 복사하여 새로운 임시 배열을 만들고 이 임시 배열에 쓰기 동작을 수행한 후 원본 배열을 갱신하는 것을 알 수 있다.

이 덕분에 읽기 동작은 잠금에 걸리지 않아 성능이 좋지만, 쓰기 동작은 배열 복사 작업을 하기 때문에 성능의 측면에서 손해를 볼 수 있다. 

따라서, COWL와 동일하게 변경 작업보다 반복문으로 읽어내는 일이 훨씬 빈번할 경우 효과적이다.

## ConcurrentHashMap

동기화된 HashMap를 생성하는 방법은 아래 두 가지가 있다.

- Collections.synchronizedMap(new HashMap<>())
- ConcurrentHashMap

ConcurrentHashMap은 HashMap과 동일하게 Hash를 기반으로 하는 Map이다. synchronizedMap에 비해 더욱 효율적인 방법으로 동시성을 보장한다. 

자바 8 이전에는 ReentrantLock을 상속 받는 Segment를 이용해, 영역을 구분하여(기본 16개) 영역 별로 잠금을 하는 방식이였다. 

자바 8 이후부터는 각 테이블 버킷을 독립적으로 잠그는 방식을 사용한다. 빈 버킷에 노드를 삽입할 경우에는 잠금(Lock) 대신 CAS 알고리즘을 사용하고, 그 외의 변경(수정, 삭제)은 각 버킷의 첫 번째 노드를 기준으로 부분적인 잠금(synchronized block)을 획득하여 스레드 경합을 최소화 하며 동시성을 보장한다.

ConcurrentHashMap에 새로운 노드를 삽입하는 `putVal()` 메소드의 코드를 보며 동시성을 어떻게 보장하는지 확인해보자. 참고로 아래 예시는 자바 11 기준이다.

`putVal()` 메소드는 크게 아래와 같이 2가지 경우(분기는 총 4 부분)로 나눌 수 있다.

- 빈 해시 버킷에 노드를 삽입하는 경우
- 해시 버킷에 이미 노드가 존재하는 경우

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh; K fk; V fv;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
						**//(1)**
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
								**//(2)**
                if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                    break;
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else if (onlyIfAbsent // check first node without acquiring lock
                     && fh == hash
                     && ((fk = f.key) == key || (fk != null && key.equals(fk)))
                     && (fv = f.val) != null)
                return fv;
						**//(3)**
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
																**//(4)**
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
																**//(5)**
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key, value);
                                    break;
                                }
                            }
                        }
												**//(6)**
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                        else if (f instanceof ReservationNode)
                            throw new IllegalStateException("Recursive update");
                    }
                }
								...
            }
        }
				...
    }
```

### **빈 해시 버킷에 노드를 삽입하는 경우**

**(1) -** 새로운 노드를 삽입하기 위해, 해당 버킷 값(`tabAt()`)을 가져와 비어 있는지 확인한다.

```java
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
		return (Node<K,V>)U.getObjectAcquire(tab, ((long)i << ASHIFT) + ABASE);
}
```

**(2)** - 노드에 담겨 있는 volatile 변수에 접근하여 기존 값(null)과 비교한 후 같으면 새로운 노드를 저장하고, 아니면 `for()` 문으로 돌아간다(재시도). 이 방식은 이전 글에서 설명한 CAS(Compare And Swap) 알고리즘이다.

```java
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i, Node<K,V> c, Node<K,V> v) {
		return U.compareAndSetObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
```

CAS 알고리즘을 사용해 원자성과 가시성 문제를 해결해, 동시성을 보장한다.

### **해시 버킷에 이미 노드가 존재하는 경우**

**(3)** - 이미 노드가 존재하는 경우에는 synchronized block을 사용해 하나의 스레드만 접근할 수 있도록 제어한다. 이때, 비어있지 않은 `Node<K,V>` 타입의 해시 버킷에 잠금을 걸기 때문에 동일한 버킷에 접근하는 스레드들은 Blocking 된다.

**(4)** - 새로운노드로 교체된다.

**(5)** - 해시 충돌이 일어난 경우 Separate Chaing에 추가한다.

**(6)** - 해시 충돌이 일어난 경우 트리에 추가한다.

# 참고

- https://tyboss.tistory.com/entry/Java-SynchronizedCollections-vs-ConcurrentCollections
- http://asuraiv.blogspot.com/2020/02/java-synchronizedlist-vs.html
- https://jdm.kr/blog/197
- https://pplenty.tistory.com/17
- http://egloos.zum.com/ryukato/v/731585
- https://sungjk.github.io/2016/11/02/Queue.html

# 면접 질문 및 답변

### Q. Vector, HashTable, Collections.SynchronziedXxx의 문제점은?

Vector, HashTable, SynchronziedXxx 클래스들은 synchronized 메소드 또는 블록을 사용하며, 하나의 잠금 객체를 공유한다. 따라서, 컬렉션에 하나의 스레드가 잠금을 획득하는 경우 다른 스레드들은 모든 메소드를 사용하지 못하고 Blocking 하게 된다. 이는 애플리케이션 성능 저하의 원인이 될 수 있다.

### Q. SynchronizedList와 CopyOnArrayList의 차이점은?

SynchronizedList는 읽기와 쓰기 동작시 인스턴스 자체에 잠금이 걸린다. 그러나 CopyOnArrayList는 쓰기 동작시 해당 블록에 잠금을 걸고 원본 배열에 있는 요소를 복사하여 새로운 임시 배열을 만들고, 이 임시 배열에 쓰기 동작을 수행한 후 원본 배열을 갱신한다. 이 덕분에 읽기 동작은 잠금이 걸리지 않아 SynchronizedList보다 읽기 성능이 좋다. 그러나 쓰기 동작은 비용이 높은 배열 복사 작업을 하기 때문에 SynchronizedList보다 쓰기 성능이 떨어진다.

따라서, 변경 작업보다 읽는 작업이 많을 경우 CopyOnArrayList를 사용하는 것이 더 효과적이다.

### Q. SynchronizedMap과 ConcurrentHashMap의 차이점은?

SynchronziedMap은 읽기와 쓰기 동작시 인스턴스 자체에 잠금이 걸린다. 그러나 ConcurrentHashMap은 각 테이블 버킷을 독립적으로 잠그는 방식을 사용한다. 만약, 빈 버킷에 노드를 삽입할 경우 잠금(Lock) 대신 CAS 알고리즘을 사용하고, 그 외의 변경은 접근한 버킷에만 잠금이 걸려 스레드 경합을 최소화하며 동시성을 보장해준다.
