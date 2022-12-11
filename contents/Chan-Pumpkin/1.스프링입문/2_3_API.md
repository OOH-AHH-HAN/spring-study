html로 내리는지, API 방식으로 내리는지 두가지 방법

## @ResponseBody

http에서 헤더부와 바디부가 있는데

바디부의 이 데이터를 직접 넣어주겠다는 뜻임.

![Untitled 11](https://user-images.githubusercontent.com/62877858/206881402-ffc8099f-e6df-461a-8029-c36e7f814a43.png)

![Untitled 12](https://user-images.githubusercontent.com/62877858/206881408-1655d65a-940c-4216-96e6-1cc1fe87f22f.png)


페이지 소스보기

![Untitled 13](https://user-images.githubusercontent.com/62877858/206881410-94bd733a-9bfd-4a5d-8542-132ce68f594e.png)

html 태그 같은 거 없이 문자가 있는 것을 확인할 수 있다.

json API 방식은 그대로 데이터를 내려준다.

## Json 방식
![2_3_4](https://user-images.githubusercontent.com/62877858/206881415-9e278555-ba72-43b7-aa1c-471a32e89f2e.png)

![Untitled 14](https://user-images.githubusercontent.com/62877858/206881425-b617dc29-fa25-4e7a-9ba8-75fd130e5e67.png)


json이라는 방식이다.

json 키 밸류로 이루어진 구조

xml 방식은 열고 닫고 태그를 두번 써야하고 무겁다

json은 심플해서 json 방식으로 주로 사용한다.

스프링도 기본으로 json으로 반환하는게 default로 되어있다.

(2020년 기준)

## get set

```
//private이라 못써서 아래 get,set 메소드로 접근한다.
//java bean 표준 방식, 프로퍼티 방식
```

## ResponseBody 사용원리

1)웹브라우저에서 [localhost:8080/hello-api](http://localhost:8080/hello-api) 침

2)내장 톰캣서버 스프링으로 보냄

3)Controller 매핑 된 거 발견

4)responseBody 어노테이션 발견 

http에 응답에 넣은게 되겠군

문자면? 바로 http에 넣어서 바로 줬음

객체를 줌..? 스프링 입장에서는 기본 default가 json방식으로 데이터를 만들어서 http에 반환하겠다가 기본 정책

5)HttpMessageConverter 동작 (responseBody 어노테이션이 있기때문에 동작)

단순 문자면 StringConverter 동작

객체면 JsonConverter 동작

객체를 Json 형태로 바꾼다.

서버-웹 브라우저에 보내준다.

- responseBody 사용하면 viewResolver 대신에 HttpMessageConverter가 동작
- Json을 객체로 바꿔주는 여러 라이브러리가 있음(Jackson라이브러리, Gson라이브러리 스프링에서는 Jackson을 기본적으로 사용함.)
- XML라이브러리 꽂아놓고 실행하면 XML로 반환이 된다.
- 참고: 클라이언트의 HTTP Accept 해더와 서버의 컨트롤러 반환 타입 정보 둘을 조합해서
HttpMessageConverter 가 선택된다.
