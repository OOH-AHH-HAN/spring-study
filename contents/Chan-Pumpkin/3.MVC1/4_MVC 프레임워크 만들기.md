# 1. 프론트 컨트롤러 패턴

## 프론트 컨트롤러 패턴 특징

- 프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받음 프론트 컨트롤러 앞에 서블릿이 하나 있다고 생각하면 됨.
- 프론터 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출
- 공통 처리 가능
- 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아됨 프론트 컨트롤러가 직접 호출해주기 때문에

## 스프링 MVC의 핵심

### FrontController : MVC 핵심

- DispatcherServlet이 FrontController 패턴으로 구현 되어있음

# 2. V1-프론트 컨트롤러 도입

## V1 구조 순서

1) 클라이언트 HTTP 요청

2) Front Controller라는 서블릿이 요청을 받음

3) URL 매핑 정보에서 컨트롤러 조회

4) 컨트롤러 호출

5) 컨트롤러에서 JSP forward 호출

6) JSP에서 HTML 응답

## V1 구조 패키지

web

→ frontcontroller

→ v1 : 

- controllerV1(인터페이스)
- FrontControllerServletV1

→ controller : 

- MemberFormControllerV1
- MemberListControllerV1
- MemberSaveControllerV1

## 구현

### controllerV1(인터페이스)

```java
package hello.servlet.web.frontcontroller.v1;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

//구현을 여러개 한다 요청이 오고, 매핑정보에서 찾아서 호출할 때,
//일관성 있게 프론트 컨트롤러는 인터페이스에 의존하면서 편리하게 컨트롤러에 호출할 수 있다.
public interface ControllerV1 {

   void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;

}
```

### MemberFormControllerV1

```java
package hello.servlet.web.frontcontroller.v1.controller;

import hello.servlet.web.frontcontroller.v1.ControllerV1;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class MemberFormControllerV1 implements ControllerV1 {
    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispathcer = request.getRequestDispatcher(viewPath);
        dispathcer.forward(request, response);
    }
}
```

### MemberSaveControllerV1

```java
package hello.servlet.web.frontcontroller.v1.controller;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import hello.servlet.web.frontcontroller.v1.ControllerV1;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class MemberSaveControllerV1 implements ControllerV1 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        request.setAttribute("member", member);

        //뷰로 던지기
        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

### MemberListControllerV1

```java
package hello.servlet.web.frontcontroller.v1.controller;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import hello.servlet.web.frontcontroller.v1.ControllerV1;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

public class MemberListControllerV1 implements ControllerV1 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        List<Member> members = memberRepository.findAll();

        request.setAttribute("members", members);

        String viewPath = "/WEB-INF/views/members.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);

    }
}
```

### FrontControllerServletV1

```java
package hello.servlet.web.frontcontroller.v1;

import hello.servlet.web.frontcontroller.v1.controller.MemberFormControllerV1;
import hello.servlet.web.frontcontroller.v1.controller.MemberListControllerV1;
import hello.servlet.web.frontcontroller.v1.controller.MemberSaveControllerV1;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

// 'front-controller/v1/*' : v1 하위에 어떤 URL이 들어와도 이 서블릿이 무조건 호출이 된다.
@WebServlet(name = "FrontControllerServletV1", urlPatterns = "/front-controller/v1/*")
public class FrontControllerServletV1 extends HttpServlet {
    //매핑정보
    private Map<String, ControllerV1> controllerMap = new HashMap<>();

    //서블릿 생성을 할 때, 값을 controllerMap 넣어두게 된다.
    //매핑정보 넣기
    public FrontControllerServletV1() {
        controllerMap.put("/front-controller/v1/members/new-form", new MemberFormControllerV1());
        controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV1());
        controllerMap.put("/front-controller/v1/members", new MemberListControllerV1());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("FrontControllerServletV1.service");
        //URI 받음
        String requestURI = request.getRequestURI();
        //요청받은 URI 매핑정보 찾는다.
        ControllerV1 controller = controllerMap.get(requestURI);
        //매핑정보가 없으면
        if(controller == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }
        controller.process(request, response);
    }
}
```

### URLPatterns

urlPatterns = "/front-controller/v1/*" : /front-controller/v1 를 포함한 하위 모든 요청은 이 서블릿에서 받아들인다.

### controllerMap

- key: 매핑 URL
- value: 호출될 컨트롤러

### service()

- requestURI를 조회해서 실제 호출할 컨트롤러 controllerMap에서 찾음
- controllerMap에서 없다면 404 상태 코드를 반환함
- 있다면 controller.process(request, response)를 호출해서 해당 컨트롤러를 실행함.

# 3. V2-view 분리

## 구조 순서

1) 클라이언트 HTTP 요청

2) Front Controller라는 서블릿이 요청을 받음

3) URL 매핑 정보에서 컨트롤러 조회

4) 컨트롤러 호출

5) MyView(객체) 반환

6) render()호출

7) MyView에서 JSP forward 호출

8) JSP HTML 응답

## 소스 순서

- 화면을 렌더링하기 위한 MyView(객체)를 활용함.

1) `FrontControllerServletV2` - 서블릿 URL 호출 받음

2) 매핑 컨트롤러 찾아와서 process 호출

3) 매핑 컨트롤러 이동

4) MyView(”경로”) 반환

5) MyView(”경로”).render 호출

6) `MyView`-viewPath 가지고 forward

## 구현

### MyView

```java
package hello.servlet.web.frontcontroller;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

// 각 컨트롤러에서 했던 로직을 MyView에 넣어서 하는 것
// 뷰 렌더링
public class MyView {
    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request,response); 
    }
}
```

### ControllerV2(인터페이스)

```java
package hello.servlet.web.frontcontroller.v2;

import hello.servlet.web.frontcontroller.MyView;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public interface ControllerV2 {
    //MyView를 만들어서 넘기는 식
    MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```

### MemberFormControllerV2

```java
package hello.servlet.web.frontcontroller.v2.controller;

import hello.servlet.web.frontcontroller.MyView;
import hello.servlet.web.frontcontroller.v2.ControllerV2;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class MemberFormControllerV2 implements ControllerV2 {

    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        return new MyView("/WEB-INF/views/new-form.jsp");
    }
}
```

### MemberSaveControllerV2

```java
package hello.servlet.web.frontcontroller.v2.controller;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import hello.servlet.web.frontcontroller.MyView;
import hello.servlet.web.frontcontroller.v2.ControllerV2;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class MemberSaveControllerV2 implements ControllerV2 {
    private MemberRepository memberRepository = MemberRepository.getInstance();
    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        request.setAttribute("member", member);

        //뷰로 던지기
        return new MyView("/WEB-INF/views/save-result.jsp");
    }
}
```

### MemberListControllerV2

```java
package hello.servlet.web.frontcontroller.v2.controller;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import hello.servlet.web.frontcontroller.MyView;
import hello.servlet.web.frontcontroller.v2.ControllerV2;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

public class MemberListControllerV2 implements ControllerV2 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        List<Member> members = memberRepository.findAll();

        request.setAttribute("members", members);

        return new MyView("/WEB-INF/views/members.jsp");
    }
}
```

### FrontControllerServletV2

```java
package hello.servlet.web.frontcontroller.v2;

import hello.servlet.web.frontcontroller.MyView;
import hello.servlet.web.frontcontroller.v2.controller.MemberFormControllerV2;
import hello.servlet.web.frontcontroller.v2.controller.MemberListControllerV2;
import hello.servlet.web.frontcontroller.v2.controller.MemberSaveControllerV2;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

// 'front-controller/v2/*' : v2 하위에 어떤 URL이 들어와도 이 서블릿이 무조건 호출이 된다.
@WebServlet(name = "FrontControllerServletV2", urlPatterns = "/front-controller/v2/*")
public class FrontControllerServletV2 extends HttpServlet {
    //매핑정보
    private Map<String, ControllerV2> controllerMap = new HashMap<>();

    //서블릿 생성을 할 때, 값을 controllerMap 넣어두게 된다.
    //매핑정보 넣기
    public FrontControllerServletV2() {
        controllerMap.put("/front-controller/v2/members/new-form", new MemberFormControllerV2());
        controllerMap.put("/front-controller/v2/members/save", new MemberSaveControllerV2());
        controllerMap.put("/front-controller/v2/members", new MemberListControllerV2());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //URI 받음
        String requestURI = request.getRequestURI();
        //요청받은 URI 매핑정보 찾는다.
        ControllerV2 controller = controllerMap.get(requestURI);
        //매핑정보가 없으면
        if(controller == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }
        MyView view = controller.process(request, response);
        view.render(request, response);
    }
}
```

# 4. V3-Model 추가

## 서블릿 종속성 제거

- 현재 컨트롤러 입장에서 request, response 정보가 당장 필요해 보이지 않음.
- 요청 파라미터 정보는 자바의 Map으로 대신 넘기도록 하면 지금 구조에서 컨트롤러가 서블릿 기술을 몰라도 동작할 수 있음.

## 뷰 이름 중복 제거

- 컨트롤러에서 지정하는 뷰 이름에 중복이 있는 것을 확인할 수 있음.
- 컨트롤러는 뷰의 논리 이름을 반환하고, 실제 물리 위치의 이름은 프론트 컨트롤러에서 처리하도록 단순화 해보자 향후 뷰의 폴더 위치가 함께 이동해도 프론트 컨트롤러만 고치면 된다.

## 구조 순서

1) HTTP 요청

2) 프론트 컨트롤러 요청 받음

3) 프론트 컨트롤러→매핑 정보 조회→반환

4) 프론트 컨트롤러→컨트롤러 호출

5) 컨트롤러에서 ModelView반환→프론트 컨트롤러

6) 프론트 컨트롤러→viewResolver 호출→MyView 반환

7) 프론트 컨트롤러→render(model) 호출→MyView

8) MyView→HTML 응답

## ModelView

서블릿의 종속성을 제거하기 위해 Model을 직접 만들 것

## 소스

### ModelView

```java
package hello.servlet.web.frontcontroller;

import java.util.HashMap;
import java.util.Map;

public class ModelView {
    private String viewName;
    private Map<String, Object> model = new HashMap<>();

    public ModelView(String viewName) {
        this.viewName = viewName;
    }

    public String getViewName() {
        return viewName;
    }

    public void setViewName(String viewName) {
        this.viewName = viewName;
    }

    public Map<String, Object> getModel() {
        return model;
    }

    public void setModel(Map<String, Object> model) {
        this.model = model;
    }
}
```

### ControllerV3

```java
package hello.servlet.web.frontcontroller.v3;

import hello.servlet.web.frontcontroller.ModelView;

import java.util.Map;

public interface ControllerV3 {
    ModelView process(Map<String, String> paramMap);
}
```

### MemberFormControllerV3

```java
package hello.servlet.web.frontcontroller.v3.controller;

import hello.servlet.web.frontcontroller.ModelView;
import hello.servlet.web.frontcontroller.v3.ControllerV3;

import java.util.Map;

public class MemberFormControllerV3 implements ControllerV3 {

    @Override
    public ModelView process(Map<String, String> paramMap) {
        //뷰의 전체 패스가 아닌, 논리적인 이름만
        return new ModelView("new-form");
    }
}
```

### MemberSaveControllerV3

```java
package hello.servlet.web.frontcontroller.v3.controller;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import hello.servlet.web.frontcontroller.ModelView;
import hello.servlet.web.frontcontroller.MyView;
import hello.servlet.web.frontcontroller.v3.ControllerV3;

import java.util.Map;

public class MemberSaveControllerV3 implements ControllerV3 {
    private MemberRepository memberRepository = MemberRepository.getInstance();
    @Override
    public ModelView process(Map<String, String> paramMap) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        ModelView mv = new ModelView("save-result");
        mv.getModel().put("member", member);

        //뷰로 던지기
        return mv;
    }
}
```

### MemberListControllerV3

```java
package hello.servlet.web.frontcontroller.v3.controller;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import hello.servlet.web.frontcontroller.ModelView;
import hello.servlet.web.frontcontroller.MyView;
import hello.servlet.web.frontcontroller.v3.ControllerV3;

import java.util.List;
import java.util.Map;

public class MemberListControllerV3 implements ControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public ModelView process(Map<String, String> paramMap) {
        List<Member> members = memberRepository.findAll();
        ModelView mv = new ModelView("members");
        mv.getModel().put("members", members);

        return mv;
    }
}
```

### FrontControllerServletV3

```java
package hello.servlet.web.frontcontroller.v3;

import hello.servlet.web.frontcontroller.ModelView;
import hello.servlet.web.frontcontroller.MyView;
import hello.servlet.web.frontcontroller.v3.controller.MemberFormControllerV3;
import hello.servlet.web.frontcontroller.v3.controller.MemberListControllerV3;
import hello.servlet.web.frontcontroller.v3.controller.MemberSaveControllerV3;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

// 'front-controller/v3/*' : v3 하위에 어떤 URL이 들어와도 이 서블릿이 무조건 호출이 된다.
@WebServlet(name = "FrontControllerServletV3", urlPatterns = "/front-controller/v3/*")
public class FrontControllerServletV3 extends HttpServlet {
    //매핑정보
    private Map<String, ControllerV3> controllerMap = new HashMap<>();

    //서블릿 생성을 할 때, 값을 controllerMap 넣어두게 된다.
    //매핑정보 넣기
    public FrontControllerServletV3() {
        controllerMap.put("/front-controller/v3/members/new-form", new MemberFormControllerV3());
        controllerMap.put("/front-controller/v3/members/save", new MemberSaveControllerV3());
        controllerMap.put("/front-controller/v3/members", new MemberListControllerV3());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //URI 받음
        String requestURI = request.getRequestURI();
        //요청받은 URI 매핑정보 찾는다.
        ControllerV3 controller = controllerMap.get(requestURI);
        //매핑정보가 없으면
        if(controller == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }
        //paramMap
        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        // 물리 이름을 논리 이름으로 받아야 함.
        String viewName = mv.getViewName();//논리 이름 new-form
        MyView view = viewResolver(viewName);

        view.render(mv.getModel(),request, response);
    }
    //뷰를 해결해준다
    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```

### MyView

```java
package hello.servlet.web.frontcontroller;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Map;

// 각 컨트롤러에서 했던 로직을 MyView에 넣어서 하는 것
// 뷰 렌더링
public class MyView {
    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request,response); 
    }

    public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        modelToRequestAttribute(model, request);
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request,response);
    }

    //model를 requestAttribute로 바꾼다.
    private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
        //model에 있는 데이터를 다 꺼내야함.
        model.forEach((key, value) -> request.setAttribute(key,value));
    }
}
```

### 순서

1) urlPatterns으로 FrontControllerServletV3 호출

2) createParamMap 으로 파라미터 다 뽑음

3) 논리 이름 반환

4) 논리 이름 가지고, viewResolver 호출

5) Myview 반환

6) render(model) 호출

7) model에 담긴 데이터를 다 꺼내서 request.setAttribute에 다 넣는다.

- JSP는 HttpRequest에 넣어야 함.

### 뷰 리졸버

컨트롤러가 반환한 논리 뷰 이름을 실제 물리 뷰 경로로 변경한다. 그리고 실제 물리 경로가 있는 MyView객체를 반환한다.

논리 뷰 이름: members

물리 뷰 이름: /WEB-INF/views/members.jsp

### view.render(mv.getModel(), request, response)

- 뷰 객체를 통해서 HTML 화면을 렌더링 함.
- JSP는 request.getAttribute()로 데이터를 조회하기 때문에, 모델의 데이터를 꺼내서 request.setAttribute로 담아둔다.
- JSP로 포워드 해서 JSP를 렌더링 함.

# 5. V4-단순하고 실용적인 컨트롤러

## 구조

- 기본적인 구조는 V3와 같지만, ViewName만 반환하는 것만 다름.
- ControllerV4가 model을 만들어서 model를 파라미터로 넘긴다.
- V4는 인터페이스에 ModelView가 없음. model 객체는 파라미터로 전달되기 때문에 그냥 사용하면 됨. 결과로 뷰의 이름만 반환해주면 된다.

### ControllerV4

```java
package hello.servlet.web.frontcontroller.v4;

import java.util.Map;

public interface ControllerV4 {
    /**
     *
     * @param paramMap
     * @param model
     * @return viewName
     */
    String process(Map<String, String> paramMap, Map<String, Object> model);
}
```

### MemberFormControllerV4

```java
package hello.servlet.web.frontcontroller.v4.controller;

import hello.servlet.web.frontcontroller.v4.ControllerV4;

import java.util.Map;

public class MemberFormControllerV4 implements ControllerV4 {

    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        //모델은 생성할 필요가 없음 FrontController에서 만들어서 넘겨줄 것이다
        return "new-form";
    }
}
```

### MemberSaveControllerV4

```java
package hello.servlet.web.frontcontroller.v4.controller;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import hello.servlet.web.frontcontroller.ModelView;
import hello.servlet.web.frontcontroller.v3.ControllerV3;
import hello.servlet.web.frontcontroller.v4.ControllerV4;

import java.util.Map;

public class MemberSaveControllerV4 implements ControllerV4 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        model.put("member", member);
        return "save-result";
    }
}
```

### MemberListControllerV4

```java
package hello.servlet.web.frontcontroller.v4.controller;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import hello.servlet.web.frontcontroller.ModelView;
import hello.servlet.web.frontcontroller.v3.ControllerV3;
import hello.servlet.web.frontcontroller.v4.ControllerV4;

import java.util.List;
import java.util.Map;

public class MemberListControllerV4 implements ControllerV4 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        List<Member> members = memberRepository.findAll();

        model.put("members", members);
        return "members";
    }
}
```

### FrontControllerServletV4

```java
package hello.servlet.web.frontcontroller.v4;

import hello.servlet.web.frontcontroller.ModelView;
import hello.servlet.web.frontcontroller.MyView;
import hello.servlet.web.frontcontroller.v3.ControllerV3;
import hello.servlet.web.frontcontroller.v4.controller.MemberFormControllerV4;
import hello.servlet.web.frontcontroller.v4.controller.MemberListControllerV4;
import hello.servlet.web.frontcontroller.v4.controller.MemberSaveControllerV4;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

// 'front-controller/v4/*' : v4 하위에 어떤 URL이 들어와도 이 서블릿이 무조건 호출이 된다.
@WebServlet(name = "FrontControllerServletV4", urlPatterns = "/front-controller/v4/*")
public class FrontControllerServletV4 extends HttpServlet {
    //매핑정보
    private Map<String, ControllerV4> controllerMap = new HashMap<>();

    //서블릿 생성을 할 때, 값을 controllerMap 넣어두게 된다.
    //매핑정보 넣기
    public FrontControllerServletV4() {
        controllerMap.put("/front-controller/v4/members/new-form", new MemberFormControllerV4());
        controllerMap.put("/front-controller/v4/members/save", new MemberSaveControllerV4());
        controllerMap.put("/front-controller/v4/members", new MemberListControllerV4());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //URI 받음
        String requestURI = request.getRequestURI();
        //요청받은 URI 매핑정보 찾는다.
        ControllerV4 controller = controllerMap.get(requestURI);
        //매핑정보가 없으면
        if(controller == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }
        //paramMap
        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>();

        String viewName = controller.process(paramMap, model);

        MyView view = viewResolver(viewName);
        view.render(model,request, response);
    }
    //뷰를 해결해준다
    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```

### 모델 객체 전달

`Map<String, Object> model = new HashMap<>();`

- 모델 객체를 프론트 컨트롤러에서 생성해서 넘겨준다.
- 컨트롤러에서 모델 객체에 값을 담으면 여기에 그대로 담겨있게 된다.

### 뷰의 논리 이름을 직접 반환

```java
String viewName = controller.process(paramMap, model);
MyView view = viewResolver(viewName);
```

컨트롤러가 직접 뷰의 논리 이름을 반환하므로 이 값을 사용해서 실제 물리 뷰를 찾을 수 있음.

### 요약

- 프레임워크나 공통 기능이 수고로워야 사용하는 개발자가 편리해진다.

# 6. V5-유연한 컨트롤러1

## 만약 하고싶은 방식이 v3방식과 v4방식으로 개발하는 방법으로 나뉜다면?

- ControllerV3와 ControllerV4는 호환이 불가능한 완전히 다른 인터페이스
- 어댑터 패턴을 사용하면 된다.
- 어댑터 패턴을 사용하면 다양한 방식의 컨트롤러를 처리할 수 있다.

## V5 구조

핸들러라는 개념은 컨트롤러라고 생각하면 된다.

1) HTTP 요청→Front Controller

2) 핸들러 매핑 정보에서 핸들러 조회한다.

3) 핸들러 어댑터 목록에서 핸들러를 처리할 수 있는 핸들러 어댑터 조회한다.

- 매핑 정보를 가져왔을 때, 해당 매핑 정보를  처리할 수 있는 어댑터 조회

4) Front Controller→[handle(handle)]→ 핸들러 어댑터

- 어댑터를 통해서 핸들러를 호출해야함. 그 중간 과정.

5) 핸들러 어댑터→[handler 호출]→핸들러(컨트롤러)

6) 핸들러 어댑터→[ModelView 반환]→Front Controller

7) Front Controller→[viewResolver 호출]→viewResolver

8) viewResolver→[MyView 반환]→Front Controller

9) Front Controller→[render(model) 호출]→MyView

10) MyView→HTML 응답

## 핸들러

### 핸들러 어댑터

중간에 어댑터 역할을 수행하는 것으로, 다양한 종류의 컨트롤러를 호출할 수 있도록 도와준다.

### 핸들러

컨트롤러의 이름을 더 넓은 범위인 핸들러로 변경함 이제 해당 어댑터만 있으면 다 처리할 수 있기 때문이다.

## 구현

### MyHandlerAdapter.java

```java
package hello.servlet.web.frontcontroller.v5;

import hello.servlet.web.frontcontroller.ModelView;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public interface MyHandlerAdapter {

    boolean supports(Object handler);

    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}
```

### ControllerV3HandlerAdapter.java

```java
package hello.servlet.web.frontcontroller.v5.adapter;

import hello.servlet.web.frontcontroller.ModelView;
import hello.servlet.web.frontcontroller.v3.ControllerV3;
import hello.servlet.web.frontcontroller.v5.MyHandlerAdapter;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class ControllerV3HandlerAdapter implements MyHandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV3);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        ControllerV3 controller = (ControllerV3) handler;

        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        return mv;
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```

### FrontControllerServletV5.java

```java
package hello.servlet.web.frontcontroller.v5;

import hello.servlet.web.frontcontroller.ModelView;
import hello.servlet.web.frontcontroller.MyView;
import hello.servlet.web.frontcontroller.v3.ControllerV3;
import hello.servlet.web.frontcontroller.v3.controller.MemberFormControllerV3;
import hello.servlet.web.frontcontroller.v3.controller.MemberListControllerV3;
import hello.servlet.web.frontcontroller.v3.controller.MemberSaveControllerV3;
import hello.servlet.web.frontcontroller.v5.adapter.ControllerV3HandlerAdapter;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@WebServlet(name = "frontControllerServletV5", urlPatterns = "/front-controller/v5/*")
public class FrontControllerServletV5 extends HttpServlet {

    //기존 : 컨트롤러 타입에 해당 컨트롤러가 들어갔음.
    //변경 : Object를 넣어서 어떤 컨트롤러가 들어갈 수 있음.
    private final Map<String, Object> handlerMappingMap = new HashMap<>();
    private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();

    public FrontControllerServletV5() {
				//핸들러 매핑 초기화
        initHandlerMappingMap();
        //어댑터 매핑 초기화
        initHandlerAdapters();
    }

    private void initHandlerMappingMap() {
        //매핑 정보를 넣는다.
        handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());
    }

    private void initHandlerAdapters() {
        handlerAdapters.add(new ControllerV3HandlerAdapter());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //요청 정보를 가지고 핸들러를 찾아온다
        Object handler = getHandler(request);
        //핸들러가 없으면
        if(handler == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }
        //핸들러 어댑터 찾기
        MyHandlerAdapter adapter = getHandlerAdapter(handler);
        //핸들러 호출
       ModelView mv = adapter.handle(request, response, handler);
        //뷰네임 얻어서 뷰리졸버 호출
        String viewName = mv.getViewName();//논리 이름 new-form
        MyView view = viewResolver(viewName);
        //뷰 렌더 호출
        view.render(mv.getModel(),request, response);
    }

    private MyHandlerAdapter getHandlerAdapter(Object handler) {
        for (MyHandlerAdapter adapter : handlerAdapters) {
            if (adapter.supports(handler)) {
                return adapter;
            }
        }
        throw new IllegalArgumentException("핸들러 어댑터를 찾을 수 없습니다. 핸들러="+handler);
    }

    private Object getHandler(HttpServletRequest request) {
        //URI 받음
        String requestURI = request.getRequestURI();
        //요청받은 URI를 가지고, 핸들러를 찾는다.
        return handlerMappingMap.get(requestURI);
    }

    //뷰를 해결해준다
    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}
```

# 7. V5-2번째 시간

ContorllerV4 기능도 추가

## 소스

### FrontControllerServletV5.java

```java
package hello.servlet.web.frontcontroller.v5;

import hello.servlet.web.frontcontroller.ModelView;
import hello.servlet.web.frontcontroller.MyView;
import hello.servlet.web.frontcontroller.v3.ControllerV3;
import hello.servlet.web.frontcontroller.v3.controller.MemberFormControllerV3;
import hello.servlet.web.frontcontroller.v3.controller.MemberListControllerV3;
import hello.servlet.web.frontcontroller.v3.controller.MemberSaveControllerV3;
import hello.servlet.web.frontcontroller.v4.controller.MemberFormControllerV4;
import hello.servlet.web.frontcontroller.v4.controller.MemberListControllerV4;
import hello.servlet.web.frontcontroller.v4.controller.MemberSaveControllerV4;
import hello.servlet.web.frontcontroller.v5.adapter.ControllerV3HandlerAdapter;
import hello.servlet.web.frontcontroller.v5.adapter.ControllerV4HandlerAdapter;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@WebServlet(name = "frontControllerServletV5", urlPatterns = "/front-controller/v5/*")
public class FrontControllerServletV5 extends HttpServlet {

    //기존 : 컨트롤러 타입에 해당 컨트롤러가 들어갔음.
    //변경 : Object를 넣어서 어떤 컨트롤러가 들어갈 수 있음.
    private final Map<String, Object> handlerMappingMap = new HashMap<>();
    private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();

    public FrontControllerServletV5() {
        //핸들러 매핑 초기화
        initHandlerMappingMap();
        //어댑터 매핑 초기화
        initHandlerAdapters();
    }

    private void initHandlerMappingMap() {
        //매핑 정보를 넣는다.
        handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());

        //V4 추가
        handlerMappingMap.put("/front-controller/v5/v4/members/new-form", new MemberFormControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members/save", new MemberSaveControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members", new MemberListControllerV4());
    }

    private void initHandlerAdapters() {
        handlerAdapters.add(new ControllerV3HandlerAdapter());
        handlerAdapters.add(new ControllerV4HandlerAdapter());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //요청 정보를 가지고 핸들러를 찾아온다
        Object handler = getHandler(request);
        //핸들러가 없으면
        if(handler == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }
        //핸들러 어댑터 찾기
        MyHandlerAdapter adapter = getHandlerAdapter(handler);
        //핸들러 호출
        ModelView mv = adapter.handle(request, response, handler);
        //뷰네임 얻어서 뷰리졸버 호출
        String viewName = mv.getViewName();//논리 이름 new-form
        MyView view = viewResolver(viewName);
        //뷰 렌더 호출
        view.render(mv.getModel(),request, response);
    }

    private MyHandlerAdapter getHandlerAdapter(Object handler) {
        for (MyHandlerAdapter adapter : handlerAdapters) {
            if (adapter.supports(handler)) {
                return adapter;
            }
        }
        throw new IllegalArgumentException("핸들러 어댑터를 찾을 수 없습니다. 핸들러="+handler);
    }

    private Object getHandler(HttpServletRequest request) {
        //URI 받음
        String requestURI = request.getRequestURI();
        //요청받은 URI를 가지고, 핸들러를 찾는다.
        return handlerMappingMap.get(requestURI);
    }

    //뷰를 해결해준다
    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}
```

### ControllerV4HandlerAdapter.java

```java
package hello.servlet.web.frontcontroller.v5.adapter;

import hello.servlet.web.frontcontroller.ModelView;
import hello.servlet.web.frontcontroller.v4.ControllerV4;
import hello.servlet.web.frontcontroller.v5.MyHandlerAdapter;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class ControllerV4HandlerAdapter implements MyHandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV4);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        ControllerV4 controller = (ControllerV4) handler;

        Map<String, String> paramMap = createParamMap(request);
        HashMap<String, Object> model = new HashMap<>();

        String viewName = controller.process(paramMap, model);

        //어댑터 역할
        //어댑터는 ModelView로 만들어서 형식을 맞추어 반환함.
        ModelView mv = new ModelView(viewName);
        mv.setModel(model);

        return mv;
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```
