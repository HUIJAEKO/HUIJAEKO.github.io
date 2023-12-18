---
layout: single
title: "조건문과 반복문"
categories: java
---

- if문

```java
public static void main(String[] args){
  int score = 93;

  if(score>=90){
    System.out.println("점수가 90점보다 큽니다")
  }
}
```

- switch문

```java
public static void main(String[] args){
  int num = (int)(Math.random()*6)+1;

  switch(num){
    case1:
      System.out.println("1번이 나왔습니다");
      break;
    case2:
      System.out.println("2번이 나왔습니다");
      break;
    case3:
      System.out.println("3번이 나왔습니다");
   }
}
```

- for문

```java
public static void main(String[] args){
  int sum = 0;
  for(int i=0; i<=100; i++){
    sum = sum+1;
  }
}
```

- while문

```java
public static void main(String[] args){
  int i = 1;
  while(i<=10){
    System.out.print(i);
    i++;
  }
}
```


