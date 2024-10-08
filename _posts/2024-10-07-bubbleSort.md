---
layout: single
title: "버블 정렬(Bubble Sort)"
categories: algorithm
---

## 버블 정렬이란?

버블 정렬은 배열을 정렬하는 간단한 알고리즘 중 하나이다. 

인접한 두 요소를 비교하여 교환하는 방식으로 동작하며, 이 과정을 반복하면서 가장 큰 또는 작은 값이 배열의 끝으로 이동하는 정렬 알고리즘이다.

## 버블 정렬의 동작 원리

1. 배열의 첫 번째 요소부터 인접한 두 개의 요소를 비교한다.

2. 앞의 요소가 뒤의 요소보다 크면 두 요소의 위치를 교환한다.

3. 배열의 끝까지 이 과정을 반복한다. 이 과정을 한 번 끝내면 가장 큰 요소가 배열의 끝으로 이동하게 된다.

4. 같은 과정을 반복하는데, 매번 끝부분에서 정렬된 요소를 제외한 나머지 부분에서만 다시 비교한다.

5. 배열이 정렬될 때까지 이 과정을 반복한다.

### 예시

예시 배열: [5, 2, 9, 1, 5, 6]

**1차 패스**

[5, 2, 9, 1, 5, 6] -> [2, 5, 9, 1, 5, 6] (5와 2를 교환)

[2, 5, 9, 1, 5, 6] -> [2, 5, 9, 1, 5, 6] (5와 9는 교환 없음)

[2, 5, 9, 1, 5, 6] -> [2, 5, 1, 9, 5, 6] (9와 1을 교환)

[2, 5, 1, 9, 5, 6] -> [2, 5, 1, 5, 9, 6] (9와 5를 교환)

[2, 5, 1, 5, 9, 6] -> [2, 5, 1, 5, 6, 9] (9와 6을 교환)

1차 패스 후: [2, 5, 1, 5, 6, 9] (가장 큰 숫자 9가 끝으로 이동)

**2차 패스**

[2, 5, 1, 5, 6, 9] -> [2, 5, 1, 5, 6, 9] (2와 5는 교환 없음)

[2, 5, 1, 5, 6, 9] -> [2, 1, 5, 5, 6, 9] (5와 1을 교환)

[2, 1, 5, 5, 6, 9] -> [2, 1, 5, 5, 6, 9] (5와 5는 교환 없음)

[2, 1, 5, 5, 6, 9] -> [2, 1, 5, 5, 6, 9] (6과 9는 이미 정렬됨)

2차 패스 후: [2, 1, 5, 5, 6, 9]

**3차 패스**

[2, 1, 5, 5, 6, 9] -> [1, 2, 5, 5, 6, 9] (2와 1을 교환)

[1, 2, 5, 5, 6, 9] -> [1, 2, 5, 5, 6, 9] (나머지는 이미 정렬됨)

3차 패스 후: [1, 2, 5, 5, 6, 9]

**4차 패스** 

이미 정렬되어 있으므로, 교환이 일어나지 않는다.

## 시간 복잡도

버블 정렬의 시간 복잡도는 비교 및 교환 연산의 횟수를 기준으로 분석할 수 있다. 이 알고리즘은 배열의 모든 요소를 반복하면서 인접한 두 요소를 비교하고, 필요할 경우 교환하는 방식으로 작동하므로, 요소의 개수에 따라 수행해야 하는 작업의 수가 달라진다.

### 1. 최악의 경우 : O($N^2$)

버블 정렬의 최악의 경우는 배열이 완전히 역순으로 정렬되어 있는 경우이다. 이 경우, 모든 요소를 비교하고 교환해야 하므로 연산의 횟수가 가장 많다.

배열의 크기가 N일 때, 다음과 같은 과정이 진행된다

- 첫 번째 패스에서 배열의 마지막까지 N−1번 비교하여 가장 큰 값을 배열 끝으로 이동시킨다.

- 두 번째 패스에서 N−2번 비교하여 그 다음 큰 값을 배열의 끝에서 두 번째 위치로 이동시킨다.

- 이 과정을 반복하여, 마지막 요소를 제외한 모든 요소를 정렬한다.

따라서, 비교의 횟수는 다음과 같다.

![bubble](/images/bubble.png)

이는 시간 복잡도로 O($N^2$)이다.

### 2. 최선의 경우 : O(N)

버블 정렬의 최선의 경우는 배열이 이미 정렬된 상태인 경우이다. 이 경우, 첫 번째 패스에서 비교는 이루어지지만 교환이 일어나지 않는다.

최선의 경우, 한 번의 패스만으로 정렬이 완료되었음을 인식하고 알고리즘이 종료된다. 따라서 최선의 경우에는 N−1번의 비교만 필요하므로 시간 복잡도는 O(N)이다.

### 3. 평균적인 경우 : O($N^2$)

버블 정렬의 평균적인 경우는 배열이 무작위로 정렬된 상태이다. 이 경우에도 최악의 경우와 비슷하게 N(N−1)/2번의 비교가 발생할 수 있으며, 교환도 상당수 이루어진다.

따라서 평균적인 경우의 시간 복잡도는 최악의 경우와 동일하게 O($N^2$)이다.

## 백준2750

![2750](/images/2750.png)

```java
import java.util.Scanner;

public class Sort {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        int N = input.nextInt();  // 입력받을 숫자의 개수
        int[] arr = new int[N];   // 숫자를 저장할 배열

        // 입력된 숫자를 배열에 저장
        for (int i = 0; i < N; i++) {
            arr[i] = input.nextInt();
        }

        // 버블 정렬 구현
        for (int i = 0; i < N - 1; i++) {
            for (int j = 0; j < N - 1 - i; j++) {  // 이미 정렬된 부분을 제외하기 위해 N - 1 - i
                if (arr[j] > arr[j + 1]) {  // 인접한 두 요소 비교
                    // 두 요소를 교환
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                }
            }
        }

        // 정렬된 결과 출력
        for (int i = 0; i < N; i++) {
            System.out.println(arr[i]);
        }
    }
}
```

![2750result](/images/2750result.png)

