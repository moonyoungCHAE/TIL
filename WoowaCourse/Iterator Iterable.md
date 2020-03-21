<h4> Iterator 정의 </h4>

- 자바의 컬렉션 프레임웍에서 **컬렉션에 저장되어 있는 요소들을 읽어오는 방법을 표준화** 하였는데 그 중 하나가 Iterator이다.

- 어떤 컬렉션이라도 동일한 방식으로 접근이 가능하여 그 안에 있는 항목들에 접근할 수 있는 방법을 제공 (다형성)

- Collection 인터페이스에서는 Iterator 인터페이스를 구현한 클래스의 인스턴스를 반환하는 ```iterator()``` 메소드를 정의하여 각 요소에 접근하도록 하고 있다.

- Interface

  - Collection에서 제공하는 iterator() 메소드를 통해서만 Iterator 객체를 전달 받을 수 있다.
  - 이 때, Iterator 객체는 컬렉션 객체의 기능을 버리고 **반복을 위한 기능만 남겨놓은 새로운 객체 **

  

---

<h4> Iterator Interface의 선언과 구현 </h4>

[Iterator 구현](https://m.blog.naver.com/PostView.nhn?blogId=writer0713&logNo=220877874725&proxyReferer=https%3A%2F%2Fwww.google.com%2F)

```
public interface Iterator<E> { 

​	boolean hasNext(); 

​	E next(); 

​	default void remove() { 

​		throw new UnsupportedOperationException("remove"); 

​	}

​	default void forEachRemaining (Consumer<? super E> action) {

​		Objects.requireNonNull(action);

​		while (hasNext())

​			action.accept(next());

​	} 

}
```

![Untitled (3)](https://user-images.githubusercontent.com/37133536/77233176-1ad5e980-6be9-11ea-93f6-53c77b06b9cc.png)


* Iterable, Collection, List 모두 `iterator()` 함수를 선언

* 실제 `iterator()` 구현 부: ArrayList와 같은 구현체

----



<h4> Iterator 작동 방식 </h4>

![Untitled (1)](https://user-images.githubusercontent.com/37133536/77232974-aea6b600-6be7-11ea-8de0-f7f3728581f4.png)

- Iterator: Collection 객체와 구별되는 새로운 객체

![Untitled](https://user-images.githubusercontent.com/37133536/77232959-95056e80-6be7-11ea-87af-2bd6590464d7.png)

![Untitled (2)](https://user-images.githubusercontent.com/37133536/77232972-acdcf280-6be7-11ea-90fe-0d099c4e3702.png)

- List가 Element 자체에 접근한다면,

  Iterator는 Element 사이 사이에 접근한다.

- List get(index)로 접근시, index 0 부터 해당 index를 검색한다. → Iterator 사용 시, 성능 개선 可

---

### List Iterator

- Iterator 인터페이스를 상속받아 여러 기능을 추가한 인터페이스

- 양방향 이동 가능 (↔Iterator: 순방향으로만 이동)

  ```
    hasPrevious();
    /*
    이 리스트 반복자가 해당 리스트를 역방향으로 순회할 때
    다음 요소를 가지고 있으면 true를 반환하고,
    더 이상 다음 요소를 가지고 있지 않으면 false를 반환함.
    */
    
    previous();
    /*
    리스트의 이전 요소를 반환하고, 
    커서(cursor)의 위치를 역방향으로 이동시킴.
    */
    
    
    previousIndex();
    /*
    리스트의 이전 요소를 반환하고,
    커서(cursor)의 위치를 역방향으로 이동시킴.
    */
  ```

----

### Enumeration

- Iterator 인터페이스와 같은 동작을 하는 인터페이스

- 현재에는 기존 코드와의 호환성을 위해서만 남아있으므로, **Enumeration 인터페이스보다는 Iterator 인터페이스를 사용하는 것이 좋다**.

- Iterator와의 차이

  - Iterators allow the caller to remove elements from the underlying collection during the iteration with well-defined semantics.

  - Method names have been improved.

  - Enumeration thread-safe한 클래스에서 제공

    Iterator thread-safe하지 않은 클래스에서 제공

  Enumeration<Double> doubleE = doubleVector.elements(); while(doubleE.hasMoreElements()) { doubleE.nextElement(); }

- JDK 1.5부터 추가된 Enhanced for 문을 사용하도록 권장하고 있다.

  // Enhanced for문 for (String name : names) { // do something }

------

### Iterable

[iterator와 iterable의 차이](https://92bluemoon.netlify.com/posts/iterator-iterable/)

- Iterable: Iterator를 사용하는 메서드를 제공하는 인터페이스

  - (대표적인 예) **for-each** 같은 경우 내부적으로 iterator가 작동한다.
  - Iterable을 implement하는 클래스는
    1. 내부적으로 iterator를 재정의 (오버라이드) 해야 한다.
    2. 재정의한 iterator를 사용해서 Iterable의 메서드를 제공한다.
  - Iterator가 나온 후, Iterable이 나왔음.

  public interface Iterable<T> { Iterator<T> iterator();

  ```
   default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
  
   default Spliterator<T> spliterator() {
       return Spliterators.spliteratorUnknownSize(iterator(), 0);
   }
  ```

  }

Iterator와 Iterable의 차이

1. Iterable은 Iterator를 사용하는 인터페이스

   - Iterable을 구현하려면 Iterator가 필요하다.

2. Iterator는 iteration(위치 정보) 可

   Iterable은 위치 정보를 가질 수 없음

   - 때문에, Iterable은 메서드가 호출 될 때 마다 새로운 Iterator를 생성해야 한다.

     Iteration 상태를 유지해야 하기 때문에 같은 iterator를 두 번 사용하면 안된다.
