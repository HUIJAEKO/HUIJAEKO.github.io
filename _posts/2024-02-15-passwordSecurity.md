---
layout: single
title: "비밀번호 암호화하기"
categories: spring
---

회원가입 시 비밀번호를 **암호화**하는 방법에 대해서 작성해보고자 한다.

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
```

먼저 `BCryptEncoder` 타입의 객체에 대한 의존성 주입을 요청한다.

이후 `Entity`의 패스워드를 `bCryptPasswordEncoder.encode(userDTO.getPassword())`를 통해 암호화한다.

`passwordEncoder.encode(...)` 메서드는 인자로 받은 비밀번호를 암호화하여, 저장하거나 사용할 수 있는 형태의 문자열로 변환한다.

이후 회원가입을 진행하면 암호화가 완료된 것을 볼 수 있다.

![암호화실행](/images/passwordSecurity.png)



