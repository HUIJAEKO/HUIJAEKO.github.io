---
layout: single
title: "문자열"
---

**문자열 다루기**

- **문자 추출**: charAt() 사용

```java
String subejct = "자바 프로그래밍";
char charValue = subject.charAt(3);
```

- **문자 길이**: length()사용

```java
String subejct = "자바 프로그래밍";
int length = subject.length();
```

- **문자열 대체**: replace()사용

```java
String subejct = "자바 프로그래밍";
String newStr = subject.replace("자바", "JAVA");
```

- **문자열 잘라내기**: substring()사용

```java
String subejct = "0102030403";
String cutNumber = subject.substring(0,6); //인덱스0부터 5까지
```

- **문자열 분해**: split()사용

```java
String subejct = "자바, 프로그래밍";
String[] arr = subject.split(",");
```

- **배열**: 변수에 여러 값을 저장할 때 사용. 

(같은 타입의 값만 저장, 배열의 길이는 늘리거나 줄일 수 없음)

실수, 정수, 문자열 타입별로 배열을 선언할 수 있고, null로 초기화도 가능하다.

```java
int[] intArray;
double[] doubleArray;
String[] stringArray;

타입[] 변수 = null;

season[1] = "여름"
//대입 연산자로 배열의 값을 바꿀 수도 있음
```

배열 변수를 선언한 후에는 값을 변수에 대입할 수 없고, new타입[]을 사용해야 한다.

```java
타입[] 변수;
변수 = {10,20,30}; //컴파일 에러

변수 = new타입[]{10,20,30,40}
```

- **배열 복사**

System.arraycopy(원본배열, 원본배열 복사 시작 인덱스, 새 배열, 새 배열 붙여넣기 시작 인덱스, 항목 수)


