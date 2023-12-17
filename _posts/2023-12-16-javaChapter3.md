---
layout: single
title: "calculate"
---

증감 연산자: ++i 변수를 1증가 시킨 후 다른 연산 수행, i++ 연산을 끝낸 후 변수를 1증가

오버플로우, 언더플로우: 타입이 허용하는 최대값을 벗어나는 것, 타입이 허용하는 최소값을 벗어나는 것

**정학한 계산은 정수 연산으로 하는 것이 좋다**

ArithmeticException: 연산값이 intinity이거나 nan일때 발생, Double.isInfinite와 Double.isNaN을 사용해서 확인하는 것이 좋다.

삼항 조건 연산자
```java
public static void main(String[] args){
  int score = 85;
  char grade = (score>90) ? 'A' : (score>80) ? 'B' : 'C');
}
```

**연산의 우선순위도 생각해야한다**
