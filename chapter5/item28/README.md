`제네릭 (5장)`

# [ITEM28] 배열보다는 리스트를 사용하라

## 1. 배열은 공변 / 리스트는 불공변 ?
___
* **공변** : 자기 자신과 자식 객체로 타입 변환을 허용해주는 것
```java
/*
   배열은 공변이라서 Long 이 Object 하위 타입으로 인식되어 컴파일 오류 발생 X 
   but, 런타임 오류 발생 
 */
    Object[] objectArray = new Long[1];
    objectArray[0] = "String !"; // 컴파일러에게 String 은 Object 하위니까
```

* **불공변** : `List<String>`과 `List<Object>`가 있을 때 두 개의 타입은 전혀 관련이 없다는 뜻

``` java
public class Test {
    public static void test(List<Object> list) {}
    
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("suyeon");
        test(list);   // 컴파일 에러
    } 
}
```

## 1-1. 왜 배열을 공변으로 만들었을까.
```text
: 배열을 불공변으로 만들면 '다형성' 의 이점을 살릴 수 없기 때문 !

* 초기의 자바에서는 제네릭의 개념이 없었는데, 타입 오류의 위험성이 발생하더라도 다형성으로 얻을 수 있는
 이점이 더 많다고 생각했기 때문이었을 것이다.
 이후 자바에서 제네릭(jdk 1.5)이 도입되고, 와일드카드 등의 도구를 통해 다형성까지 챙길 수 있게 되었으므로
 ex. Arrays 클래스의 swap 메소드 참고
 
* 배열보다는 컴파일 시점에서 더 안전한 리스트를 사용하는 것이 옳은 코드 !
```
https://hwan33.tistory.com/24
___
## 2. `List<String>` 는 왜 실체화 불가 타입이지?
> 166p. E, `List<E>`,  `List<String>` 같은 타입을 실체화 불가 타입이라고 한다.
> 쉽게 말해, 실체화되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입이다.
___

## 3. 제네릭 타입 소거

* 타입을 컴파일 타임에서만 검사하고, `런타임에는 해당 타입 정보를 알 수 없는 것(= 타입에 대한 정보 소거)` 이다.
* 즉, 컴파일이 된 파일(.class) 에는 제네릭 타입에 대한 정보가 전혀 없다
* 목적 :  `하위 호환성` 때문이다. 
  * 제네릭이 도입된 JDK 5 이전의 코드를 깨뜨리지 않고 제네릭을 지원하기 위해 로 타입(Raw Type - 타입 파라미터를 사용하지 않는 제네릭 클래스) + 타입 소거 방법을 도입했다.
####
* Java 컴파일러가 타입소거를  적용하는 방법
  * unbounded Type`(<?>, <T>)`는 Object로 변환한다.
  * bound type(`<E extends Comparable>`)의 경우는 Object가 아닌 Comprarable로 변환한다.(한정된 타입으로.)
  * 제네릭 타입을 사용할 수 있는 일반 클래스, 인터페이스, 메소드에만 소거 규칙을 적용한다.
  * 타입 안정성 보존을 위해 필요하다면 type casting을 넣는다.
  * 확장된 제네릭 타입에서 다형성을 보존하기 위해 자바 컴파일러가 bridge method를 생성한다.     


* ex
``` java
// case1. unbounded type
// 컴파일 할 때 (타입 소거 전) 
public class Test<T> {
    public void test(T test) {
        System.out.println(test.toString());
    }
}

// 런타임 때 (타입 소거 후)
public class Test {
    public void test(Object test) {
        System.out.println(test.toString());
    }
}
```

``` java
// case2. bounded type
public class Test<T extends Comparable<T>> {
    private T data;

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
}

// 런타임 때 (타입 소거 후)
public class Test {
    private Comparable data;

    public Comparable getData() {
        return data;
    }

    public void setData(Comparable data) {
        this.data = data;
    }
}
```

```java
// case3. bridge method

// 컴파일
public class MyComparator implements Comparator<Integer> {
  public int compare(Integer a, Integer b) {
    //
  }
}

// 런타임 (소거 완료)
public class MyComparator implements Comparator {
  public int compare(Integer a, Integer b) {
    //
  }
}

// 컴파일러가 메소드 시그니처 사이에 불일치를 없애기 위해서 브릿지 메소드를 만듬.
public class MyComparator implements Comparator {
  public int compare(Integer a, Integer b) {
    //
  }

  //THIS is a "bridge method"
  public int compare(Object a, Object b) {
    return compare((Integer)a, (Integer)b);
  }
}

```

## 3-1. 타입 소거로 인해 제네릭에 원시 타입을 사용할 수가 없다
```java
  List<int> intList; //이런 선언은 불가능
```
* 제네릭 클래스는 타입 소거의 첫번째 규칙에 의해 타입 파라미터를 Object로 교체하는데 primitive 타입은 Object의 하위 타입이 아니기 때문에 제네릭에서 사용하는 것이 불가능하다.

###
https://jyami.tistory.com/99 
https://devlog-wjdrbs96.tistory.com/263

## 4. 제네릭의 장점
 * 객체의 타입 안정성을 높일 수 있다 (컴파일시 타입 체크)
 * 반환값의 타입 변환, 타입 검사에 대한 비용을 생략할 수 있다.

## 5. 실체화
* 구체화 타입(reifiable type): 자신의 타입 정보를 런타임에도 알고 있는 것
* 비 구체화 타입(non-reifiable type): 런타임에는 소거(erasure)가 되기 때문에 컴파일 타임보다 정보를 적게 가지는 것
___
`내용 정리`
> 배열은 공변이고 실체화((reify)되는 반면, 제네릭은 불공변이고 타입 정보가 소거된다. 그 결과 배열은 런타임에는 타입 안전하지만 컴파일타임에는 그렇지 않다. 제네릭은 반대다. 그래서 둘을 섞어 쓰기란 쉽지 않다. 둘을 섞어 쓰다가 컴파일 오류나 경고를 만나면, 가장 먼저 배열을 리스트로 대체하는 방법을 적용해보자 !
> 
> 아무튼 결론은 배열보다는 리스트 쓰자.(for 타입 안전성) 

