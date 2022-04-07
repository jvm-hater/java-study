## 인자(Argument)

- 메소드 호출 시에 전달되는 값

## 매개변수(Parameter)

- 메소드에서 전달 받은 값

## Call by value

- 메소드를 호출할 때 넘겨주고 싶은 변수(인자)를 지정하면, 메소드의 매개변수가 지정한 **변수 값의 복사본**으로 초기화되는 것이다.
- 따라서 함수 내에서의 변경은 메소드를 호출할 때 지정한 변수(원본)에 아무 영향을 미치지 않는다.

## Call by reference

- 메소드를 호출할 때 넘겨주고 싶은 변수(인자)를 지정하면, 메소드의 매개변수가 지정한 **변수의 레퍼런스**로 초기화되는 것이다.
- 따라서 함수 내에서의 변경은 메소드를 호출할 때 지정한 변수(원본)에 영향을 미친다.

## Java는 Call by value 일까? 아니면 Call by reference일까?

Java에서 객체를 전달받고, 그 객체를 수정하면 원본도 같이 수정되니 이것이 Call by reference라고 생각할 수도 있다. 그러나 **Java는 Call by value**이다. 예시를 보며 그 이유를 살펴보자.

![image](https://user-images.githubusercontent.com/55661631/162172178-88976719-b2b3-4dcd-b091-6867e5ac9b41.png)

여기서 주의깊게 봐야 하는 부분은 a1의 value가 111로 변경된 것이다. arg1의 value를 변경하니 원본 a1의 값도 변경되었으니 이 것을 call by reference라고 헷갈릴 수도 있다. 그러나 **a1에서 arg1으로 매개변수를 넘기는 과정에서 직접적인 참조를 넘긴 게 아닌, 주소 값을 복사해서 넘기기 때문에 이는 call by value이다**. 복사된 주소 값으로 참조가 가능하니 **주소 값이 가리키는 객체의 내용** 변경되는 것이다.

좀 더 자세히 알아보기 위해 JVM의 메모리(런타임 데이터) 영역을 살펴보자.

1. `run()` 메소드 실행 전

![image](https://user-images.githubusercontent.com/55661631/162172247-16ffeee6-b68e-442d-b8db-6ac3f472d926.png)

1. `run()` 메소드 실행 후

![image](https://user-images.githubusercontent.com/55661631/162172282-cf22d0b4-6604-464e-8cc9-ffcd2a1e6cc0.png)

- arg1은 a1이 가지고 있는 주소값을 복사하여 독자적으로 가진다.
- arg2도 마찬가지로 a2가 가지고 있는 주소 값을 복사하여 독자적으로 가진다.
- 주소 값을 복사하여 가져 가는 call by value가 발생한다.

1. `run()` 메소드의 첫 번째 줄 실행

![image](https://user-images.githubusercontent.com/55661631/162172325-de1c4532-b6b5-41d1-9ef1-beec67ccf0c7.png)

- arg1을 통해 value의 값을 변경한다면 arg1이 가지고 있는 주소 값을 통해 객체의 값을 변경한다.

1. `run()` 메소드의 두 번째 줄 실행

![image](https://user-images.githubusercontent.com/55661631/162172392-b6206757-1c36-4e21-913a-207d3209d369.png)

- arg2가 arg1이 가진 주소 값을 복사하여 저장한다.
- a2와 arg2는 독립된 변수이기 때문에 a2는 변경되지 않는다.

## 예상 면접 질문 및 답변

### Q. 인자(Argument)란?

메소드 호출 시에 전달되는 값

### Q. 매개변수(Parameter)란?

메소드에서 전달 받은 값

### Q. Call by value와 Call by reference의 차이

Call by value는 메소드를 호출할 때 넘겨주고 싶은 변수(인자)를 지정하면, 메소드의 매개변수가 지정한 변수 값의 복사본으로 초기화되는 것이다.

Call by reference는 메소드를 호출할 때 넘겨주고 싶은 변수(인자)를 지정하면, 메소드의 매개변수가 지정한 변수의 레퍼런스로 초기화되는 것이다.

### Q. Java는 Call by reference일까 Call by value일까?

자바에서 매개변수를 넘기는 과정에 직접적인 참조를 넘긴 게 아닌, 주소 값을 복사해서 넘기기 때문에 이는 call by value이다.

## 참고

- [https://deveric.tistory.com/92](https://deveric.tistory.com/92)
- [https://jaimemin.tistory.com/1479](https://jaimemin.tistory.com/1479)
- [https://imasoftwareengineer.tistory.com/104](https://imasoftwareengineer.tistory.com/104)
