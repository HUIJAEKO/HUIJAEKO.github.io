---
layout: single
title: "인프런 스프링 입문(프로젝트 환경 설정과 웹 개발 기초)"
categories: spring
---

#### `이 포스트는 인프런 김영한님의 강의 및 자료를 사용합니다`

## 1. 프로젝트 환경 설정

첫번째 글은 프로젝트 설정, 그리고 웹 개발 기초 강의에 관한 글이다.

기본적으로 스프링 프로젝트를 만들 때에는 `start.spring.io`를 이용한다. 

이 페이지에서 사용할 언어, 스프링부트의 버전, 파일 이름, 사용할 라이브러리 등을 설정한다.

프로젝트를 처음 만들고 자신이 사용하는 프레임워크를 이용해 파일을 열게 되면, 가장 중요한 부분이 `gradle` 혹은 `maven`이다. 

이 파일은 우리가 처음 설정한 라이브러리와 관련된 파일이다. 여기서 `gradle`과 같은 파일의 역할은, 의존관계에 있는 라이브러리들을 함께 끌어와주는 역할을 한다.

예를 들어, 처음에 `spring-boot-starter-web`이라는 라이브러리를 설정했다고 가정해보자. 

그렇다면, 이 라이브러리와 의존관계가 있는 `spring-boot-starter-tomcat`, `spring-webmvc` 과 같은 라이브러리를 끌어온다.

아래 `gradle` 파일을 보게 되면, 자바와 스프링부트의 버전, 사용할 라이브러리, 그리고 그 라이브러리를 어디서 가져올 것인지 등을 알 수 있다.

더 자세한 내용은 스프링 공부를 더 하면서 알아보도록 하자!

```java
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.2.4'
	id 'io.spring.dependency-management' version '1.1.4'
}

group = 'hello'
version = '0.0.1-SNAPSHOT'

java {
	sourceCompatibility = '17'
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
	useJUnitPlatform()
}
```

## 2. 웹 개발 기초

스프링 웹 개발의 기본적인 내용에는 `정적 컨텐츠`, `MVC`, `API`가 있다. 

관련된 내용들을 짧게 나마 정리할 예정이다.

#### **1. 정적 컨텐츠 : 파일을 사용자 화면에 그대로 나타내는 것이다. 아무런 변화 없이 나타나기 때문에 정적이라는 의미가 있다.**

정적 컨텐츠는 기본적으로 `/main/resources/static` 하위에 파일을 생성한다.

아래 코드 기본적인 `html`파일이며, 파일 이름은 `hello-static.html`이다.

```html
<!DOCTYPE HTML>
<html>
    <head>
        <title>Hello</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    </head>
    <body>
    Hello
    </body>
</html>
```

사용자로부터 `localhost:8080/hello.html` 요청이 들어오면 위의 코드를 기반으로 화면이 클라이언트 쪽으로 나타나게 된다.

원리를 보자면, 

![정적 컨텐츠](/images/static.png)

가장 먼저 톰캣이라는 내장 서버를 거치게 된다.

이후 스프링 컨테이너에서 `hello-static` 관련 컨트롤러가 있는지 확인한다. 

정적 컨텐츠기 때문에 관련 컨트롤러가 없어 `/resource/static`폴더 하위에서 `hello-static.html` 파일을 찾아 반환해준다.

결과

![api결과](/images/staticResult.png)

#### **2. MVC : MVC방식은 템플릿 엔진을 Model, View, Controller로 나누고, View를 템플릿 엔진으로 프로그래밍하여 렌더링 된 것을 반환하는 것이다.**

기본적으로 `@Controller`와 `@GetMapping`이 필요하다.

```java
@Controller
public class helloController {

  @GetMapping("hello-mvc")
  public String helloMvc(@RequestParam("name") String name, Model model) {
    model.addAttribute("name", name);
    return "hello-template";
  }
}
```

```html
<html xmlns:th="http://www.thymeleaf.org">
<body>
<p th:text="'hello ' + ${name}">hello! empty</p>
</body>
</html>
```

`localhost:8080/hello-mvc?name=spring` 요청을 받게 되면 클라이언트 쪽에서는 `hello spring` 문자열이 나타나있는 화면이 나타나게 된다.

원리를 알아보기전에 `@RequestParam` 어노테이션에 대해서 알아보도록 하겠다.

- `@RequestParam` : 클라이언트가 보낸 HTTP 요청 파라미터를 컨트롤러 메서드의 파라미터로 쉽게 받아올 수 있게 해주는 어노테이션

쉽게 말해서 특정 경로로 요청을 보낼때, 예를 들어 `localhost:8080/hello-mvc?name=spring`으로 요청을 보낼때, `name=spring`부분에서 `spring`이라는 문자열을 `name`에 담아주는 어노테이션이다.

이러한 `name`의 속성을, `model.addAttribute("name", name)`를 통해 모델에 담아 클라이언트쪽으로 전달해준다.

다음으로 MVC의 원리를 보자면,

![mvc](/images/mvc.png)

마찬가지로 톰켓 내장 서버를 거친후, 이번에는 `hello-mvc`관련 컨트롤러가 있기 때문에,

컨테이너에서 `@Controller`에 의해 `helloController`가 실행된다. 

따라서 `helloMvc` 메소드가 실행 된다. 

`hello-template`으로 반되기 때문에 `viewResolver`는 `hello-template.html`파일을 뷰 템플릿인 `Thymeleaf`에게 처리해달라 넘기고,

`Thymeleaf`는 `${name}`를 통해 `name`에 저장된 데이터를 가져와 값을 변환하고, 변환된 `HTML`파일을 웹 브라우저에게 반환한다.

따라서 클라이언트쪽에서는 `hello spring` 문자열이 나타나있는 화면이 나타나게 된다.

변환된 파일을 반환하기 때문에 동적이다.

결과

![api결과](/images/mvcResult.png)

#### **3. API : 데이터를 JSON 스타일로 바꿔서 반환하는 것이다. View를 거치지 않으며, MVC방식과 마찬가지로 동적이다.**

기본적으로 `@ResponseBody`를 사용한다.

여기서도 먼저 `@ResponseBody`에 대해서 알아보도록 하겠다.

- `@ResponseBody` : 컨트롤러 메서드에서 반환하는 값을 HTTP 본문으로 직접 사용하겠다는 것을 나타낸다.

직접 사용하기 때문에, MVC방식과는 다르게 `viewResolver`를 사용하지 않는다. 이때, 데이터는 클라이언트에 전송될 적절한 형식(JSON, XML 등)으로 변환된다.

먼저 아래 코드를 봐보자. 

```java
@Controller
public class HelloController {
  @GetMapping("hello-string")
  @ResponseBody
  public String helloString(@RequestParam("name") String name) {
  return "hello " + name;
  }
}
```

`localhost:8080/hello-string?name=spring` 요청을 받게 되면, 마찬가지로 클라이언트 쪽에서는 `hello spring` 문자열이 나타나있는 화면이 나타나게 된다.

원리를 보자면,

![api](/images/api.png)

톰켓 내장 서버를 거친후, `@Controller`에 의해 `helloController`가 실행된다. 

`helloString()` 메소드가 실행되면 `@RequestParam`을 통해 전달받은 값을 이름으로 설정한다.

`@ResponsBody`가 있기 때문에, `viewResolver`가 아닌 `HttpMessageConverter`로 처리된다. 

그 중에서도 단순 문자라면 `StringConverter`, `JsonConverter`로 처리된다. 

이후 http로 직접 반환된다. 반환된 값은 `{"name":"spring"}` 형태로 나타난다.

결과

![api결과](/images/apiResult.png)











