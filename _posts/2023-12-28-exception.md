---
layout: single
title: "예외처리"
categories: java
---

### 예외 : 잘못된 사용 또는 코딩으로 인한 오류. 오류에는 두 가지 종류가 있다.

1. 일반 예외: 컴파일러가 예외 처리 코드 여부를 검사하는 예외

2. 실행 예외: 컴파일러가 예외 처리 코드 여부를 검사하지 않는 예외

- 예외 처리 코드: 예외가 발생했을 때 프로그램의 갑작스러운 종료를 막고 정상 실행을 유지할 수 있도록 처리하는 코드'

`try-catch-finally` 블록으로 이루어진다.

```java
try{
  //예외 발생 가능 코드
}catch(예외클래스 e){
  //예외 처리
}finally{
  //항상 실행
}
```

`try`블록에서 예외가 발생하면 `catch`블록이 실행되고, `finally`블록은 마지막에 항상 실행되며 생략 가능하다.

예를 들어 아래의 코드를 보자

```java
public class ExceptionExample1 {
	public static void printLength(String data) {
		try{
			int result = data.length();
			System.out.println(result);
		}catch(NullPointerException e){
			System.out.println(e.getMessage());
		}finally {
			System.out.println("마무리\n");
		}
	}

	public static void main(String[] args) {
		System.out.println("시작\n");
		printLength(null);
		printLength("안녕안녕");
		System.out.println("종료");
		
	}
}
```

이 코드는 문자 수를 확인하는 코드이다. 따라서 null이 들어올 시 `NullPointException` 이 발생한다.

예외가 발생하면 코드 실행이 바로 종료되지만, 예외 처리를 해주어서 `catch` 블록이 실행된 뒤, 정상적으로 실행된다.

예외 정보 출력에는 3가지가 있다.

1. `e.getMessage()`: 예외가 발생한 이유만 리턴

2. `e.toString()`: 예외의 종류까지 리턴

3. `e.printStachTrace()`: 예외가 어디서 발생했는지 추적한 내용까지 리턴


처리해야 할 예외 클래스들이 상속관계에 있을 때는 하위 클래스 catch블록 먼저 작성해야 한다.

```java
try{
  //NumberFormatException발생
}catch(NumberFormatException e){
  //예외 처리1
}catch(Exception e){
  //예외 처리2
}
```

표준 라이브러리에 없는 예외가 많기 때문에 직접 예외 클래스를 정의해서 사용해야 할 때도 많다.

예를 들어 잔고 부족 예외처리가 있다.

예외 클래스를 선언하기 위해서는 먼저 예외를 선언한 뒤 두 개의 생성자를 선언해준다.

```java
public class InsufficientException extends Exception{
	public InsufficientException() {
	}
	
	public InsufficientException(String message){
		super(message);
	}
}
```

이 후 예외를 발생시켜서 호출한 곳으로 예외를 떠넘긴다.

```java
public class Account {
	private long balance;
	
	
	public long getBalance() {
		return balance;
	}
	
	public void deposit(int money) {
		balance+=money;
	}
	
	public void withdraw(int money) throws InsufficientException{//예외 떠넘기기
		if (balance<money) {
			throw new InsufficientException("잔고부족:" + (money-balance) + "모자람");//예외 발생
		}
		balance-=money;
	}
}
```

이후 메인 클래스에서 선언한 예외를 사용한다.

```java
public class AccountExample {

	public static void main(String[] args) {
		Account account = new Account();
		
		account.deposit(10000);
		System.out.println("예금액은" + account.getBalance());
		
		try {
			account.withdraw(30000);
		}catch(InsufficientException e){//선언한 예외 사용
			String message = e.getMessage();
			System.out.println(message);
		}
	}
}
```











