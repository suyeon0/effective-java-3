`제네릭 (5장)`

# [ITEM30] 이왕이면 제네릭 메서드로 만들라

## 1. 제네릭 메소드란
___
` 메소드의 선언부에 적은 제네릭으로 리턴 타입, 파라미터 타입이 정해지는 메소드 `
![img.png](img.png)

## 2. static 과 제네릭 메소드
### static 변수는 제네릭을 사용할 수 없다.
```java
public class Student<T> {
    static T name;
}
```
* static 변수는 클래스가 인스턴스가 되기 전에 메모리에 올라가는데, 이 때 name 의 타입인 T 가 결정되지 않기 때문에 위 코드처럼 사용 불가.

###
### 제네릭 메소드는 static 이 가능하다.
`이유` 제네릭 메소드는 호출 시에 매개 타입을 지정하기 때문에 static 이 안될 이유가 없다.
```java
public class Student<T> {
    
    static <T> T getOneStudent(T id) {
        return id;
    }
}
```
* 주의사항
  * Student 클래스에 지정한 제네릭 타입 `<T>` 와 제네릭 메소드에 붙은 `<T>` 는 같은 T 를 사용하더라도 **전혀 별개** 라는 것이다.(클래스에 지정된 타입 파라미터와 제네릭 메서드에 정의된 타입 파라미터는 전혀 상관이 없기 때문에, 일반 클래스에서도 제네릭 메소드를 사용할 수 있다)
  * 클래스에 표시하는 `<T>` 는 **인스턴스 변수** 라고 생각하자. 인스턴스가 생성될 때마다 지정되기 때문이다.
  * 제네릭 메소드에 붙은 T 는 **지역변수** 를 선언한 것과 같다고 생각하자. **(메소드의 붙은 모든 T는 클래스에 붙은 T와 다르다)**
  * 그리고 혼동 주지 않도록 다른 타입의 이름으로 선언하자 !
###
### 제네릭 메소드를 사용하는 이유가 뭘까

```java
private static List<String> add(List<String> list, String element) {
    list.add(element);
    return list;
}

private static List<Integer> add(List<Integer> list, Integer element) {
    list.add(element);
    return list;
}

// 이렇게 오버로딩 계속 해야 하므로
private static <T> List<T> add(List<T> list, T element) {
    list.add(element);
    return list;
}
```

### 
* [제네릭 메소드란?](https://devlog-wjdrbs96.tistory.com/201)
* [사용이유 참고 예제](https://okky.kr/article/474500)
###




