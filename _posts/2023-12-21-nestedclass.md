---
layout: single
title: "중첩 클래스와 익명 객체"
categories: java
---

- **중첩클래스**: 클래스 내부에 선언된 클래스이다.

크게 **인스턴스 멤버 클래스, 정적 멤버 클래스, 로컬 클래스**로 나누어진다.

- **인스턴스 멤버 클래스**: A객체를 생성해야 B객체를 생성할 수 있다.

```java
class A{
  class B{}
}

//생성
A a = new A();
A.B b = a.new B();
```

- **정적 멤버 클래스**: A객체를 생성하지 않아도 B객체를 생성할 수 있다.

```java
class A{
  static class B{}
}

//생성
A.B b = new A.B();
```

- **로컬 클래스**: method가 실행될 때만 B객체를 생성할 수 있다.

```java
class A{
  void method(){
    class B{}
  }
}

//생성
A a = new A();
b.method();
```

로컬 클래스에서의 변수는 **final 특성을 갖게 되어** 읽을 수만 있고 수정할 수 없다.

- **바깥 멤버 접근**: 중첩 클래스는 바깥 클래스의 멤버(필드, 메소드)에 접근할 수 있다.

```java
public class A{
  int field1 = 1

  public class B{
    int field2 = 2
    void print(){
      //B객체의 필드 이용
      System.out.println(this.field2);
      //A객체의 필드 이용
      System.out.println(A.this.field1);
    }
  }
}
```

- **중첩 인터페이스**: 클래스와 마찬가지로 인터페이스도 중첩해서 선언할 수 있다.

```java
class A{
  public interface B{
    //상수 필드
    //추상 메소드
    //디폴트 메소드
    //정적 메소드
  }
}
```

- **익명 객체**: 이름이 없는 객체이다. 클래스를 상속하거나 인터페이스를 구현해야 만 생성될 수 있다.

**익명 자식 객체**와 **익명 구현 객체**로 나누어진다.

- **익명 자식 객체**: 부모 클래스를 상속받아 생성된다.

```java
public class Tire{
  public void roll(){
    System.out.println("타이어가 굴러간다.");
  }
}

//다음과 같이 사용할 수 있다.

public class Car{
  //익명 자식 객체 생성
  private Tire tire1 = new Tire(){
    @Override
    public void roll(){
      System.out.println("타이어1이 굴러간다.");
    }
  };
}
```

또한 **매개변수를 이용한 메소드**에도 사용할 수 있다.

```java
public void run(Tire tire){
  tire.roll()
}

//다음과 같이 사용할 수 있다.

Car car = new Car();

car.run(new tire(){
  @Override
    public void roll(){
      System.out.println("타이어2가 굴러간다.");
    }
};
```

**익명 구현 객체에서도 같은 방법으로 사용**할 수 있으며, 

익명 자식 객체보다는 **익명 구현 객체가 더 많이 사용된다.**


