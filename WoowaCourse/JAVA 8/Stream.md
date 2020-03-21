## Stream
### 스트림이란?

- (참고) InputStream / OutputStream과 다른 Stream이다.
- 주로 Collection, Arrays에 사용된다.
- 데이터 소스를 추상화 & 데이터를 다루는데 자주 사용되는 메서드 정의

    → **어떤 데이터 소스든지 같은 방식으로 처리**할 수 있기 때문에 👍**코드의 재사용성**

- 객체 집합(Collections).스트림 생성().중개연산().최종연산()

    names.stream().filter(x -> x.contains("o"));

- 자주 사용되는 중개연산

   ```
   // Filter: 요소들을 조건에 따라 걸러내는 작업
    names.stream().filter(x -> x.contains("o"));
    
    // Map: 요소들을 특정 조건에 해당하는 값으로 변환
    List<String> names = Arrays.asList("jeong", "pro", "jdk", "java");
    names.parallelStream().map((x) ->{return x.concat("s");}).forEach(x -> System.out.println(x));
    //jeongs, pros, jdks, javas
    
    // Sorted: 요소들을 정렬해주는 작업
	```

- 스트림은 1회용이다.

    한 번 탐색한 스트림은 다시 탐색할 수 없다. 다시 탐색하려면 새로운 스트림을 만들어야 한다.

- 스트림은 데이터 소스를 변경하지 않는다.

    Why? Collection과 같이 데이터를 다루는 것이 목적이 아니기 때문이다. 

    Collection은 데이터를 다루는 것이 목적이기 때문에, 데이터를 add, remove하지만

    스트림은 데이터를 한 꺼번에 담는 방식으로 진행한다.

- *명시적이고 선언적이다.*
- *또 하나의 장점은 간단하게 병렬처리(multi-threading)가 가능하다는 점입니다. 하나의 작업을 둘 이상의 작업으로 잘게 나눠서 동시에 진행하는 것을 병렬 처리(parallel processing)라고 합니다. 즉 쓰레드를 이용해 많은 요소들을 빠르게 처리할 수 있습니다.*
- **스트림은 작업을 내부 반복으로 처리한다.**

    *스트림을 이용한 작업이 간결할 수 있는 비결중의 하나가 바로 '내부 반복'이다. 내부 반복이라는 것은 반복문을 메서드의 내부에 숨길 수 있다는 것을 의미한다.*

### Stream과 Collection

- 공통점: 연속된 값 집합
- 차이점
    - Stream의 목적: 연산 / 자료구조를 다루는 역할
    - Collection의 목적: 데이터 / 자료구조의 구현체
	
	
### IntStream
* range(startNumber, endNumber)
  * startNumber ~ endNumber - 1
* rangeClosed(startNumber, endNumber)
  * startNumber ~ endNumber
* boxed()
  * int, long, double → Integer, Long, Double로 바꾸어 Stream 생성
  
  
 사용 예
 ```java
 Map<Integer, LottNumber> lottoNumbers = IntStream.rangeClosed(MIN_LOTTO_NUMBER, MAX_LOTTO_NUMBER)
                .boxed()
                .collect(Collectors.toMap(Function.identity(), LottoNumber::new));
				```