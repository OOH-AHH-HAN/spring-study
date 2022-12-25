# URI(Uniform Resource Identifier)

리소스를 식별하는 방법?

## URI

URI는 로케이터(locator), 이름(name) 또는 둘 다 추가로 분류할 수 있다.

URI는 가장 큰 개념

자원이 어디있는지 식별하는 방법

URL : 리소스의 위치

URN : 리소스의 이름 (리소스의 이름을 대면 위치를 찾아갈 수 있다.)
![Untitled 3](https://user-images.githubusercontent.com/62877858/209468480-31c467a5-c10d-4f8c-b7f4-705558d83071.png)


문제는 이름을 부여하면 찾을 수가 없음 리소스가 매핑이 되어 있어야하는데 사실상 어렵다.

- Uniform: 리소스 식별하는 통일된 방식
- Resource: 자원, URI로 식별할 수 있는 모든 것(제한 없음)
- Identifier: 다른 항목과 구분하는데 필요한 정보
- URL: Uniform Resource Locator : 리소스가 있는 위치 지정
- URN: Uniform Resource Name : 리소스에 이름 부여

위치는 변할 수 있지만, 이름은 변하지 않는다.

URN 이름만으로 실제 리소스를 찾을 수 있는 방법이 보편화 되지 않음.

### URL 전체 문법

[https://www.google.com/search?q=hello&hl=ko](https://www.google.com/search?q=hello&hl=ko)

## URL 전체 문법

scheme://[userinfo@]host[:port][/path][?query][#fragment]
[https://www.google.com:443/search?q=hello&hl=ko](https://www.google.com/search?q=hello&hl=ko)

- 프로토콜(https)
- 호스트명(www.google.com)
- 포트 번호(443)
- 패스(/search)
- 쿼리 파라미터(q=hello&hl=ko)

## URL userinfo

URL에 사용자정보를 포함해서 인증

거의 사용하지 않음.

## URL host

- 호스트명
- 도메인명 또는 IP주소 직접 사용가능

## URL PORT

- 포트
- 접속 포트
- 일반적으로 생략 (생략시 http는 80, https는 443)

## URL path

리소스 경로, 계층적 구조

## URL query

- key=value 형태
- 키의 밸류 값을 검색할 때 주로 사용
- query parameter, query string 등으로 불림

## URL - fragment

html 내부 북마크 등에 사용
