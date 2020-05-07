- persisent data : db에 저장된 데이터

### 등장배경

JPA : 너무 많은 일을 한다. → 개발자들 혼란 → Spring JDBC 등장

JPA의 복잡한 단점을 개선하기 위한 Spring JDBC의 특징

1. entity를 load하면 바로 Database에 반영된다. (↔ JPA는 그렇지 않다.)
2. entity를 table로 mapping해주는 model이 있다. 이 모델을 따라야 한다.

### 작동 방식

1. domain object를 생성한다.
2. table의 data와 object를 mapping한다.
3. Object 생성

- object를 생성하면 Spring Data JDBC는 자동적으로 그 object에 맞는 table을 찾는다.
- object 생성할 때
  - if 기본 생성자가 존재 : 그 생성자 사용
  - else if arg가 있는 생성자 1 : 그 생성자 사용
  - else if arg가 있는 생성자 多 : persistenceContructor anntation있는 생성자 사용
- Reflection : 객체를 통해 클래스 정보를 분석하는 것
  - (예) 클래스의 생성자, 멤버 변수, 클래스 변수 정보를 얻을 수 있다.
  - 자바 lib로 구현되어있다.
- 

------

[Introduction to Spring Data JDBC](https://lumberjackdev.com/spring-data-jdbc)

### One to One (1 : 1)

```java
/*
Address Table - id
*/
class Address {
	Long id;
}

/*
UserAccount Table - address_id references address(id)
*/
class UserAccount {
	// 참고하고자 하는 객체 table의 column 넣는다.
	@MappedCollection(idColumn = "id")
	private Address address;
}
```

Save하는 객체가 참조하는 모든 객체 한번에 저장 (참조하는 객체 중 저장된 게 있다면 삭제 후 다시 저장)

- Address 저장 → UserAccount에 저장된 Address 추가해서 저장 (X)
- UserAccount에 Address 추가해서 저장(O)

### One to Many (1:多)

```java
/*
Article Table - Comment에 대한 정보 없다.
*/
class Article {
	Set<Comment> commenets;
}

/*
Comment Table - Long article : Article의 주소를 가지고 있다.
*/
class Comment {
	// 객체에는 Article에 대한 정보 없다.
}
```

### Many to One, Many to Many

최대한 지양하는 것이 좋다.

- Spring Data JDBC를 사용할 때 지양하는 것이 좋다.

- Spring Data JDBC에서는 참조하는 객체를 삭제 후 다시 저장하는 데,

  많은 객체가 하나의 객체를 참조하면 의도치 않는 삭제가 발생할 수 있다.
