# 홈 화면 추가
## HomeController

```java
package hello.hellospring.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/")
    public String home(){
        return "home";
    }
}
```

여기서 “/”로 통해 매핑이 되어, 해당 메소드가 동작한다.

## 정적 페이지? 페이지 우선순위

아무것도 없으면 index.html로 간다고 했는데?

- `localhost:8080/~~~` 쳤을 때, 해당 컨트롤러가 없으면 resources-static에 있는 페이지로 넘어간다.
- 매핑된게 있으면, 컨트롤러 호출되고, 정적 페이지는 무시 되는 것이다.

      
# 회원 웹 기능-등록      
## 등록

MemberController.java

```java
package hello.hellospring.controller;

import hello.hellospring.domain.Member;
import hello.hellospring.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;

@Controller
public class MemberController {

    private MemberService memberService;

    @Autowired
    public MemberController(MemberService memberSerivce){
        this.memberService = memberSerivce;
    }

    @GetMapping("/members/new")
    public String createForm(){
        return "members/createMemberForm";
    }

    @PostMapping("/members/new")
    public String create(MemberForm form){
        Member member = new Member();
        member.setName(form.getName());

        memberService.join(member);

        return "redirect:/";
        //생성하고 홈으로 리턴
    }
}
```

MemberForm.java

```java
package hello.hellospring.controller;

public class MemberForm {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

createMemberForm.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
  <form action="/members/new" method="post">
    <div class="form-group">
      <label for="name">이름</label>
      <input type="text" id="name" name="name" placeholder="이름을 입력하세요">
    </div>
    <button type="submit">등록</button>
  </form>
</div> <!-- /container -->
</body>
</html>
```

html에서 이름(ex 펌킨)을 넣고 등록 버튼을 누르면

form안에 input의 name=”name”이라는 키와 value가 서버로 넘어가게 된다.

## 순서

1) 회원가입 들어가면 /member/new로 매핑 

url를 직접 치면 Get방식으로 들어와서 매핑

2) `return "members/createMemberForm";` 으로 리턴해서 뷰리졸버로 통해 선택이 되어 타임리프엔진이 렌더링한다.

3) form은 값을 입력할 수 있는 것이다.

```html
<form action="/members/new" method="post">
```

등록 버튼을 누르면 form안에 url의 /members/new에 post 방식으로 보내게 된다.

4) post 방식으로 보내기 때문에, post 매핑으로 매핑 시켜준다.

```java
@PostMapping("/members/new")
    public String create(MemberForm form){
        Member member = new Member();
        member.setName(form.getName());

        memberService.join(member);

        return "redirect:/";
        //생성하고 홈으로 리턴
    }
```

url은 같은 메소드가 있지만,  postMapping이기 때문에 create 메소드가 호출이 된다.

매개변수 MemberForm타입(getter setter)에서 name에 값이 들어가는 것이다.

get으로 name으로 꺼내게 된다.

- post 매핑은 데이터를 주로 전달할 때, 쓴다.
- get은 주로 조회할 때 쓴다.      

     
#  회원 웹 기능 - 조회    
MemberController.java

```java
@GetMapping("/members")
    public String list(Model model){
        List<Member> members = memberService.findMembers();
        model.addAttribute("members", members);
        return "members/memberList";
    }
```

memberList.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
    <div>
        <table>
            <thead> <tr>
                <th>#</th>
                <th>이름</th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="member : ${members}">
                <td th:text="${member.id}"></td>
                <td th:text="${member.name}"></td>
            </tr>
            </tbody>
        </table>
    </div>
</div> <!-- /container -->
</body>
</html>
```

`<tr th:each="member : ${members}">` 자바의 foreach와 같이 루프를 돌면서 보여준다.

다만 런을 다시 돌리면, 메모리를 활용하기 때문에 데이터가 다 지워진다.
