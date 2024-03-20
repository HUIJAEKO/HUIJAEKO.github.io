---
layout: single
title: "사용자에 따른 게시글 조회"
categories: spring
---

이번 글은 블로그 글을 작성한 사람일 경우와 아닐 경우로 나누어 상세조회할 시 어떻게 코드를 짰는지에 대해서 작성하는 글이다. 

작성자가 아닌 다른 사람은 글을 삭제, 수정할 수 없도록 하고 작성자만 가능하도록 하였다.

내가 작성한 방법이 맞는 방법인지는 모르겠지만 일단 기록해보겠다.

추가적으로 아직 댓글에 대한 기능은 구현하지 않은 상태이다.

**1. 먼저 `PostDetail.html` 파일과 `PostDetailNotWriter.html` 파일로 나누었다.** 

`PostDetail.html` 파일에만 수정, 삭제버튼을 추가해주었다.

```html
<button th:onclick="'location.href=\'/post/edit/' + ${post.id} + '\''">수정하기</button>
<button th:onclick="'location.href=\'/post/delete/' + ${post.id} + '\''">삭제하기</button>
<button onclick="history.back()">뒤로가기</button>
```

**2. `PostService`에 블로그 글을 불러오고, 작성자를 불러올 수 있도록 구현하였다.**

```java
public UserEntity getPostWriter(Long id){
  Optional<PostEntity> postEntityOptional = postRepository.findById(id);
  if (postEntityOptional.isPresent()) {
    PostEntity postEntity = postEntityOptional.get();
    return postEntity.getUserEntity();
  } else {
    return null; // 혹은 적절한 예외 처리
  }
}

public PostDTO postDetail(Long id) {
  Optional<PostEntity> post = postRepository.findById(id);
  if(post.isPresent()){
    PostEntity postEntity = post.get();
    PostDTO postDTO = PostDTO.toPostDTO(postEntity);
    return postDTO;
  }else{
    return null;
  }
}
```

먼저 위에 있는 메서드는 글의 작성자를 불러오는 메서드이다. 글이 있으면 글의 정보를 받아와서 작성자를 리턴하도록 하였다.

아래 있는 메서드는 블로그 글을 `DTO`로 전환하여 리턴하는 메서드이다.

**3. 마지막으로 `PostController`를 수정해야 한다.**

컨트롤러를 수정하는 것이 가장 어려웠던 것 같다. 주석을 달아 각 부분에 대한 설때 가장 고민되는 것이 작성자가 아닌 다른 사용자가 블로그 글을 수정하거나 삭제하면 어떡할지였다. 

하지만 생각해보니 현재 사용자가 블로그 글 작성자일 시와 아닐 시로 나누어서 코드를 짜면 되는 것이었다.

따라서, `PostService`를 통하여 블로그 작성자에 대한 정보를 가져와서 비교한 뒤, 작성자가 맞는지 확인할 수 있도록 구현하였다
