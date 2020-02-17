final

- 재할당 금지
- 불변을 보장하지는 않는다.

final 변수

- final 변수: 변경 불가
- final Object: 변경 불가
  * final Object의 속성: 변경 가능
  * final Collection의 속성: 변경 가능

```JAVA
        final int a = 1;
        // a = 2; [에러]
        
        final Person person = new Person();
        // person = new Person(); [에러]
        person.name = "John" // [에러X]
        
        final int[] arr = new int[] {1, 2, 3];
        arr[0] = 100; // [에러X] 
```

final 매소드

- 오버라이드 X

final 클래스

- 상속 X