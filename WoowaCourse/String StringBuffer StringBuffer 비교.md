### String / StringBuffer / StringBuilder 비교

#### String : immutable
객체를 생성한 후 변경할 수 없는 것, char 배열 + final로 값을 저장한다

#### String Buffer, String Builder : mutable
* String과 다르게 기존 데이터에 새로운 데이터를 더하는 방식을 취하기 때문에 속도가 더 빠르다.
* 메모리에 append하는 방식
* 따라서 긴 문자열을 더하는 상황이 발생하는 경우 StringBuffer/Builder를 활용해 구현한다

#### String Buffer  : Thread-Safe (synchronzied)
#### String Builder : Non-Thread-Safe
* 속도: String Buffer > String Builder
(String Buffer에서 Synchrozied만 제외한 것: String Builder)
* Multi Thread일 때 : String Buffer
* Single Thread일 때 : String Builder