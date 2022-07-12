`람다 (7장)`

# [ITEM33] 스트림은 주의해서 사용하라

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