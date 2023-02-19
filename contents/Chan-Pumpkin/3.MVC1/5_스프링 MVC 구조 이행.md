# 1. 스프링 MVC 전체 구조

## 차이점

### 직접 만든 MVC 프레임워크 구조

![Untitled 3](https://user-images.githubusercontent.com/62877858/218307934-d45176c6-fa1e-4121-8d60-03a04fae35cf.png)

### SpringMVC 구조

![Untitled 4](https://user-images.githubusercontent.com/62877858/218307940-ab76d59f-f016-4765-924b-d37dad64c40c.png)

### 비교

- FrontController → DispatcherServlet
- handlerMappingMap → HandlerMapping
- MyHandlerAdapter → HandlerAdapter
- ModelView → ModelAndView
- viewResolver → ViewResolver : 스프링은 ViewResolver를 인터페이스로 만들어놨음
- MyView → View

### SpringMVC 동작 순서

1) 핸들러 조회 : 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.

2) 핸들러 어댑터 조회 : 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.

3) 핸들러 어댑터 실행: 핸들러 어댑터를 실행한다.

4) 핸들러 실행: 핸들러 어댑터가 실제 핸들러를 실행한다.

5) ModelAndView 반환: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서
반환한다.

6) viewResolver 호출: 뷰 리졸버를 찾고 실행한다.

- JSP의 경우: InternalResourceViewResolver 가 자동 등록되고, 사용된다.

7) View 반환: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.

- JSP의 경우 InternalResourceView(JstlView) 를 반환하는데, 내부에 forward() 로직이 있다.

8) 뷰 렌더링: 뷰를 통해서 뷰를 렌더링 한다.

## DispatcherServlet구조

### DispatcherServlet 서블릿 등록

- DispacherServlet 도 부모 클래스에서 HttpServlet 을 상속 받아서 사용하고, 서블릿으로 동작한다.
- DispatcherServlet FrameworkServlet HttpServletBean HttpServlet
- 스프링 부트는 DispacherServlet 을 서블릿으로 자동으로 등록하면서 모든 경로`( urlPatterns="/" )`에 대해서 매핑한다.
- 참고: 더 자세한 경로가 우선순위가 높다. 그래서 기존에 등록한 서블릿도 함께 동작한다.

### 요청 흐름

- 서블릿이 호출되면 HttpServlet 이 제공하는 serivce() 가 호출된다.
- 스프링 MVC는 DispatcherServlet 의 부모인 FrameworkServlet 에서 service() 를 오버라이드
해두었다.
- FrameworkServlet.service() 를 시작으로 여러 메서드가 호출되면서 DispacherServlet.doDispatch() 가 호출된다.

### DispatcherServlet.doDispatch()

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse 
response) throws Exception {

	HttpServletRequest processedRequest = request;
	HandlerExecutionChain mappedHandler = null;
	ModelAndView mv = null;

	// 1. 핸들러 조회
	mappedHandler = getHandler(processedRequest);
	if (mappedHandler == null) {
		noHandlerFound(processedRequest, response);
		return;
	}
	// 2. 핸들러 어댑터 조회 - 핸들러를 처리할 수 있는 어댑터
	HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
	// 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환
	mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

	processDispatchResult(processedRequest, response, mappedHandler, mv,
	dispatchException);
}

private void processDispatchResult(HttpServletRequest request,
HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView 
mv, Exception exception) throws Exception {
	// 뷰 렌더링 호출
	render(mv, request, response);
}

protected void render(ModelAndView mv, HttpServletRequest request,
HttpServletResponse response) throws Exception {

	View view;
	String viewName = mv.getViewName();
	// 6. 뷰 리졸버를 통해서 뷰 찾기, 7. View 반환
	view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
	// 8. 뷰 렌더링
	view.render(mv.getModelInternal(), request, response);
}
```

### 주요 인터페이스 목록

- 핸들러 매핑: org.springframework.web.servlet.HandlerMapping
- 핸들러 어댑터: org.springframework.web.servlet.HandlerAdapter
- 뷰 리졸버: org.springframework.web.servlet.ViewResolver
- 뷰: org.springframework.web.servlet.View

스프링 MVC의 큰 강점은 DispatcherServlet 코드의 변경 없이, 원하는 기능을 변경하거나 확장할 수 있다는 점이다. 지금까지 설명한 대부분을 확장 가능할 수 있게 인터페이스로 제공한다.

이 인터페이스들만 구현해서 DispatcherServlet 에 등록하면 여러분만의 컨트롤러를 만들 수도 있다.

# 2. 핸들러 매핑과 핸들러 어댑터

## OldController.java

```java
package hello.servlet.web.springmvc.old;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component("/springmvc/old-controller")
public class OldController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("OldController.handleRequest");
        return null;
    }
}
```

- `import org.springframework.web.servlet.mvc.Controller;` 의 Controller를 써야함.
- `@Controller` 어노테이션 사용x : 핸들러 매핑을 따로 처리해야함.

### 호출 방식

HandlerMapping(핸들러 매핑)

- 핸들러 매핑에서 이 컨트롤러를 찾을 수 있어야 한다.
- 예) 스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸들러 매핑이 필요하다.

> 0 = RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
1 = BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러를 찾는다.
> 

HandlerAdapter(핸들러 어댑터)

- 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.
- 예) Controller 인터페이스를 실행할 수 있는 핸들러 어댑터를 찾고 실행해야 한다.

> 0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
2 = SimpleControllerHandlerAdapter : Controller 인터페이스(애노테이션X, 과거에 사용)
처리
> 

## MyHttpRequestHandler.java

```java
package hello.servlet.web.springmvc.old;

import org.springframework.stereotype.Component;
import org.springframework.web.HttpRequestHandler;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component("/springmvc/request-handler")
public class MyHttpRequestHandler implements HttpRequestHandler {
    @Override
    public void handleRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("MyHttpRequestHandler.handleRequest");
    }
}
```

# 3. 뷰 리졸버

### OldController.java

```java
package hello.servlet.web.springmvc.old;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component("/springmvc/old-controller")
public class OldController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("OldController.handleRequest");
        return new ModelAndView("new-form");
    }
}
```

- View를 사용할 수 있도록 return에 코드를 수정
- 이렇게 되면, 웹 브라우저에서 에러 페이지가 뜨지만, 콘솔에서는 정상 출력이 될 것이다.
- 아래 코드를 추가해보자

### application.properties

```java
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

스프링 부트는 InternalResourceViewResolver 라는 뷰 리졸버를 자동으로 등록하는데, 이때
application.properties 에 등록한 spring.mvc.view.prefix , spring.mvc.view.suffix 설정
정보를 사용해서 등록한다.

참고로 권장하지는 않지만 설정 없이 다음과 같이 전체 경로를 주어도 동작하기는 한다.

`return new ModelAndView("/WEB-INF/views/new-form.jsp");`

## 뷰 리졸버 동작 방식

> 경우
1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성
기능에 사용)
2 = InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환한다.
> 

1) 핸들러 어댑터 호출

- 핸들러 어댑터를 통해 new-form 이라는 논리 뷰 이름을 획득한다.

2) ViewResolver 호출

- new-form 이라는 뷰 이름으로 viewResolver를 순서대로 호출한다.
- BeanNameViewResolver 는 new-form 이라는 이름의 스프링 빈으로 등록된 뷰를 찾아야 하는데 없다.
- InternalResourceViewResolver 가 호출된다.

3) InternalResourceViewResolver

- 이 뷰 리졸버는 InternalResourceView 를 반환한다.

4) 뷰 - InternalResourceView

- InternalResourceView 는 JSP처럼 포워드 forward() 를 호출해서 처리할 수 있는 경우에 사용한다.

5) view.render()

- view.render() 가 호출되고 InternalResourceView 는 forward() 를 사용해서 JSP를 실행한다.

# 4. 스프링 MVC 시작하기

스프링이 제공하는 컨트롤러는 애노테이션 기반으로 동작해서 매우 유연하고 실용적임.

### `@RequestMapping`누군가 인식을 해서 찾아줘야 하지 않나?

가장 우선순위가 높은 핸들러 매핑과 핸들러 어댑터

- RequestMappingHandlerMapping
- RequestMappingHandlerAdapter

### @Controller

- 스프링이 자동으로 스프링 빈 등록한다.

`@Component` 애노테이션이 있으면 자동으로 빈이 등록이 되는데, `@Controller` 안에 들어가보면 `@Component` 애노테이션이 붙어 있음.

- 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식함.

### @RequestMapping

요청 정보를 매핑한다. 해당 URL이 호출되면 이 메서드가 호출이 되고, 애노테이션을 기반으로 동작하기 때문에 메서드의 이름은 임의로 지으면 된다.

### @ModelAndView

모델과 뷰 정보를 담아서 반환하면 된다.

### RequestMappingHandlerMapping.java

RequestMappingHandlerMapping은 스프링 빈 중에서 @RequestMapping 또는 @Controller가 클래스 레벨에 붙어 있는 경우에 매핑 정보로 인식함.

```java
@Override
	protected boolean isHandler(Class<?> beanType) {
		return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) ||
				AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class));
	}
```

`Controller.class`, `RequestMapping.class`클래스 레벨에 있으면 처리할 수 있는 대상으로 인식함.

## 구현

application.properties

```java
logging.level.org.apache.coyotey.http11=debug

spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

### SpringMemberFormControllerV1.java

```java
package hello.servlet.web.springmvc.v1;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class SpringMemberFormControllerV1 {

    @RequestMapping("/springmvc/v1/members/new-form")
    public ModelAndView process() {
        //뷰 리졸버에서 찾아서 렌더링
        return new ModelAndView("new-form");
    }
}
```

### SpringMemberSaveControllerV1.java

```java
package hello.servlet.web.springmvc.v1;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import hello.servlet.web.frontcontroller.ModelView;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Map;

@Controller
public class SpringMemberSaveControllerV1 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members/save")
    public ModelAndView process(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member", member);

        //뷰로 던지기
        return mv;
    }
}
```

### SpringMemberListControllerV1.java

```java
package hello.servlet.web.springmvc.v1;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import hello.servlet.web.frontcontroller.ModelView;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import java.util.List;
import java.util.Map;
@Controller
public class SpringMemberListControllerV1 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members")
    public ModelAndView process() {
        List<Member> members = memberRepository.findAll();
        ModelAndView mv = new ModelAndView("members");
        mv.addObject("members", members);

        return mv;
    }
}
```

# 5. 컨트롤러 통합

`@RequestMapping`은 메서드 단위로 사용할 수 있다 

다만, 하나의 컨트롤러에서 연관된 메서드(기능)들이 있어야 한다.

## URL 통합

### 1. /springmvc/v1/members/new-form

```java
@RequestMapping("/springmvc/v1/members")
public class SpringMemberControllerV2 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/new-form")
    public ModelAndView newForm() {
```

### 2. /springmvc/v1/members

- URL 더할 게 없으면 @RequestMapping에 안붙여도 된다.

```java
@RequestMapping("/springmvc/v1/members")
public class SpringMemberControllerV2 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping
    public ModelAndView members() {
```

## SpringMemberControllerV2

```java
package hello.servlet.web.springmvc.v2;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.List;

@Controller
@RequestMapping("/springmvc/v2/members")
public class SpringMemberControllerV2 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/new-form")
    public ModelAndView newForm() {
        return new ModelAndView("new-form");
    }

    @RequestMapping("/save")
    public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member", member);

        //뷰로 던지기
        return mv;
    }

    @RequestMapping
    public ModelAndView members() {
        List<Member> members = memberRepository.findAll();
        ModelAndView mv = new ModelAndView("members");
        mv.addObject("members", members);

        return mv;
    }

}
```

# 6. 스프링 MVC - 실용적인 방식

## 코드

```java
package hello.servlet.web.springmvc.v3;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@Controller
@RequestMapping(value = "/springmvc/v3/members", method = RequestMethod.GET)
public class SpringMemberControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    //RequestMapping은 제약이 없기 때문에,
    //해당된 HTTP 메소드의 맞는 어노테이션 사용할 것.
    @GetMapping("/new-form")
    //문자로 반환해도 뷰인 것을 알고 리턴됨.
    public String newForm() {
        return "new-form";
    }

    @PostMapping("/save")
    public String save(@RequestParam("username") String username,
                       @RequestParam("age") int age,
                       Model model) {
        Member member = new Member(username, age);
        memberRepository.save(member);

        model.addAttribute("member", member);

        return "save-result";
    }

    @GetMapping
    public String members(Model model) {
        List<Member> members = memberRepository.findAll();

        model.addAttribute("members", members);
        return "members";
    }
}
```

## View를 String 반환?

### 변경 전

```java
		@RequestMapping("/new-form")
    public ModelAndView newForm() {
        return new ModelAndView("new-form");
    }
```

### 변경 후

```java
		@GetMapping("/new-form")
    public String newForm() {
        return "new-form";
    }
```

ModelAndView 타입으로 뷰를 리턴했었지만, String 타입으로 문자로 반환해도 뷰인 것을 알고 동작이 정상적으로 이루어진다.

## @GetMapping @PostMapping

PostMan으로 테스트를 했을 때, get, post 둘다 진행해도 정상이다. 

실제 HTTP 메소드가 뭐가 오든 스프링에서 안막는 것을 알 수가 있는데, @RequestMapping 안에 아래와 같이 메소드를 제약을 걸 수가 있다.

`@RequestMapping(value = "/~~/~~/~~", method = RequestMethod.GET)`

### 예시

```java
@Controller
@RequestMapping(value = "/springmvc/v3/members", method = RequestMethod.GET)
public class SpringMemberControllerV3 {
```

### 결과

Get으로 테스트를 하면 정상이지만, Post로 테스트를 하면 아래와 같은 결과가 나오므로, Get으로 제약 걸어두었다는 것을 확인할 수 있다.

```java
{
    "timestamp": "2023-02-14T12:15:01.861+00:00",
    "status": 405,
    "error": "Method Not Allowed",
    "path": "/springmvc/v3/members/new-form"
}
```

### 제약 걸어두는 이유?

위와 같이 조회하는 메소드 사용하는데, Post을 허용하면 앞단 캐시문제부터 여러가지 문제가 발생할 수가 있다 기능에 맞게 HTTP 메소드 제약을 거는 것이 더 좋은 설계다.

### 메소드 제약

```
@RequestMapping(value = "/~~/~~/~~", method = RequestMethod.GET)
@RequestMapping(value = "/~~/~~/~~", method = RequestMethod.POST)
```

위와 같이 RequestMapping에서 제약을 걸 수 있지만, 더 편한 방법으로 애노테이션을 제공해준다.

### @GetMapping @PostMapping

```
@GetMapping(”/~~/~~/~~”)
@PostMapping(”/~~/~~/~~”)
```

- @GetMapping 안을 열어보면..

`@RequestMapping(method = RequestMethod.GET)`

- @PostMapping 안을 열어보면..

`@RequestMapping(method = RequestMethod.POST)`
