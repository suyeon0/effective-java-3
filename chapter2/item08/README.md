# 객체 생성과 파괴 (2장)

## # [ITEM08] finalizer와 cleaner 사용을 피하라


> 궁금했던 점과 해결 내용


## 1. WeakHashMap

``` java
@Test
public void weakHashMap_test() {
    WeakHashMap<String, Integer> weakHashMap = new WeakHashMap<>();
    String key1 = "test1";
    String key2 = "test2";

    weakHashMap.put(key1, 1);
    weakHashMap.put(key2, 2);

    key1 = null;

    System.gc();

    weakHashMap.entrySet().stream().forEach(
        System.out::println
    );
}
```

### 질문

WeakHashMap 은 WeakReference 을 사용했기 때문에
객체 참조가 끊어지면 ( = null 로 만들면 ) 바로 gc 대상이 된다는 걸 배웠다.
테스트 코드를 위와 같이 짜보았는데
`System.gc();`  실행 이후에도
weakHashMap.put 에 “test1” 이 그대로 남아있는게 이해가 되지 않았다.

```
// 결과
test1=1
test2=2
```

### 해결

* 위의 문제가 발생한 이유는 `String Content Pool` 때문이었다.
* String 은 literal(“”) 로 생성되면 heap 영역 내 “String Constant Pool” 에 저장되어 그 주소값이 재사용된다.  따라서 항상 strong Reference 로 남아있는다.
* 하지만 new 연산자를 사용하여 String 객체를 만들 땐 Constant Pool 에 저장되지 않고 새로 객체가 만들어진다. 따라서 테스트를 아래 코드로 했을 땐, gc 대상이 되는걸 확인할 수 있었다.
```
String key1 = new String("test1");
String key2 = new String("test2");
```



### 기타
  * WeakHashMap 은 안드로이드나 IoT 처럼 메모리가 작아서 메모리 관리가 필요한 곳에서 주로 사용한다고 한다. 잘못 사용하면 NPE 를 발생시키기 좋으므로 iot 처럼 메모리 걱정 해야 하는 경우가 아니라면 굳이 ‘사용해서 장애 내지는 말자’

### 참고
  [WeakHashMap 에 대해 제대로 이해하자.](https://aroundck.tistory.com/3057)


---

> 기록

## 1. Finalizer Attack 막는 방법

item08을 읽다보면 아래와 같은 내용이 있다.
``` text
 finalizer 를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다. 
 (하위 클래스에서 악의적인 finalizer 가 수행되면서 발생하는 공격)
 
(1) final 클래스들은 그 누구도 하위 클래스를 만들 수 없으니 이 공격에서 안전하다
(2) final 이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalizer 메소드를 만들고 final 로 선언하자.
```
해결을 위해 클래스와 메소드에 Final 을 적용하는 부분이 있는데 읽으면서
`final 클래스` , `final 메소드`  의 좋은 예시인 것 같다고 생각했다.

* 방법1. 부모 클래스 자체를 final 로 만든다
`final 클래스는 상속 자체가 불가하니까`

* 방법2. 부모 클래스의 finalize 메소드를 final  로 선언한다. 
`final 메소드는 오버라이딩이 불가하니까`

``` java
public class Parent {
    @Override
    protected final void finalize() throws Throwable {}
}

public class Child extends Parent{
	/* 오버라이딩 불가
    @Override
    protected final void finalize() throws Throwable {
        // 악의적인 코드~~
    }
*/
}
```

### 기타
* finalize 는 자바9부터 Deprecated 되었다
