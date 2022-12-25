# 1. AOP가 필요한 상황

### 모든 메소드의 호출 시간을 측정하고 싶다면?

- 메소드 마다 시간 측정하는 것을 아래와 같이 넣는건 시간 낭비가 크다.
- 시간을 측정하는 로직은 공통 기능이다.

```java
/**
     * 회원 가입
     */
    public Long join(Member member){

        long start = System.currentTimeMillis();
        try {
            validateDuplicateMember(member); //중복 회원 검증
            memberRepository.save(member);
            return member.getId();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("join " + timeMs + "ms");
        }
    }
```

# 2. AOP 적용

Aspect Oriented Programming

공통 관심 사항과 핵심 관심 사항 분리

시간 측정 로직을 공통 관심 사항 적용

AOP는 `@Aspect` 라고 적어줘야 함.

@Component 스캔 사용해도 되긴 하지만

스프링 bean에 등록하는 것을 선호 함.

왜냐면 AOP 걸어서 쓰는구나 라는 것을 알 수 있기에

```java
@Bean
    public TimeTraceAop TimeTraceAop(){
        return new TimeTraceAop();
    }
```

### TimeTraceAop

```java
package hello.hellospring.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class TimeTraceAop {

    @Around("execution(* hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable{
        long start = System.currentTimeMillis();
        System.out.println("START: "+ joinPoint.toString());
        try{
            return joinPoint.proceed();
        }finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END:" + joinPoint.toString() + " " + timeMs + "ms");
        }
    }
}
```

`@Around()` 어노테이션

타켓팅을 해줄 수가 있다.

`@Around("execution(* hello.hellospring..*(..))")`

안에 원하는 조건을 넣을 수 있음

예시로 위에는 패키지 모든 적용을 쓴 것이다.
![Untitled 4](https://user-images.githubusercontent.com/62877858/209468813-35f8a40b-e238-480a-a8ea-94a7fbd7c59a.png)


동작 하나하나에 적용 되는 것을 볼 수 있다.

## 실제 동작 방식

### AOP 적용 전 의존관계

helloController→memberService

### AOP 적용 후 의존관계

helloController→memberService(프록시) 가짜 memberService → memberServuce실제 호출

### AOP 적용 후 그림
![Untitled 5](https://user-images.githubusercontent.com/62877858/209468819-7968beab-5079-444a-9451-7fe0eda6b0e4.png)
