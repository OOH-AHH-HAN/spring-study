# 1. HTTP API

## 요구사항

회원 정보 관리 API를 만들어라

- 회원 목록 조회
- 회원 조회
- 회원 등록
- 회원 수정
- 회원 삭제

## API URL 설계

### URL

- 회원 목록 조회 /read-member-list
- 회원 조회 /read-member-by-id
- 회원 등록 /create-member
- 회원 수정 /update-member
- 회원 삭제 /delete-member

이렇게 해서 개발 한다?

이것은 좋은 URI 설계인가?

가장 중요한 것은 리소스 식별이다.

## API URI 고민

### 리소스의 의미는 무엇일까

회원이라는 개념 자체가 바로 리소스

회원을 조회해라가 아닌 회원 자체가 리소스

### 리소스를 어떻게 식별하는게 좋을까

회원 등록하고 수정하고 조회하는 것을 모두 배제

회원이라는 리소스만 식별하면 된다. → 회원 리소스를 URI에 매핑

## API URL 설계

### URL

- 회원 목록 조회 /members/
- 회원 조회 /members/{id}
- 회원 등록 /members/{id}
- 회원 수정 /members/{id}
- 회원 삭제 /members/{id}

조회, 등록, 수정, 삭제 어떻게 구분하지..?

## 리소스와 행위를 분리해야함.

URI는 리소스만 식별

리소스와 해당 리소스 대상으로 하는 행위를 분리해야함.

리소소는 명사, 행위는 동사

행위는 어떻게 구분?

HTTP 메서드를 활용할 것이다.

# 2. HTTP 메서드 - GET, POST

클라이언트가 서버에 행동을 요청함.

## 메서드 종류

GET : 리소스 조회

POST :  요청 데이터 처리, 주로 등록에 사용

PUT : 리소스를 대체, 해당 리소스가 없으면 생성

PATCH : 리소스 부분 변경

DELETE : 리소스 삭제

HEAD: GET이랑 동일하지만, 메시지 부분을 제외하고 상태 줄과 헤더만 반환

OPTIONS: 대상 리소스에 대한 통신 가능 옵션을 주로 설명

CONNECT: 대상 자원으로 식별되는 서버에 대한 터널을 설정

TRACE: 대상 리소스에 대한 경로를 따라 메시지 루프백 테스트 수행

최근에는 리소스 대신 Representation으로 변경 됨

## GET

- 리소스 조회
- GET / ~~~~~~~ 이렇게 요청 보냄
- 서버에 전달하고 싶은 데이터는 쿼리 파라미터를 통해서 전달
- 메시지 바디를 사용해서 데이터를 전달할 수 있지만, 지원하지 않는 곳이 많아서 권장하지 않음

## GET 메시지 전달

1) 클라이언트가 GET 메시지 보냄

2) 서버에서 GET 메시지 받음

3) 서버에서 응답 메시지 보냄

## POST

- 요청 데이터 처리
- 메시지 바디를 통해 서버로 요청 데이터 전달
- 요청 데이터를 주면서 이것을 처리해달라는 요청
- 서버는 요청 데이터를 받아서 처리 함.
- 메시지 바디를 통해 들어온 데이터를 처리하는 모든 기능 처리함
- 주로 등록에 사용함.

## POST 메시지 전달

1) 미리 POST의 대한 행동을 정함.

2) 클라이언트가 POST 메시지 보냄.

3) 서버에서 POST 메시지 받음

4) 서버에서 미리 정한 POST 행동을 함.

5) 서버가 클라이언트에 응답함. (등록된 자원의 데이터도 보냄)

## POST 요청 데이터를 어떻게 처리하라는 뜻?

스펙 : POST 메서드는 대상 리소스가 리소스의 고유 한 의미 체계에 따라 요청에 포함 된 표헌을 처리하도록 요청함.

- post의 의미가 많음

정리 : 이 리소스 URI에 POST 요청이 오면 요청 데이터를 어떻게 처리할지 리소스마다 따로 정해야 함.

## POST 정리

1. 새 리소스 생성(등록)
2. 요청 데이터 처리
- 값 변경을 넘어 프로세스의 상태가 변경되는 경우
- POST의 결과로 새로운 리소스가 생성되지 않을 수도 있음
- 리소스만으로 안될 때도 있음. 아래와 같이 동사로 정해주는 경우도 있음. ex) POST /member/{id}/start-create
1. 다른 메서드로 처리하기 애매한 경우
- 조회 데이터를 넘겨야 하는데, GET 메시지로 바디 부분을 처리하는 것을 지원하지 않는 서버들이 많음 이럴 때, POST
- 애매하면 POST

# 3. PUT,PATCH,DELETE

## PUT

- 리소스가 있으면 대체, 없으면 생성

폴더에 파일을 넣었을 때, 없으면 새로 생기지만, 있으면 덮어씌우는 것과 같은 원리

- 클라이언트가 리소스를 식별

클라이언트가 리소스 위치를 알고 URI 지정(POST와 차이점)

## PUT 리소스가 있음

1) 클라이언트가 PUT 보냄

2) 서버에서 PUT 받음

3) 리소스가 있으면 대체

## 리소스가 없음

1) 클라이언트가 PUT 보냄

2) 서버에서 PUT 받음

3) 리소스가 없으므로 생성

## PUT 리소스를 완전히 대체함

대체 했을 때, 기존 리소스를 완전히 덮어버림 내용까지도

## PATCH

- 리소스 부분 변경
- PATCH로 보내게 되면, 부분 변경하게 됨(해당 내용만 변경됨)

ex) name : 20 → name만 변경 됨.

## DELETE

- 리소스 제거

# 4. HTTP 메서드의 속성

- 안전(Safe Methods)
- 멱등(Idempotent Methods)
- 캐시가능(Cacheable Methods)
![Untitled 4](https://user-images.githubusercontent.com/62877858/209468580-30e3d453-62a7-4922-8605-f15263b21f39.png)

## 안전

- 호출해도 리소스가 변경되지 않음.
- GET, HEAD는 안전하다
- 해당 리소스가 변하는지, 안변하는지 이것만 고려함. 로그 고려 안함.

## 멱등

- 몇 번 호출해도 결과가 똑같다.
- GET : 똑같은 리소스를 조회할 뿐 변하지 않음.
- PUT : 똑같은 파일을 업로드를 했다고 하면 덮어버리기 때문에 여러번 해도 최종 결과는 같다.
- DELETE : 같은 요청을 여러번 해도 삭제된 결과는 똑같다.
- POST : 두 번 호출하면 같은 결제가 중복해서 발생할 수도 있다.

ex) 두번 결제 했으면 중복 결제로 문제 발생

### 활용으로 자동 복구 메커니즘으로 사용할 수 있음

똑같은 요청을 해도 결과는 똑같으니까

ex) delete를 했는데 응답을 못 받았다? 또 시도 가능 어차피 결과는 같으니까

### 재요청 중간에 다른 곳에서 리소스 변경해버리면?

멱등은 외부 요인으로 중간에 리소스가 변경되는 것까지는 고려하지 않음.

ex) GET→PUT→GET

## 캐시 가능

캐시 가능: 웹 브라우저 내에 이미지를 저장하고 있을 수 있는가?

응답 결과 리소스를 캐시해서 사용해도 되는가?

- GET, HEAD, POST, PATCH 캐시가능
- POST, PATCH는 본문 내용까지 캐시 키로 고려해야 하는데, 구현이 쉽지 않음.
