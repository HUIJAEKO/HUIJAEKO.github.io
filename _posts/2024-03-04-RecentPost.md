---
layout: single
title: "메인페이지에 최신글 띄우기"
categories: spring
---

이제 글을 작성하는 코드는 마무리했고, 다음으로 글을 작성했을 시 메인페이지에 최신글 순서로 5개씩 띄어지도록 코드를 짜보았다.

**1. 먼저 메인화면 이동관련하여 `UserController`를 수정하였다.**

```java
@GetMapping("/user/main")
	public String LoginSuccess(Model model) {
		Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
		//authentication을 통해 정보 가져오기
		Object principal = authentication.getPrincipal();
		if(principal instanceof CustomUserDetails) {
			CustomUserDetails userDetails = (CustomUserDetails) principal;
			model.addAttribute("username", userDetails.getName());
			model.addAttribute("posts", postService.find5Post());
		}
		return "user/main";
	}
```

`model.addAttribute("posts", postService.find5Post());` 이 부분을 추가해주었다.

**2. 이후 `find5Post()`메서드를 작성하기 위해 `PostService`와 `PostRepository`를 수정하였다.**

```java
public List<PostEntity> find5Post(){
		return postRepository.findAllByOrderByPostCreatedTimeDesc(PageRequest.of(0, 5));
	}

List<PostEntity> findAllByOrderByPostCreatedTimeDesc(Pageable pageable);
```

`find5Post`라는 최신글 5개를 찾는 메서드를 통해 `model`에 `posts`라는 이름으로 최신글을 담는다.

**3. 다음으로 클라이언트에 직접적으로 나타내지는 부분이기 때문에 `html`과 `javascript`를 수정해야 한다.**

```html
<div class="posts-container">
  <h3>최신 글</h3>
  <table>
    <thead>
      <tr>
        <th>제목</th>
        <th>지역</th>
        <th>하위지역</th>
        <th>작성자</th>
      </tr>
    </thead>
    <tbody>
      <tr th:each="post : ${posts}" th:id="'post-' + ${post.id}"
      class="clickable-row" th:data-post-id="${post.id}">
        <td th:text="${post.postTitle}">제목</td>
        <td th:text="${post.region}">지역</td>
        <td th:text="${post.subregion}">하위 지역</td>
        <td th:text="${post.userEntity.name}">작성자</td>
      </tr>
    </tbody>
  </table>
</div>
```

`controller`에서 `posts`라는 이름으로 담은 글들을 `table`형식으로 나타내려고 하였다.

`for`문을 통하여 각각의 `post`들에 정보를 담고, `${post.postTitle}`등을 통해 글과 관련된 정보들을 나타내주었다.

```javascript
document.addEventListener("DOMContentLoaded", function() {
  document.querySelectorAll('.clickable-row').forEach(row => {
    row.addEventListener('click', () => {
      const postId = row.getAttribute('data-post-id');
    window.location.href = '/post-detail?postId=' + postId;
    });
  });
});
```

이후 `javascript`를 통하여 클릭시 글 상세조회가 될 수 있도록 설정하였다.

하지만 이렇게 구현을 했더니, 초기에 **로그인을 하여 메인페이지로 이동할 때는 최신글 목록이 나타났지만,** 

**글을 작성하고 메인페이지로 이동할 때는 최신글 목록이 보이지 않았다.**

따라서 `PostController`를 살펴보았다.

```java
@PostMapping("/post/newPost")
	public String postSave(@ModelAttribute PostDTO postDTO, Model model) {
		Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
		//authentication을 통해 정보 가져오기
		Object principal = authentication.getPrincipal();
		if(principal instanceof CustomUserDetails) {
			CustomUserDetails userDetails = (CustomUserDetails) principal;
			model.addAttribute("username", userDetails.getName());
			postService.save(postDTO, userDetails.getId());		
		}
		return "redirect:/user/main";
	}
```

여기서 `return`값을 `/user/main`에서 `redirect:/user/main`으로 수정해보았고, 글 작성이 완료되고 메인페이지로 이동했을 때 글이 추가된 것을 확인할 수 있었다. 

그 이유를 조금 찾아보았는데, `/user/main`을 사용하면 현재의 요청 처리 과정에서 바로 뷰를 반환한다. 그렇게 때문에 이를 사용하려면 `@PostMapping`에서 `posts`를 `model`에 담는 과정이 필요하다.

하지만 `redirect:/user/main`을 사용하면, `/user/main`이라는 `GET`요청을 새로하고, `@GetMapping("/user/main")`이 새로이 실행된다.

따라서 `redirect`를 통해 더 간결하게 코드를 작성할 수 있게 된다.
