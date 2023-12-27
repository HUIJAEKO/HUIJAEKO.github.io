---
layout: single
title: "애니메이션 효과 구현"
categories: css
---

`애니메이션 효과 구현에 대하여 메모해두려고 한다.`

- `animation-name` : 애니메이션의 중간 상태를 지정하기 위한 이름을 정의한다. 중간 상태는 @keyframes 규칙을 이용하여 기술한다.

- `animation-duration` : 한 싸이클의 애니메이션이 얼마에 걸쳐 일어날지 지정한다.

- `animation-delay` :엘리먼트가 로드되고 나서 언제 애니메이션이 시작될지 지정한다.

- `animation-direction` : 애니메이션이 종료되고 다시 처음부터 시작할지 역방향으로 진행할지 지정한다.

- `animation-iteration-count` : 애니메이션이 몇 번 반복될지 지정한다. infinite 로 지정하여 무한히 반복할 수 있다.

- `animation-play-state` : 애니메이션을 멈추거나 다시 시작할 수 있다.

- `animation-timing-function` : 중간 상태들의 전환을 어떤 시간간격으로 진행할지 지정한다.

- `animation-fill-mode` : 애니메이션이 시작되기 전이나 끝나고 난 후 어떤 값이 적용될지 지정한다.



애니메이션 속성들을 설정하고 이후에 @keyframe으로 시간에 따른 효과들을 구현할 수 있다. 

예를 들어 간단하게 이런 방식으로 할 수 있다.

```javascript
button:hover{animation:sunday 0.5s; animation-fill-mode:forwards;}

@keyframes sunday{
    
    0%{background:greenyellow}
    
    100%{background:green}}
```
