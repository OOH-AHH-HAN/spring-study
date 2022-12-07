# Spring 프로젝트 환경설정

## 프로젝트 생성

- IDE : IntelliJ
- Java 17
- Spring-boot : `3.0`
	* SNAPSHOT, M1 : 정식 release 버전이 아님

- 빌드 관리 도구 : `Gradle`
	* 빌드 관리 도구란?
		+ 필요한 라이브러리를 가져오고 빌드한 라이프 사이클을 관리하는 툴
	* Gradle vs Maven
		+ 요즘 추세는 Gradle

- Dependencies : Spring Web, Thymeleaf
	* Dependency란?
		+ 어떤 라이브러리를 가져올 것인지
		+ Dependency에 라이브러리를 추가해주면 알아서 `의존관계`에 있는 라이브러리까지 다 가져옴

- Spring 프로젝트 구조
	* `main/java` : java
	* `main/resources` : xml, html, 프로젝트 설정 파일, java를 제외한 파일
	* `main/test` : **테스트 코드 (매우 중요)**
	* `.gitignore` : 공개되면 안되는 설정 및 암호를 포함한 소스를 제외하는 파일
	* `build.gradle` : 스프링 부트에 필요한 설정 추가하여 빌드 파일 설정
		+ sourceCompatibility : 자바 소스를 컴파일 시키는 역할. 자바 version 명시
		+ dependencies : 라이브러리(의존 관계에 있는 라이브러리 포함)
		+ repositories : dependency를 다운받아올 곳 명시
		+ IntelliJ에서 프로젝트를 열 때 이 파일을 open함

- Spring-boot 라이브러리
	* spring-boot-starter-tomcat : 톰캣(웹서버)
	* spring-webmvc : 스프링 웹 mvc
	* spring-boot-starter-thymeleaf : 타임리프 템플릿 엔진
	* `spring-boot-starter(공통)` : 스프링 부트 + 스프링 코어 + 로깅(심각한 에러 모아보기, 로그 관리 가능)
		+ spring-boot
			- spring-core
		+ spring-boot-starter-logging
			- `logback`, slf4j			

- 테스트 라이브러리
	* spring-boot-starter-test
		+ `junit` : 테스트 프레임워크 (5로 추세가 넘어감)
		+ mockito : 목 라이브러리
		+ assertj : 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리
		+ spring-test : 스프링 통합 테스트 지원
		
<br>

## Java 11 설치(맥북 m1)


- 현재 사용중인 java version 확인

```bash
java -version
```

<br>

- 현재 로컬 java version 확인

```bash
/usr/libexec/java_home -V
```

<br>

- java 다른 버전으로 변경

```bash

#java 8
export JAVA_HOME=$(/usr/libexec/java_home -v 1.8)
source ~/.bash_profile

#java 11
export JAVA_HOME=$(/usr/libexec/java_home -v 11)

```
<br>


- 환경변수 오류

```bash
#환경변수 설정 파일
vim ~/.bash_profile

#설정 파일 안에 경로 설정(~/.bash_profile 안에)
# ----------------------------------------------------
  #Java Paths
  export JAVA_HOME_17=$(/usr/libexec/java_home -v17)
		
  #Java 17
  export JAVA_HOME=$JAVA_HOME_17
# ----------------------------------------------------

#변경한 설정 파일 적용
source .bash_profile

#설정 파일 잘 적용됐는지 확인
echo $JAVA_HOME

```

<br>

## View 환경설정

- 다른 경로가 없을 때는 `static/index.html`을 가장 먼저 찾음
- 동작
	1. 웹 브라우저에 `localhost:8080/hello` 입력 (`get` 방식)
	2. 내장된 톰캣 서버에서 요청을 받아 스프링 컨테이너의 Controller의 `hello`를 찾아 매핑
	3. Controller에서 `model` 객체의 값, `return` 뒤에 화면을 반환(뮨자)s
	4. viewResolver가 문자를 받아 `resources:templates/{viewName}+.html`형식으로 매핑


<br>

## 빌드 및 실행

- Terminal
	1. `./gradlew build` 
		- build시 오류 발생하면 `./gradlew clean build` 		( 완전히 지우고 build ) 
	2. `cd build/libs`
	3. `ls -arlth` ( .을 포함한 경로 안의 모든 파일과 디렉토리, 		하위 경로와 그 안에 있는 모든 파일의 내용을 자세히 출력)
	4. `java -jar hello-spring-0.0.1-SNAPSHOT.jar` 		(빌드)




