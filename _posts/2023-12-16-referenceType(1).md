---
layout: single
title: "참조 타입"
categories: java
---

`참조 타입`: 객체의 번지를 참조하는 타입

 `배열 타입`, `열거 타입`, `클래스`, `인터페이스` 등이 있다. 정수나 실수, 논리 타입은 기본 타입에 속한다.

- 스택 영역과 힙 영역: 변수들은 모두 스택이라는 메모리 영역에 저장된다. 

- 기본 타입 변수는 직접 값을 저장하고 있지만, 참조 타입 변수는 객체 번지를 힙 영역에 저장한다.

`힙 영역에 객체가 생성된다`

```java
arr1 = new int[]{1,2,3};
arr2 = new int[]{1,2,3};
```
객체를 비교할 때에는 대입되는 번지를 확인해야 한다. 위에 같은 경우에 저장하는 항목은 같지만,

서로 다른 객체로 저장되므로 `arr1 != arr2` 가 된다.

- 참조 타입 변수는 null을 가질 수 있다.

```java
int[] intArray = null;
intArray[0] = 10;
//NullPointException이 된다.
```

`기존에 사용중이던 참조 타입의 번지를 없애거나 바꿀 수 있고, 그렇게 되면 기존의 객체는 쓰레기 객체가 된다.`

- 문자열에서 리터럴이 동일하면 객체를 공유한다.

하지만 new연산자를 사용하면 `서로 다른 객체`를 사용하게 된다.

```java
String name1 = "홍길동";
String name2 = "홍길동";
//name1 == name2

String name3 = new String("홍길동");
String name4 = new String("홍길동");
//name3 != name4
```

하지만 내부 분자열만을 비교하고 싶을 때에는 `equals()메소드`를 이용한다

```java
boolean result = str1.equals(str2);
```




