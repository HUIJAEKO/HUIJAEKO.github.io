---
layout: single
title: "What is java"
---

- **파일 실행: 고급 언어가 컴파일되어 기계어로 변환된 뒤 컴퓨터가 사용**

**캐멀스타일**: 자바 소스 파일명은 대문자로 시작(MemberGrade.java), 변수명은 소문자로 시작(mathScore)

**변수 선언**: 하나의 값을 저장할 수 있는 메모리 번지에 붙여진 이름

**정수 타입**: byte, char, short, int, long

**실수 타입**: float, double

**논리**: boolean


- **자동 타입 변환**: 허용 범위가 작은 타입이 허용 범위가 큰 타입으로 대입될 때 발생

```java
byte byteValue = 10;
int intValue = byteValue;
```

- **강제 타입 변환**: 큰 허용 범위 타입은 작은 허용 범위 타입으로 쪼개어서 저장하는 것

```java
int intValue = 10;
byte byteValue = (byte)intValue;
```

- **변환 타입**: 문자열을 기본 타입으로 변환할 수도 있다.
```java
String str = "200";
int value = Integer.parseInt(str);
```

- **변수 사용 범위:{}내에서 선언된 변수는 해당 중괄호 블록 내에서만 사용할 수 있다.**
```java
public static void main(String[] args){
  int var1;

  if(){
    int var2
    //var1과 var2사용 가능
  }

  for(){
    int var3
    //var1과 var3사용 가능, var2사용 불가능
  }

  //var1만 사용 가능
}
```


