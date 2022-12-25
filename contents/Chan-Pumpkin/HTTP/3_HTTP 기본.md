# 1. 모든 것이 HTTP

## HTTP(HyperText Transfer Protocol)

문서간의 전송하는 프로토콜

## HTTP 메시지에 모든 것을 전송

- HTTP 메시지에 모든 것을 전송
- HTML, TEXT, 이미지, 음성, 영상, 파일 , JSON, XML

## HTTP 역사

HTTP/1.1 

- 1997년에 나옴
- 가장 많이 사용
- 우리에게 가장 중요한 버전

HTTP/2

- 2015년 성능 개선

HTTP/3

- 진행중
- TCP 대신에 UDP 사용

## 기반 프로토콜

### TCP : HTTP/1.1, HTTP/2

### UDP :  HTTP/3

속도가 너무 느려서 UDP 프로토콜 위에 애플리케이션 성능 최적화해서 나온 것

### TIP

인터넷 창에서 F12 눌러서 네트워크 탭에서 프로토콜 쪽에 보면 HTTP 몇을 사용하고 있는지 확인할 수 있음

## HTTP 특징

- 클라이언트 서버 구조
- 무상태 프로토콜, 비연결성
- HTTP 메시지
- 단순함, 확장 가능

# 2. 클라이언트 서버 구조

클라이언트와 서버 구조로 되어있음(Request Response 구조)

클라이언트는 서버에 요청 보내고 응답을 대기

서버가 요청에 대한 결과를 만들어서 응답

비즈니스 로직은 서버에 다 넣음

클라이언트는 UI에 집중

클라이언트와 서버가 각각 독립적으로 진화할 수 있다는 장점이 있음.

# 3. Stateful, Stateless

## Stateless - 무상태 프로토콜

- 서버가 클라이언트의 상태를 보존하지 않음
- 장점은 서버 확장성 높음
- 단점은 클라이언트가 추가 데이터 전송
- 무상태일 때는 무상태라서 데이터를 하나하나 자세히 넘긴다.

## Stateful - 상태 유지

- 서버가 중간에 바뀌면 시스템 장애가 발생함

## 무상태, 상태 차이

### 상태유지

- 중간에 다른 서버로 바뀌면 안된다.
- 중간에 다른 서버로 바뀔 때, 상태 정보를 미리 알려줘야함. 무상태처럼 데이터를 자세히 받아야하기 때문에
- 만약 서버 A가 문제가 생겨서 꺼지게 되면 장애가 발생함.

### 무상태

- 중간에 다른 서버로 바뀌어도 된다.
- 서버 A가 문제가 생겨서 꺼지게 되어도 서버 B로 사용하는데 지장이 없음. 어차피 무상태니까!

## 무상태의 한계

- 상태 유지 해야하는 것

예) 로그인

- 데이터를 너무 많이 보냄
- 그래서 상태 유지는 최소한으로만 사용해야함.

# 4. 비연결성

## 연결을 유지하는 모델

서버 연결 유지하는 자원을 많이 사용한다.

## 연결을 유지하지 않는 모델

서버 연결 유지하는 자원을 최소한으로 사용한다.

## 비연결성

- HTTP는 기본적으로 연결을 유지하지 않는 모델
- 빠른 속도로 응답
- 서버 자원을 매우 효율적으로 사용할 수 있다.

## 비연결성 단점

- TCP/IP 연결을 새로 맺어야 한다.

즉 3 way handshake 시간이 추가 된다.

- 수 많은 자원이 함께 다운로드가 됨.
- 연결하고 → 수많은 자원 다운 받고 → 끊고
- 지금은 HTTP 지속 연결(Persistent Connection)으로 문제 해결
- HTTP/2, HTTP/3에서는 보완이 되었음.

## HTTP 초기

- 연결하고 → 요청 → 응답 → 종료 여러번 반복

## HTTP 지속 연결(Persistent Connections)

연결 → 요청/응답 (연결 유지) → 요청/응답(연결 유지) → 요청/응답 → 종료

## 동시간 대용량 트래픽

Stateless 방식으로 설계하면 자원을 최소한으로 쓸 수 있기에 무조건 Stateless 방식으로 설계 해야한다.

# 5. HTTP메시지

## HTTP 메시지에 모든 것을 전송

- HTTP 메시지에 모든 것을 전송
- HTML, TEXT, 이미지, 음성, 영상, 파일 , JSON, XML

## HTTP 메시지 구조

1) 시작라인

2) 헤더

3) 공백 라인

4) 메시지 바디

## HTTP 요청 메시지

1) 시작라인 : GET /search?q=hello&hl=ko HTTP/1.1

2) 헤더 : Host: [www.google.com](http://www.google.com/)

3) 공백라인

TIP!) 요청 메시지도 body 본문을 가질 수 있음

## HTTP 응답 메시지

1) 시작라인 : HTTP/1.1 200 OK

2) 헤더 : 

Content-Type: text/html;charset=UTF-8
Content-Length: 3423

3) 공백라인

4) 메시지 바디 :

<html>
<body>...</body>
</html>

## 공식 스펙
![Untitled 3](https://user-images.githubusercontent.com/62877858/209468533-b27292f4-1a73-4ff4-8330-31e5896db58a.png)
## 시작 라인

### 요청 메시지

- start-line = request-line / status-line
- request-line = method SP(공백) request-target SP HTTP-version CRLF(엔터)

### 요청 메시지 - HTTP 메서드

- 종류 : GET, POST, PUT, DELETE
- 서버가 수행해야 할 동작 지정

GET : 리소스 조회

POST :  요청 내역 처리

### 요청 메시지 - 요청 대상

- 절대 경로 = “/”로 시작하는 경로
- 절대 경로로 시작함.

### 응답 메시지

- start-line = request-line / status-line
- status-line = HTTP-version SP status-code SP reason-phrase CRLF
- HTTP 버전
- HTTP 상태 코드: 요청 성공, 실패를 나타냄
• 200: 성공
• 400: 클라이언트 요청 오류
• 500: 서버 내부 오류
• 이유 문구: 사람이 이해할 수 있는 짧은 상태 코드 설명 글

## HTTP 헤더

header-field = field-name “:” OWS field-value OWS (OWS: 띄워쓰기 허용)

- field-name은 대소문자 구분 없음
- : 전에 띄우는 건 안됨 : 후에 띄우는 건 됨

ex) Host: www.google.com

## HTTP 헤더 용도

- HTTP 전송에 필요한 모든 부가 정보가 들어가 있음.
- 메시지 바디의 내용, 바디의 크기, 압축, 인증, 요청 클라이언트 정보 등등
- 표준 헤더가 너무 많음
- 필요시 임의의 헤더 추가 기능 다만 임의의 헤더를 추가하면 약속한 클라이언트와 서버만 알 수 있음.

## HTTP 메시지 바디

- 실제 전송할 데이터
- byte로 표현할 수 있는 모든 데이터 전송 가능
- HTML, 이미지, 영상 JSON 등등

## 단순함

- 단순해서 확장 가능
- HTTP 메시지도 매우 단순함.
