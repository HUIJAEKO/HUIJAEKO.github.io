---
layout: single
title: "비밀번호 암호화와 비밀번호 변경, @Transactional?"
categories: spring
---

### 회원가입 시 비밀번호를 **암호화**하는 방법에 대해서 작성해보고자 한다.

비밀번호를 암호화하기 위해서는 `PasswordEncoder` 인터페이스를 구현하는 클래스를 사용해야 한다.

가장 일반적으로는 `BCryptPasswordEncoder`이 있다.

따라서 이를 `@Bean`으로 등록한다.

```java
@Configuration
public class SecurityConfig {
  @Bean
  public BCryptPasswordEncoder bCryptPasswordEncoder() {		
    return new BCryptPasswordEncoder();
  }
}
```

이제 `Controller`와 `Service`에 로그인 로직을 작성한다

```java
@Controller
public class UserController {
  @Autowired
  private UserService userService;
  
  @PostMapping("/signup")
  public String MakeSignUp(@ModelAttribute UserDTO userDTO) {
    userService.save(userDTO);
    return "login";
  }
}
```

```java
@Service
public class UserService {
  @Autowired
  private UserRepository userRepository;
  
  @Autowired
  private BCryptPasswordEncoder bCryptPasswordEncoder;
  
  public void save(UserDTO userDTO) {
    UserEntity userEntity = UserEntity.toUserEntity(userDTO);
    userEntity.setPassword(bCryptPasswordEncoder.encode(userDTO.getPassword()));
    userRepository.save(userEntity);
  }
}
```

먼저 `BCryptEncoder` 타입의 객체에 대한 의존성 주입을 요청한다.

이후 `Entity`의 패스워드를 `bCryptPasswordEncoder.encode(userDTO.getPassword())`를 통해 암호화한다.

`passwordEncoder.encode(...)` 메서드는 인자로 받은 비밀번호를 암호화하여, 저장하거나 사용할 수 있는 형태의 문자열로 변환한다.

이후 회원가입을 진행하면 암호화가 완료된 것을 볼 수 있다.

![암호화실행](/images/passwordSecurity.png)


### 이번에는 비밀번호 변경 기능을 넣어보았다. 현재 비밀번호 확인, 새 비밀번호 입력, 새 비밀번호 확인 기능을 넣어 구현하였다.

```html
<form th:action="@{/user/editPassword}" th:object="${userDTO}"
  method="post">
  <div>
  <label>현재 비밀번호</label> <input type="password" id="currentPw"
      name="currentPw">
  <div id="currentPw-error-message" style="color: red;"></div>
  </div>
  <div>
  <label>비밀번호 입력</label> <input type="password" id="newPw"
    name="newPw">
  </div>
  <div>
  <label>비밀번호 확인</label> <input type="password" id="newPwConfirm"
    name="newPwConfirm">
  <div id="newPwConfirm-error-message" style="color: red;"></div>
  </div>
  <div>
    <button type="submit" id="edit_password" >비밀번호
      변경하기</button>
  </div>
</form>
```

먼저 **`form`** 태그를 이용하여 **`input`** 과 **`button`** 을 만들어준다.

**`form`** 태그에서 **`PATCH`** 를 직접적으로 쓸 수 없기 때문에 **`Ajax`** 를 사용하여 **`method`** 를 우회한다.

```javascript
$(document).ready(function() {
  $("#edit_password").click(function() {
    event.preventDefault();
    var currentPw = $("#currentPw").val();
    var newPw = $("#newPw").val();
    var newPwConfirm = $("#newPwConfirm").val();

    $.ajax({
      url: '/user/editPassword',
      type: 'PATCH',
      contentType: 'application/json',
      data: JSON.stringify({
        currentPw: currentPw,
        newPw: newPw,
        newPwConfirm: newPwConfirm
      }),
      success: function(response) {
        alert("비밀번호가 성공적으로 변경되었습니다.");
        window.location.href = "/user/login";
      },
      error: function(xhr, status, error) {
        var errorMessage = xhr.responseJSON.error;
        if (errorMessage.includes("현재 비밀번호가 일치하지 않습니다.")) {
          $("#currentPw-error-message").text(errorMessage);
        } else if (errorMessage.includes("새 비밀번호와 비밀번호 확인이 일치하지 않습니다.")) {
          $("#newPwConfirm-error-message").text(errorMessage);
        }
        setTimeout(function() {
          $("#currentPw-error-message").empty();
          $("#newPwConfirm-error-message").empty();
        }, 1000);
      }
    });
  });
});
```

이후 **`Ajax`** 를 사용한다.

먼저 버튼을 클릭하면 기능이 수행되도록 설정하고, **`event.preventDefault()`** 를 사용한다. 

사용하지 않을 시 경고문이 띄어짐과 동시에 폼 제출, 페이지가 새로고침 되어 사용자가 경고문을 인식하지 못하게 된다.

이후 사용자가 입력한 현재 비밀번호, 새 비밀번호, 그리고 새 비밀번호 확인 값을 각각의 변수에 저장한 뒤 **`Ajax`** 요청을 보낸다.

요청이 성공적으로 완료됐을 때에는 사용자에게 비밀번호가 변경되었다는 알림을 보여주고 **`/user/login`** 페이지로 리디렉션한다.

요청이 실패했을 때에는 에러 메시지를 분석하여 해당하는 부분 아래에 에러 메시지를 표시한다. 에러 메시지는 1초 후에 사라지도록 **`setTimeout`** 함수를 사용한다.

**`var errorMessage = xhr.responseJSON.error`** 이 부분은 **`Controller`** 에서 설정된 **`error`** 메시지를 **`errorMessage`** 에 담는 역할을 한다.

이후 **`passwordChangeRequest`** 클래스와 **`Controller`** 를 작성한다.

```java
public class PasswordChangeRequest {
  private String currentPw;
  private String newPw;
  private String newPwConfirm;
  
  public String getCurrentPw() {
    return currentPw;
  }
  public void setCurrentPw(String currentPw) {
    this.currentPw = currentPw;
  }
  public String getNewPw() {
    return newPw;
  }
  public void setNewPw(String newPw) {
    this.newPw = newPw;
  }
  public String getNewPwConfirm() {
    return newPwConfirm;
  }
  public void setNewPwConfirm(String newPwConfirm) {
    this.newPwConfirm = newPwConfirm;
  }
}
```

```java
@PatchMapping("/user/editPassword")
public ResponseEntity<?> bCryptPasswordEncoder(@RequestBody PasswordChangeRequest request, @AuthenticationPrincipal UserDetails userDetails) {
    CustomUserDetails currentUser = (CustomUserDetails) userDetails;

    // 현재 비밀번호 확인
    if (!bCryptPasswordEncoder.matches(request.getCurrentPw(), currentUser.getPassword())) {
        return ResponseEntity.badRequest().body(Map.of("error", "현재 비밀번호가 일치하지 않습니다."));
    }

    // 새 비밀번호와 새 비밀번호 확인 일치 검사
    if (!request.getNewPw().equals(request.getNewPwConfirm())) {
        return ResponseEntity.badRequest().body(Map.of("error", "새 비밀번호와 비밀번호 확인이 일치하지 않습니다."));
    }else {
        // 비밀번호 변경 로직 실행
        userService.changePassword(currentUser.getUsername(), request.getNewPw());
    }
    return ResponseEntity.ok().body(Map.of("message", "비밀번호가 성공적으로 변경되었습니다."));
}
```

먼저 비밀번호만 변경할 수 있도록 **`PATCH`** 를 사용한다.

이후 **`@RequestBody PasswordChangeRequest request`** 를 통하여 클라이언트로부터 받은 JSON 데이터를 **`request`** 에 저장한다.

현재 비밀번호가 일치하는지를 확인할때 **`!bCryptPasswordEncoder.matches(request.getCurrentPw(), currentUser.getPassword())`** 를 이용하는데,

**`matches`** 메서드는 첫 번째 매개변수로 전달된 비밀번호를 해시하고, 이 해시가 두 번째 매개변수로 전달된 저장된 해시와 일치하는지 확인한다.

또한 새 비밀번호와 새 비밀번호 확인이 일치하는지 확인한다. 

위의 두 과정을 모두 통과하면 **`userService.changePassword(currentUser.getUsername(), request.getNewPw())`** 메소드를 호출하여 사용자의 비밀번호를 새 비밀번호로 변경한다.

마지막으로, **`userService`** 를 작성한다.

```java
@Transactional
public void changePassword(String username, String newPassword) {
  UserEntity user = userRepository.findByname(username);
  if (user != null) {
    String encodedPassword = bCryptPasswordEncoder.encode(newPassword);
    user.setPassword(encodedPassword);
    userRepository.save(user);
  } else {
    throw new UsernameNotFoundException("User not found with username: " + username);
  }
}
```

먼저 **`userRepository.findByname(username)`** 을 사용하여 사용자를 조회한다. 

조회된 사용자가 존재할 경우에는 새 비밀번호를 **`bCryptPasswordEncoder.encode(newPassword)`** 를 사용하여 암호화한다. 이후 이를 사용자 엔티티의 비밀번호로 설정한 후 DB에 저장한다.

추가적으로, 여기서 **`@Transactional`** 을 처음써봐서, 왜 써야하는지 잠깐 정리해보겠다.

트랜잭션을 사용하는 이유는 **데이터의 일관성과 무결성**을 보장하기 위해서이다. 데이터베이스 작업을 수행할 때 여러 연산이 함께 수행되어야 할 경우가 많고, 이러한 연산들은 전체가 성공적으로 완료되거나, 하나라도 실패할 경우 전체가 취소되어야 한다. 이를 위해 트랜잭션을 사용한다.

**`@Transactional`** 어노테이션을 메소드에 적용하면, 해당 메소드에서 이루어지는 데이터베이스 연산들이 하나의 트랜잭션으로 묶인다. 이 경우, 메소드 내의 모든 데이터베이스 연산은 다음 두 가지 조건 중 하나에 해당할 때까지 임시적인 상태로 유지된다.

**1. Commit:** 메소드 내의 모든 연산이 성공적으로 완료되었을 때, 변경 사항이 데이터베이스에 영구적으로 반영된다.

**2. Rollback:** 메소드 실행 중 어떤 연산이라도 실패하면, 메소드 내에서 이루어진 모든 데이터베이스 변경 사항이 취소되고, 데이터베이스는 트랜잭션 이전 상태로 복원된다.

비밀번호 변경 기능에서 사용하는 이유는 사용자의 비밀번호를 변경하는 과정이 안전하게 처리되도록 하기 위함이다. 예를 들어, 비밀번호 변경 과정에서 오류가 발생하여 데이터베이스의 상태가 일관되지 않는 상황이 발생할 수 있다. 이때, 트랜잭션을 사용하면 오류 발생 시 모든 변경 사항을 롤백하여 데이터베이스의 일관성과 무결성을 유지할 수 있다.





