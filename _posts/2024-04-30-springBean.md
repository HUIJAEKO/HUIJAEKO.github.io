---
layout: single
title: "인프런 스프링 입문(스프링빈과 웹MVC개발)"
categories: spring
---

#### `이 포스트는 인프런 김영한님의 강의 및 자료를 사용합니다`

## 스프링 빈과 의존관계

이전 포스트를 보면 알 수 있는 것처럼 스프링의 기본 구조는 다음과 같다.

![structure](/images/structure.png)

따라서, `Controller`에서 `Service`와 `Repository`를 사용할 수 있도록 해주어야하고, 이를 위해서 `Service`와 `Repository`를 스프링 빈으로 등록해주어야한다.

먼저, 등록을 하기 전에 코드를 짜보자.

```java
@Controller
public class MemberController {
    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
    this.memberService = memberService;
  }
}
```

여기서 `@AutoWired`라는 것은 연관된 객체를 스프링 컨테이너에서 찾아서 주입해주는 기능을 한다. 여기서 중요한 개념인 `DI(Dependency Injection)`가 등장하게 된다.

**DI란?**

- DI(Dependency Injection, 의존성 주입)은 소프트웨어 설계 패턴 중 하나로, 컴포넌트 간의 의존성을 외부에서 주입하는 방식을 말한다. 이 패턴은 객체가 필요로 하는 의존성(다른 객체나 서비스)을 직접 생성하지 않고, 외부(프레임워크나 컨테이너)에서 제공받아 사용하게 된다.

하지만 이대로 실행을 하게 되면, `Consider defining a bean of type 'hello.hellospring.service.MemberService' in your configuration`라는 오류가 발생하게 된다.

이유는 `memberService`가 스프링 빈으로 등록되어있지 않기 때문에, `Controller`에서 `Service`를 주입할 수 없기 때문이다.

![bean](/images/nobean.png)

이제 스프링 빈으로 등록을 해보자. 스프링 빈으로 등록하는 방법에는 두 가지가 있다.

## 1. 컴포넌트 스캔

첫번째는 상대적으로 더 많이 사용하는 컴포넌트 스캔이다. 

원리를 설명해보자면, `@Component` 어노테이션이 있으면 스프링 빈으로 자동 등록된다.

따라서 `@Component` 를 포함하는 아래와 같은 어노테이션들은 스프링 빈으로 자동 등록된다.

- `@Controller`

- `@Service`

- `@Repository`

이를 실제로 아래와 같이 사용할 수 있다.

```java
@Controller
public class MemberController {
    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
    this.memberService = memberService;
  }
}
```
```java
@Service
public class MemberService {
    private final MemberRepository memberRepository;
    @Autowired
    public MemberService(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
    }
}
```
```java
@Repository
public class MemoryMemberRepository implements MemberRepository {}
```

## 2. 자바 코드로 직접 등록

다음은 컴포넌트 스이 아닌 자바 코드로 직접 등록하는 방법이다.

먼저 코드를 보자.

```java
@Configuration
public class SpringConfig {
  @Bean
  public MemberService memberService() {
  return new MemberService(memberRepository());
  }
  @Bean
  public MemberRepository memberRepository() {
    return new MemoryMemberRepository();
  }
```

`@Configuration` 어너테이션이 클래스에 적용되어 있다. 이는 해당 클래스가 스프링의 설정 정보를 담고 있는 클래스임을 나타내며, 스프링 컨테이너가 이 클래스를 구성 정보로 사용하게 된다.

`public MemberService memberService()` 메서드에 `@Bean` 애너테이션이 붙어 있습니다. 이는 메서드가 실행되어 반환하는 객체를 스프링 컨테이너의 빈으로 등록하게 된다.

`public MemberRepository memberRepository()` 메서드도 위와 동일하다.

언뜻 봐도 알 수 있는 것처럼 코드로 직접 등록하는 것보다는 컴포넌트 스캔을 이용하는 것이 훨씬 편리하다. 

그렇다면 코드로 직접 등록하는건 어떤 장점이 있을까?

- 명확성 및 제어: 자바 코드로 직접 빈을 등록하면, 어떤 빈이 어떻게 구성되어 있는지 명확하게 보여줄 수 있다. 또한, 빈의 생성과 의존성 주입 과정을 완벽히 제어할 수 있어 복잡한 조건이나 특정 상황에 따라 다르게 빈을 설정해야 할 때 유용하다.

- 조건부 빈 등록: 특정 조건에 따라 다른 빈을 등록해야 할 때, 이를 쉽게 처리할 수 있다.

- 동적 빈 등록: 실행 시점에 의존성을 결정하거나 여러 구성 옵션 중에서 선택해야 하는 경우, 자바 코드로 빈을 정의하면 이러한 동적인 설정을 더 쉽게 처리할 수 있다.

의존성 주입과 컴포넌트 스캔은 이후 강의에서 자세히 다룰 예정이다.

의존성 주입이 가능해졌으니, 다음으로 이를 이용하여 여러 기능들을 추가해준다.

크게 홈 화면 추가, 회원 등록, 회원 조회의 기능을 추가한다.

## 1. 홈 화면 추가

먼저, 홈 화면 추가이다. 단순히 초기화면을 설정하는 메서드이기 떄문에 간단하다.

```java
@Controller
public class HomeController {
  @GetMapping("/")
  public String home() {
    return "home";
  }
}
```

위의 코드에서는 `@GetMapping("/")` 어노테이션을 이용한다. 여기서는 `/` 경로에 대한 GET 요청을 처리한다. 즉, 처음 사이트에 들어갔을 때 초기화면을 설정해줄 수 있다.

`return "home"`으로 설정하였기 때문에 이 메서드는 `home.html`파일을 반환하게 된다.

현재 `home.html`에는 회원 가입과 회원 목록 버튼을 만들어 두었다.

## 2. 회원 가입

다음은 회원가입이다. 먼저, 회원가입을 하기 위해서는 회원 가입 폼으로 이동해야한다. 따라서 위와 동일하게 `GetMapping`을 이용한다.

```java
@Controller
public class MemberController {
  private final MemberService memberService;
  @Autowired
  public MemberController(MemberService memberService) {
    this.memberService = memberService;
  }
  @GetMapping(value = "/members/new")
  public String createForm() {
    return "members/createMemberForm";
  }
}
```

회원가입 폼으로 이동하여 정보를 입력하고 완료 버튼을 누르게되면, `POST`요청을 하게 된다.

이는 `html`파일에 `<form action="/members/new" method="post">`라는 폼 태그가 있기 때문이다. 

결국, 완료 버튼을 누르면 `POST` 방식으로 `/members/new` 경로로 데이터를 전송하는 폼을 생성한다.


```java
@PostMapping(value = "/members/new")
public String create(MemberForm form) {
  Member member = new Member();
  member.setName(form.getName());
  memberService.join(member);
  return "redirect:/";
}
```

`@PostMapping(value = "/members/new")` 어노테이션은 POST 요청을 처리한다. 여기서는 `/members/new` 경로에 대한 POST 요청을 처리한다.

`public String create(MemberForm form)` 메서드는 `MemberForm` 객체를 매개변수로 받아들이며, 경로에 대한 POST 요청을 처리한다. 따라서 새 회원을 생성하는 역할을 한다.

이후 `Member`객체를 만들어서 이름을 `MemberForm`에서 가져온 이름으로 설정한다.

마지막으로 `memberService.join(member)`은 `MemberService`클래스의 `join` 메서드를 호출하여 새 회원을 등록하며, 

`return "redirect:/"`를 통하여 작업 완료 후 사용자를 `/` 경로로 리다이렉트하여 홈 페이지로 이동한다.

## 3. 회원 조회

다음은 회원 조회이다. 아래는 회원 목록을 조회하고, 이를 뷰로 전달하여 사용자에게 보여주는 메서드이다.

```java
@GetMapping(value = "/members")
public String list(Model model) {
  List<Member> members = memberService.findMembers();
  model.addAttribute("members", members);
  return "members/memberList";
}
```

`List<Member> members = memberService.findMembers()`는 `MemberService` 클래스의 `findMembers` 메서드를 호출하여 회원 목록을 가져온다.

가져온 목록을 `model.addAttribute("members", members)`를 통해 Model에 members라는 이름으로 회원 목록을 추가한다. 이를 통해 뷰에서 회원 목록에 접근할 수 있게 된다.

마지막으로 `return "members/memberList"`를 통하여 `members/memberList`뷰가 렌더링되어 실제로 사용자에게 회원 목록이 보여지게 된다.










