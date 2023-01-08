## SOLID

클린코드로 유명한 로버트 마틴이 좋은 객체지향 설계의 5개의 원칙

## SRP(Single responsibility principle) 단일 책임 원칙

> 한 클래스는 하나의 책임만 가져야 한다.
> 

하지만, 하나의 책임이라는 것은 모호한데, 중요한 기준은 변경이다. 변경이 있을 때, 파급 효과가 적으면 단일 책임 원칙을 잘 따른 것.

예를 들어 변경이 있을 때, 하나의 클래스 or 하나의 지점만 고치면 되는 것이 단일 책임 원칙을 잘 지키는 것

## OCP(Open/closed principle) 개방-폐쇄 원칙

> 소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다.
> 

다형성을 활용하면, 확장에는 열려있고, 변경에는 닫혀있을 수 있으므로, 개방-폐쇄 원칙을 잘 지킬 수 있다.

### OCP의 문제점

```java
public class MemberService{
	//private MemberRepository memberRepository = new MemoryMemberRepository();
	private MemberRepository memberRepository = new JdbcMemberRepository();
}
```

위와 같이 Service 클라이언트가 구현 클래스를 직접 선택하는 상황에서, JdbcMemberRepository로 변경을 하려면, 클라이언트 코드를 변경을 해야하는 문제가 생긴다. 이렇게 되면 OCP 원칙을 지킬 수가 없는 것이다.

객체를 생성하고 연관관계를 맺어주는 별도의 조립, 설정자가 필요하다.

## LSP(Liskov substitution principle) 리스코프 치환 원칙

> 인터페이스의 규약을 지켜야하는 것으로 다형성을 지원하기 위한 원칙
> 

### 예시

A의 인터페이스의 규약은 ‘자동차는 앞으로 가라’ 하지만 ‘자동차는 뒤로 가라’로 만들 수도 있다 그래도 A의 인터페이스 규약을 지켜서 ‘자동차는 앞으로 가라’로 만들어야 한다. A의 인터페이스 규약에 맞게 기능을 구현하지 않으면, 위배하는 것

## ISP(Interface segregation principle) 인터페이스 분리 원칙

> 특정 클라이어트를 위한 여러 개의 인터페이스가 범용 인터페이스 하나보다 낫다.
> 

### 예시

자동차 인터페이스 → 운전 인터페이스, 정비 인터페이스로 분리

사용자 클라이언트 → 운전자 클라이언트, 정비사 클라이언트로 분리

위와 같이 분리를 하게 되면, 정비에 대한 문제가 생겼을 시에 정비와 정비사 클라이언트만 바꾸면 되고, 운전과 운전자 클라이언트에 영향이 없는 장점이 있다.

이로 인해 인터페이스가 명확해지고, 대체 가능성이 높아진다.

## DIP(Dependency inversion principle) 의존관계 역전 원칙

> 프로그래머는 추상화에 의존해야 하고, 구체화에 의존하면 안된다.
> 

클라이언트 코드가 구현 클래스를 바라보지말고, 인터페이스를 바라봐야한다는 뜻

운전자는 자동차 역할에 대해서 바라봐야하지, K3, 테슬라, 아반떼에 바라보면 안된다.

역할에 의존을 해야하고, 구현에 의존하면 전혀 안되고, 구현체에 의존하게 되면 변경이 아주 어려워진다.

### 의존한다?

```java
public class MemberService{
	private MemberRepository memberRepository = new MemoryMemberRepository();
}
```

MemberService는 인터페이스에 의존하지만, MemoryMemberRepository에도 의존하고 있는 것

MemoryMemberRepository를 다른 것으로 변경하려고 할 때, 코드를 변경해야한다.

여기서 의존한다는 것으저 코드에 대해서 안다는 것 자체가 의존한다.

DIP위반 하는 것 추상화에 의존해야하는데, 구현체에도 의존하고 있으므로 DIP 위반

## 총 정리

객체 지향의 핵심은 다형성

다형성 만으로는 OCP, DIP를 지킬 수가 없다.
