---
layout: single
title: "class상속, final class, protected 접근 제한자"
categories: java
---

- `클래스 상속`: 부모 클래스의 필드와 메소드를 자식 클래스에게 물려줌

자식이 부모를 `선택`하며, 다중 상속을 `허용하지 않는다.`

```java
public class 자식클래스 extends 부모클래스{
}
```

자식 객체를 생성하면, `부모 객체가 먼저 생성된 후` 자식 객체가 생성된다.

부모 생성자는 자식 생성자 맨 첫줄에 있는 `super()` 에 의해 호출.

super은 코드를 넣지 않아도 컴파일 과정에 추가된다.

```java
public 자식클래스(){
  super();
}
```

하지만 부모 클래스가 매개 변수를 갖는 생성자가 있으면 `super코드를 직접 입력해야한다.`

```java
public class Phone{
  public Phone(String model){
    this.model = model;
  }
}
//이러한 경우 상속할 때 super를 직접 입력

public class SmartPhone extends Phone{
  public SmartPhone(String model){
    super(model);
  }
}
```

- `오버라이딩`: 부모의 메소드를 자식 클래스에서 재정의 하는 것. 다음과 같은 조건이 있다.

1. 부모 메소드의 선언부와 동일해야함
2. 접근 제한을 더 강하게 오버라이딩 불가능
3. 새로운 예외를 throws할 수 없다.

다음은 오버라이딩의 한 예시이다.

```java
public class Calculator{
  public double area(double r){
    return r*r*3.14
  }
}

//자식 클래스에서 오버라이딩
public class CorrectCalculator expends Calculator{
  public double area(double r){
    return r*r*Math.PI
  }
}
```

`super.method()` 를 사용한다면, 메소드를 오버라이딩할 때 부모 메소드의 재사용

그리고 추가적으로 자신이 하고 싶은 것을 추가해 실행할 수 있다.

- `final클래스와 메소드`: final클래스는 상속이 불가하며, final메소드는 오버라이딩이 불가하다.

```java
public final class 클래스{}
public fonal void stop(){}
```

- `protected 접근 제한자`: 같은 패키지에서는 사용 가능하나, 다른 패키지에서는 자식 클래스만 접근 허용




