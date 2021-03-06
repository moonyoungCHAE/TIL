### 계층형 구조

```
  └── src
        ├── main
        │   ├── java
        │   │   └── com
        │   │       └── example
        │   │           └── demo
        │   │               ├── DemoApplication.java
        │   │               ├── config
        │   │               ├── controller
        │   │               ├── dao
        │   │               ├── domain
        │   │               ├── exception
        │   │               └── service
        │   └── resources
        │       └── application.properties
  ```
  * 계층형 구조: 각 계층을 대표하는 디렉터리를 기준으로 구성
  * 👍 계층형 구조의 장점은 해당 프로젝트에 이해가 상대적으로 낮아도 전체적인 구조를 빠르게 파악할 수 있음 
  * 👎 디렉터리에 클래스들이 너무 많이 모이게 된다.

  ### 도메인형 구조
  
  ```
  └── src
        ├── main
        │   ├── java
        │   │   └── com
        │   │       └── example
        │   │           └── demo
        │   │               ├── DemoApplication.java
        │   │               ├── coupon
        │   │               │   ├── controller
        │   │               │   ├── domain
        │   │               │   ├── exception
        │   │               │   ├── repository
        │   │               │   └── service
        │   │               ├── member
        │   │               │   ├── controller
        │   │               │   ├── domain
        │   │               │   ├── exception
        │   │               │   ├── repository
        │   │               │   └── service
        │   │               └── order
        │   │                   ├── controller
        │   │                   ├── domain
        │   │                   ├── exception
        │   │                   ├── repository
        │   │                   └── service
        │   └── resources
        │       └── application.properties
  ```
        
  * 도메인형 구조: 도메인 디렉터리 기준으로 코드 구성 
  * 👍 관련된 코드들이 응집
  * 👎 프로젝트에 대한 이해도가 낮을 경우, 전체적인 구조를 파악하기 어려움
