---
layout: single
title: "회원 탈퇴하기, 그리고 Ajax통신에 대하여"
categories: spring
---

회원 탈퇴 과정을 다뤄볼 예정이다.

**1. 먼저 html에 탈퇴버튼을 만들어준다.**

```html
<button type="button" id="delete_account"
th:onclick="confirmDelete()">탈퇴하기</button>
```

**2. 이후 Ajax통신을 이용한다.**

```javascript
function confirmDelete() {
    var confirmation = confirm("정말로 탈퇴하시겠습니까?");
    if (confirmation) {
            $.ajax({
                url: '/user/delete', 
                type: 'DELETE',
                success: function(result) {
                    alert("탈퇴 처리가 완료되었습니다.");
                    window.location.href = '/user/login'; 
                },
                error: function(xhr, status, error) {
                    alert("탈퇴 처리 중 문제가 발생했습니다.");
                }
            });
        }
    }
```

먼저 탈퇴버튼을 누를 시 정말로 탈퇴할 지 경고창을 띄운다.

확인을 누르면 Ajax통신을 시작한다.

먼저, Ajax를 사용하면서 갑자기 왜 Ajax통신을 이용하는지에 관한 의문이 들어 잠깐 정리하고자 한다.

**왜 Ajax를 사용하는가?**

여러 이유가 있겠지만 나같은 경우에는 비동기적인 특징을 위해 사용한다.

Ajax의 가장 큰 특징은 비동기적으로 서버와 통신한다는 점이다. 즉, 사용자가 웹 페이지의 특정 부분에 대한 요청을 보낼 때, 전체 페이지를 새로고침하지 않고도 해당 부분만을 업데이트할 수 있다.

결국, 전체 페이지를 새로고침하지 않고 필요한 데이터만을 교환할 수 있기 때문에 서버 트래픽이 감소한다. 

그리고 Ajax의 작동 원리이다.

**1. 버튼 클릭, 폼 제출 등에 의해 이벤트가 발생**

**2. 이벤트가 발생하면, JavaScript를 통해 XMLHttpRequest 객체를 사용하여 비동기 요청을 생성**

**3. 생성된 요청을 서버로 전송**

**4. 서버는 받은 요청을 처리하고, 필요한 데이터를 조회하거나 수정한 후 응답 준비**

**5. 서버는 처리한 결과를 JSON, XML, HTML 등의 형식으로 클라이언트에게 응답**

**6. 클라이언트는 서버로부터의 응답을 받아 JavaScript를 사용하여 웹 페이지의 특정 부분만을 동적으로 업데이트**

**3. Controller 작성**

```java
@DeleteMapping("/user/delete")
	public String DeleteUser(@AuthenticationPrincipal CustomUserDetails user) {
	userService.deleteUser(user.getId());		
		SecurityContextHolder.clearContext();
		return "user/login";
	}
```

**`@AuthenticationPrincipal`** 어노테이션으로 현재 인증된 사용자의 상세 정보를 가져온 뒤, **`CustomUserDetails`** 타입으로 현재 인증된 사용자의 정보를 주입받는다. 

**`UserEntity`** 를 직접 사용하면 사용자 인증 정보뿐만 아니라 엔티티에 정의된 모든 정보에 접근할 수 있지만, 여기서 **`UserEntity`** 를 사용하지 않는 이유는, 이는 직접적으로 스프링 시큐리티와 통합되어 있지 않기 때문에 **`@AuthenticationPrincipal`** 과 함께 사용하려면 추가적인 설정이 필요하기 때문이다. 

예를 들어, **`UserEntity`** 를 **`UserDetails`** 로 변환하는 로직을 추가해야 한다. 

이후 **`UserService`** 의 **`deleteUser`** 메소드를 이용해 사용자 ID를 기반으로 해당 사용자를 삭제하는 로직을 수행한다.

마지막으로 **`SecurityContextHolder.clearContext()`** 를 이용하여 인증된 사용자의 정보를 세션에서 클리어한다. 

**4. UserService작성**

```java
public void deleteUser(Long id) {
  userRepository.deleteById(id);
}
```

 **`JpaRepository`** 를 확장시킨 **`userRepository`** 의 **`deleteById`** 메서드를 이용해 해당 사용자를 데이터베이스에서 삭제한다.
