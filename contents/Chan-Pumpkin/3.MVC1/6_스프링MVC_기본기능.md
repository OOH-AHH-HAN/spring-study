# 1. 프로젝트 생성

## 프로젝트 열기

> project open→해당 프로젝트 [gradle.build](http://gradle.build) 파일 클릭→open as project
> 
- 스프링 3.0은 자바 17버전 필수

## jar로 왜 했나?

- JSP를 사용하지 않음
- War는 톰캣을 별도 설치해서 이 빌드된 파일을 넣어야할 때 사용해야함 War를 사용하면 내장 서버도 사용하지만, 주로 외부 서버에 배포하는 목적으로 사용됨
- Jar는 내장 톰캣을 사용하고, webapp 경로도 사용하지 않고, 내장 서버 사용에 최적화 되어있는 기능 주로 이 방식을 사용함.

## lombok 셋팅

Settings(mac: Preferences)→Annotation Processors→Enable annotation procession 체크

## Welcome 페이지

스프링 부트에 Jar 를 사용하면 /resources/static/ 위치에 index.html 파일을 두면 Welcome 페이지로 처리해준다. (스프링 부트가 지원하는 정적 컨텐츠 위치에 /index.html 이 있으면 된다

# 2. 로깅 간단히 알아보기

운영 시스템에서는 System.out.println() 같은 시스템 콘솔을 사용해서 필요한 정보를 출력하면 안된다. 별도의 로깅 라이브러리를 사용해서 로그를 출력한다.

## 로깅 라이브러리

수 많은 로그 라이브러리가 있는데, 그것을 통합한 인터페이스를 제공하는 것이 SLF4J 라이브러리이다.

실무에서는 스프링 부트가 기본으로 제공하는 Logback을 대부분 사용한다.

SLF4J : [http://www.slf4j.org](http://www.slf4j.org/)
Logback : [http://logback.qos.ch](http://logback.qos.ch/)

## 로그 사용법

### 로그 선언 방법

```java
@RestController
public class LogTestController {
    private final Logger log = LoggerFactory.getLogger(getClass());
```

`import org.slf4j.Logger;`의 Logger를 사용해야함.

### 로그 어노테이션

```java
@Slf4j
@RestController
public class LogTestController {
    //@Slf4j 어노테이션이 있으면 생략 @Slf4j 어노테이션이 대신 해줌.
    //private final Logger log = LoggerFactory.getLogger(getClass());
```

`@Slf4j` 을 사용하면 로그를 선언하지 않아도 사용할 수 있다.

## 잠깐! @RestController?

```java
@Slf4j
@RestController
public class LogTestController {
    //@Slf4j 어노테이션이 있으면 생략 @Slf4j 어노테이션이 대신 해줌.
    //private final Logger log = LoggerFactory.getLogger(getClass());
```

`@Controller`: 어노테이션은 반환을 할 때 뷰를 찾는다.
`@RestController`: 어노테이션은 문자를 반환하면 HTTP messagebody에 스트링이 바로 반환이 된다.

## System.out.print와 logger의 결과 차이

```
name = Spring
2023-02-18 18:07:01.053  INFO 61056 --- [nio-8080-exec-3] hello.springmvc.basic.LogTestController  :  info log=Spring
```

- INFO 61056: 프로세스 아이디
- [nio-8080-exec-3]: 현재 실행한 스레드
- hello.springmvc.basic.LogTestController: 현재 나의 컨트롤러

## 로그 사용

```java
package hello.springmvc.basic;

import lombok.extern.slf4j.Slf4j;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Slf4j
@RestController
//@Controller 어노테이션은 반환을 할 때 뷰를 찾지만,
//@RestController 어노테이션은 문자를 반환하면 HTTP messagebody에 스트링이 바로 반환이 된다.
public class LogTestController {
    //@Slf4j 어노테이션이 있으면 생략 @Slf4j 어노테이션이 대신 해줌.
    //private final Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping("/log-test")
    public String logTest() {
        String name = "Spring";

        System.out.println("name = " + name);

        //로그 레벨
        log.trace("trace log={}", name);
        log.debug("debug log={}", name);
        log.info("info log={}", name);
        log.warn("warn log={}", name);
        log.error("error log={}", name);

        return "ok";
    }
}
```

### 로그 레벨

> LEVEL : TRACE > DEBUG > INFO > WARN > ERROR
> 
- 개발서버는 debug 레벨
- 운영서버는 info 레벨

### 로그 레벨 설정

application.properties

```
#전체 로그 레벨 설정(기본 info)
logging.level.root=info

#hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=trace
```

### 오잉? trace와 debug는 왜 안나오지..?

```sql
name = Spring
2023-02-18 18:16:03.562  INFO 62648 --- [nio-8080-exec-1] hello.springmvc.basic.LogTestController  : info log=Spring
2023-02-18 18:16:03.562  WARN 62648 --- [nio-8080-exec-1] hello.springmvc.basic.LogTestController  : warn log=Spring
2023-02-18 18:16:03.563 ERROR 62648 --- [nio-8080-exec-1] hello.springmvc.basic.LogTestController  : error log=Spring
```

로그 레벨 기본이 INFO로 설정되어있기 때문이다. 로그 레벨을 설정해서 TRACE, DEBUG 볼 수 있게 하면 된다.

## system.out.print를 쓰지 않는 이유?

system.out.print를 사용하면, 설정을 할 수가 없어서 운영에서는 무조건 로그에 찍힌다. 

한마디로, 운영서버에는 로그 폭탄을 받을 수 있기에 system.out.print를 운영에서 사용하는 것을 지양한다.

## 로그 설정 주의점?

```sql
#전체 로그 레벨 설정(기본 info)
logging.level.root=info
```

application.properties에서 로그 레벨 설정을 debug나 trace로 하면 모든 라이브러리들의 로그들도 많이 찍히기 때문에 로그 레벨 설정은 info로 설정하거나, 건들지 않는 것이 좋다.

## 올바른 로그 사용법

`log.debug("data="+data)`
로그 출력 레벨을 info로 설정해도 해당 코드에 있는 "data="+data가 실제 실행이 되어 버린다. 결과적으로 문자 더하기 연산이 발생한다.
`log.debug("data={}", data)`
로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다. 따라서 앞과 같은 의미없는 연산이 발생하지 않는다.

# 3. 요청 매핑

요청이 왔을 때, 어떤 컨트롤러에 매핑을 해야하는지

## 매핑 정보

`@RestController`

- `@Controller` 는 반환 값이 String 이면 뷰 이름으로 인식된다. 그래서 뷰를 찾고 뷰가 랜더링 된다.
- `@RestController` 는 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력한다. 따라서 실행 결과로 ok 메세지를 받을 수 있다.
- `@RequestMapping`("/hello-basic")
`/hello-basic` URL 호출이 오면 이 메서드가 실행되도록 매핑한다.
대부분의 속성을 배열[] 로 제공하므로 다중 설정이 가능하다. `{"/hello-basic", "/hello-go"}`

### 스프링 부트 3.0 이전 -둘다 허용

다음 두가지 요청은 다른 URL이지만, 스프링은 다음 URL 요청들을 같은 요청으로 매핑한다.

- 매핑: /hello-basic
- URL 요청: /hello-basic , /hello-basic/

### 스프링 부트 3.0 이후

스프링 부트 3.0 부터는 /hello-basic , /hello-basic/ 는 서로 다른 URL 요청을 사용해야 한다.

기존에는 마지막에 있는 / (slash)를 제거했지만, 스프링 부트 3.0 부터는 마지막의 / (slash)를 유지한다. 따라서 다음과 같이 다르게 매핑해서 사용해야 한다.

- 매핑: /hello-basic → URL 요청: /hello-basic
- 매핑: /hello-basic/ → URL 요청: /hello-basic/

## 매핑 예시

```java
package hello.springmvc.basic.requestmapping;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.*;

@RestController
public class MappingController {

    private Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping(value = "/hello-basic")
    public String helloBasic() {
        log.info("helloBasic");
        return "ok";
    }
    /**
     * method 특정 HTTP 메서드 요청만 허용 * GET, HEAD, POST, PUT, PATCH, DELETE
     *
     * 여기에 POST 요청을 하면 스프링 MVC는 HTTP 405 상태코드(Method Not Allowed)를
     * 반환한다
     */
    @RequestMapping(value = "/hello-basic-get-v1", method = RequestMethod.GET)
    public String mappingGetV1() {
        log.info("mappingGetV1");
        return "ok";
    }

    /**
     * 편리한 축약 애노테이션 (코드보기)
     * @GetMapping
     * @PostMapping
     * @PutMapping
     * @DeleteMapping
     * @PatchMapping
     *
     * HTTP 메서드를 축약한 애노테이션을 사용하는 것이 더 직관적이다. 코드를 보면 내부에서
     * @RequestMapping 과 method 를 지정해서 사용하는 것을 확인할 수 있다.
     */
    @GetMapping(value = "/mapping-get-v2")
    public String mappingGetV2() {
        log.info("mapping-get-v2");
        return "ok";
    }

    /**
     * PathVariable 사용
     * 변수명이 같으면 생략 가능
     *
     * @PathVariable("userId") String userId -> @PathVariable userId
     * url 자체에 값이 들어가있는 것
     * 경로 변수(쿼리 파라미터 아님.)
     *
     * @PathVariable 의 이름과 파라미터 이름이 같으면
     * @PathVariable 의 이름을 생략할 수 있다
     */
    @GetMapping("/mapping/{userId}")
    public String mappingPath(@PathVariable("userId") String data) {
        log.info("mappingPath userId={}", data);
        return "ok";
    }

    /**
     * PathVariable 사용 다중
     */
    @GetMapping("/mapping/users/{userId}/orders/{orderId}")
    public String mappingPath(@PathVariable String userId, @PathVariable Long
            orderId) {
        log.info("mappingPath userId={}, orderId={}", userId, orderId);
        return "ok";
    }

    /**
     * 파라미터로 추가 매핑
     * params="mode",
     * params="!mode"
     * params="mode=debug"
     * params="mode!=debug" (! = )
     * params = {"mode=debug","data=good"}
     *
     * 요청할 때, mode를 빼면 400 발생
     */
    @GetMapping(value = "/mapping-param", params = "mode=debug")
    public String mappingParam() {
        log.info("mappingParam");
        return "ok";
    }

    /**
     * 특정 헤더로 추가 매핑
     * headers="mode",
     * headers="!mode"
     * headers="mode=debug"
     * headers="mode!=debug" (! = )
     *
     * 헤더에 mode에 debug값을 넣어줘야함
     */
    @GetMapping(value = "/mapping-header", headers = "mode=debug")
    public String mappingHeader() {
        log.info("mappingHeader");
        return "ok";
    }

    /**
     * Content-Type 헤더 기반 추가 매핑 Media Type
     * consumes="application/json"
     * consumes="!application/json"
     * consumes="application/*"
     * consumes="*\/*"
     * MediaType.APPLICATION_JSON_VALUE
     *
     * 설명:
     * Headers에 content-type이 application/json타입이여야 호출 가능
     * HTTP 요청의 Content-Type 헤더를 기반으로 미디어 타입으로 매핑한다.
     * 만약 맞지 않으면 HTTP 415 상태코드(Unsupported Media Type)을 반환한다
     * 
     * 사용예시:
     * consumes = "text/plain"
     * consumes = {"text/plain", "application/*"}
     * consumes = MediaType.TEXT_PLAIN_VALUE
     * */
    @PostMapping(value = "/mapping-consume", consumes = "application/json")
    public String mappingConsumes() {
        log.info("mappingConsumes");
        return "ok";
    }

    /**
     * Accept 헤더 기반 Media Type
     * produces = "text/html"
     * produces = "!text/html" * produces = "text/*"
     * produces = "*\/*"
     *
     * Headers에 Accept가 text/html타입말고는 호출 불가능
     *
     * HTTP 요청의 Accept 헤더를 기반으로 미디어 타입으로 매핑한다.
     * 만약 맞지 않으면 HTTP 406 상태코드(Not Acceptable)을 반환한다.
     */
    @PostMapping(value = "/mapping-produce", produces = "text/html")
    public String mappingProduces() {
        log.info("mappingProduces");
        return "ok";
    }
}
```

# 4. 요청 매핑-API 예시

```java
package hello.springmvc.basic.requestmapping;

import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/mapping/users")
//공통 url 설정할 수 있음.
public class MappingClassController {

    @GetMapping
    public String user() {
        return "get users";
    }

    @PostMapping
    public String addUser() {
        return "post users";
    }

    @GetMapping("/{userId}")
    public String findUser(@PathVariable String userId) {
        return "get userId="+ userId;
    }

    @PatchMapping("/{userId}")
    public String updateUser(@PathVariable String userId) {
        return "get userId="+ userId;
    }

    @DeleteMapping("/{userId}")
    public String deleteUser(@PathVariable String userId) {
        return "get userId="+ userId;
    }
}
```

- @RequestMapping("/mapping/users")을 클래스 레벨에 매핑 정보를 두면 메서드 레벨에서 해당 정보를 조합해서 사용할 수 있다.

# 5. HTTP 요청 - 기본, 헤더 조회

```java
package hello.springmvc.basic.request;

import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpMethod;
import org.springframework.util.MultiValueMap;
import org.springframework.web.bind.annotation.CookieValue;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Locale;

@Slf4j
@RestController
public class RequestHeaderController {

    @RequestMapping("/headers")
    public String headers(HttpServletRequest request,
                          HttpServletResponse response,
                          HttpMethod httpMethod,
                          Locale locale,
                          @RequestHeader MultiValueMap<String, String> headerMap,
                          @RequestHeader("host") String host,
                          @CookieValue(value = "myCookie", required=false) String cookie
                            ) {
        log.info("request={}", request);
        log.info("response={}", response);
        log.info("httpMethod={}", httpMethod);
        log.info("locale={}", locale);
        log.info("headerMap={}", headerMap);
        log.info("host={}", host);
        log.info("cookie={}", cookie);

        return "ok";
    }
}
```

### 결과

```java
2023-02-19 18:47:40.161  INFO 17272 --- [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet        : Completed initialization in 2 ms
2023-02-19 18:47:40.237  INFO 17272 --- [nio-8080-exec-2] h.s.b.request.RequestHeaderController    : request=org.apache.catalina.connector.RequestFacade@4a9b3bd
2023-02-19 18:47:40.239  INFO 17272 --- [nio-8080-exec-2] h.s.b.request.RequestHeaderController    : response=org.apache.catalina.connector.ResponseFacade@20b8199b
2023-02-19 18:47:40.239  INFO 17272 --- [nio-8080-exec-2] h.s.b.request.RequestHeaderController    : httpMethod=GET
2023-02-19 18:47:40.240  INFO 17272 --- [nio-8080-exec-2] h.s.b.request.RequestHeaderController    : locale=ko_KR
2023-02-19 18:47:40.240  INFO 17272 --- [nio-8080-exec-2] h.s.b.request.RequestHeaderController    : headerMap={mode=[debug], content-type=[application/json], user-agent=[PostmanRuntime/7.31.0], accept=[*/*], postman-token=[0eb4dbf8-46ee-4fc2-bc52-c0c9762c44bb], host=[localhost:8080], accept-encoding=[gzip, deflate, br], connection=[keep-alive], content-length=[27]}
2023-02-19 18:47:40.240  INFO 17272 --- [nio-8080-exec-2] h.s.b.request.RequestHeaderController    : host=localhost:8080
2023-02-19 18:47:40.240  INFO 17272 --- [nio-8080-exec-2] h.s.b.request.RequestHeaderController    : cookie=null
```

### HttpMethod

Http 메서드를 조회함.

### Locale

스프링에서 우선순위 가장 높은 Locale정보, 로케일 리졸버로 수정할 수 있음. 

### @RequestHeader MultiValueMap<String, String> headerMap

모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.

### MultiValueMap

- Map과는 유사하지만, 하나의 키에 여러 값을 받을 수 있다.
- Http header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용함.
- 배열로 반환이 됨.

### @RequestHeader("host") String host

- 특정 HTTP 헤더를 조회한다.
- 속성

필수 값 여부: required

기본 값 속성: defaultValue

### @CookieValue(value = "myCookie", required = false) String cookie

- 특정 쿠키를 조회한다.
- 속성

필수 값 여부: required

기본 값: defaultValue

### @Controller의 사용 가능한 파라미터 목록 메뉴얼 참고

[Web on Servlet Stack](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-methods)

> Spring Web MVC → 1.3 Annotated Controllers → 1.3.3. Handler Methods → Method Arguments
> 

# 6. 쿼리 파라미터, HTML Form

## 서버로 요청 데이터 전달 3가지 방법

- GET 쿼리 파라미터
- POST HTML Form
- HTTP message body에 데이터를 직접 담아서 요청

## 예제

```java
package hello.springmvc.basic.request;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
@Controller
public class RequestParamController {

    @RequestMapping("/request-param-v1")
    public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));
        log.info("username={}, age={}", username, age);

        response.getWriter().write("ok");
    }
}
```

### 결과

```java
2023-02-19 19:14:13.385  INFO 39628 --- [nio-8080-exec-2] h.s.b.request.RequestParamController     : username=test, age=12
```

### request.getParameter

HttpServletRequest가 제공하는 방식으로 요청 파라미터를 조회함.

## PostForm 페이지 생성

```java
<!DOCTYPE html>
<html>
<head>
 <meta charset="UTF-8">
 <title>Title</title>
</head><body>
 <form action="/request-param-v1" method="post">
 username: <input type="text" name="username" />
 age: <input type="text" name="age" />
 <button type="submit">전송</button>
 </form>
</body>
</html>
```

- 리소스는 /resources/static 아래에 두면 스프링 부트가 자동으로 인식한다.
- 요청할 때, 해당 페이지 경로 url를 입력하면 됨.

## 참고

Jar를 사용하면 webapp 경로를 사용할 수 없다. 이제부터 정적 리소스도 클래스 경로에 함께 포함해야 함.

# 7. HTTP 요청 파라미터 - @RequestParam

```java
package hello.springmvc.basic.request;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Map;

@Slf4j
@Controller
public class RequestParamController {

    @RequestMapping("/request-param-v1")
    public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));
        log.info("username={}, age={}", username, age);

        response.getWriter().write("ok");
    }

    /**
     * @RequestParam 사용
     *
     * */
    @ResponseBody
    @RequestMapping("/request-param-v2")
    public String requestParamV2(
            @RequestParam("username") String memberName,
            @RequestParam("age") int memberAge) {

        log.info("username={}, age={}", memberName, memberAge);
        return "ok";
    }

    /**
     * @RequestParam 변수명만 사용
     *
     * */
    @ResponseBody
    @RequestMapping("/request-param-v3")
    public String requestParamV3(
            @RequestParam String username,
            @RequestParam int age) {

        log.info("username={}, age={}", username, age);
        return "ok";
    }

    /**
     * @RequestParam 없이 변수명만 사용
     *
     * */
    @ResponseBody
    @RequestMapping("/request-param-v4")
    public String requestParamV4(
            String username,
            int age) {

        log.info("username={}, age={}", username, age);
        return "ok";
    }

    /**
     * @RequestParam required : 파라미터 필수 여부
     *
     * */
    @ResponseBody
    @RequestMapping("/request-param-required")
    public String requestParamRequired(
            @RequestParam(required = true) String username,
            @RequestParam(required = false) Integer age) {

        log.info("username={}, age={}", username, age);
        return "ok";
    }

    /**
     * @RequestParam defaultValue : 기본 값 적용
     *
     * */
    @ResponseBody
    @RequestMapping("/request-param-default")
    public String requestParamDefault(
            @RequestParam(defaultValue = "guest") String username,
            @RequestParam(defaultValue = "-1") int age) {

        log.info("username={}, age={}", username, age);
        return "ok";
    }

    /**
     * @RequestParam Map : 파라미터를 한번에 담을 수 있다.
     *
     * */
    @ResponseBody
    @RequestMapping("/request-param-map")
    public String requestParamMap(@RequestParam Map<String, Object> paramMap) {

        log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));
        return "ok";
    }
}
```

- @ResponseBody : View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력

## 파라미터 필수 여부 - requestParamRequired

### @RequestParam.required

- true 필수
- false 없어도 됨.
- “” 빈문자는 통과

### 주의

- 기본형 int형은 null이 들어갈 수가 없다 Integer로 바꿔줘야함

## 기본 값 적용 - requestParamDefault

```java
@RequestParam(required = false, defaultValue = "-1") int age
```

- 값이 안들어가면 defaultValue의 값이 들어감.
- 빈문자의 경우도 설정한 값이 들어감.

## 파라미터를 Map으로 조회하기 - requestParamMap

- 파라미터의 값이 1개가 확실하다면 Map 을 사용해도 되지만, 그렇지 않다면 MultiValueMap 을 사용하자.

## 영한쌤의 조언

어노테이션을 생략하면서 개발을 하기엔 나중에 코드 읽기가 더 어려워질 수도 있다.

# 8. HTTP 요청 파라미터 - @ModelAttribute

### HelloData.java

```java
package hello.springmvc.basic;

import lombok.Data;

@Data
public class HelloData {
    private String username;
    private int age;
}
```

### RequestParamController.java

```java
@ResponseBody
    @RequestMapping("/model-attribute-v1")
    public String modelAttributeV1(@ModelAttribute HelloData helloData) {
        log.info("helloData={}", helloData);

        return "ok";
    }

    @ResponseBody
    @RequestMapping("/model-attribute-v2")
    public String modelAttributeV2(HelloData helloData) {
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        log.info("helloData={}", helloData);

        return "ok";
    }
```

### @ModelAttribute

- HelloData 객체를 생성한다.
- 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩) 한다.
- 예) 파라미터 이름이 username 이면 setUsername() 메서드를 찾아서 호출하면서 값을 입력한다.

### @ModelAttribute 생략

- @ModelAttribute 는 생략할 수 있다.
- @RequestParam 도 생략할 수 있다.

### 생략 규칙

- String , int , Integer 같은 단순 타입 = @RequestParam
- 나머지 = @ModelAttribute (argument resolver 로 지정해둔 타입 외)

### 주의

파라미터 타입을 안맞게 넣으면 BindException이 발생함.

예) int형 파라미터에 문자열 넣기

# 9. HTTP 요청 메시지 - 단순 텍스트

## HTTP message body에 데이터를 직접 담아서 요청

- HTTP API에서 주로 사용, JSON, XML, TEXT
- 데이터 형식은 주로 JSON 사용
- POST, PUT, PATCH
- `@RequestParam`, `@ModelAttribute`를 사용할 수 없다.

### RequestBodyStringController.java

```java
package hello.springmvc.basic.request;

import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpStatus;
import org.springframework.http.RequestEntity;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.util.StreamUtils;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.InputStream;
import java.io.Writer;
import java.nio.charset.StandardCharsets;

@Slf4j
@Controller
public class RequestBodyStringController {

    @PostMapping("/request-body-string-v1")
    public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ServletInputStream inputStream = request.getInputStream();
        //스트림 바이트 코드라서, UTF-8 지정해줘야함
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody={}", messageBody);

        response.getWriter().write("ok");
    }

    @PostMapping("/request-body-string-v2")
    public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
        //스트림 바이트 코드라서, UTF-8 지정해줘야함
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody={}", messageBody);

        responseWriter.write("ok");
    }

    @PostMapping("/request-body-string-v3")
    public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) throws IOException {

        String messageBody = httpEntity.getBody();

        log.info("messageBody={}", messageBody);

        return new HttpEntity<>("ok");
    }

    @ResponseBody
    @PostMapping("/request-body-string-v4")
    public String requestBodyStringV4(@RequestBody String messageBody) throws IOException {
        log.info("messageBody={}", messageBody);
        return "ok";
    }
}
```

## V1

- postman로 body에 내용 담아서 요청 테스트

결과

```java
2023-02-19 20:43:37.289  INFO 69184 --- [nio-8080-exec-3] h.s.b.r.RequestBodyStringController      : messageBody={"name": "Park", "age": 11}
2023-02-19 20:44:26.407  INFO 69184 --- [nio-8080-exec-4] h.s.b.r.RequestBodyStringController      : messageBody=hello
```

## Input, Output 스트림, Reader - requestBodyString (V2)

- InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
- OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력

### 결과

```java
2023-02-19 20:50:47.906  INFO 70296 --- [nio-8080-exec-2] h.s.b.r.RequestBodyStringController      : messageBody=hello
```

### HttpEntity - requestBodyString (V3)

### 결과

```java
2023-02-19 20:53:22.688  INFO 73192 --- [nio-8080-exec-1] h.s.b.r.RequestBodyStringController      : messageBody=hello
```

### HttpEntity:

- HTTP header, body 정보를 편리하게 조회
- 메시지 바디 정보를 직접 조회
- 요청 파라미터를 조회하는 기능과 관계 없음 @RequestParam X, @ModelAttribute X
- HttpEntity는 응답에도 사용 가능
- 메시지 바디 정보 직접 반환
- 헤더 정보 포함 가능
- view 조회X

### HttpEntity 상속 받은 객체

- RequestEntity : HttpMethod, url 정보가 추가, 요청에서 사용
- ResponseEntity : HTTP 상태 코드 설정 가능, 응답에서 사용
`return new ResponseEntity<String>("Hello World", responseHeaders,
HttpStatus.CREATED)`

```java
@PostMapping("/request-body-string-v3")
    public HttpEntity<String> requestBodyStringV3(RequestEntity<String> httpEntity) throws IOException {

        String messageBody = httpEntity.getBody();

        log.info("messageBody={}", messageBody);

        return new ResponseEntity<String>("ok", HttpStatus.CREATED);
    }
```

## @RequestBody - requestBodyString (V4)

### @RequestBody

- @RequestBody 를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다.
- 헤더 정보가 필요하다면 HttpEntity 를 사용하거나 @RequestHeader 를 사용하면 된다.
- 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 `@RequestParam`, `@ModelAttribute`와는 전혀 관계가 없다.

## 요청 파라미터 vs HTTP 메시지 바디

요청 파라미터를 조회하는 기능: @RequestParam , @ModelAttribute
HTTP 메시지 바디를 직접 조회하는 기능: @RequestBody

## @ResponseBody

@ResponseBody 를 사용하면 응답 결과를 HTTP 메시지 바디에 직접 담아서 전달할 수 있다. 이 경우에도 view를 사용하지 않는다.

## 메시지 컨버터

스프링MVC 내부에서 HTTP 메시지 바디를 읽어서 문자나 객체로 변환해서 전달해주는데, 이때 HTTP 메시지 컨버터( HttpMessageConverter )라는 기능을 사용한다. 

# 10. HTTP 요청 메시지-JSON

```java
package hello.springmvc.basic.request;

import com.fasterxml.jackson.databind.ObjectMapper;
import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpEntity;
import org.springframework.stereotype.Controller;
import org.springframework.util.StreamUtils;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.nio.charset.StandardCharsets;

/**
 * {"username":"hello", "age":20}
 * content-type: application/json
 */
@Slf4j
@Controller
public class RequestBodyJsonController {
    //Json
    private ObjectMapper objectMapper = new ObjectMapper();

    @PostMapping("/request-body-json-v1")
    public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody={}", messageBody);
        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());

        response.getWriter().write("ok");
    }

    @ResponseBody
    @PostMapping("/request-body-json-v2")
    public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {
        log.info("messageBody={}", messageBody);
        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());

        return "ok";
    }

    @ResponseBody
    @PostMapping("/request-body-json-v3")
    public String requestBodyJsonV3(@RequestBody HelloData helloData) throws IOException {
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());

        return "ok";
    }

    @ResponseBody
    @PostMapping("/request-body-json-v4")
    public String requestBodyJsonV4(HttpEntity<HelloData> httpEntity) throws IOException {
        HelloData data = httpEntity.getBody();
        log.info("username={}, age={}", data.getUsername(), data.getAge());
        return "ok";
    }

    @ResponseBody
    @PostMapping("/request-body-json-v5")
    public HelloData requestBodyJsonV5(@RequestBody HelloData data){
        log.info("username={}, age={}", data.getUsername(), data.getAge());
        return data;
    }
}
```

## @RequestBody 객체 파라미터(메시지 컨버터)

- HttpEntity , @RequestBody 를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
- HTTP 메시지 컨버터는 문자 뿐만 아니라 JSON도 객체로 변환해주는데, 우리가 방금 V2에서 했던 작업을 대신 처리해준다
- MappingJackson2HttpMessageConverter가 동작을 함.
`HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);`대체해줌.

## @RequestBody는 생략 불가

생략하면 @ModelAttribute가 붙기 때문엔 안됨.

## V5

응답도 요청한 JSON 형식 그대로 보여줌.

# 11. 응답 - 정적 리소스, 뷰 템플릿

## 서버에서 응답 데이터를 만드는 방법 3가지

### 정적 리소스

- 예) 웹 브라우저에 정적인 HTML, CSS, js를 제공할 때는, 정적 리소스를 사용함.

### 뷰 템플릿 사용

- 예) 웹 브라우저에 동적인 HTML을 제공할 때는 뷰 템플릿을 사용함.

### HTTP 메시지 사용

- HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보냄.

## 경로

### `src/main/resources` :

- 리소스를 보관하는 곳
- 클래스패스의 시작 경로
- 해당 디렉토리에 리소스를 넣어두면 스프링 부트가 정적 리소스로 서비스가 제공함.

### `resources/static` : 정적 페이지

### `resources/templates`: 뷰 템플릿

## 정적 리소스

스프링 부트는 클래스패스의 다음 디렉토리에 있는 정적 리소스를 제공한다.

- `/static`
- `/public`
- `/resources`
- `/META-INF/resources`

### 정적 리소스 경로

1) `src/main/resources/static` 다음 경로에 파일이 들어있으면
2) `src/main/resources/static/basic/hello-form.html`  웹 브라우저에서 다음과 같이 실행하면 된다.
3) [http://localhost:8080/basic/hello-form.html](http://localhost:8080/basic/hello-form.html) 정적 리소스는 해당 파일을 변경 없이 그대로 서비스하는 것이다.

## 뷰 템플릿

`resources/templates`: 뷰 템플릿

### 예제

- 뷰 템플릿

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<!--${data}의 값을 empty에 치환해서 넣어줌-->
<p th:text="${data}">empty</p>
</body>
</html>
```

- ResponseViewController - 뷰 템플릿을 호출하는 컨트롤러

```java
package hello.springmvc.basic.response;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class ResponseViewController {

    @RequestMapping("/response-view-v1")
    public ModelAndView responseViewV1(){
        ModelAndView mav = new ModelAndView("response/hello")
                .addObject("data", "hello");
        return mav;
    }

    @RequestMapping("/response-view-v2")
    public String responseViewV2(Model model){
        model.addAttribute("data", "hello");
        return "response/hello";
    }

    //권장하지 않는 방법.(명시성이 떨어짐)
    //뷰 경로의 이름과 메소드 호출 URL과 같은 방법
    @RequestMapping("/response/hello")
    public void responseViewV3(Model model){
        model.addAttribute("data", "hello");
    }
}
```

- 메소드가 String 타입일 경우, Model이 필요함
- @ResponseBody를 쓸 경우 return 뷰 페이지로 가지 않고, return값을 화면에 딱 보여줌.

### String을 반환하는 경우 - View or HTTP 메시지

- `@ResponseBody` 가 없으면 `response/hello`로 뷰 리졸버가 실행되어서 뷰를 찾고, 렌더링 한다.
- `@ResponseBody`가 있으면 뷰 리졸버를 실행하지 않고, HTTP 메시지 바디에 직접 `response/hello` 라는 문자가 입력된다.

### Void를 반환하는 경우

조건 : `@Controller` 를 사용하고, `HttpServletResponse` , `OutputStream(Writer)` 같은 HTTP 메시지 바디를 처리하는 파라미터가 없으면 요청 URL을 참고해서 논리 뷰 이름으로 사용

### 템플릿 설명

```java
<p th:text="${data}">empty</p>
```

- empty 텍스트 부분이 ${data}로 바뀐다.

## Thymeleaf 스프링 부트 설정

- 프로젝트 만들 때, 타임리프 추가하면 자동등록 되어있음.

### build.gradle

```java
`implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'`
```

스프링 부트가 자동으로 ThymeleafViewResolver 와 필요한 스프링 빈들을 등록한다. 그리고 다음 설정도 사용한다. 이 설정은 기본 값 이기 때문에 변경이 필요할 때만 설정하면 된다.

### application.properties

```java
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

# 12. HTTP 응답 - HTTP API, 메시지 바디에 직접 입력

```java
package hello.springmvc.basic.response;

import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
@Controller
//@ResponseBody
//@RestController
public class ResponseBodyController {

    /**
     *
     * 서블릿을 직접 다룰 때처럼,
     * HttpServletResponse 객체를 통해서 HTTP 메시지 바디에 직접 ok 응답 메시지를 전달한다
     * */
    @GetMapping("/response-body-string-v1")
    public void responseBodyV1(HttpServletResponse response) throws IOException{
        response.getWriter().write("ok");
    }

    /**
     * ResponseEntity 엔티티는 HttpEntity 를 상속 받았는데,
     * HttpEntity는 HTTP 메시지의 헤더, 바디 정보를 가지고 있다.
     * ResponseEntity 는 여기에 더해서 HTTP 응답 코드를 설정할 수 있다.
     * */
    @GetMapping("/response-body-string-v2")
    public ResponseEntity<String> responseBodyV2() throws IOException{
        return new ResponseEntity<>("ok", HttpStatus.OK);
    }

    /**
     * @ResponseBody 를 사용하면 view를 사용하지 않고,
     * HTTP 메시지 컨버터를 통해서 HTTP 메시지를 직접
     * 입력할 수 있다.
     * ResponseEntity 도 동일한 방식으로 동작한다.
     * */
    //@ResponseBody
    @GetMapping("/response-body-string-v3")
    public String responseBodyV3() throws IOException{
        return "ok";
    }

    /**
     * ResponseEntity 를 반환한다.
     * HTTP 메시지 컨버터를 통해서 JSON 형식으로 변환되어서 반환된다.
     * */
    @GetMapping("/response-body-json-v1")
    public ResponseEntity<HelloData> responseBodyJsonV1() {
        HelloData helloData = new HelloData();
        helloData.setUsername("userA");
        helloData.setAge(20);

        return new ResponseEntity<>(helloData, HttpStatus.OK);
    }

    /**
     * ResponseEntity 는 HTTP 응답 코드를 설정할 수 있는데, @ResponseBody 를 사용하면 이런 것을 설정하기 까다롭다.
     * @ResponseStatus(HttpStatus.OK) 애노테이션을 사용하면 응답 코드도 설정할 수 있다.
     * 물론 애노테이션이기 때문에 응답 코드를 동적으로 변경할 수는 없다.
     * 프로그램 조건에 따라서 동적으로 변경하려면 ResponseEntity 를 사용하면 된다
     * */
    @ResponseStatus(HttpStatus.OK)
    //@ResponseBody
    @GetMapping("/response-body-json-v2")
    public HelloData responseBodyJsonV2() {
        HelloData helloData = new HelloData();
        helloData.setUsername("userA");
        helloData.setAge(20);

        return helloData;
    }
}
```

- `@Controller`+ `@ResponseBody`  = `@RestController`
- 클래스 단위에 @ResponseBody를 사용하면 클래스 안에 모든 메소드에 @ResponseBody가 적용된다.

### RestController

@ResponseBody 는 클래스 레벨에 두면 전체 메서드에 적용되는데, @RestController 에노테이션 안에 @ResponseBody 가 적용되어 있다

# 13. HTTP 메시지 컨버터

## `@ResponseBody` 사용시

- HTTP의 BODY에 문자 내용을 직접 반환
- viewResolver 대신에 HttpMessageConverter가 동작
기본 문자 처리 : StringHttpMessageConverter
기본 객체 처리 : MappingJackson2HttpMessageConverter
- byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음.

## HTTP 메시지 컨버터 적용

> HTTP Accept 헤더 : 클라이언트에서 웹서버로 요청시 요청메시지에 담기는 헤더로, 자신에게 이러한 데이터 타입만 허용하겠다는 뜻
출처: [https://dololak.tistory.com/630](https://dololak.tistory.com/630)
> 

### HTTP 요청

- @RequestBody
- HttpEntity(RequestEntity)

### HTTP 응답

- @ResponseBody
- HttpEntity(ResponseEntity)

### HTTP 메시지 컨버터 인터페이스

- HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용된다.
- canRead() , canWrite() : 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크
- read() , write() : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

## 스프링 부트 기본 메시지 컨버터

> 미디어 타입이라는 것은 Content Type이 무슨 타입인지 알려준다. 그 타입의 메시지 컨버터가 선택이 되는 것
> 

### byteArrayHttpMessageConverter :

- byte[] 데이터 처리
- 클래스 타입: byte[]
- 미디어 타입: * / *
- 요청 예) @RequestBody byte[] data
응답 예) @ResponseBody return byte[] 쓰기 미디어타입 application/octet-stream

### StringHttpMessageConverter :

- 문자로 데이터 처리
- 클래스 타입: String , 미디어타입: * / *
- 요청 예) @RequestBody String data
응답 예) @ResponseBody return "ok" 쓰기 미디어타입 text/plain

### MappingJackson2HttpMessageConverter :

- 객체를 JSON으로 바꾸는 것
- JSON요청 메시지온 것을 객체로 바꿔서 사용해주는 것
- 클래스 타입: 객체 또는 HashMap , 미디어타입 application/json 관련
- 요청 예) @RequestBody HelloData data
응답 예) @ResponseBody return helloData 쓰기 미디어타입 application/json 관련

## 예시

### StringHttpMessageConverter

```java
content-type: application/json

@RequestMapping
void hello(@RequestBody String data) {}
```

### MappingJackson2HttpMessageConverter

```java
content-type: application/json
@RequestMapping
void hello(@RequestBody HelloData data) {}
```

# 14. 요청 매핑 핸들러 어댑터 구조

애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할 수 있었는데, 어떻게 사용할 수 있었는지 알아보자

## RequestMappingHandlerAdapter

### RequestMappingHandlerAdapter 동작 방식

1) argumentResolver를 통해서 해당 타입의 객체가 나옴.(파라미터 처리)

2) 핸들러 어댑터는 매개변수 넘겨줄 파라미터가 준비가 되면, 생성된 파라미터를 핸들러(컨트롤러)에게 넘겨줌.

3) ReturnValueHandler가 컨트롤러의 응답 값을 변환하고 처리한다.

### ArgumentResolver

- HandlerMethodArgumentResolver를 줄여서 ArgumentResolver 라고 부른다.
- 컨트롤러가 필요로 하는 다양한 파라미터의 객체를 생성한다.
- 스프링은 30개가 넘는 ArgumentResolver를 기본으로 제공함.
- 소스로 찾아보고 싶을 땐, HandlerMethodArgumentResolver 인터페이스를 참고한다.
- 공식 메뉴얼 : [https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-annarguments](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-annarguments)

### ReturnValueHandler

- HandlerMethodReturnValueHandler를 줄여서 ReturnValueHandler 라 부른다.
- 응답 값을 변환하고 처리함.
- 컨트롤러에서 String으로 뷰 이름을 반환해도 동작하는 이유
- 공식 메뉴얼 : [https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-annreturn-types](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-annreturn-types)

## HTTP 메시지 컨버터

### 요청

- ArgumentResolver들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성함.
- 이때 HTTP 메시지 컨버터의 read를 사용한다.

### 응답

- ReturnValueHandler가 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.
- 이때 HTTP 메시지 컨버터의 write를 사용한다.

## 확장

### 인터페이스로 제공

- HandlerMethodArgumentResolver
- HandlerMethodReturnValueHandler
- HttpMessageConverter

### 확장이 필요하면?

- WebMvcConfigurer를 검색해서 기능을 확장하면 된다.
