`클래스와 인터페이스 (4장)`

# [ITEM15] 클래스와 멤버의 접근 권한을 최소화하라

----
`Q&A`
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
[UnmodifiableList은 만능이 아니다](https://ecsimsw.tistory.com/entry/unmodifiableList%EC%9D%80-%EB%A7%8C%EB%8A%A5%EC%9D%B4-%EC%95%84%EB%8B%88%EB%8B%A4)
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
  
  <img width="700" src="https://user-images.githubusercontent.com/83405795/167402836-6c2c5dcd-728a-4e41-8cb3-8712018d358f.png">

####
* 아래와 같이 명령어를 쳤을 때 참조하고 있는 경로가 아래와 같이 달랐다
  * `case1. Jdk11 사용`
  <img src="https://user-images.githubusercontent.com/83405795/167402863-0d805943-3569-44a4-a564-a439a6f9105b.png">

  * `case2. Jdk8 사용 ` : rt.jar 참조
  <img src="https://user-images.githubusercontent.com/83405795/167402854-a5866ef1-3c45-47a7-852c-3f4255bbcf5c.png">


### 참고
[jdk9 모듈 설명](https://www.youtube.com/watch?v=cfs2wjfp5xE)


----

## 4. Serializable
> 98p. private 과 package-private 멤버는 모두 해당 클래스의 구현에 해당하므로 보통은 공개 API 에 영향을 주지 않는다. 단, Serializable 을 구현한 클래스에서는 그 필드들도 의도치 않게 공개 API 가 될 수 있다

### [Q]
___
* Serializable 이 무엇이고, 직접 관리하지 않아도 되는 이유는?

### [A]
___
`(1) Serializable 이란?`
* 직렬화는 객체를 데이터 스트림으로 만드는 것을 뜻하고,
* 직렬화 가능한 객체를 만드는 방법으로 *Serializable* 이 있습니다.
* 따라서 작성한 클래스가 파일로 저장, 읽고 쓸 수 있도록 하거나, 다른 서버로 보내거나 받고자 할 때 필요한게 Serializable 이고,
  이를 사용하기 위해서는 Serializable 인터페이스를 구현해야 합니다.
* Serializable 인터페이스를 구현하면 JVM에서 해당 객체는 저장하거나 다른 서버로 전송할 수 있도록 해줍니다.

`(2) Serializable 을 구현한 클래스에서는 그 필드들도 의도치 않게 공개 API 가 될 수 있다 의미`
* Serializable 인터페이스를 구현한 클래스는 모든 멤버변수가 직렬화 대상이 되기 때문입니다.

  * static 은 제외
    * 객체 직렬화는 인스턴스에 대해 적용되기 때문에 클래스 자체 정보인 static 멤버는 여기에 포함되지 않는다.
    * Static 변수는 클래스의 모든 인스턴스에서 공유되는 하나의 필드니까!
  * 멤버변수 중 직렬화 대상에서 제외하고 싶다면 **transient** 키워드를 사용하여 제외시킨다
    * transient가 붙은 인스턴스 변수의 값은 그 타입의 기본값으로 직렬화된다.


``` java
import java.io.Serializable;
import lombok.Getter;

@Getter
public class User implements Serializable {

    private final String name;
    private final String team;
    private final transient int age;

    public User(String name, String team, int age) {
        this.name = name;
        this.team = team;
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
            "name='" + name + '\'' +
            ", team='" + team + '\'' +
            ", age=" + age +
            '}';
    }
}
```
``` java
public class Sample {

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        FileOutputStream fos = new FileOutputStream("objectfile.ser");
        ObjectOutputStream out = new ObjectOutputStream(fos);
        out.writeObject(new User("수연", "고도화", 30));

        FileInputStream fis = new FileInputStream("objectfile.ser");
        ObjectInputStream in = new ObjectInputStream(fis);
        User user = (User) in.readObject();
        System.out.println(user); // User{name='수연', team='고도화', age=0}
    }
}
```
* 참고
  * [Java 직렬화(Serialization)란 무엇일까? :: Gyun’s 개발일지](https://devlog-wjdrbs96.tistory.com/268)
  * [Static 사용을 피해야 하는 이유](https://kellis.tistory.com/127)
