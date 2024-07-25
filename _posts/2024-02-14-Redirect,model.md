---
layout: single
title: "Model객체 사용이유, 그리고 model객체에 username을 넣고 Redirect를 하려 했는데 안되는 현상"
categories: spring
---

## Model 객체를 사용하는 이유?

컨트롤러에서 생성하거나 조회한 데이터를 뷰에 전달하여, 사용자에게 데이터를 보여주거나 사용자 입력을 위한 폼 데이터를 제공하기 위해서이다.

주로 `model.addAttribute`를 이용한다.

```java
@Controller
public class UserController {

    @GetMapping("/user")
    public String getUser(Model model) {
        User user = new User("HJ", "HJ@naver.com"); 
        model.addAttribute("username", user.getUsername());
        return "userProfile"; 
    }
}
```

이 예시에서는 /user 경로로 GET 요청이 들어올 때, 모델에 `username`라는 이름으로 유저의 이름을 추가한다. 그 후, `userProfile` 뷰 이름을 반환한다. 이후 `Thymeleaf`와 같은 뷰 템플릿에서 `username`을 사용한다.

```html
<span th:text="${username} + '님, 환영합니다!'">사용자님, 환영합니다!</span>
//로그인 성공 시 ~님 환영합니다 반환
```

**@ModelAttribute**: 스프링 MVC에서 모델 데이터를 뷰에 전달하거나 요청 파라미터를 객체에 바인딩할 때 사용하는 어노테이션

이 어노테이션은 크게 메소드 파라미터에 적용하는 방식과 메소드 레벨에 적용하는 방식으로 사용된다.

메소드 파라미터에 적용하는 방식만 사용해봐서, 지금은 이 방식만 예시를 들어 설명해보겠다.

```java
@PostMapping("/signup")
public String SignUp(@ModelAttribute User user) {
    // user 객체에는 폼으로부터 전송된 데이터가 바인딩되어 있음
    userRepository.save(user);
    return "main";
}
```

이 예시에서, `/signup`경로로 POST요청을 하게 되면, `@ModelAttribute`를 통해 객체에 자동으로 바인딩된다. 

`@ModelAttribute`를 사용하면, 별도의 요청 파라미터를 하나씩 받아오고, 객체에 수동으로 할당하는 번거로움 없이, 간결한 코드로 객체 바인딩을 처리할 수 있다.

**그럼 둘의 차이는 무엇일까?**

`model.addAttribute`는 컨트롤러 메소드 실행 중에 특정 데이터를 모델에 추가할 때 사용하는 명령적 방식이다.

`@ModelAttribute`는 요청 데이터를 객체에 바인딩하거나 모든 뷰에서 공통으로 사용할 데이터를 모델에 자동으로 추가할 때 사용되는 선언적 방식이다.

### 그리고 이를 이용하여 운동메이트 만들기 프로젝트를 진행하던 중 로그인을 했을 때 메인 화면에 이름을 띄우게 하고싶었다.

```java
@PostMapping("/login")
public String login(@ModelAttribute UserDTO userDTO,HttpSession session, Model model) {
  UserDTO loginResult = userService.login(userDTO);  
  if (loginResult != null) {
    session.setAttribute("loginInformation", loginResult);
    model.addAttribute("loginName", loginResult.getName());
    return "redirect:/main";
  } else {
    model.addAttribute("loginError", "아이디 또는 비밀번호가 일치하지 않습니다.");
    return "redirect:/login";
}
```

기존에 이런 식으로 코드를 짰었다. 하지만 로그인 성공 후 메인 페이지로 이동할 때 `loginName`에 자꾸 null이 떴다.

### 원인

`redirect:/main`을 사용할 때, `redirect`는 새로운 HTTP 요청을 서버에 보내 `/main` 경로로 이동하게 한다. 
이 과정에서 `Model`에 추가된 속성들은 유지되지 않는다.
`Model`에 추가된 속성들은 해당 요청 동안만 유효하고, 리다이렉트를 통해 새로운 요청이 발생하면 속성들은 소실된다. 
따라서, `model.addAttribute("loginName", loginResult.getName())`에 의해 추가된 `loginName` 속성은 `/main`페이지로 리다이렉트된 후에는 사용할 수 없다.

### 해결방안

리다이렉트를 뺄 수 있으면 빼고, 써야하는 상황이면 다음과 같이 `GET`요청을 수정한다.

```java
@GetMapping("/main")
public String main(HttpSession session, Model model) {
    UserDTO loginInformation = (UserDTO) session.getAttribute("loginInformation");
    if (loginInformation != null) {
        model.addAttribute("loginName", loginInformation.getName());
    }
    return "main";
}
```


