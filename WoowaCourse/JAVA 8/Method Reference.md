하나의 메서드만 호출하는 람다식은 ‘메서드 레퍼런스’로 간단히 할 수 있다.

사용 예
```
// Before
(number) -> ReferenceToStaticMethodExample.isPrime((int) number)
// After
ReferenceToStaticMethodExample::isPrime 

```