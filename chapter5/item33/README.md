`제네릭 (5장)`

# [ITEM33] 타입 안전 이종 컨테이너를 고려하라

## 1. 타입 안전 이종 컨테이너 란
___
```java

public class Main {
    public static void main(String[] args) {

        Favorites favorites = new Favorites();

        favorites.put(String.class, "스트링");
        //밑에 코드는 컴파일 에러가 난다. 타입안정성을 얻을 수 있음!
        favorites.put(Integer.class, "이것도 되나.");

        //타입 형변환도 자동으로 해줘서 이런 코드가 필요 없다.
        Integer o = (Integer) favorites.get(Integer.class);
        //이렇게만 작성하면 된다.
        Integer o2 = favorites.get(Integer.class);

    }
}
//타입 안전 이종 컨테이너
class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void put(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T get(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}

```
* 장점
  * 컴파일타임에 타입안정성을 보장해준다.
  * map에서 꺼내올 때, 타입캐스팅을 클라이언트쪽에서 해주지 않아도 되서 더욱 깔끔하다.

### 그래서 타입 안전 이종 컨테이너란?
* 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공한다. 
* 이렇게 하면 제너릭 타입 시스템이 값의 타입이 키와 같음을 보장해주는데, 이러한 설계 방식을 타입 안전 이종 컨테이너 패턴이라고 한다.

###
*  [타입 안전 이종 컨테이너](https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/04/24/%ED%83%80%EC%9E%85-%EC%95%88%EC%A0%84-%EC%9D%B4%EC%A2%85-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88/)
  `정리`



## 2. cast 메소드
> cast 메서드는 형변환 연산자의 동적 버전이다. 이 메서드는 단수히 주어진 인수가 Class 객체가 알려주는 타입의
> 인스턴스인지를 검사한 다음, 맞다면 그 인수를 그대로 반환하고,
> 아니면 ClassCastException 을 던진다.


* 우리가 많이 하는건 정적 형변환이다. ex) UserVO userVO = (UserVO)object;
* 정적 캐스팅은 이미 객체(object)가 어떤 클래스인지 알고 있어야 가능하다.
#####
* JDK 1.5부터 캐스트 구문을 대체할 수 있게 Class에서 cast() 메서드를 지원하고 있다.
```java
public T cast(Object obj) {
  if (obj != null && !isInstance(obj))
  throw new ClassCastException(cannotCastMsg(obj));
  return (T) obj;
 }
```
* UserVO userVO = UserVO.class.cast(object); 로 변경 가능.


## 2-1. 동적 캐스팅이 필요한 경우?
[필요한 케이스](https://carrotweb.tistory.com/96)
