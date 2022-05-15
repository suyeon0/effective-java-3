`클래스와 인터페이스 (4장)`

# [ITEM16] public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라


----
`Q&A`
## 1.
> 104p. 하지만 package-private 클래스나 private 중첩 클래스에서는 종종(불변이든 가변이든) 필드를
> 노출하는 편이 나을 때도 있다.

### [Q]
___
어떤 경우에 노출하는 편이 좋은지 그리고 노출함으로써 얻는 장점은 무엇일까요?

### [A]
___
: 제가 생각한 장점과 노출 판단의 기준은 `가독성` 입니다! 

package-private 클래스, private 중첩 클래스 에서는 접근에 제한이 생겨서 public 필드 사용에 대한 위험 요소가 낮아지기 때문입니다.

```java
public class Point {
	public double x;
	public double y;
}
```
* 위와 같이 클래스 작성시 아래와 같은 문제점이 발생합니다
  * (1) 외부에서 인스턴스 필드에 직접 접근할 수 있게 된다 -> 불변식 보장 불가
  * (2) 외부에서 필드에 접근할 때 부수 작업을 수행할 수 없다.


* 아래와 같이 private 중첩 클래스 를 작성했을 때, 위의 문제점을 발생시키지 않으면서 접근자 메소드 없이 깔끔한 코드를 작성할 수 있습니다.
```java
public class TopPoint {
    /**
     *  private 클래스를 중첩시키면 
     *  TopPoint 클래스에서는 얼마든지 Point 클래스의 필드를 조작할 수 있지만,
     *  외부 클래스에서는 Point 클래스의 필드에 직접 접근할 수 없습니다.
     *  오히려 코드 작성 면에서 x, y 필드의 getter 메소드 작성 없이 깔끔할 수 있습니다
     *  (TopPoint 내부에서만 동작하기 때문)
     */

    private class Point {
      public int x;
      public int y;

      public Point(int x, int y) {
        this.x = x;
        this.y = y;
      }
    }

    public double getMaxPoint(int a, int b) {
      Point point = new Point(a,b);
      return Integer.max(point.x, point.y);
    }
}
```
```java
// 참고 : private static 클래스 중첩 이용한 싱글톤 구현(질문과 상관 없음!)
public class S4DemandHolder {
  private S4DemandHolder(){}

  public static S4DemandHolder getInstance(){
    return SingletonLazyHolder.INSTANCE;
  }

  private static class SingletonLazyHolder{ // static class로 선언함으로써 외부에서 객체 생성 없이 호출 가능
    private static final S4DemandHolder INSTANCE = new S4DemandHolder();
  }
}
```
* 노출해도 된다 가 아니라, 노출하는 편이 나을때도 있다고 한 이유는
* package-private 클래스의 경우 같은 패키지에 있는 클래스에서는 public 필드 접근이 가능하기 때문인 것 같습니다
