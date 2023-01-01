# 1. 클라이언트에서 서버로 데이터 전송

## 클라이언트에서 서버로 데이터 전송 데이터 전달 방식

### 1) 쿼리 파라미터를 통한 데이터 전송

- GET에서 많이 사용함.
- 정렬 필터(검색어)에서 사용 많이 함.

### 2) 메시지 바디를 통한 데이터 전송

- POST,PUT,PATCH
- 회원 가입, 상품 주문, 리소스 등록, 리소스 변경

## 데이터 전송 4가지 상황

### 1) 정적 데이터 조회

- 이미지를 클라이언트에게 내려주는 것으로 URL 경로에 넣어서 보내면, 서버에서 이미지 리소스를 만들어서 클라이언트에게 내려줌.
- 추가적인 데이터가 필요가 없음.
- 쿼리파라미터 없이 조회가 가능함.

### 2) 동적 데이터 조회

- 검색어 or 추가 조건 전달해야하는데, 그때 쿼리 파라미터 사용함.

`?/search?q=hello&hl=ko`

- 서버에서 받게 되면, 쿼리 파라미터 기반으로 뒷단에서 결과를 찾아서 결과를 동적으로 생성해서 보내줌
- 검색을 하거나 게시판 목록에서 정렬 필터할 때, 사용함.
- 조회조건을 줄여주는 필터, 조회 결과를 정렬하는 정렬 조건에 주로 사용
- 조회는 GET을 사용하고, GET은 쿼리 파라미터 사용해서 데이터 전달
- HTTP 스펙상 GET도 메시지 바디를 사용해서 쓸 수 있으나, 지원하지 않은 곳이 많기에 권장하지 않음.

### 3) HTML Form을 통한 데이터 전송

1) POST

```html
<form action="/create" method="post">
	<input type="text" name="name"/>
	<input type="text" name="job"/>
	<button type="submit">가입</button>
</form>
```

HTML Form 태그안에 있는 input에 값을 쓰고 가입 버튼을 누르면, 서버에 아래와 같은 요청 메시지가 전달이 된다. (여기서 input name값이 키와 같은 역할을 하기 때문에 중요함.) 

```
POST /create HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded
name=kim&job=cook
```

`name=kim&job=cook` 쿼리 파라미터와 유사한 형식

post라서 메시지 바디안에 쿼리파라미터를 넣음.

2) GET

```html
<form action="/create" method="get">
	<input type="text" name="name"/>
	<input type="text" name="job"/>
	<button type="submit">가입</button>
</form>
```

```
POST /create?name=kim&job=cook HTTP/1.1
Host: localhost:8080
```

- GET은 메시지 바디를 쓰지 않음.
- url안에 쿼리 파라미터를 넣어서 사용함.
- 주로 정렬필터(검색어) 용도로 사용함.
- 리소스가 생성 혹은 변경이 하는 것은 GET 메소드를 사용하면 안됨.
- 조회에서만 사용해야함.

3) multipart/form-data

```html
<form action="/create" method="multipart/form-data">
	<input type="text" name="username"/>
	<input type="text" name="age"/>
	<input type="file" name="file1"/>
	<button type="submit">가입</button>
</form>
```

- 웹 브라우저가 생성한 요청 HTTP 메시지

Content-Type: multipart/form-data;

enctype을 multipart/form-data 한다면 자동으로 username, age, file1의 값과 이미지에 대한 바이트 정보를 경계선으로 구분하여 보낸다.

여러 개의 컨텐츠의 타입들을 데이터로 보낼 수 있는 것이 multipart

multipart/form-data는 바이너리 데이터를 전송할 때 사용함.

### HTML Form submit시 POST전송

- Content-Type: application/x-www-form-urlencoded 사용
- form의 내용을 메시지 바디를 통해서 전송(key=value, 쿼리 파라미터 형식)
- 전송 데이터를 url encoding 처리
- HTML Form은 GET 전송으로 가능하지만, 저장할 때 사용하면 안되고, 쿼리 파라미터 형식으로 보낸다.
- 멀티파트의 Content-Type: multipart/form-data;
- 바이너리 데이터 전송하거나 다른 종류의 여러 파일관 폼의 내용 함께 전송할 때 사용
- HTML Form 전송은 GET, POST만 지원

### 4) HTML API를 통한 데이터 전송

```
POST /create HTTP/1.1
Content-Type: application/json
{
 "name": "kim",
 "job": "cook"
}
```

- 클라이언트에서 서버로 데이터를 바로 전송해야할 때 사용
- 서버 to 서버 백엔드 시스템 통신할 때 많이 사용함.
- 자바스크립트를 통한 통신에 사용(AJAX)
- React, VueJs 등이 웹 클라이언트와 API 통신을 사용함.
- HTTP  API 데이터 전송을 할 때, POST, PUT, PATCH를 사용하여 메시지 바디를 통해 데이터 전송
- GET으로 조회 쿼리 파라미터로 데이터를 전달한다.
- 예전에는 Content-Type: application/xml을 많이 쓰였지만, 지금은 Content-Type: application/json을 주로 사용하고 표준임.

# 2. HTTP API 설계 예시

URI 리소스를 식별해야하지 다른 것을 식별하면 안된다. 리소스를 가지고 행위하는 것에 행위는 HTTP 메소드로 하면 된다.

### 목록

회원 목록 /members -> GET
데이터를 내려줌.

다만 내려줘야하는 데이터가 100만개다 그럼 검색할 수 있는 검색어가 들어가야한다. 쿼리 파라미터를 이용해서 

이렇게 설계 한다.

### 등록

회원 등록 /members -> POST

/members 컬렉션에 회원을 관리하는 uri에 회원을 넣어주면 추가로 등록 되도록 설계

### 조회

회원 조회 /members/{id} -> GET
/members 하위에 아이디를 넣어주면 단건 조회 가능하도록 설계

### 삭제

회원 삭제 /members/{id} -> DELETE

/members 하위에 아이디를 넣어주면 삭제 되도록 설계

### 수정

회원 수정 /members/{id} -> PATCH, PUT, POST

조금 고민을 해봐야함

put은 기존 리소스를 지우고 덮는다.

patch는 기존 데이터 부분 수정

완전히 덮어버려도 되면 put이지만, 그럴 상황은 거의 없다.

클라이언트에서 회원 데이터를 전부 보내야 덮을 수 있는 것이다.

그래서 patch를 사용한다.

게시글을 수정한다. 그러면 put을 수정할 수 있다.

왜냐면 게시글의 정보를 다 불러와서 전체를 다시 수정하기 때문에

## 신규 자원 등록 특징 POST 기반 등록

### post로 리소스 등록

1) 클라이언트 - 신규 데이터 등록 요청

POST/members

2) 서버

/members/{신규리소스 식별자 생성}

ex) /members/200

신규 리소스를 서버가 만들어준다.

3) 응답 데이터

Location: /members/200

- post로 데이터를 등록할 때, 서버에서 리소스 URI를 만들어준다.

### 특징

- 클라이언트는 등록될 리소스의 URI를 모른다.
- 서버가 새로 등록된 리소스 URI를 생성해준다.
- 서버가 리소스의 URI를 생성하고 관리하는 리소스 디렉토리를 컬렉션(Collection)이라 한다.
ex) /members

## 파일 관리 시스템 PUT 기반 등록

### 파일 목록

/files -> GET

### 파일 단건 조회

/files/{filename} -> GET

### 파일 등록

/files/{filename} -> PUT

파일을 등록할 때, put을 사용함.

기존 파일이 있으면 지우고, 다시 올려야하기 때문에

### 파일 삭제

/files/{filename} -> DELETE

### 파일 대량 등록

/files -> POST

### PUT - 신규 자원 등록 특징

클라이언트가 리소스 URI를 알고 있어야 한다.

파일을 등록할 때, 파일 네임을 넣어준다. 해당 URI를 클라이언트가 알고 있는 것이다.

EX) URI: /files/ball.jpg

post는

/members를 넘기면 서버가 리소스 URI를 생성한다. 

EX) /members/{리소스 생성}

put은 클라이언트가 URI를 알고 등록을 해야함.

클라이언트가 등록될 리소스의 URI를 직접 관리한다.

스토어

- 클라이언트가 관리하는 리소스 저장소
- 클라이언트가 리소스의 URI를 알고 관리
- 여기서 스토어는 `/files`

대부분은 post 기반 등록을 사용함. 

파일 업로드 서버를 다룰 때, put을 사용함.

## HTML FORM 사용

- GET, POST만 지원
- AJAX 같은 기술을 사용해서 해결 가능
- GET, POST만 지원하므로 제약이 있음.

### 회원 목록

/members -> GET

### 회원 등록 폼

/members/new -> GET

### 회원 등록

/members/new, /members -> POST

두가지 방법

1) /members/new로 두가지 url을 맞추는 스타일일 수도 있음

2) 폼: /members/new 등록:/members

밸리데이션에 걸려서 다시 회원등록 폼으로 보내야하는 경우가 있을 수가 있어서

영한님은 1번 스타일을 선호 하심.

### 회원 조회

/members/{id} -> GET

회원 상세 HTML로 이동할 때

### 회원 수정 폼

/members/{id}/edit -> GET

폼 자체를 보는 것은 변경이 일어나지 않기 때문에, GET

### 회원 수정

/members/{id}/edit, /members/{id} -> POST

폼 데이터가 서버로 전송이 되어야 하는데, 등록가 비슷함.

/members/{id}/edit,

/members/{id} 

폼의 URI과 맞추는 것을 선호하심.

### 회원 삭제

/members/{id}/delete -> POST

HTML FORM에서 DELETE 메소드를 사용할 수 없다.

어쩔 수 없이

/delete 이렇게 컨트롤 URI를 써야한다.

URI 내에 delete를 넣어줘서 표현해줘야 함.

### 컨트롤 URI

- 이런 제약을 해결하기 위해 동사로 된 리소스 경로 사용
- POST의 /new, /edit, /delete가 컨트롤 URI
- HTTP 메서드로 해결하기 애매한 경우 사용(HTTP API 포함)
- 사용할 수 있는 메서드가 GET, POST만 지원하기 때문에, 한정적이라서 컨트롤 URI를 사용함.
- 최대한 리소스라는 개념을 가지고 설계를 하고 컨트롤 URI를 대체제로 사용하는 것.
- 실무에서는 HTTP 메서드만으로 해결하기 어려워서 컨트롤 URI를 사용하는 경우가 많음.
- 동사를 사용함.

## 참고하면 좋은 URI 설계 개념

문서

- 단일 개념(파일 하나, 객체 인스턴스, 데이터베이스 row)
- EX) /members/100, /files/star.jpg

컬렉션

- 서버가 관리하는 리소스 디렉터리
- 서버가 리소스의 URI를 생성하고 관리
- EX)/members/{리소스}

스토어

- 클라어인트가 관리하는 자원 저장소
- 클라이언트가 리소스의 URI를 알고 관리
- EX) /files

컨트롤러(controller), 컨트롤 URI

- 문서, 컬렉션, 스토어로 해결하기 어려운 추가 프로세스 실행
- 동사를 직접 사용
- EX) /members/{id}/delete

최대한 문서와 컬렉션을 활용하고, 안되면 컨트롤 URI를 활용한다.
