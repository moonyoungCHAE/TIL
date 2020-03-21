### Mock Object

테스트하고자 하는 부분이 다른 클래스에 많이 의존하는 경우, 단위 테스트하기 어렵다.

Mock

- 실제 객체를 만들어 사용하기에 시간, 비용 등의 Cost가 높거나

  혹은 객체 서로간의 의존성이 강해 구현하기 힘들 경우 가짜 객체를 만들어 사용하는 방법

- ex 특정 함수에 대한 return 값을 지정해준다.

```java
// chaining : 3가지 결과를 임의로 생성할 수 있다. 
when(mock.someMethod(anyString())) 
    .thenReturn("foo") 
    .thenReturn("bar") 
    .thenThrow(new RuntimeException());

// multi arguments
when(mock.someMethod(anyString())) 
    .thenReturn("one", "two");
```



---



Issue :  mock을 적용할 때는 함수의 호출관계를 고려하자.

```
function A() {
	B();
}

function B() {
}
```

- 다음과 같은 상황에서 function B가 의도와 다르게 작동했다.
- 그래서 Test Code에서 function B를 mock쳤다.
- 그런데 실제 프로덕션 코드의 function A에서 부르는 function B는 Test Code에서 mock친 것 때문에 아예 호출도 안하고, 의도한 결과도 안나옴