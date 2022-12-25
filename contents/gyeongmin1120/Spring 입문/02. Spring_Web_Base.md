# 스프링 웹 개발 종류(기초)


## 정적 컨텐츠

- 파일을 그대로 웹 브라우저에 나타내줌
- Spring 프로젝트에서 `resources/static/hello-static.html` 생성 
- 동작 순서 
	1. 웹 브라우저에 `localhost:8080/hello-static.html` 입력
	2. 내장된 톰캣 서버가 `hello-static`과 관련된 **Controller**를 찾음(Controller	   가 우선순위를 가짐)
	3. 관련 **Controller**가 없다면 `resources:static/hello-static.html`을 찾음
	4. 찾은 html을 웹 브라우저에 나타내줌


## MVC와 템플릿 엔진
- MVC : Model(데이터를 담는 객체), View(화면), Controller(비즈니스 로직)
- MVC 패턴은 화면과 비즈니스 로직을 구분했다는 것에 의의
- Contoller는 비즈니스 로직에, View는 화면을 그리는데 모든 역량을 집중해야 함
- 동작 순서 
	1. 웹 브라우저에 `localhost:8080/hello-mvc` 입력
	2. 내장된 톰캣 서버가 **Controller**에서 `hello-mvc`를 찾음
	3. **Controller**가 return하는 `model` 및 `hello-template`(문자)을 	   **viewResolver**에게 전달
	4. **viewResolver**가 `hello-template`를 .html 형식으로 변환
	5. **Thymeleaf 템플릿 엔진**이 `html`로 변환 후 웹 브라우저에 나타내줌(정적 컨텐츠일	   때는 변환하지 않음)

### Source

- Controller : `PracticeController`
	```
	@Controller
	public class PracticeController {
		@GetMapping("hello-mvc")
    	public String helloMvc(@RequestParam("name") String name, Model model){
        		//RequestParam은 클라이언트로부터 받아와야 하는 변수를 나타냄
			//(required = false)을 추가하면 값이 없더라도 에러가 발생하지 않음
			model.addAttribute("name", name);
       		return "hello-template";
    	}
	}
	```
- View : `resources/templates/hello-template.html`

	```
	<!--타임리프 명시-->
	<html xmlns:th="http://www.thymeleaf.org">
		<body>
			<!--th:text 부분이 뒤에 hello! empty 부분을 치환-->
			<p th:text="'hello ' + ${name}">hello! empty</p>
		</body>
	</html>
	```

	+ 타임리프 장점 : 서버없이 파일 그대로 열어도 결과 볼 수 있음


## API

- `@ResponseBody` 사용 시 뷰 리졸버(viewResolver) 사용 안함
	+ 템플릿 엔진과의 차이 : view를 내려주지 않고 바로 데이터를 내려줌
- http body부에 리턴 데이터를 직접 넣어주겠다는 의미
- 동작 순서
	1. 웹 브라우저에 `localhost:8080/hello-api` 입력
	2. 내장된 톰캣 서버가 **Controller**에서 `hello-api`를 찾음
	3. **Controller**에 `@ResponseBody`가 붙어있다면 **HttpMessageConverter**		에게 데이터를 반환(Spring에서 기본적으로 setting)
	4. 반환하는 데이터가 문자라면 **StringConverter**가 처리하고 객체라면 		MappingJackson2HttpMessageConverter**가 처리한다.
		- 객체가 반환되면 기	본적으로 `json`형식으로 만들어서 http 응답에 반환하는게 기본 정책
		- Jackson이란? 
			+ 객체를 json형식으로 변환해주는 라이브러리
			+ Gson도 많이 쓰임
			+ Jackson이 spring 기본

### Source

- Controller : `PracticeController`

```
 @GetMapping("hello-string")
    @ResponseBody
    public String helloString(@RequestParam("name") String name){
        return "hello " + name; //"hello spring"
    }


    @GetMapping("hello-api")
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name){
        Hello hello = new Hello();
        hello.setName(name);
        return hello;
    }

    //class 안에 class
    static class Hello{
        private String name;
	
	//java bean 표준 property 접근 방식
	//private로 변수 선언 후 직접 접근하지 못하니 getter, setter로 접근
        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
```
