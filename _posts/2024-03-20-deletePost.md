---
layout: single
title: "포스트 삭제, client로 정보를 잘 전달하자"
categories: spring
---

이번 글은 게시글 삭제에 대하여 다룰 예정이다. 

기존에 회원탈퇴를 구현해본적 있어서 익숙한 코드를 사용할 수 있었지만, 회원탈퇴는 스프링 시큐리티를 통해 저장되있는 유저정보를 불러와 `db`에서 삭제를 하고, 

포스트삭제는 `@PathVariable`을 통해 `db`에서 삭제를 한다는 차이점이 있었다. 그리고 구현을 하며 아쉬웠던 부분에 대해서도 다뤄볼 예정이다.

**1. 먼저 PostController를 구현하였다.**

```java
@DeleteMapping("/post/delete/{id}")
  public String deletePost(@PathVariable("id") Long id, Model model){
    PostDTO postDTO = postService.deletePost(id);
    model.addAttribute("post", postDTO);
    return "user/main";
  }
```

**2. 이후 PostService를 구현한다.**

```java
public PostDTO deletePost(Long id) {
  Optional<PostEntity> post = postRepository.findById(id);
    if (post.isPresent()) {
      PostEntity postEntity = post.get();
      PostDTO postDTO = PostDTO.toPostDTO(postEntity);
      postRepository.deleteById(id);
      return postDTO;
    } else {
      return null;
    }
  }
}
```

로직에 대하여 설명을 하자면, `PostService`에 있는 `postRepository.findById(id)`를 통해 `id`값을 통하여 그에 맞는 게시글을 불러온다. 

이후 `postRepository.deleteById(id)`를 통해 게시글을 삭제한다. 하지만 여기서 유심히 봐야할 부분은 `Entity`를 `DTO`로 변환하여 삭제를 하는 부분이다.

일반적으로 `Entity`를 바로 삭제하는 것이 효율적이지만, `DTO`를 통하여 삭제를 하는 이유는, 삭제 전에 특정 필드에 따라 삭제를 막아야 할 상황이 있을수도 있기 때문이다.

또한, 삭제하기 전 사용자에게 이메일을 통해 관련 정보를 보내야하는 상황 등이 있다면, `DTO`를 통해 필요한 정보만을 보낼 수도 있기 때문이다.

`Controller`에서 유심히 봐야할 부분은 `{id}`, `@PathVariavle("id")`이다.

요청 경로인 `url`에서의 `{id}`값은 동적으로 사용자의 요청에 따라 값이 변한다. 

그렇기 때문에 이러한 id값을 컨트롤에 전달해주는 것이 필요한데, 이 역할을 하는 것이 `@PathVariavle("id")`이다.

이렇게 구현을 했다면, 다음으로 `Html`파일과 `Javascript`파일을 손봐야한다.

**3. Html, Javascript를 수정한다.**

```html
<button type="button" id="delete_post"
th:onclick="'confirmDelete(' + ${post.id} + ')'">삭제하기</button>
```

```javascript
function confirmDelete(id) {
  var confirmation = confirm("정말로 삭제하시겠습니까?");
  if (confirmation) {
      $.ajax({
        url: '/post/delete/' + id,
        type: 'DELETE',
        success: function(result) {
          alert("삭제가 완료되었습니다.");
          window.location.href = '/user/main';
        },
        error: function(xhr, status, error) {
          alert("삭제 처리 중 문제가 발생했습니다.");
        }
    });
  }
}
```

먼저, `html`파일을 통해서 삭제를 할 수 있는 버튼을 만들어야한다.

여기서는 서버에서 사용자가 선택한 게시글의 아이디값을 가져오는 `${post.id}`로직이 있어야한다.

이 로직이 없다면, `ajax`의 `url: '/post/delete/' + id` 부분에서 id값이 null이 된다.

이렇게 구현을 완료했다면, 글삭제가 정상적으로 진행된다.

이 로직을 구현하면서 한가지 실수로 시간을 많이 쓰게 되었다. 

그 부분은 `Controller`에서 `model.addAttribute("post", postDTO)`부분을 빼먹은 것이다. 

이게 어떻게보면 당연히 넣어야하는 코드인데, 회원탈퇴를 구현했던 것을 생각하며 코드를 짜니 당연한 것을 잊고있었다.

회원탈퇴 로직에서는 스프링 시큐리티에서 자동으로 유저의 정보를 받아와 `Service`와 `Repositoty`를 통해 삭제할 수 있다.

하지만, 게시글 삭제에서는 스프링 시큐리티의 개입이 없기 때문에, `Html`로 정보를 넘겨줘야한다.

그렇지 않으면, id값이 null이 되는 현상이 반복된다. 

아까 말했던 것처럼 `url`은 동적으로 변하는데, 동적으로 변하기 위해서는 서버에서 클라이언트로 id값을 넘겨줘야 사용자가 선택한 게시글의 id값이 다시 `Controller`로 넘어올 수 있다.

따라서 `Controller`에서 `model`을 통해 `DTO`정보를 넘겨주고, 클라이언트에서는 그 `DTO`의 `id`값을 받아야한다.

처음부터 생각을 유심히 하면서 코드를 짰으면 이런 일이 발생하지 않았겠지만, 집중하지 않고 코드를 짠 것에 대해서 반성하며 글을 작성해본다.


