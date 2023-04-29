# 1. 시작

HTML 페이지의 경우 오류페이지만 있으면 문제를 해결할 수 있지만, API의 경우에는 각 오류 상황에 맞는 오류 응답 스펙을 정하고, JSON으로 데이터를 내려주어야 한다

### WebServerCustomizer

```java
package hello.exception;

import org.springframework.boot.web.server.ConfigurableWebServerFactory;
import org.springframework.boot.web.server.ErrorPage;
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;

@Component
public class WebServerCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
    @Override
    public void customize(ConfigurableWebServerFactory factory) {
        //404오류가 발생하면 /error-page/400 페이지로 이동
        ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404");
        //500오류가 발생하면 /error-page/500 페이지로 이동
        ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");
        //Runtime 예외(500오류, runtime 자식에러까지)가 발생하면 /error-page/500 페이지로 이동
        ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500");

        //등록
        factory.addErrorPages(errorPage404, errorPage500, errorPageEx);

    }
}
```

### ApiExceptionController

```java
package hello.exception.api;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@Slf4j
@RestController
public class ApiExceptionController {

    @GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id) {
        if (id.equals("ex")) {
            throw new RuntimeException("잘못된 사용자");
        }
        return new MemberDto(id, "hello" + id);
    }

    @Data
    @AllArgsConstructor
    static class MemberDto {
        private String memberId;
        private String name;
    }
}
```

### ErrorPageController

```java
@RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<Map<String, Object>> errorPage500Api(HttpServletRequest request, HttpServletResponse response) {
        log.info("API errorPage 500");

        HashMap<String, Object> result = new HashMap<>();
        Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION);
        result.put("status", request.getAttribute(ERROR_STATUS_CODE));
        result.put("message", ex.getMessage());

        Integer statusCode = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
        return new ResponseEntity<>(result, HttpStatus.valueOf(statusCode));
    }
```

- `produces = MediaType.APPLICATION_JSON_VALUE` : 클라이언트가 요청하는 HTTP Header의
Accept 의 값이 application/json 일 때 해당 메서드가 호출된다는 것이다. 결국 클라어인트가 받고 싶은 미디어타입이 json이면 이 컨트롤러의 메서드가 호출된다
- 응답 데이터를 위해서 Map 을 만들고 status , message 키에 값을 할당했다.
- Jackson 라이브러리는 Map 을 JSON 구조로 변환할 수 있다.
- ResponseEntity 를 사용해서 응답하기 때문에 메시지 컨버터가 동작하면서 클라이언트에 JSON이 반환된다

### 테스트

- api니까 postman을 실행 시켜줘야 함.

```
{
 "message": "잘못된 사용자",
 "status": 500
}
```

# 2. 스프링 부트 기본 오류 처리

### BasicErrrorController

```java
@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse 
response) {}

@RequestMapping
public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {}
```

- `errorHtml()` : `produces = MediaType.TEXT_HTML_VALUE` 클라이언트 요청의 Accept 해더 값이 text/html 인 경우에는 errorHtml() 을 호출해서 view를 제공한다.
- `error()` : 그외 경우에 호출되고 ResponseEntity 로 HTTP Body에 JSON 데이터를 반환한다.

### 스프링 부트 예외 처리

- 스프링 부트의 기본 설정은 오류 발생시 `/error`를 오류 페이지로 요청한다.
- `BasicErrorController` 는 server.error.path 로 수정 가능, 기본 경로 /
error 경로를 기본으로 받는다.

### 오류 정보 추가

`application.properties`

- server.error.include-binding-errors=always
- server.error.include-exception=true
- server.error.include-message=always
- server.error.include-stacktrace=always

### HTML 페이지 vs API 오류

- 스프링 부트가 제공하는 BasicErrorController 는 HTML 페이지를 제공하는 경우에는 매우 편리하다. 4xx, 5xx 등등 모두 잘 처리해준다
- API 오류 처리는 다른 차원의 이야기이다. API 마다, 각각의 컨트롤러나 예외마다 서로 다른 응답 결과를 출력해야 할 수도 있다

# 3. HandlerExceptionResolver 시작

예외가 발생해서 서블릿을 넘어 WAS까지 예외가 전달되면 HTTP 상태코드가 500으로 처리된다.

발생하는 예외에 따라서 다른 상태코드로 처리하고 싶을 때, 오류 메시지, 형식등을 API마다 다르게 처리하고 싶을 때 어떻게 해야하는가

`IllegalArgumentException`를 처리하지 못해서 컨트롤러 밖으로 남어가는 일이 발생하면 HTTP 상태코드를 400으로 처리하고 싶다 (클라이언트의 잘못을 400으로 명시하고싶은 것.)

`IllegalArgumentException` : Argument가 잘못 넘어온 것

### 예외처리 ExceptionResolver 적용 전 순서

![image](https://user-images.githubusercontent.com/62877858/235317333-78c023de-4f44-439f-865a-b00c164898e1.png)

1) WAS에서 Dispatcher Servlet 호출

2) Dispatcher Servlet → preHandle 호출

3) Dispatcher Servlet → 핸들러 어댑터 호출

4) 핸들러 어댑터 → 핸들러 호출

5) 핸들러에서 예외 발생

6) 핸들러 어댑터 → Dispatcher Servlet으로 예외 전달

7) Dispatcher Servlet → afterCompletion 호출

8) Dispatcher Servlet → WAS로 예외 전달 

### ExceptionResolver 적용 후 순서

![image](https://user-images.githubusercontent.com/62877858/235317347-e97f447b-7730-4c29-8fdf-83853916f62f.png)

1) WAS에서 Dispatcher Servlet 호출

2) Dispatcher Servlet → preHandle 호출

3) Dispatcher Servlet → 핸들러 어댑터 호출

4) 핸들러 어댑터 → 핸들러 호출

5) 핸들러에서 예외 발생

6) 핸들러 어댑터 → Dispatcher Servlet으로 예외 전달

7) Dispatcher Servlet → ExceptionResolver 호출해서 예외 해결 시도

8) Dispatcher Servlet → View : model을 가지고 렌더링

9) afterCompletion 호출

10) WAS 정상 응답

- ExceptionResolver : 핸들러에서 생긴 예외를 해결해주는 역할

### MyHandlerExceptionResolver

```java
package hello.exception.resolver;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        try {
            //Exception이 IllegalArgumentException이면
            if(ex instanceof IllegalArgumentException){
                //IllegalArgumentException 예외가 터지면 상태 코드를 400으로 바꿀 것이다.
                log.info("IllegalArgumentException resolver to 400");
                //Exception을 바꿔줌.
                response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage());
                return new ModelAndView();
            }
        } catch (IOException e) {
            log.error("resolver ex", e);
        }

        return null;
    }
}
```

- ExceptionResolver 가 ModelAndView 를 반환하는 이유는 마치 try, catch를 하듯이, Exception을 처리해서 정상 흐름 처럼 변경하는 것이 목적이다. 이름 그대로 Exception 을 Resolver(해결)하는 것이 목적이다
- `IllegalArgumentException`이 발생하면 `response.sendError(400)`를 호출해서 HTTP
상태 코드를 400으로 지정하고, 빈 `ModelAndView` 를 반환한다

### WebConfig

```java
@Override
    public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        resolvers.add(new MyHandlerExceptionResolver());
    }
```

- `WebMvcConfigurer` 통해 등록
- `configureHandlerExceptionResolvers(..)` 를 사용하면 스프링이 기본으로 등록하는
`ExceptionResolver` 가 제거되므로 주의, `extendHandlerExceptionResolvers` 를 사용하자.
- ExceptionResolver를 여러 개 등록할 수 있다.

### 반환 값에 따른 동작 방식

- 빈 ModelAndView: new ModelAndView() 처럼 빈 ModelAndView 를 반환하면 뷰를 렌더링 하지 않고, 정상 흐름으로 서블릿이 리턴된다.
- ModelAndView 지정: ModelAndView 에 View , Model 등의 정보를 지정해서 반환하면 뷰를 렌더링 한다.
- null: null 을 반환하면, 다음 ExceptionResolver 를 찾아서 실행한다. 만약 처리할 수 있는
ExceptionResolver 가 없으면 예외 처리가 안되고, 기존에 발생한 예외를 서블릿 밖으로 던진다.

### ExceptionResolver 활용법

예외 상태 코드 변환

- 예외를 response.sendError(xxx) 호출로 변경해서 서블릿에서 상태 코드에 따른 오류를
처리하도록 위임
- 이후 WAS는 서블릿 오류 페이지를 찾아서 내부 호출, 예를 들어서 스프링 부트가 기본으로 설정한 /error 가 호출됨

뷰 템플릿 처리

- ModelAndView 에 값을 채워서 예외에 따른 새로운 오류 화면 뷰 렌더링 해서 고객에게 제공

API 응답 처리

- response.getWriter().println("hello"); 처럼 HTTP 응답 바디에 직접 데이터를 넣어주는
것도 가능하다. 여기에 JSON 으로 응답하면 API 응답 처리를 할 수 있다.

### 테스트

Postman으로 실행해보면,

- http://localhost:8080/api/members/ex HTTP 상태 코드 500
- http://localhost:8080/api/members/bad HTTP 상태 코드 400

# 4. HandlerExceptionResolver

예외가 발생하면 WAS까지 예외가 던져지고, WAS에서 오류 페이지 정보를 찾아서 다시 /error를 호출하는 과정은 매우 복잡함.

ExceptionResolver를 활용하면 예외가 발생했을 때 이런 복잡한 과정없이 WAS까지 가지 않고, 문제를 깔끔하게 해결할 수 있다.

### UserException

```java
package hello.exception.exception;

public class UserException extends RuntimeException{
    public UserException() {
        super();
    }

    public UserException(String message) {
        super(message);
    }

    public UserException(String message, Throwable cause) {
        super(message, cause);
    }

    public UserException(Throwable cause) {
        super(cause);
    }

    protected UserException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}
```

### ApiExceptionController

```java
@GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id) {
        if (id.equals("ex")) {
            throw new RuntimeException("잘못된 사용자");
        }
        if (id.equals("bad")) {
            throw new IllegalArgumentException("잘못된 입력 값");
        }
        if (id.equals("user-ex")) {
            throw new UserException("사용자 오류");
        }
        return new MemberDto(id, "hello" + id);
    }

```

- 예외 추가함

### UserHandlerExceptionResolver

```java
package hello.exception.resolver;

import com.fasterxml.jackson.databind.ObjectMapper;
import hello.exception.exception.UserException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

@Slf4j
public class UserHandlerExceptionResolver implements HandlerExceptionResolver {

    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        try{
            if (ex instanceof UserException) {
                log.info("UserException resolver to 400");
                //acceptHeader 꺼내기
                String acceptHeader = request.getHeader("accept");
                //상태코드 400으로
                response.setStatus(HttpServletResponse.SC_BAD_REQUEST);

                if ("application/json".equals(acceptHeader)) {
                    Map<String, Object> errorResult = new HashMap<>();
                    errorResult.put("ex", ex.getClass());
                    errorResult.put("message", ex.getMessage());

                    String result = objectMapper.writeValueAsString(errorResult);

                    response.setContentType("application/json");
                    response.setCharacterEncoding("utf-8");
                    response.getWriter().write(result);

                    //서블릿 컨테이너까지 가지 않고 여기서 끝나고, 정상 리턴을 함
                    return new ModelAndView();
                } else {
                    // TEXT/HTML
                    // acceptHeader가 json이 아닌 경우에
                    // templates/error/500 경로의 뷰를 렌더링
                    return new ModelAndView("error/500");
                }
            }
        } catch (IOException e){
            log.error("resolver ex", e);
        }
        return null;
    }
}
```

- HTTP 요청 해더의 ACCEPT 값이 application/json 이면 JSON으로 오류를 내려주고, 그 외 경우에는 error/500에 있는 HTML 오류 페이지를 보여준다

### WebConfig

```java
@Override
    public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        resolvers.add(new MyHandlerExceptionResolver());
        resolvers.add(new UserHandlerExceptionResolver());
    }
```

- UserHandlerExceptionResolver 추가

### 정리

- 컨트롤러에서 예외가 발생해도 `ExceptionResolver` 를 사용하면 `ExceptionResolver` 에서 예외를 처리해준다.
- 서블릿 컨테이너까지 예외가 전달되지 않고, 스프링 MVC에서 예외 처리는 끝이 남
- WAS에서는 정상 처리가 됨

# 5. 스프링이 제공하는 ExceptionResolver

`ExceptionResolver` 는  `HandlerExceptionResolverComposite` 에 다음 순서로 등록

1) ExceptionHandlerExceptionResolver

2) ResponseStatusExceptionResolver

3) DefaultHandlerExceptionResolver (우선 순위가 가장 낮다)

### ExceptionHandlerExceptionResolver

@ExceptionHandler 을 처리한다. API 예외 처리는 대부분 이 기능으로 해결한다. 조금 뒤에 자세히 설명한다.

### ResponseStatusExceptionResolver

HTTP 상태 코드를 지정해준다.
예) @ResponseStatus(value = HttpStatus.NOT_FOUND)

### DefaultHandlerExceptionResolver

스프링 내부 기본 예외를 처리한다.

## ResponseStatusExceptionResolver

예외에 따라서 HTTP 상태 코드를 지정해주는 역할을 한다

### 두 가지 경우에 처리 한다.

- @ResponseStatus 가 달려있는 예외
- ResponseStatusException 예외

### BadRequestException

```java
package hello.exception.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "오류!")
public class BadRequestException extends RuntimeException {

}
```

- BadRequestException 예외가 컨트롤러 밖으로 넘어가면 ResponseStatusExceptionResolver 예외가 해당 애노테이션을 확인해서 오류 코드를 HttpStatus.BAD_REQUEST (400)으로 변경하고, 메시지도 담는다
- ResponseStatusExceptionResolver 코드를 확인해보면 결국 response.sendError(statusCode,
resolvedReason) 를 호출하는 것을 확인할 수 있다. sendError(400) 를 호출했기 때문에 WAS에서 다시 오류 페이지( /error )를 내부 요청한다.
- 예외 발생한 것을 sendError로 바꿔준 것일 뿐

### ApiExceptionController

```java
@GetMapping("/api/response-status-ex1")
    public String responseStatusEx1() {
        throw new BadRequestException();
    }

    @GetMapping("/api/response-status/ex2")
    public String responseStatusEx2(){
        //상태코드와 메시지를 한번에 처리할 수 있음.
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "error.bad", new IllegalArgumentException());
    }
```

### 메세지 기능

- BadRequestException

```java
package hello.exception.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "error.bad")
public class BadRequestException extends RuntimeException {

}
```

- messages.properties

```java
error.bad=잘못된 요청 오류입니다. 메시지 사용
```

- reason을 가져와서 message 소스를 찾아본다.

### 메시지 한글 깨짐

messages.properties에서 한글 문자가 깨져서 나오면,

설정 - 파일 인코딩 - 프로퍼티 파일에 대한 디폴트 인코딩 UTF-8로 설정

# 6. API 예외 처리 - 스프링이 제공하는 ExceptionResolver2

대표적으로 파라미터 바인딩 시점에 타입이 맞지 않으면 내부에서 TypeMismatchException 이
발생하는데, 이 경우 예외가 발생했기 때문에 그냥 두면 서블릿 컨테이너까지 오류가 올라가고, 결과적으로 500 오류가 발생한다.

그런데 파라미터 바인딩은 대부분 클라이언트가 HTTP 요청 정보를 잘못 호출해서 발생하는 문제이다. HTTP 에서는 이런 경우 HTTP 상태 코드 400을 사용하도록 되어 있다.
DefaultHandlerExceptionResolver 는 이것을 500 오류가 아니라 HTTP 상태 코드 400 오류로
변경한다. 스프링 내부 오류를 어떻게 처리할지 수 많은 내용이 정의되어 있다

### ApiExceptionController

```java
@GetMapping("/api/default-handler-ex")
public String defaultException(@RequestParam Integer data) {
 return "ok";
}
```

### 테스트 결과

```java
{
    "timestamp": "2023-04-21T08:44:25.924+00:00",
    "status": 400,
    "error": "Bad Request",
    "exception": "org.springframework.web.method.annotation.MethodArgumentTypeMismatchException",
    "message": "Failed to convert value of type 'java.lang.String' to required type 'java.lang.Integer'; nested exception is java.lang.NumberFormatException: For input string: \"qqq\"",
    "path": "/api/default-handler-ex"
}
```

실행 결과를 보면 HTTP 상태 코드가 400인 것을 확인할 수 있음

# 7. @ExceptionHandler

### HTML 화면 예외

- 웹 브라우저에 HTML 화면을 제공할 때는 오류가 발생하면 BasicErrorController 를 사용해서 해당 오류의 오류 화면만 보여주면 편하다

### API 예외 처리 - @ExceptionHandler

스프링은 API 예외 처리 문제를 해결하기 위해 @ExceptionHandler 라는 애노테이션을 사용하는 매우 편리한 예외 처리 기능을 제공하는데, 이것이 바로 ExceptionHandlerExceptionResolver 이다. 스프링은 ExceptionHandlerExceptionResolver 를 기본으로 제공하고, 기본으로 제공하는
ExceptionResolver 중에 우선순위도 가장 높다. 실무에서 API 예외 처리는 대부분 이 기능을 사용한다

## 코드

### ErrorResult

```java
package hello.exception.exhandler;

import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
public class ErrorResult {
    private String code;
    private String message;
}
```

### ApiExceptionV2Controller

```java
package hello.exception.api;

import hello.exception.exception.UserException;
import hello.exception.exhandler.ErrorResult;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@Slf4j
@RestController
public class ApiExceptionV2Controller {

    /**
     * IllegalArgumentException 오류가 터지면, 잡고 해당 로직이 수행됨.
     * ErrorResult는 그대로 반환된다.
     *
     * IllegalArgumentException 예외의 자식까지 잡아준다.
     * */
    @ResponseStatus(HttpStatus.BAD_REQUEST)//상태값 변경, 상태값 변경 없이 성공(200)으로 뜬다.
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandler(IllegalArgumentException e) {
        log.error("[exceptionHandler] ex", e);
        return new ErrorResult("BAD", e.getMessage());
    }

    /**
     * UserException 예외의 자식까지 잡아준다.
     *
     * @ExceptionHandler 지정을 생략하게 되면, 메서드 파라미터의 예외가 지정이 된다.
     * */
    @ExceptionHandler
    public ResponseEntity<ErrorResult> userExHandler(UserException e) {
        log.error("[exceptionHandler] ex", e);
        ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
        return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);

    }
    /**
     * @ExceptionHandler는 해당 컨트롤러 안에서만 적용이 된다.
     *
     * Exception : 나머지 예외의 자식들은 여기로 넘어온다.
     * */
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ErrorResult exHandler(Exception e) {
        log.error("[exceptionHandler] ex", e);
        return new ErrorResult("EX", "내부 오류");
    }

    @GetMapping("/api2/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id) {
        if (id.equals("ex")) {
            throw new RuntimeException("잘못된 사용자");
        }
        if (id.equals("bad")) {
            throw new IllegalArgumentException("잘못된 입력 값");
        }
        if (id.equals("user-ex")) {
            throw new UserException("사용자 오류");
        }
        return new MemberDto(id, "hello" + id);
    }

    @Data
    @AllArgsConstructor
    static class MemberDto {
        private String memberId;
        private String name;
    }

}
```

- @ExceptionHandler 애노테이션을 선언하고, 해당 컨트롤러에서 처리하고 싶은 예외를 지정해주면 된다. 해당 컨트롤러에서 예외가 발생하면 이 메서드가 호출된다.

### 우선순위

자세한 것이 우선권을 가진다.

```java
@ExceptionHandler(부모예외.class)
public String 부모예외처리()(부모예외 e) {}

@ExceptionHandler(자식예외.class)
public String 자식예외처리()(자식예외 e) {}
```

- 더 자세한 자식예외처리가 호출된다.

### 다양한 예외

```java
@ExceptionHandler({AException.class, BException.class})
public String ex(Exception e) {
 log.info("exception e", e);
}
```

### 예외 생략

```java
@ExceptionHandler
public ResponseEntity<ErrorResult> userExHandle(UserException e) {}
```

- 메서드 파라미터의 예외가 지정된다.

### HTML 오류 화면

```java
@ExceptionHandler(ViewException.class)
public ModelAndView ex(ViewException e) {
 log.info("exception e", e);
 return new ModelAndView("error");
}
```

- 이렇게 잘 사용 안한다.

# 8. @ControllerAdvice

@ExceptionHandler 를 사용해서 예외를 깔끔하게 처리할 수 있게 되었지만, 정상 코드와 예외 처리 코드가 하나의 컨트롤러에 섞여 있다. @ControllerAdvice 또는 @RestControllerAdvice 를 사용하면 둘을 분리할 수 있다.

### ExControllerAdvice

```java
package hello.exception.exhandler.advice;

import hello.exception.exception.UserException;
import hello.exception.exhandler.ErrorResult;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RestControllerAdvice;
/**
 * 컨트롤러에 있는 것들을 여기에 모아서 처리해주는 것
 * */
@Slf4j
@RestControllerAdvice(basePackages = "hello.exception.api")
public class ExControllerAdvice {
    /**
     * IllegalArgumentException 오류가 터지면, 잡고 해당 로직이 수행됨.
     * ErrorResult는 그대로 반환된다.
     *
     * IllegalArgumentException 예외의 자식까지 잡아준다.
     * */
    @ResponseStatus(HttpStatus.BAD_REQUEST)//상태값 변경, 상태값 변경 없이 성공(200)으로 뜬다.
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandler(IllegalArgumentException e) {
        log.error("[exceptionHandler] ex", e);
        return new ErrorResult("BAD", e.getMessage());
    }
    /**
     * UserException 예외의 자식까지 잡아준다.
     *
     * @ExceptionHandler 지정을 생략하게 되면, 메서드 파라미터의 예외가 지정이 된다.
     * */
    @ExceptionHandler
    public ResponseEntity<ErrorResult> userExHandler(UserException e) {
        log.error("[exceptionHandler] ex", e);
        ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
        return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);

    }
    /**
     * @ExceptionHandler는 해당 컨트롤러 안에서만 적용이 된다.
     *
     * Exception : 나머지 예외의 자식들은 여기로 넘어온다.
     * */
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ErrorResult exHandler(Exception e) {
        log.error("[exceptionHandler] ex", e);
        return new ErrorResult("EX", "내부 오류");
    }
}
```

### ApiExceptionV2Controller

코드에 있는 @ExceptionHandler 모두 제거

```java
package hello.exception.api;

import hello.exception.exception.UserException;
import hello.exception.exhandler.ErrorResult;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@Slf4j
@RestController
public class ApiExceptionV2Controller {
    
    @GetMapping("/api2/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id) {
        if (id.equals("ex")) {
            throw new RuntimeException("잘못된 사용자");
        }
        if (id.equals("bad")) {
            throw new IllegalArgumentException("잘못된 입력 값");
        }
        if (id.equals("user-ex")) {
            throw new UserException("사용자 오류");
        }
        return new MemberDto(id, "hello" + id);
    }

    @Data
    @AllArgsConstructor
    static class MemberDto {
        private String memberId;
        private String name;
    }

}
```

### @ControllerAdvice

- @ControllerAdvice 는 대상으로 지정한 여러 컨트롤러에 @ExceptionHandler , @InitBinder 기능을 부여해주는 역할을 한다.
- @ControllerAdvice 에 대상을 지정하지 않으면 모든 컨트롤러에 적용된다. (글로벌 적용)
- @RestControllerAdvice 는 @ControllerAdvice 와 같고, @ResponseBody 가 추가되어 있다.
@Controller , @RestController 의 차이와 같다.

### 대상 컨트롤러 지정 방법

```java
// Target all Controllers annotated with @RestController
// 컨트롤러 지정
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// Target all Controllers within specific packages
// 패키지 지정
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
// 클래스, 혹은 부모 클래스 지정
@ControllerAdvice(assignableTypes = {ControllerInterface.class,
AbstractController.class})
public class ExampleAdvice3 {}
```

실무에서는 예외를 공통으로 묶는게 중요하다. 그래서 자주 이 방법을 사용한다.
