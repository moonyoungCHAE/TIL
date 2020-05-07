> [Effective JAVA 아이템8] C++ 프로그래머라면 다음의 내용에 주의하자. 자바의 finalizer와 cleaner는 C++의 파괴자(destructor)와 다른 개념이다. C++에서의 파괴자는 특정 객체와 관련된 자원을 회수하는 보편적인 방법이다. 자바에서는 접근할 수 없게 된 객체를 회수하는 역할을 가비지 컬렉터가 담당하고, 프로그래머에게는 아무런 작업도 요구하지 않는다. C++의 파괴자는 비메모리 자원을 회수하는 용도로도 쓰인다. 하지만 자바에서는 try-with-resources와 try-finally를 사용해 해결한다.

### try-with-resources

- `try - with - resources` 구문을 사용하게 되면 입출력 처리시 **예외가 발생하는 경우 JVM이 자동으로 `close()`를 호출하여 자원을 반납**시켜줍니다.

- try() 안에 입출력 스트림을 생성하는 로직을 작성하는데 이때 해당 객체는 **AutoCloseable** 인터페이스를 구현한 객체여야 합니다.

  - JDBC - Statement, PreparedStatement : AutoClosable
  - 우리가 자주 사용하는 라이브러리에 있는 자원 클래스들은 대부분 AutoCloseable 인터페이스를 이미 구현하고 있다.
  - try with resource에서 AutoCloseable의 close 메서드를 호출하기 때문이다.

- 자동 close() 도중에 예외가 발생했다면 발생한 예외를 입출력 처리시 발생한 예외에 담아 던져지게 됩니다

- 자바 9 前 :  try 안에서 생성해야 close 잡을 수 있다.

  자바 9 ~ : try 밖에서 생성하고 try 안에서 호출만 해도 close 잡아준다.

  public class Class1 { public method1() throws Exception { Connection conn = DriverManager.getConnection("..."); Statement stat = conn.createStatement(); ResultSet rs = stat.executeQuery("SELECT 1 from dual")

  ```
      try (conn; stat; rs) {
          // 무조건 try 안에서 호출은 해야 AutoClosable의 close를 호출한다. 
      } catch (Exception e) {
          // ...
      } 
    }
  ```

  }

[JDBC 의 Connection, Statement, ResultSet close 잘 하기 | Hyoj blog](https://hyoj.github.io/blog/java/basic/jdbc/jdbc-try-catch-tip.html#connection-preparedstatement-resultset-닫는-가장-이상적인-방식)

https://dololak.tistory.com/67

[Java 7 - AutoCloseable 살펴보기 | Hyoj blog](https://hyoj.github.io/blog/java/basic/java7-autocloseable.html#method-summary)