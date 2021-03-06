### 오류와 예외
오류 (Error)
* 시스템으로 정상적이지 않은 상황
* 애플리케이션 레벨이 아닌 더 낮은 시스템 레벨에서 발생
* 개발자가 예측하여 대응하기 어렵다.

예외 (Exception)
* 로직에서의 에러
* 개발자가 예측하고 대응할 수 있다.


### 구체적인 예외를 선언하는 것이 좋다.
  * 공통적인 상위 클래스 (ex. ```Exception```) 사용 지양
  * 예외적으로 메인 메서드에서는 괜찮다. 오직 JVM만이 호출할 수 있는 메서드이기 때문
### 예외를 제어 흐름용으로 사용하면 안 된다.

* [Effective JAVA 아이템 70]

  * 예외는 (그 이름이 말해주듯) 오직 예외 상황에서만 써야 한다. 절대로 일상적인 제어 흐름용으로 쓰면 안 된다.

  * 특정 상태에서만 호출 할 수 있는 '상태 의존적' 메서드를 제공하는 클래스는 '상태 검사' 메서드도 함께 제공해야 한다.

    ex. 만약 A 상황에서만 수행해야 하는 메서드

    * ~~A 상황이 아니라면 exception~~

    * A 상황일 때만 수행할 수 있도록 상태 검사 (O)

    * ```java
      while(it.hasNex()) {
      	it.next();
      }
      ```

### 멀티 캐치 시 사용된 예외들은 상속 관계에서 부모와 자식 관계에 있으면 안된다.
```JAVA
// before
try{

}catch (ArithmeticException | RuntimeException e) {

}

// after
try{

}catch (RuntimeException e) {

RuntimeException 하나로 하위 예외들을 모두 처리

}
```


### 검사 예외/비검사 예외
throwable

1. 검사 예외
2. 런타임 예외
3. 에러



#### 검사 예외 (Checked Exception)

* **호출하는 쪽이 복구할 것이라고 여겨지면 검사 예외를 사용한다.**
 * 프로그램에서 반드시 처리했으면 하는 예외
 *  예외 발생 시, 컴파일 에러를 내는 것을 강요한다.
 * 개발자는 반드시 검사예외를 잡아내야 한다.
 * ```비검사 예외 (ex. RuntimeException, Error class, 이를 상속하여 만든 예외 클래스) 이외의 클래스 ```

#### 비검사 예외 (Unchekced Exception)

* 런타임 예외, 에러 ```ex. RuntimeException, Error class, 이를 상속하여 만든 예외 클래스 ```
* 프로그램에서 잡을 필요가 없거나 통상적으로 잡지 말아야 한다.
* 복구가 불가능하거나 더 실행해봐야 득보다는 실이 많다.



* Runtime Exception: 프로그래밍 오류 

  대부분 전제조건 위반 (ex. Index 범위 초과)

* 개발자가 만든 논리 오류
* 해당 예외가 발생한다고 해서, 컴파일 에러는 내지 않는다.
* 다소 타협적인 예외



* 에러: 보통 JVM이 자원 부족, 불변식 깨짐 등 더 이상 수행을 계속할 수 없는 상황을 나타낼 때 사용한다. 
  * AssertionError를 제외하고



### Custom Exception을 사용할 때

1. 예외 상황과 관련된 정보를 exception에 포함시켜야 하는 경우
2. 시스템 내 공통된 예외 처리를 하려는 경우
3. 외부 시스템으로 예외를 발생시키는 경우