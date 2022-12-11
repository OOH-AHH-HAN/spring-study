### MVC 템플릿 엔진

서버에서 프로그래밍 해서 html 동적으로 해서 전달하는 것

### MVC

Model, View, Controller

Controller: 비즈니스 로직과 관련이 있거나, 내부적인 것들을 처리하는데 집중해야함

View : 화면을 그리는데 모든 역량을 집중해야함.

Model: 필요한 것들을 담아서 화면에 전달함.

### thymeleaf의 장점

html 파일를 서버를 거치지 않고 열 수 있다.

controller requestParam에서 required default가 true이기 때문에 값을 안넣어도 됨.
![Untitled 9](https://user-images.githubusercontent.com/62877858/206881364-47ca744c-906d-4a55-aa18-7d7ea3cad59d.png)
![Untitled 10](https://user-images.githubusercontent.com/62877858/206881366-74b30d05-2393-41cb-9237-2c7dd510772f.png)

requestParam 변수 name을 추가했기 때문에 URL에 name의 값을 써줘야 함.

name의 값을 넣어서 웹브라우저에서 입력하면

controller name 변수에 값이 담기고 모델에 담아서 html에 리턴해준다.

웹브라우저 localhost:8080/hello-mvc

내장 톰캣서버 거침

스프링 매핑 되어있는 거 찾음

return: hello-template 모델에 키와 값 담아서 뷰 리졸버에 넘김

viewresolver 타임리프 템플릿 엔진 처리해서 웹브라우저에 반환함
