Comparable: 기본적으로 적용되는 정렬 기준

- JAVA에서 제공하는 정렬이 가능한 클래스는 모두 Comparable 인터페이스를 구현하고 있다.

- ```JAVA
  a.compareTo(b)
  ```

  - a < b : 음수
  - a == b : 0
  - a > b : 양수
  - 관습적으로 -1, 0, 1을 쓰지만 규칙은 아니다.
    - 단, a - b의 차이가 해당 data type의 범위 안에 드는지 확인할 것
  - `compareTo` / `isBiggerThan isSmallerThan` 동시에 구현하는 것도 방법이다.

  

Comparator: 기본 정렬과 다르게 정렬하고 싶을 때

- 주로 Comparable : 오름차순
- Comparator : 내림차순
- `compare(a, b)`
  - a < b : 양수
  - a == b : 0
  - a > b : 음수