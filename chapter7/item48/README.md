`람다 (7장)`

# [ITEM48] 스트림 병렬화는 주의해서 적용하라

## 1. 스트림 병렬화 주의사항

> 스트림을 잘못 병렬화하면 (응답 불가를 포함해) 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나
> 예상 못한 동작이 발생할 수 있다 (293p)

Q. 이유와 구체적인 예시

---

1. 이러한 케이스가 잘못된 병렬화.

> 데이터 소스가 Stream.iterate 이거나 중간 연산으로 limit 을 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다.

1-1. 이유
* Parallel() 은 병렬로 실행하기 위해 데이터 소스를 원하는 단위로 나눠서 스레드에 분배한다. 나누는 작업은 `Spliterator` 가 담당한다. (Stream 이나 Iterable 의 spliterator 메소드)


* 따라서 좋은 케이스는 분해성이 좋은 자료구조 케이스(다수 스레드 분배에 유리하기 때문)

  * ArrayList, IntStream.range, HashSet, TreeSet, 배열 ( 참조지역성 기억 - 이웃한 원소의 참조들이 메모리에 연속해서 저장)

* 나쁜 케이스는 분해성이 나쁜 자료 구조 케이스이다
  * LinkedList, Stream.iterate ( 모든 요소 탐색 필요 )

    * Stream.iterate() : 무한 스트림을 만들어내며, 본질적으로 순차적이다. 이전 연산의 결과에 따라 다음 함수의 입력이 달라지기 때문에 iterate 연산을 청크로 분할하기가 어렵다.
    -> 순차처리와 다를게 없게 되는데 스레드를 할당하는 오버헤드만 증가한다.
```java
Stream<Integer> stream5 = Stream.iterate(0, n -> n + 2).limit(5);
System.out.println("stream5");
stream5.forEach(System.out::println); //0 2 4 6 8
```

---
## 2. java.util.stream.Collectors
https://daddyprogrammer.org/post/1163/java-collectors/

---
## 3.ThreadLocalRandom, SplittableRandom
Q. p.295 ThreadLocalRandom, SplittableRandom를 거의 써본적이 없어서 이 인스턴스의 간단한 내부구조나 차이점 같은 설명이 있으면 좋을것 같습니다!

---
1) ThreadLocalRandom
   java.util.Random은 멀티 쓰레드 환경에서 하나의 인스턴스에서 전역적으로 의사 난수(pseudo random)를 반환한다. 따라서 같은 시간에 동시 요청이 들어올 경우 경합 상태에서 성능에 문제가 생길 수 있다. 반면 JDK 7부터 도입된 java.util.concurrent.ThreadLocalRandom은 java.util.Random를 상속하여 멀티 쓰레드 환경에서 서로 다른 인스턴스들에 의해 의사 난수를 반환하므로 동시성 문제에 안전하다

* ThreadLocalRandom vs Random
  java.util.concurrent.ThreadLocalRandom은 똑같이 Random API의 구현체이며, java.util.Random를 상속받는다. ThreadLocalRandom은 위의 동시성 문제를 해결하기 위해 각 쓰레드마다 생성된 인스턴스에서 각각 난수를 반환한다. 따라서 Random과 같은 경합 문제가 발생하지 않아 안전하며, 성능상 이점이 있다. Random대신 ThreadLocalRandom을 쓰자.

-> 둘다 thread-safe 하긴 하나, Random() 은 seed 를 AutomicLong 을 사용하기 때문에 멀티 쓰레드의 요청에 대해서도 순서대로 처리하여 마치 synchronized 처리처럼 동작하기 때문에 느리다.
ThreadLocalRandom 은 AutomicLong 사용하지 않고 thread 별로 seed 값을 다르게 관리하기 때문에 Random 보다 멀티쓰레드에 대해 응답이 빠르면서도 쓰레드 세이프하다.

- AtomicLong : Long 자료형을 갖고 있는 Wrapping 클래스
  Synchronized 의 lock 을 검에 따라 발생하는 blocking 으로 발생하는 성능 이슈를 비효율성을 보완할 수 있는 방법. CAS(compare and swap) 알고리즘을 사용.
- CAS : 현재 주어진 값(=현재 쓰레드에서의 데이터)과 실제 메모리에 저장된 데이터를 비교해서 두 개가 일치할때만 값을 업데이트
  [Java - Atomic변수](https://beomseok95.tistory.com/225)
  [Java atomic과 CAS 알고리즘](https://steady-coding.tistory.com/568)


2) SplittableRandom
* jdk8  등장.
* 병렬처리에 특화된 random number generator
* thread-safe 하지 않음.
* 포크조인풀이나 병렬스트림 처리를 할 때는 ThreadLocalRandom 보다도 SplittableRandom을 사용하는 것이 품질적인 측면에서 더 좋다

new Random() 으로 쓰던걸 ThreadLocalRandom.current() 로 호출하면 된다.

[Java Random - ThreadLocalRandom, SplittableRandom, SecureRandom]
[Random 대신 ThreadLocalRandom을 써야 하는 이유](https://velog.io/@sojukang/Random-%EB%8C%80%EC%8B%A0-ThreadLocalRandom%EC%9D%84-%EC%8D%A8%EC%95%BC-%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0)(http://dveamer.github.io/backend/JavaRandom.html)




---
## 4. 코드 수행 시간 측정 방법
[JMH를 사용한 gradle 환경에서의 Java 코드 벤치마킹 – 최혜선 – Not First But Best](https://hyesun03.github.io/2019/08/27/how-to-benchmark-java/)