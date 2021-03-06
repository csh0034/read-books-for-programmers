# 11장 시스템

Ray Ozzie - 마이크로소프트 최고 기술 책임자(CTO)

"복잡성은 죽음이다. 개발자에게서 생기를 앗아가며, 제품을 계획하고 제작하고 테스트하기 어렵게 만든다."

## 목록

1. [도시를 세운다면](#도시를-세운다면)
2. [시스템 제작과 시스템 사용을 분리하라](#시스템-제작과-시스템-사용을-분리하라)
    - [Main 분리](#main-분리)
    - [팩토리](#팩토리)
    - [의존성 주입](#의존성-주입)
3. [확장](#확장)
    - [횡단 관심사](#횡단-관심사)
4. [자바 프록시](#자바-프록시)
5. [순수 자바 AOP 프레임워크](#순수-자바-aop-프레임워크)
6. [AspectJ 관점](#aspectj-관점)
7. [테스트 주도 시스템 아키텍처 구축](#테스트-주도-시스템-아키텍처-구축)
8. [의사 결정을 최적화하라](#의사-결정을-최적화하라)
9. [명백한 가치가 있을 때 표준을 현명하게 사용하라](#명백한-가치가-있을-때-표준을-현명하게-사용하라)
10. [시스템은 도메인 특화 언어가 필요하다](#시스템은-도메인-특화-언어가-필요하다)
11. [결론](#결론)

## 도시를 세운다면

도시를 세운다면 세세항 사항을 혼자서 직접 관리할 수 있을까? 불가능하다.

이미 세우진 도시라도 한 사람의 힘으로는 무리다. 각 분야를 관리하는 팀이 있기에 가능하다. 도시에는 **큰 그림을 그리는 사람들**도 있으며 **작은 사항에 집중하는 사람들**도 있다.

도시는 **적절한 추상화와 모듈화**로 큰 그림을 이해하지 못할지라도 개인과 개인이 관리하는 **구성 요소**는 효율적으로 관리될 수 있도록 구성해야한다.

흔히 소프트웨어 팀도 도시처럼 구성한다. 깨끗한 코드를 구현하면 **낮은 추상화 수준에서 관심사를 분리**하기 쉬워진다.

이 장에서는 높은 추상화 수준, 즉 **시스템 수준**에서도 깨끗함을 유지하는 방법에 대해 살펴보자.

## 시스템 제작과 시스템 사용을 분리하라

우선 제작(construction)은 사용(use)과 아주 다르다. 이 사실을 명심하자.

소프트웨어 시스템은 제작과 사용을 분리해야 된다.

- **준비 과정**: 객체를 제작하고 의존성을 서로 연결
- **런타임 로직**: 준비 과정 이후, 제작된 객체를 사용하는 비즈니스 로직

> 소프트웨어 시스템은 (애플리케이션 객체를 제작하고 의존성을 서로 **연결**하는 준비 과정과 (준비 과정 이후에 이어지는) 런타임 로직을 분리해야 한다.)

**시작 단계**는 모든 애플리케이션이 풀어야 할 **관심사(concern)다.**

**관심사 분리**는 우리 분야에서 가장 오래되고 가장 중요한 설계 기법 중 하나다. ~~불행히도 대다수 애플리케이션은 시작 단계라는 관심사를 분리하지 않는다.~~

```java
// 준비 과정 코드와 런타임 로직이 뒤섞인 안좋은 예
public class LazyInitialization {
  public Service getService() {
    if (service == null) {
      service = new MyServiceImpl(...);
      //             ^-- 모든 상황에 적합한 기본 값일까?
    }
    return service;
  }
}
```

다음 코드는 초기화 지연(Lazy Initialization) 혹은 계산 지연(Lazy Evaluation)이라는 기법이다.

- 실제로 객체를 사용하기 전까지 객체를 생성하지 않는다. 불필요한 부하가 걸리지 않는다.
- `NullPointException`이 발생하지 않는다.

하지만, 다음과 같은 이유로 위 `좀스러운 설정 기법` 코드는 나쁜 코드다.

- 컴파일 시 의존성 문제로 **강한 결합**이 존재한다.
  - getter 메서드가 초기화 지연 객체(MyServiceImpl)의 **생성자 인수에 명시적으로 의존**한다.
- 테스트 코드 작성시, 테스트 전용 객체(Test Double, Mock Object)를 생성하는데 부담된다.
- 단일 책임 원칙(SRP, Single Responsibility)을 위반한다.
  - 런타임 로직에 준비 과정(객체 생성, 의존성 연결)이 섞였다. 메서드의 책임이 두 가지다.
- **모든 상황에 적합한 객체인가?** 가장 큰 우려다.

초기화 지연 기법을 한 번 정도 사용한다면 문제가 되진 않는다. 

다만, 많은 애플리케이션이 이처럼 **좀스러운 설정 기법**을 수시로 사용한다. **좀스럽고 손쉬운 기법**은 지양하자. 대게 **좀스러운 설정 기법**은 애플리케이션 곳곳에 흩어져 중복을 유발한다. 모듈성은 깨진다.

**체계적이고 탄탄한 시스템**을 만들고 싶다면 모듈성을 높여야 한다.

**설정 논리**(객체 생성, 의존성 연결)와 **실행 논리** **분리**해야 모듈성이 높아진다.

### Main 분리

시스템 생성과 사용을 분리하는 한 가지 방법은 main을 분리하는 것이다.

1. `main`은 전반적인 시스템의 준비 과정(construction)을 다룬다.
   - 즉 시스템에 관련된 모든 객체 생성과 의존성 연결을 담당한다.
2. 애플리케이션에서 필요한 객체는 `main`을 통해 얻어 사용(use)한다.

자연스레 시스템의 모든 객체의 의존성 방향은 main에서 애플리케이션 쪽으로 향한다.

### 팩토리

**실행 논리**만 다뤄야하는 애플리케이션에서 객체 생성이 필요할 때도 있다.

이 경우 [Abstract Factory 패턴](https://refactoring.guru/design-patterns/abstract-factory) 을 사용하면 **설정 논리**를 포함하지 않고, 런타임 시점에 객체 생성을 결정할 수 있다.

애플리케이션은 객체의 생성 시점을 결정하지만, 객체를 생성하는 로직(설정 논리)은 모른다.

> 애플리케이션은 실행 논리에서 필요한 객체를 factory 클래스를 통해 주입 받는다.

### 의존성 주입

**의존성 주입(DI, Dependency Injection)** 은 시스템의 제작(construction)과 사용(use)을 분리하는 강력한 메커니즘이다.  

의존성 주입은 제어 역전(IoC, Inversion of Control) 기법을 **의존성 관리**에 적용한 매커니즘이다.

**제어 역전에서는** **한 객체가 맡은 보조 책임**을 새로운 객체에게 전적으로 **떠넘긴다.** 새로운 객체는 넘겨받은 책임만 맡으므로 단일 책임 원칙(SRP, Single Responsibility Principle)을 지키게 된다.

**의존성 관리 맥락에서 객체는** 의존성 자체를 인스턴스로 만드는 책임은 지지 않는다. 대신에 이런 책임을 다른 `전담` 메커니즘에 넘긴다. 그렇게 함으로써 **제어를 역전한다.**

전담(책임질) 메커니즘으로 'main' 루틴이나 특수 **컨테이너**를 사용한다.

```java
MyService service = (MyService)(jndiContext.lookup("NameOfMyService"));
```

이처럼 **진정한 의존성 주입은** 클래스가 의존성을 해결하려 시도하지 않는다.

클래스는 완전히 수동적이다. 대신에 DI 컨테이너에 의해 필요한 객체의 인스턴스를 생성한 뒤, 생성자/setter 를 통해 의존성 주입한다.

> 대다수의 프레임워크의 DI 컨테이너는 팩토리 패턴이나, 프록시를 통해 초기화 지연이나 계산 지연과 같은 최적화 기법을 지원한다.

## 확장

**'처음부터 올바르게'** 시스템을 만들 수 있다는 믿음은 미신이다.

성장할지 모른다는 기대로 작은 규모의 프로젝트에서 확장을 고려한다면 오히려 오버엔지니어링이다.

시스템은 점진적으로 조정하고 확장해야 한다. 이것이 반복적이고 점진적인 **애자일 방식의 핵심**이다. 테스트 주도 개발(TDD, Test-driven Development)와 리펙터링으로 얻어지는 깨끗한 코드는 **코드 수준에서 시스템을 조정하고 확장하기 쉽게 만든다.**

단순한 아키텍처를 복잡한 아키텍처로 점진적으로 발전할 수 있는 첫 걸음은 **관심사를 적절히 분리해** 관리하는 것이다.

> 자바 진영의 표준 프레임워크라 할 수 있는 EJB가 망한 이유도 관심사를 분리하지 못했기 때문이다.

### 횡단 관심사

영속성과 같은 횡단 관심사는 애플리케이션의 자연스러운 객체 경계를 넘나드는 경향이 있다. 횡단 관심사는 모든 객체가 전반적으로 일관적이며, 동일한 방식으로 동작해야한다. 

횡단 관심사는 원론적으로는 모듈화되고 캡슐화된 방식으로 구상할 수 있지만, 구현한 코드들이 여러 애플리케이션의 경계를 넘나들며 온갖 객체로 흩어진다. 

관점 지향 프로그래밍(AOP, Aspect-Oriented Programming)은 **횡단 관심사에 대처해 모듈성을 확보하는 일반적인 방법론이다.** AOP에서 **관점(Aspect)** 이라는 모듈 구성 개념은 "특정 관심사를 지원하려면 시스템에서 특정 지점들이 동작하는 방식을 일관성 있게 바꿔야 한다"라고 명시한다. 

**AOP 시스템의 진정한 가치는** 시스템 동작을 간결하고 모듈화된 방식으로 명시하는 능력이다. 모든 AOP 프레임워크는 대상 코드에 영향을 미치지 않는 상태로 동작 방식을 변경한다.

## 자바 프록시

자바 프록시는 개별 객체나 클래스에서 메서드 호출을 감싸는 경우처럼 **단순한 상황**에 적합하다.

자바 프록시 구현은 인터페이스와 클래스 기반으로 구분한다. 

- JDK Dynamic Proxy: 인터페이스 기반, 자바 리플렉션 이용
- CGLib, ASM, Javassit: 클래스 기반, 바이트 코드 기반

직접 구현한 프록시 코드는 상당히 복잡하며 코드의 "양"이 크다. 깨끗한 코드를 작성하기 어렵다.

> 프록시는 진정한 AOP 해법에 필요한 시스템 단위로 실행 '지점'을 명시하는 메커니즘도 제공하지 않는다.

## 순수 자바 AOP 프레임워크

대부분의 프록시 코드는 비슷하기 때문에 도구로 자동화할 수 있다.

대표적으로 스프링 AOP, JBoss AOP가 내부적으로 프록시를 사용한 자바 프레임워크다.

이러한 순수 자바 AOP 프레임워크를 통해 전반적인 시스템의 관심사를 철저히 분리할 수 있다. 예를 들어 스프링은 비즈니스 논리를 POJO(Plain Old Java Object)로 구현한다. POJO는 순수하게 도메인에 초점을 맞춘다.

- 관심사가 분리된 코드는 테스트가 개념적으로 더 쉽고 간단하다.
- 비즈니스에 집중할 수 있다.
  - 상대적으로 단순하기 때문에 사용자 스토리를 올바로 구현하기 쉽다.
- 유지 보수가 용이하다.
  - 미래 스토리에 맞춰 코드를 보수하고 개선하기 편하다.

> 스프링 프레임워크는 디자인 패턴의 집약체다. 스프링 AOP는 [Decorator 패턴](https://refactoring.guru/design-patterns/decorator) 과 [Proxy 패턴](https://refactoring.guru/design-patterns/proxy) 을 참고하자. 

## AspectJ 관점

관심사를 관점으로 분리하는 가장 강력한 도구는 AspectJ 언어다.

AspectJ는 언어 차원에서 관점을 모듈화 구성으로 지원하는 자바 언어 확장이다.

AspectJ는 자바 순수 AOP 프레임워크 보다 강력하고 풍부한 도구 집합을 제공하긴 하지만, 러닝 커브가 높아 새 도구를 사용하고 새 언어 문법과 사용법을 익혀야 한다는 단점이 있다.

스프링 프레임워크는 AspectJ의 높은 러닝 커브를 어노테이션 기반으로 관점을 쉽게 사용하도록 다양한 기능을 제공한다. 

## 테스트 주도 시스템 아키텍처 구축

관점으로 관심사를 분리하는 방식은 그 위력이 막강하다. 

애플리케이션 도메인 논리로 POJO로 작성할 수 있다면 즉, 코드 수준에서 아키텍처 관심사를 분리할 수 있다면, 진정한 테스트 주도 아키텍처 구축이 가능해진다.

소프트웨어 나름대로 형체(physics)가 있지만, 소프트 웨어 구조가 관점을 효과적으로 분리한다면, 극적인 변화가 경제적으로 가능하다.

다시 말해, '아주 단순하면서도' 멋지게 분리된 아키텍처로 소프트웨어 프로젝트를 진행해 결과물을 재빨리 출시한 후, 기반 구조를 추가하며 조금씩 확장해나가도 괜찮다. 

**그렇다고 '아무 방향 없이'** 프로젝트에 뛰어들어도 좋다는 소리는 아니다. 프로젝트를 시작할 때는 일반적인 범위, 목표, 일정, 결과로 내놓을 시스템의 일반적인 구조도 생각해야 한다.

> 최선의 시스템 구조는 각기 POJO (또는 다른) 객체로 구현되는 모듈화된 관심사 영역(도메인)으로 구성된다. 이렇게 서로 다른 영역은 해당 영역 코드에 최소한의 영향을 미치는 관점이나 유사한 도구를 사용해 통합한다. 이런 구조 역시 코드와 마찬가지로 테스트 주도 기법을 적용할 수 있다.

## 의사 결정을 최적화하라

모듈을 나누고 관심사를 분리하면 지엽적인 관리와 결정이 가능해진다.

아주 큰 시스템에서는 한 사람이 모든 결정을 내리기 어렵다. 가장 적합한 사람에게 책임을 맡기면 가장 좋다. 

- 가능한 마지막 순간까지 결정을 미루는 방법이 최선이다. 
- 게으르거나 무책임해서가 아니다. 최대한 정보를 모아 최선의 결정을 내리기 위해서다.
- 성급한 결정은 불충분한 지식으로 내린 결정이다.
- 너무 일찍 결정하면 고객 피드백을 더 모으고, 프로젝트를 더 고민하고, 구현 방안을 더 탐험할 기회가 사라진다.

> 관심사를 모듈로 분리한 POJO 시스템은 **기민함을 제공한다.** 이런 기민함 덕택에 최신 정보에 기반해 최선의 시점에 최적의 결정을 내리기가 쉬워진다. 또한 결정의 복잡성도 줄어든다.

## 명백한 가치가 있을 때 표준을 현명하게 사용하라

가볍고 간단한 설계로 충분했을 프로젝트에서도 EJB2를 채택했다. 

이처럼 아주 과장되게 표장된 표준에 집착하는 바람에 고객 가치가 뒷전으로 밀려난 사례가 많았다.

> 표준을 사용하면 아이디어와 컴포넌트를 재사용하기 쉽고, 적절한 경험을 가진 사람을 구하기 쉬우며, 좋은 아이디어를 캡슐화하기 쉽고, 컴포넌트를 엮기 쉽다. 하지만 때로는 표준을 만드는 시간이 너무 오래 걸려 업계가 기다리지 못한다. 어떤 표준은 원래 표준을 제정한 목적을 잊어버리기도 한다.

## 시스템은 도메인 특화 언어가 필요하다

필수적인 정보를 명료하고 정확하게 전달하는 어휘, 광용구, 패턴은 중요하다.

소프트웨어 분야에서 DSL(Domain-Specific Language)은 간단한 스크립트 언어나 표준 언어로 표준 언어로 구현한 API를 가리킨다.

- 좋은 DSL은 도메인 개념과 그 개념을 구현한 코드 사이에 존재하는 **'의사소통 간극'** 을 줄여준다.
- 도메인 전문가가 사용하는 언어로 도메인 논리를 구현하면 도메인을 잘못 구현할 가능성이 줄어든다.
- 효과적으로 사용한다면 DSL은 추상화 수준을 코드 관용구나 디자인 패턴 이상으로 끌어올린다.

> DSL은 개발자가 적절한 추상화 수준에서 코드 의도를 표현할 수 있는 수단이다.
> DSL를 사용하면 고차원 정책에서 저차원 세부사항에 이르기까지 모든 추상화 수준과 모든 도메인을 POJO로 표현할 수 있다.

## 결론

시스템 역시 깨끗해야 한다. 깨끗하지 못한 아키텍처는 도메인 논리를 흐리며 기민성을 떨어뜨린다. 

도메인 논리가 흐려지면 제품 품질이 떨어진다. 버그가 숨어들기 쉬워지고, 스토리를 구현하기 어려워지는 탓이다. 기민성이 떨어지면 생산성이 낮아져 TDD가 제공하는 장점이 사라진다. 

시스템을 설계하든 개별 모듈을 설계하든, 실제로 돌아가는 가장 단순한 수단을 사용해야 한다는 사실을 명심하자.