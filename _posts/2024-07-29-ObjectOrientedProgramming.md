---
layout: single
title: "좋은 객체 지향 설계의 5가지 원칙"
categories: spring
---

#### `이 포스트는 인프런 김영한님의 강의 및 자료를 사용합니다`

## 좋은 객체 지향 설계의 원칙은?

#### Single Responsibility Principle(단일 책임 원칙) 

- 한 클래스는 하나의 책임만 가져야 한다.

#### Open/Closed Principle(개방-폐쇄 원칙)

- 소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다.

#### Liskov Subsitution Principle(리스코프 치환 원칙)

- 프로그램의 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 바꿀 수 있어야 한다.

#### Interface Segregation Principle(인터페이스 분리 원칙)

- 특정 클라이언트를 위한 인터페이스 여러 개가 범용 인터페이스 하나보다 낫다.

#### Dependency Inversion Principle(의존관계 역전 원칙)

- 프로그래머는 추상화에 의존해야지, 구체화에 의존하면 안된다.

####  이제, 이 원칙에 따라 코드를 작성해보겠다.

## 비즈니스 요구사항과 설계

**회원**

- 회원을 가입하고 조회할 수 있다.

- 회원은 일반과 VIP 두 가지 등급이 있다.

- 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)

**주문과 할인 정책**

- 회원은 상품을 주문할 수 있다.

- 회원 등급에 따라 할인 정책을 적용할 수 있다.

- 할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액 할인을 적용해달라. (나중에 변경 될 수 있다.)

- 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루
고 싶다. 최악의 경우 할인을 적용하지 않을 수 도 있다. (미확정)

## 회원 도메인 설계

![MemberDiagram](/images/MemberDiagram.png)

```java
//회원등급
public enum Grade {
 BASIC,
 VIP
}

//회원 엔티티
public class Member {
 private Long id;
 private String name;
 private Grade grade;

//회원 저장소 인터페이스
public interface MemberRepository {
 void save(Member member);
 Member findById(Long memberId);
}

//메모리 회원 저장소 구현체
public class MemoryMemberRepository implements MemberRepository {
 private static Map<Long, Member> store = new HashMap<>();
 @Override
 public void save(Member member) {
 store.put(member.getId(), member);
 }
 @Override
 public Member findById(Long memberId) {
 return store.get(memberId);
 }
}

//회원 서비스 인터페이스
public interface MemberService {
 void join(Member member);
 Member findMember(Long memberId);
}

//회원 서비스 구현체
public class MemberServiceImpl implements MemberService {
 private final MemberRepository memberRepository = new MemoryMemberRepository();
 public void join(Member member) {
  memberRepository.save(member);
 }
 public Member findMember(Long memberId) {
  return memberRepository.findById(memberId);
 }
}
```

## 주문과 할인 도메인 설계

![OrderDiagram](/images/OrderDiagram.png)

```java
//할인 정책 인터페이스
public interface DiscountPolicy {
 int discount(Member member, int price);
}

//정액 할인 구현체
public class FixDiscountPolicy implements DiscountPolicy {
 private int discountFixAmount = 1000; //1000원 할인
 @Override
 public int discount(Member member, int price) {
  if (member.getGrade() == Grade.VIP) {
   return discountFixAmount;
  } else {
   return 0;
  }
 }
}

//주문 엔티티
public class Order {
 private Long memberId;
 private String itemName;
 private int itemPrice;
 private int discountPrice;

 public int calculatePrice() {
  return itemPrice - discountPrice;
 }
}

//주문 서비스 인터페이스
public interface OrderService {
 Order createOrder(Long memberId, String itemName, int itemPrice);
}

//주문 서비스 구현체
public class OrderServiceImpl implements OrderService {
 private final MemberRepository memberRepository = new MemoryMemberRepository();
 private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
 @Override
 public Order createOrder(Long memberId, String itemName, int itemPrice) {
  Member member = memberRepository.findById(memberId);
  int discountPrice = discountPolicy.discount(member, itemPrice);
  return new Order(memberId, itemName, itemPrice, discountPrice);
 }
}
```

여기까지 기본적인 설계를 끝냈다. 

하지만, 만약 새로운 할인 정책을 도입해야 한다면 어떻게 해야할까?

우리는 `DiscountPolicy`라는 인터페이스를 만들어놓았기 때문에, 단순히 이를 구현하는 `RateDiscountPolicy`를 작성하면 문제가 해결된다.

```java
//새로운 할인 정책
public class RateDiscountPolicy implements DiscountPolicy {
 private int discountPercent = 10; //10% 할인
 @Override
 public int discount(Member member, int price) {
  if (member.getGrade() == Grade.VIP) {
   return price * discountPercent / 100;
  } else {
   return 0;
  }
 }
}
```

## 새로운 할인 정책 적용의 문제점?

그렇다면, 새로운 할인 정책을 적용해보겠다. `OrderServiceImpl`을 수정해주면 된다.

```java
public class OrderServiceImpl implements OrderService {
// private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
 private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
}
```

이렇게 적용을 해보면, 간단하게 적용이 되기 때문에 문제가 없어 보인다. 왜냐하면!

- 인터페이스와 구현을 분리하였고, 다형성까지 잘 활용하였기 때문이다.

- 하지만, OCP, DIP 같은 원칙을 충실히 준수하지는 못하였다.

이를 이미지를 이용하여 설명해보겠다.

먼저 DIP 위반이다. 위에서 주문클래스 다이어그램을 보게 되면, `OrderServiceImpl`클래스는  `DiscountPolicy` 인터페이스만 의존한다고 생각된다.

하지만, 실제 의존관계를 보게 되면 아래와 같다.

![RealOrderDiagram](/images/RealOrderDiagram.png)

코드를 자세히 봐보면, `OrderServiceImpl` 이 `DiscountPolicy` 인터페이스 뿐만 아니라 `FixDiscountPolicy` 인 구체 클래스도 함께 의존하고 있다. 따라서 DIP를 위반한다.

다음은 OCP 위반이다.

아래 그림과 코드를 보면 알 수 있듯이, `FixDiscountPolicy` 를 `RateDiscountPolicy` 로 변경하는 순간 `OrderServiceImpl`의 코드도 함께 변경해야 한다. 따라서 OCP 위반이다.

![OCPDiagram](/images/OCPDiagram.png)

## 문제 해결

문제 해결은 생각보다 간단하다.

`OrderServiceImpl`이 인터페이스에만 의존하면 된다.

```java
public class OrderServiceImpl implements OrderService {
 //private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
 private DiscountPolicy discountPolicy;
}
```

하지만 이렇게 코드를 변경하면, 구현체가 없기 때문에 null pointer exception이 발생한다.

따라서, 누군가가 구현 객체를 대신 생성하고 주입해주어야 한다.

이 누군가는 이제 `AppConfig`로 설정해보겠다.

그렇게 되면 이제 다이어그램은 아래와 같이 변한다.

![AppConfig](/images/AppConfig.png)

생성자 주입을 해주어야 하기 때문에 당연히 `MemberServiceImpl` 과 `OrderServiceImpl`클래스에는 생성자가 하나 만들어져야 한다.

```java
//멤버서비스 구현체
public class MemberServiceImpl implements MemberService {
 private final MemberRepository memberRepository;
 public MemberServiceImpl(MemberRepository memberRepository) {
  this.memberRepository = memberRepository;
 }
 public void join(Member member) {
  memberRepository.save(member);
 }
 public Member findMember(Long memberId) {
  return memberRepository.findById(memberId);
 }
}

//주문 서비스 구현체
public class OrderServiceImpl implements OrderService {
 private final MemberRepository memberRepository;
 private final DiscountPolicy discountPolicy;
 public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
  this.memberRepository = memberRepository;
  this.discountPolicy = discountPolicy;
 }
 @Override
 public Order createOrder(Long memberId, String itemName, int itemPrice) {
  Member member = memberRepository.findById(memberId);
  int discountPrice = discountPolicy.discount(member, itemPrice);
  return new Order(memberId, itemName, itemPrice, discountPrice);
 }
}
```

그리고 이제 `AppConfig`를 작성하면 된다.

```java
//AppConfig
public class AppConfig {
 public MemberService memberService() {
  return new MemberServiceImpl(memberRepository());
 }
 public OrderService orderService() {
  return new OrderServiceImpl(memberRepository(), discountPolicy());
 }
 public MemberRepository memberRepository() {
  return new MemoryMemberRepository();
 }
 public DiscountPolicy discountPolicy() {
// return new FixDiscountPolicy();
  return new RateDiscountPolicy();
 }
}
```

이제 할인 정책을 변경해도, 애플리케이션의 구성 역할을 담당하는 `AppConfig`만 변경하면 된다. 클라이언트 코드인 `OrderServiceImpl`을 포함해서 사용 영역의 어떤 코드도 변경할 필요가 없다.

## 정리

#### 새로운 할인 정책 개발

- 다형성 덕분에 새로운 정률 할인 정책 코드를 추가로 개발하는 것 자체는 아무 문제가 없음

#### 새로운 할인 정책 적용과 문제점

- 새로 개발한 정률 할인 정책을 적용하려고 하니 클라이언트 코드인 주문 서비스 구현체도 함께 변경해야함

- 주문 서비스 클라이언트가 인터페이스인 `DiscountPolicy` 뿐만 아니라, 구체 클래스인 `FixDiscountPolicy`도 함께 의존, DIP 위반

#### AppConfig등장

- AppConfig는 애플리케이션의 전체 동작 방식을 구성하기 위해, 구현 객체를 생성하고, 연결하는 책임

- 이제부터 클라이언트 객체는 자신의 역할을 실행하는 것만 집중, 권한이 줄어듬

#### 새로운 구조와 할인 정책 적용

- 정액 할인 정책 -> 정률 할인 정책으로 변경

- AppConfig의 등장으로 애플리케이션이 크게 사용 영역과, 객체를 생성하고 구성하는 영역으로 분리

- 할인 정책을 변경해도 AppConfig가 있는 구성 영역만 변경하면 됨, 사용 영역은 변경할 필요가 없음. 물론 클라이언트 코드인 주문 서비스 코드도 변경하지 않음

## Ioc, DI

### 제어의 역전 IoC(Inversion of Control)

- 기존 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성하고, 연결하고, 실행했다. 한마디로 구현 객체가 프로그램의 제어 흐름을 스스로 조종했다.

- 반면에 AppConfig가 등장한 이후에 구현 객체는 자신의 로직을 실행하는 역할만 담당한다. 프로그램의 제어 흐름은 이제 AppConfig가 가져간다. 예를 들어서 OrderServiceImpl 은 필요한 인터페이스들을 호출하지만 어떤 구현 객체들이 실행될지 모른다. 

- 프로그램에 대한 제어 흐름에 대한 권한은 모두 AppConfig가 가지고 있다. 심지어 OrderServiceImpl 도 AppConfig가 생성한다. 그리고 AppConfig는 OrderServiceImpl 이 아닌 OrderService 인터페이스의 다른 구현 객체를 생성하고 실행할 수 도 있다. 그런 사실도 모른체 OrderServiceImpl 은 묵묵히 자신의 로직을 실행할 뿐이다.

- 이렇듯 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 제어의 역전(IoC)이라 한다

### 의존관계 주입 DI(Dependency Injection)

- OrderServiceImpl 은 DiscountPolicy 인터페이스에 의존한다. 실제로 어떤 구현 객체가 사용될지는 모른다.
 
- 의존관계는 정적인 클래스 의존 관계와, 실행 시점에 결정되는 동적인 객체 의존 관계를 분리해서 생각해야 한다. 

#### 정적인 클래스 의존관계

클래스가 사용하는 import 코드만 보고 의존관계를 쉽게 판단할 수 있다. 여기서는, 단순히 OrderServiceImpl는 MemberRepository , DiscountPolicy 에 의존한다는 것을 알 수 있다. 하지만, 실제 어떤 객체가 OrderServiceImpl 에 주입 될지 알 수 없다.

#### 동적인 객체 인스턴스 의존 관계

애플리케이션 실행 시점에 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계다.

애플리케이션 실행 시점에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언트와 서버의 실제 의존관계가 연결 되는 것을 의존관계 주입이라 한다.

객체 인스턴스를 생성하고, 그 참조값을 전달해서 연결된다. 의존관계 주입을 사용하면 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있다.































