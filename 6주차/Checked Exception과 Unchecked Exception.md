## 에러(Error)와 예외(Exception)

Checked Exception과 Unchecked Exception의 차이를 알아보기 전에 먼저 예외와 에러의 차이를 알아야 한다.

프로그래밍에서 **예외(Exception)**란 입력 값에 대한 처리가 불가능하거나, 프로그램 실행 중에 참조된 값이 잘못된 경우 등 정상적인 프로그램의 흐름을 어긋나는 것을 말한다. 그리고 자바에서 예외는 개발자가 직접 처리할 수 있기 때문에 예외 상황을 미리 예측하여 핸들링할 수 있다.

그러나, **에러(Error)**는 시스템에 무엇인가 비정상적인 상황이 발생한 경우에 사용된다. 주로 자바 가상머신에서 발생시키는 것이며 예외와 반대로 이를 애플리케이션 코드에서 잡을 수 없다. 에러의 예로는 OutOfMemoryError, ThreadDeath, StackOverflowError 등이 있다.

## 예외 구분

예외(Exception)은 아래 그림과 같이 Checked Exception과 Unchecked Exception으로 구분할 수 있다. 쉽게 말하자면, RuntimeException을 상속하지 않은 클래스는 Checked Exception, 반대로 상속한 클래스는 UnChecked Exception으로 분류할 수 있다.

![image](https://user-images.githubusercontent.com/55661631/149926488-51aed824-de2a-43bb-bc0f-89f8ae454ed7.png)

### UnChecked Exception

명시적인 예외 처리를 강제하지 않기 때문에 UnChecked Exception이라 한다. 명시적인 예외 처리란 try-catch로 예외를 잡거나 throw로 호출한 메소드에게 예외를 던지지 않아도 됨을 뜻한다.

### Checked Exception

명시적인 예외 처리를 강제하기 때문에 Checked Exception이라 한다. 반드시 try-catch로 예외를 잡거나 throw로 호출한 메소드에게 예외를 던져야 한다.

### 정리

아래의 표는 Checked Exception과 UnChecked Exception의 차이를 표로 정리한 부분이다.

![image](https://user-images.githubusercontent.com/55661631/149926514-28025376-eccd-40cb-b018-35a922d50656.png)

<aside>
**💡 Checked Exception이 Rollback되지 않는 이유**
  
기본적으로 Checked Exception는 복구가 가능하다는 메커니즘을 가지고 있다. 예를 들어, 특정 이미지 파일을 찾아서 전송해주는 함수에서 이미지를 찾지 못했을 경우 기본 이미지를 전송한다는 복구 전략을 가질 수 있다. 
정리하자면, 복구가 가능하니 Rollback은 진행하지 않는다는 의미라고 생각한다.

</aside>

## 예외(Checked Exception) 처리

예외를 처리하는 방법에는 **예외 복구**, **예외 처리 회피**, **예외 전환**이 있다.

### 예외 복구

예외 상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 방법이다.

```java
public sendFile(String fileName)() {
		File file;
		try {
				file = FileFindService.find(fileName);
		} catch (FileNotFoundException e){ 
				// 기본 파일을 찾아서 전송한다.
				file = FileFindService.find("default.png");
		}

		send(file);
		}
}
```

대부분의 상황에서 예외를 복구할 수 있는 경우는 거의 없기 때문에 자주 사용되지 않는다.

예를 들어, 유니크해야 하는 이메일 값이 중복돼서 SQLException이 발생할 경우 복구할 수 있는 방법은 없다. 이런 경우에는 RuntimeException을 발생시키고 입력을 다시 유도하는 것이 현실적인 방법이다.

그러나 예외 복구를 사용할 일이 생긴다면 위처럼 예외를 복구하는 방식보다는 아래처럼 코드의 흐름으로 제어하는 것이 좋다.

```java
public void sendFile(String fileName){
		if(FileFindService.existed(filename)){
				// 파일이 있는 경우 해당 파일을 찾아서 전송한다.
				send(FileFindService.find(fileName));    
		}else{
				// 파일이 없는 경우 기본 이미지를 전송한다.
				send(FileFindService.find("default.png"));    
		}
}
```

### 예외 처리 회피

예외 처리를 직접 담당하지 않고 호출한 쪽으로 던져 회피하는 방법이다. 긴밀하기 역할을 분담하고 있는 관계가 아닐 경우 예외를 던지는 것은 무책임한 방법이다.

```java
public Object someMethod() throws IOException {
		...
}
```

### 예외 전환

예외 처리 회피와 비슷하게 메서드 밖으로 예외를 던지지만, 그냥 던지지 않고 적절한 예외로 전환해서 넘기는 방법이다. 명확한 의미로 전달되기 위해 적합한 의미를 가진 예외로 변경해야 한다.

```java
public Object method() {
		try {
				...
		} catch (IOException e) {
				throw new CustomException ("IOException 발생");
		}
}

public class CustomException extends RuntimeException{
    public CustomException(String message) {
        super(message);
    }
}
```

보통 예외 처리를 하기 위해서 위와 같이 RuntimeException을 상속받은 클래스(UnChecked Exception)로 전환해 던진다.

## 정리

- 예외 복구 전략이 명확하고 복구가 가능하면 Checked Exception을 try-catch로 잡아서 예외 복구를 하거나, 코드의 흐름으로 제어하는 것이 좋다.
- 그러나 이러한 경우는 흔하지 않기 때문에 Checked Exception이 발생하면 더 구체적인 UnChecked Exception을 발생시키고 예외에 대한 메시지를 명확하게 전달하는 것이 효과적이다.
- 무책임하게 상위 메서드에 throw로 예외를 던지는 행위를 하지 않는 것이 좋다. 상위 메서드들의 책임이 그만큼 증가하기 때문이다.
- Checked Exception은 기본 트랜잭션에 속성에서 Rollback을 진행하지 않는다.

## 참고

- [https://madplay.github.io/post/java-checked-unchecked-exceptions](https://madplay.github.io/post/java-checked-unchecked-exceptions)
- [https://cheese10yun.github.io/checked-exception/](https://cheese10yun.github.io/checked-exception/)
- [https://programmer93.tistory.com/71](https://programmer93.tistory.com/71)

## 예상 면접 질문 및 답변

### Q. 에러(Error)란?

예외(Exception)란 입력 값에 대한 처리가 불가능하거나, 프로그램 실행 중에 참조된 값이 잘못된 경우 등 정상적인 프로그램의 흐름을 어긋나는 것을 말한다. 자바에서 예외는 개발자가 직접 처리할 수 있기 때문에 예외 상황을 미리 예측하여 핸들링할 수 있다.

### Q. 예외(Exception)란?

**에러(Error)**는 시스템에 무엇인가 비정상적인 상황이 발생한 경우에 사용된다. 주로 자바 가상머신에서 발생시키는 것이며 예외와 반대로 이를 애플리케이션 코드에서 잡을 수 없다.

### Q. Checked Exception에 대해 설명하라

Checked Exception은 RuntimeException을 상속하지 않은 클래스이며, 명시적인 예외 처리를 해야 한다. 컴파일 시점에 확인할 수 있고 트랜잭션 안에서 동작할 때 Checked Exception이 발생하면 롤백되지 않는다는 특징이 있다.

### Q. UnChecked Exception에 대해 설명하라

UnChecked Exception은 RuntimeException을 상속한 클래스이며, 명시적인 예외 처리를 하지 않는다. 런타임 시점에 확인할 수 있고 트랜잭션 안에서 동작할 때 UnChecked Exception이 발생하면 롤백된다는 특징이 있다.

### Q. 예외(Checked Exception)를 처리하는 방식에 대해 설명하라

예외 상황을 파악하고 문제를 해결해서 정상 상태로 복구하는 예외 복구 방식, 예외 처리를 직접 담당하지 않고 호출한 쪽으로 넘기는 예외 처리 회피, 적절한 예외로 전환해서 넘기는 예외 전환 방식이 있다.

### Q. 올바른 예외 처리 방식

예외 복구 전략이 명확하고 복구가 가능하다면 Checked Excetpion을 try-catch로 잡아서 예외를 복구하는 것이 좋다. 복구가 불가능한 Checked Exception이 발생하면 더 구체적인 UnChecked Exception을 발생시키고 예외에 대한 메시지를 명확하게 전달하는 것이 좋다. 무책임하게 상위 메서드에 throw로 예외를 던지는 것은 상위 메서드의 책임이 증가하기 때문에 좋지 않은 방법이다.
