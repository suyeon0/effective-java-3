`클래스와 인터페이스 (4장)`

# [ITEM20] 추상클래스보다는 인터페이스를 우선하라

`Q&A`
## 1. 템플릿 메소드 패턴과 인터페이스
* 자바8 이상부터 인터페이스 디폴트 메소드로 템플릿 메소드 패턴 구현이 가능하지 않나요?

### [A]
___

인터페이스 같은 경우 디폴트 메소드를 final로 정의 불가하여 오버라이딩을 가능하게 하고

모든 메소드는 public abstract 이므로 인터페이스로 구현할 경우,

템플릿 메소드 내부에서만 호출되어야 할 메소드들이 public 제어자에 의해 의도치 않은 사용처에서 호출될 위험이 있어

템플릿 메소드 패턴을 만들수 없습니다.

[참고]
[템플릿 메소드 패턴](https://joeylee.tistory.com/21)

___
`기타`
## 1. 템플릿 메소드 패턴이란

___

템플릿 메소드는 필수 처리절차를 정의한 일련의 과정을 메소드로 정의하고 final로 변경을 제한.

템플릿 메소드의 안의 메소드중 하나 이상이 추상메소드로 정의되며, 그 추상 메소드는 서브클래스에서 구현.

이렇게 하면 서브클래스에서 일부분을 구현할 수 있도록 하면서도 구조는 바꾸지 않아도 된다.


* Abstract Class -> 템플릿 메소드를 구현
* Concrete Class -> 추상 클래스 역할에서 정의되어 있는 추상 메소드를 오버라이딩하여 구현

[참고할 소스]
[Favor Skeletal Implementation in Java - A skeletal implementation gets your interface and Abstract class working together.](https://joeylee.tistory.com/21)

___

`내용 정리`
> 다중 구현으로는 인터페이스가 가장 적합
> #####
> 인터페이스와 추상클래스를 같이 사용하는 추상 골격 구현(skeletal implementation)을 통한 
> ####
> '템플릿 메소드 패턴'도 기억하자


----
### 참고

[템플릿 메소드 패턴](https://dzone.com/articles/favour-skeletal-interface-in-java)
