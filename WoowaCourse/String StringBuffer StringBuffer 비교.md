### String / StringBuffer / StringBuilder 비교

#### String : immutable

* 객체를 생성한 후 변경할 수 없다.
  * char 배열 + final로 값을 저장한다.
  * JAVA Heap 영역에
* JDK1.5이상부터 String에서 +연산으로 작성하더라도 StringBuilder로 컴파일하게 만들어 놨다지만 여전히 String클래스의 객체 생성하는 부분을 동일하므로 StringBuffer, StringBuilder 사용이 필요하다.  
https://jeong-pro.tistory.com/85

#### String Buffer, String Builder : mutable

* String과 다르게 기존 데이터에 새로운 데이터를 더하는 방식을 취하기 때문에 속도가 더 빠르다.
* 메모리에 append하는 방식
* 따라서 긴 문자열을 더하는 상황이 발생하는 경우 StringBuffer/Builder를 활용해 구현한다

#### String Buffer  : Thread-Safe (synchronzied)

#### String Builder : Non-Thread-Safe

* 속도: String Builder > String Buffer  

  * String Buffer = String Builder + synchronized

  * 그래서 String Builder가 더 빠르다

    

* Multi Thread 환경일 때 : String Buffer

* Single Thread 환경일 때 + Multi Thread여도 동기화가 필요없는 경우 : String Builder
