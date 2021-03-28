# [이펙티브 자바] Item24- 멤버 클래스는 되도록 static으로 만들라

중첩 클래스란 다른 클래스 안에 정의된 클래스를 말한다. 중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.

## 중첩 클래스의 종류

### 1. 정적 멤버 클래스

### 2. (비정적) 멤버 클래스

### 3. 익명 클래스

### 4. 지역 클래스

이 중 첫번째, 정적 멤버 클래스를 제외한 나머지를 내부 클래스(inner class)라고 한다.

## 정적 멤버 클래스

정적 멤버 클래스는 바깥 클래스의 **private 멤버에도 접근할 수 있다는 점을 제외**하고 일반 클래스와 똑같다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FHVFma%2FbtqZ4Bsu75t%2FJoUeoB1Y6ZyB2wKg3jDKe1%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbt46Y7%2FbtqZ4AG77dX%2FiUBua6k5ri2hJIZRxpkG51%2Fimg.png)

정적 멤버 클래스는 보통 바깥 클래스와 함께 쓰일 때 유용한 public 도우미 클래스로 쓰인다. 

private 정적 멤버 클래스는 바깥 클래스가 표현하는 객체의 구성요소를 나타낼 때 사용된다. 

공개된 클래스의 멤버 클래스의 접근제어자가 public이나 protected라면 정적으로 만들지를 신중하게 고려해야한다. 멤버 클래스 역시 공개 API가 되니, 향후 릴리즈에 static을 붙이게되면 하위 호환성이 깨진다.

## 비정적 멤버 클래스

구문상으로는 정적 클래스와 static이 붙어있고 없고의 차이지만, 의미상으로는 그 차이가 꽤 크다. 비정적 멤버 클래스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다. 따라서 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 참조를 가져올 수 있다.

```java
public class Something {
    void test() {
        System.out.println("something");
    }

    private class NonStaticClass {
        void call() {
            // 바깥 클래스의 메서드를 호출!
            Something.this.test();
        }
    }
}
```

만약 개념상 이러한 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들어주는 것이 좋다. **비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없기 때문이다.**

비정적 멤버 클래스 인스턴스와 바깥 클래스의 인스턴스의 관계는 멤버 클래스가 인스턴스화될 때 확립된다. 관계가 확립되고나면 더 이상 변경할 수 없다. 드물게 직접 `바깥 클래스의 인스턴스.new 비정적 클래스` 처럼 생성하기도 하지만 이는 생성시간과 비용이 많이 들기 때문에 좋지 않다.

```java
// 생성시간과 비용이 많이든다! 사용을 피하자.
Outer.new NonStaticMember();
```

### 결론

**멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.** static을 생략하면 바깥 인스턴스로의 숨은 외부 참조를 갖게되므로 시간과 공간을 많이 차지한다. 또한,  GC가 바깥 클래스의 인스턴스를 수거하지 못할 경우 메모리 누수가 발생하게 된다.

## 익명 클래스

익명 클래스는 바깥 클래스의 멤버가 아니다. 멤버와 달리 쓰이는 시점에 선언되고 인스턴스가 만들어지기 때문이다. 또한 익명 클래스는 비정적인 문맥에서만 사용될 때 바깥 클래스의 인스턴스를 참조할 수 있다.

### 익명 클래스의 제약

- 선언한  지점에서만 인스턴스를 만들 수 있다.
- instanceof 검사, 클래스의 이름이 필요한 작업을 수행할 수 없다.
- 여러 인터페이스를 구현할 수 없고, 인터페이스를 구현함과 동시에 다른 클래스를 상속할 수도 없다.
- 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수없다.

익명 클래스는 람다 이전(자바 7)에는 작은 함수 객체나 처리 객체를 만드는데 많이 사용되었다. 현재는 그 자리를 람다에게 물려주었지만, 아직 정적 팩토리 메서드를 구현할 때 쓰임새가 있다.

## 지역 클래스

네 가지 중첩 클래스 중 가장 드물게 사용된다. 지역 클래스는 지역 변수를 선언할 수 있는 곳이면 어디서든 선언할 수 있다. 따라서 유효 범위도 지역 변수와 같다.

지역 클래스는 다른 중첩 클래스들의 공통점을 하나씩 가지고 있다.

- 멤버 클래스처럼 이름을 가질수 있고 반복해서 사용할 수 있다.
- 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다.
- 정적 멤버는 가질 수 없으며, 가독성을 위해 짧게 작성되어야 한다.

# 핵심 정리

### 메서드로 정의하기에 너무 길다면, 멤버 클래스로 만들자.

### 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적 멤버 클래스로, 그렇지 않다면 정적 멤버 클래스로 만들자.

### 중첩 클래스가 한 메서드 안에서만 사용되며 해당 인스턴스를 생성하는 지점이 단 한곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 있다면 익명클래스로 만들고, 아니면 지역 클래스로 만들자.