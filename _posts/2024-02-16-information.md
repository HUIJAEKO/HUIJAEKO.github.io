---
layout: single
title: "로그인시 클라이언트로 정보 보내기, +++실패핸들러"
categories: spring
---

구현하고 싶은 기능은 아래 사진을 보면 바로 알 수 있다.

![로그인](/images/loginName.png)

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

### 다음은 로그인 실패 핸들러를 만들어보았다.

시행착오가 있었지만 만든 순서대로 작성해보고자 한다.

**1. SecurityConfig에 failureHandler를 추가한다**

```java
http
    .formLogin((auth)->auth.loginPage("/user/login")
        .loginProcessingUrl("/user/login")
        .defaultSuccessUrl("/user/main", true)
        .failureHandler(new CustomAuthenticationFailureHandler())
        .permitAll()
						)
```

가장 먼저 **`.failureHandler(new CustomAuthenticationFailureHandler())`** 을 추가해준다.

이는 스프링 시큐리티에서 인증에 실패했을 시 핸들러를 설정한 것이다.

**`CustomAuthenticationFailureHandler`** 는 **`AuthenticationFailureHandler`** 인터페이스를 구현한 클래스이다. 

이 클래스는 인증 실패 시 실행될 로직을 정의한다. 이를 통해 사용자에게 구체적인 오류 메시지를 보여주거나 인증 실패 등의 원인을 알려줄 수 있다.

**2. CustomAuthenticationFailureHandler를 통해 로그인 실패 시 동작을 커스터마이징한다.**

```java
public class CustomAuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
       
    	String errorCode = "unknownError";
    	if(exception instanceof BadCredentialsException){
    		 errorCode = "badCredentials";
    	}
    	
    	HttpSession session = request.getSession(true);
    	
    	session.setAttribute("loginMessage", errorCode);
    	session.setAttribute("messageType", "error");
    	
    	getRedirectStrategy().sendRedirect(request, response, "/user/login?error=true");
    	
    }
}
```

**`SimpleUrlAuthenticationFailureHandler`** 는 실패 시 사용자를 특정 URL로 리다이렉트하는 기능을 제공하는 클래스이다.

이는 실패 시 사용자를 특정 URL로 리다이렉트하는 기능을 제공하는 클래스이다

**`onAuthenticationFailure`** 메소드는 인증 실패 시 호출된다.

먼저 **`errorCode`** 필드를 만들어준다. 이후, 아이디와 비밀번호가 잘못되면 발생하는 **`BadCredentialsException`** 예외가 발생하면 **`badCredentials`** 를 에러 코드에 할당한다.

이후, 에러 코드와 메시지 유형을 HTTP 세션에 저장하고, **`/user/login?error=true`** 으로 리다이렉션한다.

**3. Controller에서 GET요청을 처리한다.**

```java
@GetMapping("/user/login")
public String login(@RequestParam(value = "error", required = false) Boolean error,
									HttpSession session, Model model) {
if(Boolean.TRUE.equals(error)) {
  String errorCode = (String) session.getAttribute("loginMessage");
  String errorMessage = switch (errorCode) {
    case "badCredentials" -> "아이디 또는 비밀번호가 일치하지 않습니다";
    default -> "알 수 없는 오류가 발생했습니다. 관리자에게 문의해주세요";
  };
  model.addAttribute("loginMessage", errorMessage);
  }
  return "user/login";
}
 ```

가장 먼저 **`error`** 값을 **`Boolean`** 타입으로 받아온다.

**`if(Boolean.TRUE.equals(error))`** : 이 조건문은 error 쿼리 파라미터가 true로 설정되어 있을 때만 실행된다. 

즉, 로그인 과정에서 에러가 발생하고 사용자가 **`/user/login?error=true`**로 리다이렉션된 경우에 해당한다.

그렇다면 세션에서 **`loginMessage`** 속성값을 가져와 **`errorCode`**에 저장한다.

이후 값에 따라 사용자에게 보여줄 에러 메시지를 결정하고 모델에 저장한다.

**4. HTML파일을 수정한다.**

```html
<div th:if="${loginMessage}" th:data-message="${loginMessage}"
    th:data-type="error" id="loginMessage" style="display: none;">
</div>
```

경고문을 보여주기 위해 파일을 수정한다.

**5. JS파일을 수정한다.**

```javascript
document.addEventListener('DOMContentLoaded', function () {
    var messageElement = document.getElementById('loginMessage');
    if (messageElement && messageElement.dataset.message) {
        var message = messageElement.dataset.message;
        var messageType = messageElement.dataset.type; // 'error'

        if (messageType === 'error') {
            Swal.fire({
                title: '로그인 오류',
                text: message,
                icon: messageType, // 'error'
                confirmButtonText: '닫기'
            });
        }
    }
});
```

이후 메세지 타입이 error일 경우만 경고문을 보여준다.

SweetAlert를 사용해보았다.

**- 결과**

![result](/images/loginError.png)

