## 클래스 로더란?

자바는 동적 로드, 즉 컴파일 타임이 아니라 런타임(바이트 코드를 실행시킬 때)에 클래스 로드하고 링크하는 특징이 있다. 이 동적 로드를 담당하는 부분이 JVM의 클래스 로더이다.

정리하자면, **클래스 로더는 런타임 중에 JVM의 Method 영역에 동적으로 Java 클래스를 로드하는 역할**을 한다.

클래스 로더에는 **로딩**, **링크**, **초기화** 단계로 나눠져 있는데, 간단히 설명하면 다음과 같다.

- **로드**
    - 자바 바이트 코드(.class)를 메소드 영역에 저장한다.
    - 각 자바 바이트 코드(.class)는 JVM에 의해 메소드 영역에 다음 정보들을 저장한다.
        - 로드된 클래스를 비롯한 그의 부모 클래스의 정보
        - 클래스 파일과 Class, Interface, Enum의 관련 여부
        - 변수나 메소드 등의 정보
- **링크**
    - 검증 : 읽어 들인 클래스가 자바 언어 명세 및 JVM 명세에 명시된 대로 잘 구성되어 있는지 검사한다.
    - 준비 : 클래스가 필요로 하는 메모리를 할당하고, 클래스에서 정의된 필드, 메서드, 인터페이스를 나타내는 데이터 구조를 준비한다.
    - 분석 : 심볼릭 메모리 레퍼런스를 메서드 영역에 있는 실제 레퍼런스로 교체한다.
- **초기화**
    - 클래스 변수들을 적절한 값으로 초기화 한다. 즉. static 필드들이 설정된 값으로 초기화한다.

## 클래스 로더의 종류

![image](https://user-images.githubusercontent.com/55661631/152149760-c73e2f19-7910-4a58-9705-c2fbfa70e6fa.png)

### **부트스트랩 클래스 로더(Bootstrap Class Loader)**

JVM 시작 시 가장 최초로 실행되는 클래스 로더이다. 부트스트랩 클래스 로더는 자바 클래스를 로드하는 것이 아닌, 자바 클래스를 로드할 수 있는 자바 자체의 클래스 로더와 최소한의 자바 클래스(java.lang.Object, Class, Classloader)만을 로드한다.


**Java 8**  
`jre/lib/rt.jar` 및 기타 핵심 라이브러리와 같은 JDK의 내부 클래스를 로드한다.

**Java 9 이후**  
더 이상 `/rt.jar`이 존재하지 않으며 `/lib` 내에 모듈화 되어 포함됐다. 이제는 정확하게 ClassLoader 내 최상위 클래스들만 로드한다.

### 확장 클래스 로더(Extension Class Loader)

확장 클래스 로더는 부트스트랩 클래스 로더를 부모로 갖는 클래스 로더로써, 확장 자바 클래스들을 로드한다. 

java.ext.dirs 환경 변수에 설정된 디렉터리의 클래스 파일을 로드하고, 이 값이 설정되어 있지 않은 경우 `${JAVA_HOME}/jre/lib/ext`에 있는 클래스 파일을 로드한다.


**Java 8**  
URLClassLoader를 상속하며,  `jre/lib/ext` 내 모든 클래스를 로드한다

**Java 9**  
Platfrom Loader로 변경되었으며, URLClassLoader가 아닌 BuiltinClassLoader를 상속한다. Inner Static 클래스로 구현되어 있다.


### 시스템 클래스 로더(System Class Loader)

자바 프로그램 실행 시 지정한 Classpath에 있는 클래스 파일 혹은 Jar에 속한 클래스들을 로드한다. 쉽게 말하자면, 저희가 만든 .class 확장자 파일을 로딩한다.

## 클래스 로더의 동작 방식

클래스 로더는 새로운 클래스를 로드해야 할 때, 다음과 같은 방식으로 로드를 수행한다.

1. JVM의 Method Area에 클래스가 로드되어 있는지 확인한다. 만일 로드되어 있는 경우 해당 클래스를 사용한다.
2.  Method Area에 클래스가 로드되어 있지 않을 경우, 시스템 클래스 로더에 클래스 로드를 요청한다.
3.  시스템 클래스 로더는 확장 클래스 로더에 요청을 위임한다.
4.  확장 클래스 로더는 부트스트랩 클래스 로더에 요청을 위임합니다.
5.  부트스트랩 클래스로더는 부트스트랩 Classpath(JDK/JRE/LIB)에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 확장 클래스로더에게 요청을 넘긴다.
6.  확장 클래스 로더는 확장 Classpath(JDK/JRE/LIB/EXT)에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 시스템 클래스 로더에게 요청을 넘긴다.
7.  시스템 애플리케이션 클래스로더는 시스템 Classpath에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 ClassNotFoundException을 발생시킨다.

## 클래스 로더가 지켜야할 세 가지 원칙

**위임 원칙(Delegation Principle)**

- 클래스 로더는 클래스 또는 리소스를 찾기 위해 요청을 받았을 때, 상위 클래스 로더에게 책임을 위임하는 위임 모델을 따른다.
- 위에서 설명한 클래스 로더 동작 방식을 보면 위임 법칙을 따른다는 것을 확인할 수 있다.

**가시범위 원칙(Visibility Principle)**

- 하위 클래스 로더는 상위 클래스 로더가 로드한 클래스를 볼 수 있지만, 반대로 상위 클래스 로더는 하위 클래스 로더가 로드한 클래스를 알 수 없다.
- 이로 인해 java.lang.Object 클래스 등 상위 클래스 로더에서 로드한 클래스도 하위 클래스 로더인 시스템 클래스 로더 등에서 사용할 수 있는 것이다.

**유일성의 원칙(Uniqueness Principle)**

- 하위 클래스 로더가 상위 클래스 로더에서 로드한 클래스를 다시 로드하지 않아야 한다는 원칙이다.
- 위임 원칙에 의해서 위쪽으로 책임을 위임하기 때문에 고유한 클래스를 보장할 수 있다.

## 동적 클래스 로딩(Dynamic Loading)

자바의 클래스 로딩은 **클래스 참조 시점에 JVM에 코드가 링크**되고 **실제 런타임 시점에 로딩**되는 동적 로딩을 거친다.
런타임에 동적으로 클래스를 로딩한다는 건 JVM이 미리 모든 클래스에 대한 정보를 Method 영역에 로딩하지 않다는 것을 의미한다.

동적으로 클래스를 로딩하는 방식은 두 가지가 있다.

### 로드타임 동적 로딩(Load-time Dynamic Loading)

```java
public class HelloWorld { 
		public static void main(String[] args) { 
				System.out.println("안녕하세요!"); 
		} 
}
```

위 코드의 경우 다음과 같이 동작한다.

1. JVM이 시작되고 부트스트랩 클래스 로더가 생성된 후에 모든 클래스가 상속받고 있는 Object 클래스를 읽어온다.
2. 클래스 로더는 명령행에서 지정한 HelloWorld 클래스를 로딩하기 위해, HelloWorld.class 파일을 읽는다.
3. HelloWorld 클래스를 로딩하는 과정에서 필요한 클래스인 `java.lang.String`과 `java.lang.System`을 로딩한다.

이처럼 **하나의 클래스를 로딩하는 과정에서 동적으로 다른 클래스를 로딩하는 것을 로드타임 동적 로딩**이라고 한다.

### 런타임 동적 로딩(Run-time Dynamic Loading)

```java
public class RuntimeLoading { 
		public static void main(String[] args) { 
				try { 
						Class cls = Class.forName(args[0]); 
						Object obj = cls.newInstance(); 
						Runnable r = (Runnable) obj; 
						r.run(); 
				} catch (Exception e) {
						e.printStackTrace();
				}
		}
}

```

위 코드에서 `Class.forName(className)`은 파라미터로 받은 className에 해당하는 클래스를 로딩한 후에, 객체를 반환하며, 다음과 같이 동작한다.

1. `Class.forName()` 메서드가 실행되기 전까지는 RuntimeLoading 클래스에서 어떤 클래스를 참조하는 지 알수 없다.
2. 따라서, RuntimeLoading 클래스를 로딩할 때는 어떤 클래스도 읽어오지 않고, RuntimeLoading 클래스의 `main()` 메소드가 실행되고 `Class.forName(args[0])`를 호출하는 순간에 비로소 args[0]에 해당하는 클래스를 로딩한다.

이처럼 클래스를 로딩할 때가 아닌 **코드를 실행하는 순간에 클래스를 로딩하는 것을 런타임 동적 로딩**이라고 한다.

## 클래스의 로딩, 초기화 시점 (feat. 싱글톤)

skyepodium님의 글을 읽고 결과만 작성했다. 자세한 내용은 [링크](https://velog.io/@skyepodium/%ED%81%B4%EB%9E%98%EC%8A%A4%EB%8A%94-%EC%96%B8%EC%A0%9C-%EB%A1%9C%EB%94%A9%EB%90%98%EA%B3%A0-%EC%B4%88%EA%B8%B0%ED%99%94%EB%90%98%EB%8A%94%EA%B0%80)를 참고하자.

### **클래스 로딩 시점**

- 클래스의 인스턴스 생성
- 클래스의 정적 변수 사용 (final 키워드 X)
- 클래스의 정적 메소드 호출

### **클래스가 로딩 X**

- 클래스에 접근하지 않을 때
- 클래스의 정적 변수 사용 (final 키워드 O)

### **초기화 시점**

- 클래스의 인스턴스 생성
- 클래스의 정적 메소드 호출
- 클래스의 정적 변수 사용 (final 키워드 X)

### **초기화 진행 순서**

1. 정적 블록
2. 정적 변수
3. 생성자

### **한번만 클래스가 로딩됨을 보장**

JLS(Java Language Specification)에 따르면 JVM에 클래스가 로딩되고 초기화될때는 순차적으로 동작함을 보장한다. 멀티 스레드 환경에서 여러 개의 스레드가 동시에 클래스를 로딩하려고 하면 오직 한 개의 클래스만 로딩된다.

### **싱글톤(Singleton)**

아래와 같이 **Initialization-on-demand holder idiom 방식(holder에 의한 초기화)**을 사용하면 클래스 로딩 및 초기화 과정이 스레드 세이프함을 이용해서 싱글톤 인스턴스를 생성할 수 있다.

**Holder에 의한 초기화 방식**

```java
public class HolderSingleton {

    private HolderSingleton() {
    }

    public static HolderSingleton getInstance() {
        return Holder.instance;
    }

    private static class Holder {
        public static final HolderSingleton instance = new HolderSingleton();
    }
}
```

클래스의 로딩, 초기화 시점을 알기 전까지 Holder에 의한 초기화 방식을 사용하지 않고 아래와 같이 `static final` 키워드를 사용하면 되지 않을까? 라는 의문이 들었다.

```java
public class Singleton {

    public static final Singleton instance = new Singleton ();

    private Singleton () {
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

그러나 이 방식은 해당 클래스가 로드될 때 사용하지 않는 싱글톤 변수가 로드되기 때문에 메모리가 낭비되는 방식이다. 
따라서 해당 클래스가 로드되어도 내부 클래스는 로드되지 않는다는 클래스 로더의 특성을 이용한 **Initialization-on-demand holder idiom 방식**을 사용하는 것이 메모리 낭비가 일어나지 않는다.

## 참고

- [https://velog.io/@skyepodium/클래스는-언제-로딩되고-초기화되는가](https://velog.io/@skyepodium/%ED%81%B4%EB%9E%98%EC%8A%A4%EB%8A%94-%EC%96%B8%EC%A0%9C-%EB%A1%9C%EB%94%A9%EB%90%98%EA%B3%A0-%EC%B4%88%EA%B8%B0%ED%99%94%EB%90%98%EB%8A%94%EA%B0%80)
- [https://limkydev.tistory.com/67](https://limkydev.tistory.com/67)
- [https://www.baeldung.com/java-classloaders](https://www.baeldung.com/java-classloaders)
- [https://engkimbs.tistory.com/606](https://engkimbs.tistory.com/606)

## 예상 면접 질문 및 답변

### Q. 클래스 로더란?

자바는 런타임에 클래스 로드하고 링크하는 특징이 있다. 클래스 로더는 런타임 중에 JVM의 Method 영역에 동적으로 Java 클래스를 로드하는 역할을 한다.

### Q. 클래스 로더의 동작 방식은?

1. JVM의 Method Area에 클래스가 로드되어 있는지 확인한다. 만일 로드되어 있는 경우 해당 클래스를 사용한다.
2.  Method Area에 클래스가 로드되어 있지 않을 경우, 시스템 클래스 로더에 클래스 로드를 요청한다.
3.  시스템 클래스 로더는 확장 클래스 로더에 요청을 위임한다.
4.  확장 클래스 로더는 부트스트랩 클래스 로더에 요청을 위임합니다.
5.  부트스트랩 클래스로더는 부트스트랩 Classpath(JDK/JRE/LIB)에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 확장 클래스로더에게 요청을 넘긴다.
6.  확장 클래스 로더는 확장 Classpath(JDK/JRE/LIB/EXT)에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 시스템 클래스 로더에게 요청을 넘긴다.
7.  시스템 애플리케이션 클래스로더는 시스템 Classpath에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 ClassNotFoundException을 발생시킨다.

### Q. 클래스 로더의 세 가지 원칙이란?

- 위임 원칙
- 가시범위 원칙
- 유일성의 원칙

### Q. 클래스 로더가 어떻게 클래스를 동적으로 로딩하는지?

하나의 클래스를 로딩하는 과정에서 동적으로 다른 클래스를 로딩하는 로드타임 동적 로딩 방식과 코드를 실행하는 순간에 클래스를 로딩하는 런타임 동적 로딩 방식이 있다.

### Q. 클래스의 로딩 시점

- 클래스의 인스턴스 생성
- 클래스의 정적 변수 사용 (final 키워드 X)
- 클래스의 정적 메소드 호출

### Q. 클래스의 초기화 시점

- 클래스의 인스턴스 생성
- 클래스의 정적 메소드 호출
- 클래스의 정적 변수 사용 (final 키워드 X)

### Q. 싱글톤 패턴과 클래스 로더는 어떤 연관이 있는지?

holder에 의한 초기화 방식을 사용하면 클래스 로딩 및 초기화 과정이 스레드 세이프함을 이용해 싱글톤 인스턴스를 생성할 수 있다.

자세한 내용은 본문을 참고하자. 코드와 함께 자세히 설명할 수 있어야 한다.
