---
layout: single
title: "스프링 시큐리티에서 프론트로 정보 보내기"
categories: spring
---

구현하고 싶은 기능은 아래 사진을 보면 바로 알 수 있다.

![로그인](images/loginName.png)

원래 처음 프로젝트를 만들때는 스프링 시큐리티를 이용하지 않았다.

따라서,

```java
@PostMapping("/login")
    public String login(@ModelAttribute UserDTO userDTO,HttpSession session, Model model) {
        UserDTO loginResult = userService.login(userDTO);  
        if (loginResult != null) {
            session.setAttribute("loginInformation", loginResult);
            model.addAttribute("loginName", loginResult.getName());
            return "main";
```

이런식으로 로그인을 성공했을 때 **`DTO`와 `ENTITY`** 를 이용해 **`model`** 에 이름을 넣어 뷰 템플릿에서 사용할 수 있었다.

그런데 스프링 시큐리티를 이용하니 방식을 비슷하지만 다르게 해야했다.

결론은 이렇다.

```java
@GetMapping("/main")
	public String LoginSuccess(Model model) {
		Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
		Object principal = authentication.getPrincipal();
		if(principal instanceof CustomUserDetails) {
			CustomUserDetails userDetails = (CustomUserDetails) principal;
		  model.addAttribute("username", userDetails.getName());
		}
		return "main";
	}
```

스프링 시큐리티는 로그인 성공 시 **`SecurityContextHolder`** 에 정보를 저장한다.

따라서 **`getContext()`** 메서드를 호출하여 현재 보안 컨텍스트를 가져온 뒤, **`getAuthentication()`** 메서드를 호출하여 현재 인증된 사용자의 **`Authentication`** 객체를 가져온다.

그리고 인증 객체에서 **`getPrincipal()`** 메서드를 호출하여, 인증된 사용자의 정보를 가져온다.

이후, 가져온 주요 정보 객체가 **`CustomUserDetails`** 클래스의 인스턴스인지 확인한다. 

맞다면, 주요 정보 객체를 **`CustomUserDetails`** 타입으로 변환한다. 이를 통해 **`CustomUserDetails`** 클래스에 정의된 메서드를 사용할 수 있게 된다.

마찬가지로, **`model`** 을 이용하여 사용자 이름을 불러 뷰 템플릿으로 넘겨서 사용한다.
