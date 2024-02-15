---
layout: single
title: "ajax통신으로 회원 중복 검사하기"
categories: ajax
---

운동메이트 만들기 프로젝트를 하던 중 회원가입을 할 때, **이미 존재하는 아이디인지 확인**하고 그렇다면 그 아이디로는 회원가입을 할 수 없도록 구현했다.

기본적으로 `ajax`를 이용하였다.

먼저, `html`파일의 구성도이다.

```html
 <input type="text" id="username"
  th:field="*{username}" required placeholder="xxxxx@naver.com"
  onblur="usernameDuplicate()"> <span id=username_duplicate></span>
```

`input`태그를 이용하여 `onblur = "usernameDuplicate()"`라는 함수를 만들었고,
`span`태그를 이용하여 아이디 존재 여부 확인 결과를 나타낼 수 있도록 하였다.

이후 `javascript` 파일을 작성하였다.

```javascript
const usernameDuplicate = () =>{
	const dupleName = document.getElementById("username").value;
	const checkResult = document.getElementById("username_duplicate");
	const signupButton = document.getElementById("signupButton");
	$.ajax({
		type: "post",
		dataType:"text",
		url: "/user/usernameCheck",
		data: {
			"username":dupleName
		},
		success: function(result){
			if(result =="ok"){
				checkResult.innerHTML="사용가능합니다";
				checkResult.style.color="green";
				signupButton.disabled = false;
			}else{
				checkResult.innerHTML="이미 존재하는 아이디입니다";
				checkResult.style.color="red";
				signupButton.disabled = true;
			}
		},
		
		error: function(){
			alert("ajax실패");
		}
	});
```

`usernameDuplicate()`이 작동하면, `"/user/usernameCheck"` 주소로 POST요청을 한다. 

이후 `result` 값이 `ok`이면 녹색 글씨로 사용가능합니다가 뜨고 버튼이 정상적으로 작동하도록 했다.

그렇지 않다면 빨간 글씨로 이미 존재한다는 문구가 뜨고 버튼이 작동하지 않도록 했다.

이후 `Controller`에서 POST요청을 처리했다.

```java
@PostMapping("/user/usernameCheck")
	public @ResponseBody String idCheck(@RequestParam(name = "username") String username) {
		String checkResult = userService.idcheck(username);
		return checkResult;
	}
```

먼저 사용자가 입력한 아이디를 `RequestParam`을 통해 `idcheck`메소드에서 사용한다.

그리고 `ResponseBody`를 통하여 `return`값인 `checkResult`를 응답 데이터로 클라이언트로 전송한다.

이후 `Service`에서 `idcheck`메서드를 만들었다.

```java
public String idcheck(String username){
		Optional<UserEntity> optionalUserEntity = userRepository.findByUsername(username);
		if(optionalUserEntity.isEmpty()) {
			return "ok";
		}else {
			return "no";
		}
	}
```

`idcheck`메서드는, 먼저 `Repository`를 통하여 `database`에서 사용자가 회원가입 할 때 입력한 아이디 값을 불러오도록 시도한다.

불러온 값이 존재하지 않는다면(db에 사용자가 입력한 아이디가 없었다면), 존재하지 않는다는 `ok`값을 리턴한다. 존재한다면 `no`값을 리턴한다.

이러한 리턴값이 `usernameDuplicate`로 넘어가 아이디 중복 체크를 할 수 있게 된다.









