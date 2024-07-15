---
layout: single
title: "필드 제약걸기 -> 비밀번호에 특수문자는 필수"
categories: spring
---

**1. 필드들에 제약을 걸기 위해서는 먼저 gradle 파일에 validation 라이브러리를 추가해야한다.**

`implementation 'org.springframework.boot:spring-boot-starter-validation'`

이후에 DTO에서 걸고 싶은 필드에 제약을 건다. 예를 들어, 아래와 같은 방식으로 어노테이션을 추가한다.

```java
@NotBlank(message = "비밀번호는 필수 입력 값입니다.")
@Pattern(regexp = "(?=.*[0-9])(?=.*[a-zA-Z])(?=.*\\W)(?=\\S+$).{8,16}", message = "비밀번호는 8~16자 영문 대 소문자, 숫자, 특수문자를 사용하세요.")
  private String password;
```

이는 비밀번호가 공백이면 안되고, 비밀번호는 8~16자, 영문 대 소문자, 숫자, 특수문자를 사용해야 하는 제약이다. 

이러한 조건에 부합하지 않을 시 message를 사용자에게 보여준다.

**2. 그리고 html을 작성한다**

```html
<div>
  <label for="password">비밀번호</label> <input type="password"
    id="password" th:field="*{password}" > <span
    th:if="${password_Error}" th:text="${password_Error}" class="error"></span>
</div>
```

`th:if="${password_Error}" th:text="${password_Error}" class="error"` 에서 if문을 사용하여 서버측에서 비밀번호 검증이 실패했을 시, 오류 메시지를 포함한 <span> 태그를 사용자에게 표시하게 된다.

**3. 다음은 Controller 를 작성해야 한다.**

```java
@PostMapping("/user/signup")
public String MakeSignUp(@Valid UserDTO userDTO, Errors errors, Model model) {
  if(errors.hasErrors()) {
    model.addAttribute("userDTO", userDTO);
    Map<String, String> validatorResult = userService.validateHandling(errors);
    for(String key : validatorResult.keySet()) {
      model.addAttribute(key, validatorResult.get(key));
    }
    return "user/signup";
  }
  userService.save(userDTO);
  return "user/login";
}
```

먼저, `@Valid UserDTO userDTO`을 통하여 사용자로부터 입력 받은 데이터인 `userDTO` 를 `@Valid` 어노테이션을 통해 Spring이 자동으로 검증하도록 한다. 

이때 `UserDTO`에 작성한 `@Pattern`등에에 따라 데이터가 검증한다.

`Errors errors`는 검증 과정에서 발생한 오류들을 담고 있는 객체이다. 데이터가 검증 규칙을 만족하지 못했다면, 이 객체를 통해 어떤 오류가 발생했는지 알 수 있다.

`errors.hasError()` 에러가 있다면, 아래 로직을 수행한다.

먼저 `model.addAttribute`을 통하여 에러가 발생했을 시 입력했던 것들이 초기화되지 않도록 한다.

이후 `validateHandling` 메서드를 통하여 필드 이름과 관련된 오류 메시지를 포함하는 `Map<String, String>` 타입의 `validatorResult`를 반환한다.

마지막으로 반복문을 통하여 `key`에 오류메시지를 저장한다. 

만약 모든 필드가 조건을 만족했다면 데이터를 정상적으로 `DB`에 저장한다.

**4. 마지막으로 Service를 작성한다.**

```java
public void save(UserDTO userDTO) {
  UserEntity userEntity = UserEntity.toUserEntity(userDTO);
  userEntity.setPassword(bCryptPasswordEncoder.encode(userDTO.getPassword()));
  userRepository.save(userEntity);
}
	
public Map<String, String> validateHandling(Errors errors) {
  Map<String, String> validatorResult = new HashMap<>();		
  for(FieldError error: errors.getFieldErrors()) {
    String validKeyName = String.format("%s_Error", error.getField());
    validatorResult.put(validKeyName, error.getDefaultMessage());
  }
  return validatorResult;
}
```

기본적으로 `save`메서드는 DB에 데이터를 저장하는 메서드이다.

자세히 보아야 할 것은 `validateHandling`메서드인데, 먼저 `Map`객체를 만들어준다.

이후 `errors.getErrirs()`를 통하여 모든 필드 오류를 가져와 키 이름은 `필드이름_Error`, 값은 `error내용`으로 validatorResult에 저장한다.








