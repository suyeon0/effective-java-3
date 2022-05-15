`클래스와 인터페이스 (4장)`

# [ITEM18] 상속보다는 컴포지션을 사용하라

`Q&A`
## 1.
> 119p) 래퍼 클래스는 단점이 거의 없다. 한 가지, 래퍼 클래스가 콜백 프레임워크와는 어울리지 않는다는 점만 주의하면 된다.

* 콜백 프레임워크란? (대표적인 콜백 프레임워크와 예시 코드)
* 래퍼 클래스가 콜백 프레임워크와 어울리지 않는 구체적인 이유가 무엇일까?

### [A]
___
``` java
래퍼클래스 :
- 다른 인스턴스를 감싸고 있는 클래스.
- 컴포지션은 새로운 클래스가 기존 클래스를 구성요소로 갖기 때문에 래퍼 클래스이다.

콜백 프레임워크: 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백) 때 사용하도록 하는 것
"콜백 프레임워크" 라는 단어로 구글링한 결과 명확한 설명은 따로 존재하지 않았습니다 ㅜㅜ
따라서 책에서 말하고자 하는 내용을 유추해볼 때, 콜백 프레임워크는 콜백 패턴을 사용하는 경우를 말하는 것 같습니다.
```

* 래퍼 클래스 self 문제 예제
``` java

final class SomeService {

    // callback을 전달 받아 callback.call() 호출
    void performAsync(SomethingWithCallback callback) {
        new Thread(() -> {
            perform();
            callback.call();
        }).start();
    }

    void perform() {
        System.out.println("Service is being performed.");
    }
}

public interface SomethingWithCallback {
    void doSomething();
    void call(LocalDateTime now);
}


public class WrappedObject implements SomethingWithCallback{

    private final SomeService service;

    WrappedObject(SomeService service) {
        this.service = service;
    }

    /**
     * wrapper가 무엇인지 모르니(알 방법도 없음) 
     * service의 performAsync를 비동기적으로 수행시키기 위해
     * 자기 자신을 callback으로 넘김
     */
    @Override
    public void doSomething() {
        service.performAsync(this);
    }

    @Override
    public void call() {
        System.out.println("WrappedObject callback!");
    }
}


public class Wrapper implements SomethingWithCallback{

    private final WrappedObject wrappedObject;

    Wrapper(WrappedObject wrappedObject) {
        this.wrappedObject = wrappedObject;
    }

    void doSomethingElse() {
        System.out.println("wrapped object에 데코레이션 기능을 더 추가할 수 있다!");
    }

    @Override
    public void doSomething() {
        wrappedObject.doSomething();
    }

    @Override
    public void call() {
        System.out.println("Wrapper callback!");
    }
}
```

```
SomeService service = new SomeService();
WrappedObject wrappedObject = new WrappedObject(service);
Wrapper wrapper = new Wrapper(wrappedObject);

wrappedObject.doSomething();
```


* 래퍼클래스는 새로운 클래스를 만들어 private 필드로 기존 클래스의 인스턴스를 참조하게 하는 클래스 입니다.
* 예제에서 개발자는 구성요소로 갖고 있는 wrappedObject 의 콜백함수를 실행시키는 메소드를 호출 후, 
* 콜백함수로 자기 자신의 call() 메소드("Wrapper callback!" 가 출력되는) 실행을 원했겠지만
* 자기 자신을 다음 호출 때 사용하는 콜백함수의 특징 & 기존 클래스(private 필드)의 대응하는 메서드를 호출해 그 결과를 반환하는 래퍼클래스의 특징이 결합되어
* 래퍼클래스가 아닌 내부 객체의 call() 메소드를 호출하는 결과를 낳게 됩니다. ("WrappedObject callback!" 가 출력됨)
* 따라서, 콜백과 래퍼클래스가 어울리지 않는다는 말을 한 것이라고 판단됩니다!


###

[래퍼 클래스 self 문제 예제 출처](https://coderanch.com/t/670687/java/wrapper-class-suitable-callback-framework)

___
`기타`

##1. 데코레이션 패턴
// TODO 
___


`내용 정리`
* **캡슐화**를 하는 가장 큰 **이유**는 정보 은닉에 있다.
  * 객체가 제공하는 필드와 메소드를 통해서만 접근이 가능.
  * 객체 내의 정보 손상과 오용을 방지하고 데이터가 변경되어도 다른 객체에 영향을 주지 않는다.

* 그렇기에 상속은 캡슐화를 깨트린다.
  * 모듈을 만들 때 SOLID 원칙에 의해 응집도는 높고 결합도는 낮은 모듈을 만들고, 변경사항이 있는 경우 클라이언트의 코드만 변경하는 것이 이상적이다.
  * 그러나 상속 받은 하위 클래스는 상위 클래스의 구현 내용 변경에 따라 하위 클래스의 구현 내용도 바뀔 수 있게 된다.


* 상속의 취약점을 피하려면 상속 대신 `컴포지션`과 `전달`을 사용하자 !


* `컴포지션`은 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 것을 의미한다. 새로운 클래스는 컴포지션을 이용할 때 메서드에서 기존 클래스가 제공하는 메서드를 호출하여 결과를 반환한다. 이 방식을 `전달(forwarding)`이라 하며, 새 클래스의 메서드들을 `전달 메서드(forwarding method)`라 부른다. 컴포지션과 전달의 조합은 더 넓은 의미로 위임(delegation)이라고 부른다.


* 컴포지션을 사용함으로써 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않는다.

> 상속은 코드 재사용성의 면에서 강력함을 갖고 있지만 캡슐화를 해친다. 상속을 사용할 상황이 아니거나, 상황을 파악하지 못하는 상태라면 최대한 컴포지션을 사용하여 일어날 수 있는 문제를 피하며 안전하게 하도록 하자.




----
### 참고
[컴포지션](https://ckddn9496.tistory.com/92)