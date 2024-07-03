---
layout: single
title:  "스택과 큐, 브루트포스 알고리즘"
categories: data structure and algorithm
---

## ADT

- ADT란 추상적 자료구조로, 실제로 프로그래밍 언어들에서 존재하지 않는 자료구조이다. 자료구조의 방법이 코드로 정해진 것이 아닌, 구조의 행동 양식만 정의된 것이다. 스택과 큐가 ADT의 종류이다. 

## Stack

- 스택은 `Last In, First Out (LIFO)` 원칙을 따르는 자료구조이다. 즉, 마지막에 삽입된 데이터가 가장 먼저 제거된다. 스택은 한쪽 끝에서만 데이터를 삽입하고 제거할 수 있다.

- 주요 연산

  - `push`: 스택의 맨 위에 데이터를 삽입하는 연산

  - `pop`: 스택의 맨 위에 있는 데이터를 제거하고 반환하는 연산

  - `peek`: 스택의 맨 위에 있는 데이터를 제거하지 않고 반환하는 연산

  - `isEmpty`: 스택이 비어 있는지를 확인하는 연산

```java
import java.util.ArrayList;
import java.util.List;

class Stack {
    private List<Integer> items = new ArrayList<>();

    public void push(int item) {
        items.add(item);
    }

    public int pop() {
        if (items.isEmpty()) {
            throw new IllegalStateException("Stack is empty");
        }
        return items.remove(items.size() - 1);
    }

    public int peek() {
        if (items.isEmpty()) {
            throw new IllegalStateException("Stack is empty");
        }
        return items.get(items.size() - 1);
    }

    public boolean isEmpty() {
        return items.isEmpty();
    }
}

// 스택 사용 예시
public class Main {
    public static void main(String[] args) {
        Stack stack = new Stack();
        stack.push(1);
        stack.push(2);
        stack.push(3);
        System.out.println(stack.pop());  // 출력: 3
        System.out.println(stack.peek()); // 출력: 2
        System.out.println(stack.isEmpty()); // 출력: false
    }
}
```

- 사용 예시

  - 함수 호출 스택: 프로그램 실행 중 함수 호출을 추적하는 데 사용된다.

  - 역순 문자열: 문자열을 거꾸로 뒤집을 때 사용된다.

## Queue

- 큐는 `First In, First Out (FIFO)` 원칙을 따르는 자료구조이다. 즉, 가장 먼저 삽입된 데이터가 가장 먼저 제거된다. 큐는 두 끝에서 데이터를 삽입과 제거할 수 있다: 한쪽 끝에서 데이터를 제거하고, 다른 쪽 끝에서 데이터를 삽입한다.

- 주요 연산

  - `enqueue`: 큐의 뒤쪽에 데이터를 삽입하는 연산

  - `dequeue`: 큐의 앞쪽에서 데이터를 제거하고 반환하는 연산

  - `peek`: 큐의 앞쪽에 있는 데이터를 제거하지 않고 반환하는 연산

  - `isEmpty`: 큐가 비어 있는지를 확인하는 연산

```java
import java.util.LinkedList;
import java.util.Queue;

class CustomQueue {
    private Queue<Integer> items = new LinkedList<>();

    public void enqueue(int item) {
        items.add(item);
    }

    public int dequeue() {
        if (items.isEmpty()) {
            throw new IllegalStateException("Queue is empty");
        }
        return items.poll();
    }

    public int peek() {
        if (items.isEmpty()) {
            throw new IllegalStateException("Queue is empty");
        }
        return items.peek();
    }

    public boolean isEmpty() {
        return items.isEmpty();
    }
}

// 큐 사용 예시
public class Main {
    public static void main(String[] args) {
        CustomQueue queue = new CustomQueue();
        queue.enqueue(1);
        queue.enqueue(2);
        queue.enqueue(3);
        System.out.println(queue.dequeue());  // 출력: 1
        System.out.println(queue.peek());     // 출력: 2
        System.out.println(queue.isEmpty());  // 출력: false
    }
}
```

- 사용 예시

  - 프린터 작업 대기열: 프린터에 출력할 문서들이 대기하는 데 사용된다.

  - 운영 체제의 작업 스케줄링: CPU가 실행할 프로세스들이 대기하는 데 사용된다.

## 부르트포스 알고리즘

- 브루트포스 알고리즘은 가능한 모든 경우를 전부 탐색하여 문제의 해를 찾는 방법이다. 이 알고리즘은 단순하고 구현하기 쉽지만, 시간 복잡도가 높아 비효율적일 수 있다. 문제의 크기가 커질수록 시간이 급격히 증가한다.

- 예시: 문자열에서 부분 문자열 찾기

```java
public class BruteForceSubstringSearch {
    public static int search(String txt, String pat) {
        int n = txt.length();
        int m = pat.length();

        for (int i = 0; i <= n - m; i++) {
            int j;
            for (j = 0; j < m; j++) {
                if (txt.charAt(i + j) != pat.charAt(j)) {
                    break;
                }
            }
            if (j == m) {
                return i; // 부분 문자열의 시작 인덱스 반환
            }
        }
        return -1; // 부분 문자열을 찾지 못한 경우
    }

    public static void main(String[] args) {
        String txt = "hello world";
        String pat = "world";
        int result = search(txt, pat);
        if (result == -1) {
            System.out.println("부분 문자열을 찾지 못했습니다.");
        } else {
            System.out.println("부분 문자열을 찾았습니다. 시작 인덱스: " + result);
        }
    }
}
```
