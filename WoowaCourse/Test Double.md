[읽어보기](http://haah.kr/2019/04/21/tdd-beginners-2/)



### 테스트 더블

* 실제 객체를 대신해서 테스트

* 영화 촬영 시, 위험한 역할을 스턴트 맨이 하는 것 - 스턴트 더블에서 유래

  테스트 시, 실제 객체 역할을 대신 (대역) 하는 것 - 테스트 더블

* 단위 테스트는 이상적으로 테스트 대상이 의존하는 것에 대해 독립적으로 작성되어야 한다.

  > Independent(독립적)  
  테스트는 깔끔함과 단정함을 유지해야 한다. 즉, 확실히 한 대강에 집중한 상태여야 하며, 환경과 다른 개발자들(명심하라. 다른 개발자들이 동시에 같은 테스트를 실행해 볼 수도 있다)에게서 독립적인 상태를 유지해야 한다.
  또한 독립적이라는 것은 어떤 테스트도 다른 테스트에 의존하지 않는다는 것을 의미한다. 어느 순서로든, 어떤 개별 테스트라도 실행해 볼 수 있어야 한다. 처음 것을 실행할 때 그 밖의 다른 테스트에 의존해야 하는 상황을 원하지는 않을 것이다.
  모든 테스트는 섬이어야 한다.

  *출처 :* [*실용주의 프로그래머를 위한 단위 테스트 with JUnit*](http://www.yes24.com/24/goods/1428559?scode=032&OzSrank=6)



* SUT (System Under Test) = 주요 객체 (primary object)

* 협력객체 (Collaborator) = 부차적 객체 (secondary object)

  
---
이점

* 예측 불가능한 실행 요소 제거
* 테스트 속도 개선

---

테스트 더블의 역할을 딱딱 분리하기는 어렵다. 서로가 서로의 특성을 조금씩 포함하기 때문에.

#### 테스트 더블의 종류 1. Dummy

* 해당 객체의 기능은 수행하지 않는다.
* 해당 dummy 객체의 매서드가 호출되었을 때 정상 동작을 보장하지 않는다.
* 객체는 전달되지만 사용되지 않는 객체

#### 테스트 더블의 종류 2. Stub

* 최소한의 구현 (하드 코딩된 값 반환)

  * dummpy 객체가 실제로 동작하는 것처럼 보이게 만든 객체

  * canned answer: 정해진 질문/상황에 대해 사전에 준비한 답

    > *Test stubs provide* [canned answers](https://en.wikipedia.org/wiki/Canned_response) *to calls made during the test, usually not responding at all to anything outside what’s programmed in for the test.*

* 주로 External Service를 사용하는 Code를 사용할 때 사용

  * External Service: database, web service, 외부 api etc

  * 테스트 대상: 

    * 비지니스 로직
    * ~~database와 잘 연결되었는지~~
    * ~~web service를 제대로 call하는지~~

  * 따라서 외부 서비스와 연동하는 부분은 하드 코딩으로 구현

  * 예시
    ![image](https://user-images.githubusercontent.com/37133536/77229936-10f5bb80-6bd4-11ea-9a43-9a71a4c527e3.png)
    * 아마존과 잘 연결되었다고 가정한 후 Stub에 하드코딩

      

#### 테스트 더블의 종류 3. Fake

* logic에 따라 하드코딩된 값 반환

#### 테스트 더블의 종류 4. Spy

* 어떤 메서드/객체가 호출되었는지 감시(기록)

  ex. 메일링 서비스에서 발송된 메일의 개수를 기록

#### 테스트 더블의 종류 5. Mock

* 호출에 대한 기대를 명세하고, 해당 내용에 따라 동작하도록 프로그래밍된 객체
* 행위 검증에 사용


---
### 상태 검증 vs 행위 검증

상태 기반 테스트, 행위 기반 테스트



1. 상태 검증
   * 메서드가 수행된 후 '상태'를 살펴봄으로써 올바르게 동작했는지 판단

```JAVA
SomeClass someClass = new SomeClass();
someClass.someMethod();

assertThat(someMethod.someStatus()).isEqualTo(true);
```



2. 행위 검증
   * 특정 메서드가 호출되었는지 행위를 검사하여, 올바르게 동작했는지 판단

```JAVA
SomeClass someClass = new SomeClass();

verify(someClass).someMethod();
```

