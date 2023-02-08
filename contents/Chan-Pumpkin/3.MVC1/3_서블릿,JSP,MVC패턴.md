# 1. 회원 관리 웹 애플리케이션 요구사항

## 요구사항

### 회원 정보

이름: username

나이: age

### 기능 요구사항

회원 저장

회원 목록 조회

## 코드

### Member.java

```java
package hello.servlet.domain.member;

import lombok.Getter;
import lombok.Setter;

@Getter @Setter
public class Member {

    private Long id;
    private String username;
    private int age;

    public Member() {

    }

    public Member(String username, int age) {
        this.username = username;
        this.age = age;
    }
}
```

### MemberRepository.java

```java
package hello.servlet.domain.member;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * 동시성 문제가 고려되어 있지 않아서, 실무에서는 ConcurrentHashMap, AtomicLong 사용고려
 *
 * */
public class MemberRepository {
    //static으로 선언 해서 MemberRepository를 아무리 new로 많이 받아도 하나만 사용이 된다.
    private static Map<Long, Member> store = new HashMap<>();
    private static long sequence = 0L;

    private static final MemberRepository instance = new MemberRepository();

    public static MemberRepository getInstance() {
        return instance;
    }

    //싱글톤 만들 때는 private으로 아무나 생성하지 못하게 막아야 함.
    private MemberRepository() {
    }

    public Member save(Member member){
        member.setId(++sequence);
        store.put(member.getId(), member);
        return member;
    }

    public Member findById(Long id) {
        return store.get(id);
    }

    public List<Member> findAll() {
        return new ArrayList<>(store.values());
    }

    public void clearStore() {
        store.clear();
    }
}
```

### MemberRepositoryTest.java(테스트 코드)

```java
package hello.servlet.domain.member;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;

import java.util.List;

import static org.assertj.core.api.Assertions.*;

//Junit5 부터 public 없어도 됨.
class MemberRepositoryTest {
    //싱글톤이기 때문에, new로 하면 안됨
    MemberRepository memberRepository = MemberRepository.getInstance();

    //테스트 끝나고 clear
    @AfterEach
    void afterEach(){
        memberRepository.clearStore();
    }

    //저장 테스트
    @Test
    void save() {
        //given
        Member member = new Member("park", 10);

        //when
        Member savedMember = memberRepository.save(member);

        //then
        Member findMember = memberRepository.findById(savedMember.getId());
        assertThat(findMember).isEqualTo(savedMember);
    }

    //모든 것을 저장
    @Test
    void findAll(){
        //given
        Member member1 = new Member("member1", 20);
        Member member2 = new Member("member2", 30);

        memberRepository.save(member1);
        memberRepository.save(member2);
        
        //when
        List<Member> result = memberRepository.findAll();
        
        //then
        assertThat(result.size()).isEqualTo(2);
        assertThat(result).contains(member1, member2);
    }
}
```

# 2. 서블릿으로 회원 관리 웹 애플리케이션 만들기

## 소스

### MemberFormServlet.java

```java
package hello.servlet.web.servlet;

import hello.servlet.domain.member.MemberRepository;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet(name = "memberFormServlet", urlPatterns = "/servlet/members/new-form")
public class MemberFormServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        PrintWriter w = response.getWriter();
        w.write("<!DOCTYPE html>\n" +
                "<html>\n" +
                "<head>\n" +
                " <meta charset=\"UTF-8\">\n" +
                " <title>Title</title>\n" +
                "</head>\n" +
                "<body>\n" +
                "<form action=\"/servlet/members/save\" method=\"post\">\n" +
                " username: <input type=\"text\" name=\"username\" />\n" +
                " age: <input type=\"text\" name=\"age\" />\n" +
                " <button type=\"submit\">전송</button>\n" +
                "</form>\n" +
                "</body>\n" +
                "</html>\n");

    }
}
```

### MemberSaveServlet.java

```java
package hello.servlet.web.servlet;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet(name = "memberSaveServlet", urlPatterns = "/servlet/members/save")
public class MemberSaveServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("MemberSaveServlet.service");
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        //저장
        Member member = new Member(username, age);
        memberRepository.save(member);

        //결과는 HTML
        //동적으로 코드가 들어감
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");
        PrintWriter w = response.getWriter();
        w.write("<html>\n" +
                "<head>\n" +
                " <meta charset=\"UTF-8\">\n" +
                "</head>\n" +
                "<body>\n" +
                "성공\n" +
                "<ul>\n" +
                " <li>id="+member.getId()+"</li>\n" +
                " <li>username="+member.getUsername()+"</li>\n" +
                " <li>age="+member.getAge()+"</li>\n" +
                "</ul>\n" +
                "<a href=\"/index.html\">메인</a>\n" +
                "</body>\n" +
                "</html>");
    }
}
```

### MemberListServlet.java

```java
package hello.servlet.web.servlet;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.List;

@WebServlet(name="memberListServlet", urlPatterns = "/servlet/members")
public class MemberListServlet extends HttpServlet {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        List<Member> members = memberRepository.findAll();

        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        PrintWriter w = response.getWriter();
        w.write("<html>");
        w.write("<head>");
        w.write(" <meta charset=\"UTF-8\">");
        w.write(" <title>Title</title>");
        w.write("</head>");
        w.write("<body>");
        w.write("<a href=\"/index.html\">메인</a>");
        w.write("<table>");
        w.write(" <thead>");
        w.write(" <th>id</th>");
        w.write(" <th>username</th>");
        w.write(" <th>age</th>");
        w.write(" </thead>");
        w.write(" <tbody>");
        for (Member member : members) {
            w.write(" <tr>");
            w.write(" <td>" + member.getId() + "</td>");
            w.write(" <td>" + member.getUsername() + "</td>");
            w.write(" <td>" + member.getAge() + "</td>");
            w.write(" </tr>");
        }
        w.write(" </tbody>");
        w.write("</table>");
        w.write("</body>");
        w.write("</html>");
    }
}
```

## 정리

자바 코드로 HTML을 만들어 내는 것보다 차라리 HTML 문서에 동적으로 변경해야 하는 부분만 자바 코드로 넣을 수 있다면 더 편리할 것.

html 템플릿: html에 자바코드를 중간중간에 넣어둔 것, 값을 바꿀 수 있는 기능을 제공하는 것

ex) jsp, 타임리프

# 3. JSP로 회원 관리 웹 애플리케이션 만들기

## 라이브러리 추가

- build.gradle에 추가

```
	//JSP 추가 시작
	implementation 'org.apache.tomcat.embed:tomcat-embed-jasper'
	implementation 'javax.servlet:jstl'
	//JSP 추가 끝
```

## 회원 등록

### members/new-form.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/jsp/members/save.jsp" method="post">
    username: <input type="text" name="username" />
    age: <input type="text" name="age" />
    <button type="submit">전송</button>
</form>
</body>
</html>
```

### members/save.jsp

```html
<%@ page import="hello.servlet.domain.member.MemberRepository" %>
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
  // request, response 사용 가능
  MemberRepository memberRepository = MemberRepository.getInstance();
  System.out.println("save.jsp");
  String username = request.getParameter("username");
  int age = Integer.parseInt(request.getParameter("age"));
  Member member = new Member(username, age);
  System.out.println("member = " + member);
  memberRepository.save(member);%>
<html>
<head>
  <meta charset="UTF-8">
</head>
<body>
성공
<ul>
  <li>id=<%=member.getId()%></li>
  <li>username=<%=member.getUsername()%></li>
  <li>age=<%=member.getAge()%></li>
</ul>
<a href="/index.html">메인</a>
</body>
</html>
```

## 회원 목록

### members.jsp

```html
<%@ page import="java.util.List" %>
<%@ page import="hello.servlet.domain.member.MemberRepository" %>
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    MemberRepository memberRepository = MemberRepository.getInstance();
    List<Member> members = memberRepository.findAll();
%>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<a href="/index.html">메인</a>
<table>
    <thead>
    <th>id</th>
    <th>username</th>
    <th>age</th>
    </thead>
    <tbody>
    <%
        for (Member member : members) {
            out.write(" <tr>");
            out.write(" <td>" + member.getId() + "</td>");
            out.write(" <td>" + member.getUsername() + "</td>");
            out.write(" <td>" + member.getAge() + "</td>");
            out.write(" </tr>");
        }
    %>
    </tbody>
</table>
</body>
</html>
```

## JSP 문법

- `<%@ page contentType="text/html;charset=UTF-8" language="java" %>`

: 첫 줄은 JSP문서라는 뜻이다. JSP 문서는 이렇게 시작해야 한다.

- `<%@ page import="hello.servlet.domain.member.MemberRepository" %>`

: 자바의 import 문과 같다.

- `<% ~~ %>`

: 이 부분에는 자바 코드를 입력할 수 있다.

- `<%= ~~ %>`

: 이 부분에는 자바 코드를 출력할 수 있다

## 서블릿과 JSP의 한계

- JAVA 코드, 데이터 조회하는 리포지터리 등등 다양한 코드가 노출됨
- JSP가 너무 많은 역할을 함.
- JSP 코드가 심각하게 길어질 수도 있음.

# 4. MVC 패턴

## 하나에서 비즈니스로직, 뷰를 처리하면 안좋은 점이 있다.

### 하나에 너무 많은 역할

- 여러 줄의 코드가 있어서 찾기 어려움.
- UI를 변경해야하면 비즈니스 로직까지 손을 대야함.

### 변경의 라이프 사이클

- UI 일부 수정하는 일과 비즈니스 로직을 수정하는 일은 각각 변경의 라이프 사이클이 다르다.
- 변경 주기가 다를 때, 분리 해야함.

### 기능 특화

- JSP 같은 뷰 템플릿은 화면 렌더링 하는데 최정화 되어 있기에 이 부분만 업무만 담당하는 것이 가장 효과적

## Model View Controller

### 컨트롤러

HTTP 요청 받아서 파라미터 검증하고, 비즈니스 로직을 수행해서 데이터를 모델에 담아서 뷰에 전달함.

### 모델

뷰가 필요한 데이터를 모두 모델에 담아서 전달해준다. 덕분에 뷰는 비즈니스 로직이나 데이터 접근을 신경 안쓰고 화면 렌더링에 집중 할 수 있음.

### 뷰

뷰는 모델에 있는 것을 참고해서 그린다.

## 순서

1) 클라이언트 요청

2) 컨트롤러 로직-서비스 호출

3) 서비스-비즈니스 로직-리포지터리 호출

4) 리포지터리-데이터 접근

5) 서비스-비즈니스 로직처리

6) 컨트롤러 데이터 모델이 담기

7) 모델에 담긴 데이터를 받아서 화면 렌더링

8) 응답

## 컨트롤러에 비즈니스 로직이 있나?

- 컨트롤러는 조종하는 역할
- 컨트롤러에 비즈니스 로직까지 있으면 너무 많은 역할이 부여됨
- 비즈니스 로직이 있는 서비스를 호출하는 역할

# 5. 적용

### 등록폼

MvcMemberFormServlet.java

```java
package hello.servlet.web.servletmvc;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        //컨트롤러에서 뷰로 이동할 때 사용
        RequestDispatcher dispathcer = request.getRequestDispatcher(viewPath);
        //호출하면 서블릿에서 JSP를 호출할 수 있음.
        dispathcer.forward(request, response);
    }
}
```

new-form.jsp

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<!-- 상대경로 사용, [현재 URL이 속한 계층 경로 + /save] -->
<!-- action 절대 경로: 'localhost:8080/절대' 경로 이렇게 붙음 -->
<!-- action 상대 경로: 'localhost:8080/~~/~~/aa->localhost:8080/~~/~~/상대' 경로 이렇게 붙음-->
<form action="save" method="post">
    username: <input type="text" name="username" />
    age: <input type="text" name="age" />
    <button type="submit">전송</button>
</form>
</body>
</html>
```

### 저장

MvcMemberSaveServlet.java

```java
package hello.servlet.web.servletmvc;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name="MvcMemberSaveServlet", urlPatterns = "/servlet-mvc/members/save")
public class MvcMemberSaveServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
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

save-result.jsp

```java
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <meta charset="UTF-8">
</head>
<body>
성공
<ul>
    <%-- 1번 방식
    <li>id=<%=((Member)request.getAttribute("member")).getId()%></li>
    <li>username=<%=((Member)request.getAttribute("member")).getUsername()%></li>
    <li>age=<%=((Member)request.getAttribute("member")).getAge()%></li>
    --%>
    <%-- 2번 방식 : 편하다, 임포트도 필요 없다. --%>
        <li>id=${member.id}</li>
        <li>username=${member.username}</li>
        <li>age=${member.age}</li>
</ul>
<a href="/index.html">메인</a>
</body>
</html>
```

### 목록

MvcMemberListServlet.java

```java
package hello.servlet.web.servletmvc;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

@WebServlet(name = "MvcMemberListServlet", urlPatterns = "/servlet-mvc/members")
public class MvcMemberListServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        List<Member> members = memberRepository.findAll();

        request.setAttribute("members", members);
        String viewPath = "/WEB-INF/views/members.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

member.jsp

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<a href="/index.html">메인</a>
<table>
    <thead>
    <th>id</th>
    <th>username</th>
    <th>age</th>
    </thead>
    <tbody>
    <c:forEach var="item" items="${members}">
        <tr>
            <td>${item.id}</td>
            <td>${item.username}</td>
            <td>${item.age}</td>
        </tr>
    </c:forEach>
    </tbody>
</table>
</body>
</html>
```

### 절대 경로 vs 상대경로

```
<form action="save" method="post">
```

action 절대 경로: 

- 'localhost:8080/절대' 경로 이렇게 붙음
action 상대 경로:
- 'localhost:8080/~~/~~/aa → localhost:8080/~~/~~/상대' 경로 이렇게 붙음

### dispatcher.forward()

다른 서블릿이나 JSP로 이동할 수 있는 기능, 서버 내부에서 다시 호출이 발생함.(redirect X)

### WEB-INF

이 경로안에 JSP가 있으면 외부에서 직접 JSP를 호출할 수 없다. 우리가 기대하는 것은 항상 컨트롤러를 통해서 JSP를 호출하는 것이다.

### redirect vs forward

리다이렉트는 실제 클라이언트(웹 브라우저)에 응답이 나갔다가, 클라이언트가 redirect 경로로 다시 요청한다. 따라서 클라이언트가 인지할 수 있고, URL 경로도 실제로 변경된다. 웹브라우저에서 서버로 호출이 2번 일어나는 것

포워드는 서버 내부에서 일어나는 호출이기 때문에 클라이언트가 전혀 인지하지 못한다.

# 6. MVC 패턴 한계

## MVC 컨트롤러 단점

### 포워드 중복

- View로 이동하는 코드가 항상 중복 호출되어야 함.
- 만약 jsp가 아닌 thymeleaf 같은 다른 뷰로 변경한다면 전체 코드를 다 변경해야함.

### 사용하지 않는 코드

HttpServletRequest, HttpServletResponse는 사용되지 않을 수도 있고, 테스트 케이스를 작성하기도 어렵다.

### 공통 처리가 어렵다.

공통 기능을 메서드로 뽑으면 되지만, 해당 메서드를 항상 호출해야한다.

### 정리

- 컨트롤러 호출 전에 먼저 공통 기능을 처리해야함
- 프론트 컨트롤러 패턴을 도입하면 이런 문제를 깔끔하게 해결할 수 있음.
- 입구를 하나로!
