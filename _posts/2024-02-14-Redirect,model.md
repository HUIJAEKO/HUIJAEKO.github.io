---
layout: single
title: "Model에 username을 넣고 Redirect를 하려 했는데 안되는 현상"
categories: spring
---

운동메이트 만들기 프로젝트를 진행하던 중 로그인을 했을 때 메인 화면에 이름을 띄우게 하고싶었다.

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
```

기존에 이런 식으로 코드를 짰었다. 하지만 로그인 성공 후 메인 페이지로 이동할 때 `loginName`에 자꾸 null이 떴다.

- **원인**

`redirect:/main`을 사용할 때, `redirect`는 새로운 HTTP 요청을 서버에 보내 `/main` 경로로 이동하게 한다. 
이 과정에서 `Model`에 추가된 속성들은 유지되지 않는다.
`Model`에 추가된 속성들은 해당 요청 동안만 유효하고, 리다이렉트를 통해 새로운 요청이 발생하면 속성들은 소실된다. 
따라서, `model.addAttribute("loginName", loginResult.getName())`에 의해 추가된 `loginName` 속성은 `/main`페이지로 리다이렉트된 후에는 사용할 수 없다.

- **해결방안**

리다이렉트를 뺄 수 있으면 빼고, 써야하는 상황이면 다음과 같이 GET요청을 수정한다.

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

**기본기의 중요성을 한 번 더 느끼게 됩니다.**

