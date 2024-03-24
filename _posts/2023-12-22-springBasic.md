---
layout: single
title: "인프런 스프링 입문(프로젝트 환경 설정과 웹 개발 기초)"
categories: spring
---

#### `이 포스트는 인프런 김영한님의 강의 및 자료를 사용합니다`

## 1. 프로젝트 환경 설정

첫번째 글은 프로젝트 설정, 그리고 웹 개발 기초 강의이다.

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

1. 정적 컨텐츠 : 파일을 사용자 화면에 그대로 나타내는 것이다. 아무런 변화 없이 나타나기 때문에 정적이라는 의미가 있다.

정적 컨텐츠는 기본적으로 `/main/resources/static` 하위에 파일을 생성한다.

아래 코드 기본적인 `html`파일이며, 파일 이름은 `hello.html`이다.

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

사용자로부터 localhost:8080/hello.html 요청이 들어오면 위의 코드를 기반으로 화면이 클라이언트 쪽으로 나타나게 된다.

원리를 보자면, 

가장 먼저 톰캣이라는 내장 서버를 거치게 된다.

이후 스프링 컨테이너에서 `hello관련 컨트롤러`가 있는지 확인한다. 

없다면, `static 하위`에서 폴더를 찾아 반환해준다.



- `MVC`: MVC방식은 템플릿 엔진을 `Model, View, Controller방식`으로 나누고, View를 템플릿 엔진으로 프로그래밍하여 `렌더링 된 것을 반환`하는 것이다.

MVC방식은 변환되기 때문에 `동적`이다.

기본적으로 `@Controller와 @GetMapping`이 필요하다.

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

localhost:8080/hello-mvc 요청을 받게 되면, 마찬가지로 톰켓 내장 서버를 거친후, 

컨테이너에서 `@Controller에 의해 helloController가 실행`된다. 

따라서 `helloMvc 메소드가 실행`이 된다. 

hello-template으로 return되기 때문에 `viewResolver는 hello-template.html파일을 Thymeleaf에게 처리해달라 넘기고`,

Thymeleaf는 name에 저장된 데이터를 가져와 `값을 변환`하고, 변환된 HTML을 웹 브라우저에게 반환한다.

이때 hello-template파일은 아래와 같은 방법으로 쓰인다.

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
	<body>
		<p th:text="'hello. ' + ${name}" >hello</p>
	</body>
</html>
```


- `API`: 데이터를 JSON 스타일로 바꿔서 반환하는 것이다. View를 거치지 않으며, MVC방식과 마찬가지로 `동적`이다.

기본적으로 `@ResponseBody를 사용`한다.

```java
public class helloController {
    @GetMapping("hello-api")
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name) {
    		Hello hello = new Hello();
    		hello.setName(name);
  			return hello;
    }
  		
    static class Hello{
  			private String name;
  			
  			public String getName() {
  				return name;
  			}
  			
  			public void setName(String name) {
  				this.name = name;
  			}
    }
}
```

localhost:8080/hello-api 요청을 받게 되면, 마찬가지로 톰켓 내장 서버를 거친후, 

@Controller에 의해 helloController가 실행된다. 

helloApi() 메소드가 실행되면 `hello 라는 객체를 새로 생성`하고 `setName()을 통해서 전달받은 값을 이름으로 설정`한다.

`@ResponsBody`가 있기 때문에, `viewResolver가 아닌 HttpMessageConverter로 처리`된다. 

그 중에서도 `단순 문자라면 StringConverter`, `객체라면 JsonConverter`로 처리된다. 

이후 http로 반환된다. 반환된 값은 `{"name":"spring"} 형태`로 나타난다.











