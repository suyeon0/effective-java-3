`람다 (7장)`

# [ITEM45] 스트림은 주의해서 사용하라

## 1. 스트림 Lazy Evaluation
> 스트림 파이프라인은 지연 평가(lazy evaluation) 된다. 평가는 종단 연산이 호출될 때 이뤄지며, 
> 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다. 이러한 지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠다.

Q. Lazy evaluation 이라는게 정확히 어떤 것일까. 그리고 무한 스트림에 왜 최적화되어있을까
A. 장점 - > 결과를 얻기 위해 필요하지 않은 연산은 수행하지 않게 된다.

[Lazy Evaluation - 요기 이미지 참고하기 !](https://dororongju.tistory.com/137)

---
## 2. computeIfAbsent


> 맵에 각 단어를 삽입할 때 자바 8에서 추가된 computeIfAbsent 메서드를 사용했다.

자바 8부터 HashMap 에 여러 메소드들이 추가되었고, 이런 메서드를 사용해서 HashMap 을 조금 더 간결하면서 효율적으로 사용할 수 있게 되었다.
* putIfAbsent()
* computeIfAbsent()
* compute()
* computeIfPresent()
* merge()
* getOrDefault()
#####
* [메소드 설명 및 예제](https://blog.advenoh.pe.kr/java/%EC%9E%90%EB%B0%948-HashMap-%EB%B3%B4%EB%8B%A4-%EA%B0%84%EA%B2%B0%ED%95%98%EA%B3%A0-%ED%9A%A8%EA%B3%BC%EC%A0%81%EC%9C%BC%EB%A1%9C-%EC%9E%91%EC%84%B1%ED%95%98%EA%B8%B0/
  )

* [Java computeIfAbsent 예제](https://hbase.tistory.com/53)
``` java
Map<Key, Value> map = new HashMap();

Value value = map.get(key);
if (value == null) {
	value = getNewValue(key);
	map.put(key, value);
}


// computeIfAbsent 사용하면 이렇게 간결해진다!
Map<Key, Value> map = new HashMap();

Value value = map.computeIfAbsent(key, k -> getValue(key));
```

---
## 3. 스트림이 필요한 케이스
기존 코드는 스트림을 사용하도록 리팩토링하되, 새 코드가 더 나아 보일 때만 반영하자 !
* 원소들의 시퀀스를 일관되게 변환한다
* 원소들의 시퀀스를 필터링한다
* 원소들의 시퀀스를 하나의 연산을 사용해 결합한다 (더하기, 연결하기, 최솟값 구하기 등)
* 원소들의 시퀀스를 컬렉션에 모은다 (공통된 속성 기준)
* 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

* 스트림 메소드는 복수형 & 원소의 정체를 잘 알려주는 것으로 쓰자.

#### 스트림으로 처리하기 어려운 경우
* 한 데이터가 파이프라인의 여러 단계를 통과할 때, 이 데이터의 각 단계에서의 값들에 동시에 접근하는 것.

---

## 4. flatMap
** 스트림 클래스 기준 map 메소드와 flatMap 메소드의 차이
* flatMap
  : 스트림의 형태가 배열과 같을 때, 모든 원소를 단일 원소 스트림으로 반환할 수 있기 때문에 초기에 생성된 스트림이 배열인 경우에 매우 유용하다.

```java
String[][] namesArray = new String[][]{
        {"kim", "taeng"}, {"mad", "play"},
        {"kim", "mad"}, {"taeng", "play"}};
        
Set<String> namesWithFlatMap = Arrays.stream(namesArray)
        .flatMap(innerArray -> Arrays.stream(innerArray))
        .filter(name -> name.length() > 3)
        .collect(Collectors.toSet());
        
// play, taeng 출력
namesWithFlatMap.forEach(System.out::println);
```

* flatMap VS map
  * flatMap 은 결과를 스트림으로 반환하기 때문에 flatMap 의 결과를 가지고 바로 forEach 메소드를 체이닝하여 모든 요소를 출력할 수 있다 (-> 스트림의 원소 각각을 하나의 스트림으로 매핑한 다음 그 스트림들을 다시 하나의 스트림으로 합친다 - flattening )
  * 반면에 map 의 경우에는 단일 요소로 리턴되기 때문에 map 의 결과를 가지로 forEach 메소드로 루프를 진행한 후 그 내부에서 다시 한 번 forEach 메소드를 체이닝하여 사용해야 한다.
```java
String[][] namesArray = new String[][]{
        {"kim", "taeng"}, {"mad", "play"}};

// flatMap
Arrays.stream(namesArray)
        .flatMap(inner -> Arrays.stream(inner))
        .filter(name -> name.equals("taeng"))
        .forEach(System.out::println);

// map
Arrays.stream(namesArray)
        .map(inner -> Arrays.stream(inner))
        .forEach(names -> names.filter(name -> name.equals("taeng"))
            .forEach(System.out::println));

```

[자바 map 메서드와 flatMap 메서드의 차이](https://madplay.github.io/post/difference-between-map-and-flatmap-methods-in-java)

---
## 5. 람다의 지역변수는 Final , Effectively Final

> 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다. 하지만 람다에서는 final 이거나 사실상 final 인 변수만 읽을 수 있고, 지역변수를 수정하는 건 불가능하다
> (273p)

Q. 람다에서 final 이거나 final 인 변수만 읽을 수 있는 이유, 지역변수 수정 불가능한 이유 ?

---

### Effectively Final


  자바 8에서 등장한 syntatic sugar 의 일종인데 `초기화 된 이후 한번도 값이 변경되지 않았다면` effectively final 이라고 할 수 있다.
  fianl 키워드가 붙어있지 않지만 붙어있는 것과 동일하게 컴파일러에서 처리한다.

  effectively final 은
익명 클래스나 람다식에서 코드를 좀 더 간결하게 만들어준다.
```java
// java7
public void testPlus() {
    final int number = 1;
    Addable addableImple = () -> number + 1;
}

// Java 8
public void testPlus() {
    int number = 1; // Effectively final
    Addable addableImple = () -> number + 1;
}
```
---

### lambda 규칙

자바 8에서 추가된 람다식에는 다음과 같은 규칙이 존재한다.
1. 람다는 외부 block 에 있는 변수에 접근할 수 있다.
2. 외부에 있는 변수가 **지역 변수일 경우** final 혹은 effectively final 이어야한다
3. 인스턴스 변수나 클래스 변수는 final 혹은 effective final 하지 않아도 람다식에서 사용 가능

코드예제 : https://wookcode.tistory.com/27

---
### 위와 같은 규칙이 생기는 이유 - JVM 메모리 구조
`람다 캡쳐링`

기본적으로 람다 표현식은 (파라미터) -> 동작과 같은 구조를 지니며, 
파라미터로 넘겨진 변수를 활용하여 바디에서 작업을 수행한다. 
**람다 캡처링(capturing lambda)**이란 간단히 말해 이처럼 파라미터로 넘겨받은 데이터가 아닌
"람다식 외부에서 정의된 변수"를 참조하는 변수(=자유변수)를 람다식 내부에 저장하고 사용하는 동작을 의미한다. 

아래는 그 예이다.
```java
void lambdaCapturing() {
int localVariable = 1000;

Runnable r = () -> System.out.println(localVariable);
}
```

1. 람다식에서 사용되는 외부 지역 변수는 `복사본` 이다.
  * 지역 변수는 스택 영역에 생성된다. 따라서 지역 변수가 선언된 block 이 끝나면 스택에서 제거된다
    * 메소드 내 지역 변수를 참조하는 람다식을 리턴하는 메소드가 있을 경우, 메소드 block 이 끝나면 지역 변수가 스택에서 제거 되므로 추후에 람다식이 수행될 때 참조할 수 없다
  * 지역 변수를 관리하는 쓰레드와 람다식이 실행되는 쓰레드가 다를 수 있다
    * 스택은 각 쓰레드의 고유 공간이고, 쓰레드끼리 공유되지 않기 때문에 마찬가지로 람다식이 수행될 때 값을 참조할 수 없다
      -> 람다식에서는 외부 지역 변수를 직접 참조하지 않고 복사본을 전달받아 사용한다


2. 인스턴스 변수, 클래스 변수는 아닌 이유
  * Heap 영역 혹은 메서드 영역에 선언이 되어 Thread별로 모두 접근이 가능하기때문에 람다식에서 바로 접근이 되어 복사 과정이 불필요

### 외부 지역변수가 final, e-f 가 아닐 경우 발생할 수 있는 문제
```java
public void executelocalVariableInMultiThread() {
    boolean shouldRun = true;
    executor.execute(() -> {
        while (shouldRun) {
            // do operation
        }
    });
    
    shouldRun = false;
}
```
지역 변수 값(shouldRun) 을 제어하는 쓰레드 A, 람다식을 수행되는 쓰레드 B 가 있다고 가정할 때,
쓰레드 B의 shouldRun 값이 가장 최신 값으로 복사되어 전달 됐는지 확신할 수 없다 
왜냐하면 shouldRun 은 `변경이 가능`한 지역 변수이고, 지역 변수를 쓰레드 간에 sync 해주는 것은 불가능 하기 때문이다
(지역 변수는 쓰레드 A 의 스택 영역에 존재하기 때문에 다른 쓰레드에서 접근이 불가능하다)
값이 보장되지 않는다면 매번 다른 결과가 도출 될 수 있다 -> `지역변수 수정 불가능한 이유`

`이러한 이유로 인해 외부 지역 변수는 전달되는 복사본이 변경되지 않은 최신 값 임을 보장하기 위해 fianl 혹은 effectively final 이어야 한다`

---
> 결론
> 
> 람다식에서 외부 지역 변수를 이용할 경우 final 혹은 effectively final 이어야 하는 이유는 : 지역 변수는 스택에 저장되기 때문에 람다식에서 값을 바로 참조하는 것에 제약이 있어 복사된 값을 이용하게 되는데, 이 때 멀티 쓰레드 환경에서 복사 될/복사된 값이 변경 가능 할 경우 이로 인한 동시성 이슈를 대응할 수 없기 때문이다.


* 참고
* https://github.com/woowacourse-study/2022-modern-java-in-action/issues/22
* https://vagabond95.me/posts/lambda-with-final/