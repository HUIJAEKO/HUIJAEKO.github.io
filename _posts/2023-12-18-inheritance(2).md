---
layout: single
title: "타입 변환과 다형성, 추상 클래스, 봉인된 클래스"
categories: java
---

:triangular_flag_on_post:`자동타입변환`: 자동적으로 타입 변환이 일어나는 것

자동타입변환은 자식타입객체가 부모타입객체로 변환할 때 일어난다.

```java
Class Animal{

}

Class Cat extends Animal{

}
//일때

Cat cat = new Cat();
Animal animal = cat;
```

:triangular_flag_on_post:`강제타입변환`: 부모 타입을 자식 타입으로 변환할 때 사용

```java
Animal animal = new Cat(); //자동타입변환
Cat cat = (Cat) animal;
```

:triangular_flag_on_post:`객체 타입 확인`: 어떤 객체가 매개값으로 제공되었는지 확인하는 것

```java
boolean result = 객체 instanceof 타입;
//ex
public void method(Animal animal){
  if(animal instanceof Cat){
    Child child = (Child) parent;
  }
}

//자바 12부터는 축약해서 사용 가능
if(animal instanceof Cat cat){
}
```

:triangular_flag_on_post:`다형성`: 사용 방법은 동일하지만 실행 결과가 다양하게 나오는 성질

다형성을 위해서는 `자동타입변환` 과 `메소드 오버라이딩`이 필수적으로 있어야 한다.

자식 객체들은 동일한 부모를 상속하고, 부모의 메소드를 오버라이딩한다. 

이에 따라 다른 실행 결과가 나오게 된다.

한가지 예시이다.


```java
public class Car {	
	public void run() {
		System.out.println("차량이 달립니다.");
	}
}
//부모객체생성
```

```java
public class Taxi extends Car {
	@Override
	public void run() {
		System.out.println("택시가 달립니다.");
  }
}
//자식 객체1
```

```java
public class Bus extends Car{
	@Override
	public void run() {
		System.out.println("버스가 달립니다.");
	}
}
//자식 객체2
```

```java
public class Driver {
	public void ride(Car car) {
		car.run();		
	}
}
```

```java
public class DriverExample {

	public static void main(String[] args) {
		
		Driver driver = new Driver();
		
		Bus bus = new Bus();
		driver.ride(bus);
		
		Taxi taxi = new Taxi();
		driver.ride(taxi);
	}
}
```

위 코드를 실행시킨다면, 

`버스가 달립니다.`

`택시가 달립니다.`

라는 결과가 나올 것이다. 이것이 `다형성`이다.
