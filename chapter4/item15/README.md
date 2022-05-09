`클래스와 인터페이스 (4장)`

#[ITEM15] 클래스와 멤버의 접근 권한을 최소화하라

----
## 1. 가변객체를 참조하는 Public static final 필드

> 99p) public static final 필드인데 가변객체를 참조한다면,
final 이 아닌 필드에 적용되는 모든 불이익이 그대로 적용된다


### [Q]
___
실제 예제 확인해보기

### [A]
___
``` java
public class UtilClass {

    public static final Map<Integer, String> myMap; // 가변객체
    static {
        Map<Integer, String> aMap = new HashMap<>();
        aMap.put(1, "one");
        aMap.put(2, "two");
        myMap = aMap;
    }
    
    public static final String SAMPLE_STRING = "SAMPLE_STRING"; // 대표적인 불변 객체 String

}
```
``` java
@Test
public void public_static_final_테스트() {
    // 컴파일 오류 발생 -> Cannot assign a value to final variable 'SAMPLE_STRING'
    UtilClass.SAMPLE_STRING = "new"; 
    
    UtilClass.myMap.put(1,"notOne!!!!");
    System.out.println(UtilClass.myMap.get(1)); // notOne!!!! 출력되면서 값 변경 가능 확인 
}

```
###

---


## 2. Collections.unmodifiableList

> 100p. public static final 필드(List 타입으로 가정) 나 접근자 메서드를 제공함으로써 보안 허점을 제공할 수 있는데, 이를 해결할 수 있는 방법으로 `Collections.unmodifiableList` 이 있다. public List를 private 로 만들고 public 불변 리스트를 추가하는 것이다.


### [Q] 
___
* 어떻게 해당 문제를 해결할 수 있는가? 
* = Collections.unmodifiableList를 사용해서 반환받은 리스트 레퍼런스가 Read-only 인 이유

### [A]
___
 * UnmodifiableList 타입에 set(), add(), remove() 등의 리스트를 수정하는 행위를 호출하면 UnsupportedOperationException()을 발생시키기 때문이다.
``` java
        public E get(int index) {return list.get(index);}
        public E set(int index, E element) {
            throw new UnsupportedOperationException();
        }
        public void add(int index, E element) {
            throw new UnsupportedOperationException();
        }
        public E remove(int index) {
            throw new UnsupportedOperationException();
        }
        public int indexOf(Object o)            {return list.indexOf(o);}
        public int lastIndexOf(Object o)        {return list.lastIndexOf(o);}
        public boolean addAll(int index, Collection<? extends E> c) {
            throw new UnsupportedOperationException();
        }
```  

### 기타
___
* (1) UnsupportedOperationException 이 런타임 익셉션이기 때문에 컴파일 시점에서 확인이 불가하다
####
* (2) 원본 객체를 변경시킴에 따른 객체 변경은 막을 수 없다
``` java
UnmodifiableList(List<? extends E> list) {
    super(list);
    this.list = list;
}
```

위 코드를 보면, UnmodifiableList 또한 원본 객체를 참조하기 때문에, 원본 객체를 변경시킴에 따른 객체 변경은 막을 수 없다.
그래서 책에 나와있는 것처럼 원본 객체를 private 으로 접근제한 설정하여 외부로부터 접근을 막고, 
read-only 의 성격을 지닌 UnmodifiableList 를 public 으로 제공하라고 한 것임을 이해할 수 있었다.

``` java
// 기타(2) 관련 코드 : 원본 객체 변경에 따라 unmodifiableList 리턴값도 변경됨
List<SampleVO> list = new ArrayList<>();
list.add(new SampleVO());
System.out.println("list.size() : " + list.size()); // 1

List<SampleVO> readOnlyList = Collections.unmodifiableList(list);
System.out.println("readOnlyList.size() : " + readOnlyList.size()); // 1

list.add(new SampleVO());
System.out.println("readOnlyList.size() : " + readOnlyList.size()); // 2
```
### 참고
[UnmodifiableList은 만능이 아니다] (https://ecsimsw.tistory.com/entry/unmodifiableList%EC%9D%80-%EB%A7%8C%EB%8A%A5%EC%9D%B4-%EC%95%84%EB%8B%88%EB%8B%A4)
###

----

## 3. jdk 9 - 모듈시스템


* `jdk 9 이전`까지는 자바 api 클래스들을 모두 묶어놓은 `rt.jar` 를 사용해서
JVM 이 응용프로그램을 실행할때 rt.jar 를 로딩해서 클래스들을 메모리에 올려놓았다.
(클래스로더 계층 중 최상위 우선순위를 갖는 Bootstrap ClassLoader  가
jre/lib/rt.jar 를 로드하는 특징과 같이 생각하면 좋을 것 같다고 생각했다)
###
* `Jdk9 이후` 부터는  rt.jar 를 사용하지 않고, 자바 api 를 다수의 모듈로 분할하여 제공하고 `필요한 모듈만` 조립하여 메모리에 올린다.
  * 장점 : `메모리`가 적은 시스템에서도 유연하게 동작할 수 있게 한다

  * 자바 설치 경로의 jmods 디렉토리에 모듈 파일(.jmod) 존재하고 모듈 파일에는 자바 api 의 패키지들이 들어있다. ( /Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home/jmods/java.base.jmod
  : java.lang, java.util, java.io 등이 들어있음)

####
* 아래와 같이 명령어를 쳤을 때 참조하고 있는 경로가 아래와 같이 달랐다
  * `case1. Jdk11 사용`

  * `case2. Jdk8 사용 ` : rt.jar 참조


### 참고
[jdk9 모듈 설명](https://www.youtube.com/watch?v=cfs2wjfp5xE)