### Welcome Page

resources/static/index.html

경로에 index.html을 주면 웰컴페이지가 된다.

### 1.1.5. Welcome Page

[Web](https://docs.spring.io/spring-boot/docs/current/reference/html/web.html)

Spring Boot supports both static and templated welcome pages. It first looks for an `index.html` file in the configured static content locations. If one is not found, it then looks for an `index` template. If either is found, it is automatically used as the welcome page of the application.

static에서 먼저 index.html을 찾는다고 함.

### 정적페이지

적어놓은 html 파일이 웹서버에 브라우저에 넘겨준 것이다.

### 템플릿 엔진

템플릿 엔진을 쓰면 루프를 바꿀 수 있다.

타임리프라는 템플릿 엔진을 쓸 것이다.

### 링크

thymeleaf 공식 사이트: [https://www.thymeleaf.org/](https://www.thymeleaf.org/)    
스프링 공식 튜토리얼: [https://spring.io/guides/gs/serving-web-content/](https://spring.io/guides/gs/serving-web-content/)     
스프링부트 메뉴얼: [https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/](https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/)html/spring-boot-features.html#boot-features-spring-mvc-template-engines    

스프링 부트 메뉴얼에서 템플릿 엔진 뭐 쓴다고 알려준다.

공식문서 참고 하자

### Controller

웹 어플리케이션 첫 번째 진입점이 controller이다.

Controller는 `@Controller` 라는 어노테이션을 써줘야한다.

웹어플리케이션에서 /hello로 들어오면 이 메소드로 호출을 해준다.

### View

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<p th:text="'안녕하세요. ' + ${data}" >안녕하세요. 손님</p>
</body>
</html>
```

- `<html xmlns:th="http://www.thymeleaf.org">` : 타임리프 선언

### 순서

1)웹브라우저에서 8080/hello를 던지면 스프링부트는 톰캣이라는 웹서버를 내장하고 있는데

톰캣이 스프링에게 물어본다. 

2)스프링 helloController를 보면 getMapping에서 get메소드의 get이다. get방식으로 넘어오면 hello에 매칭이 되어서 hello 메소드가 실행이 된다.

3)모델이라는 게 넘어온다. 모델에 키 data 값을 hello 넣어서 담는다.

4)리턴의 이름이 hello이다. resources-templates에 있는 hello.html의 이름과 똑같다.

hello.html의 화면을 실행 시켜라 라는 뜻으로 모델을 hello.html 화면에 넘기면서 resources-templates의 hello를 찾아서 랜더링을 한다.

### viewResolver

컨트롤러에서 리턴 값으로 문자를 반환하면 뷰 리졸버( viewResolver )가 화면을 찾아서 처리한다. 스프링 부트 템플릿엔진 기본 viewName 매핑 resources:templates/ +{ViewName}+ .html

> 참고: spring-boot-devtools 라이브러리를 추가하면, html 파일을 컴파일만 해주면 서버 재시작 없이
View 파일 변경이 가능하다.
인텔리J 컴파일 방법: 메뉴 build Recompile
>
