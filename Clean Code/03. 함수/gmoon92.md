# 3장 함수

어떤 프로그램이든 가장 기본적인 단위가 함수다. 

1. 함수가 읽기 쉽고 이해하기 쉬운 이유는 무엇일까?
2. 의도를 분명히 표현하는 함수는 어떻게 구현할 수 있을까?
3. 함수에 어떤 속성을 부여해야 처음 읽는 사람이 프로그램 내부를 직관적으로 파악할 수 있을까?

이 장은 함수를 잘 만드는 법을 소개한다.

## 목차

1. [작게 만들어라](#작게-만들어라)
    - [블록과 들여쓰기](#블록과-들여쓰기)
2. [한 가지만 해라!](#한-가지만-해라!)
3. [함수 당 추상화 수준은 하나로!](#함수-당-추상화-수준은-하나로!)
4. [위에서 아래로 코드 읽기:내려가기 규칙](#위에서-아래로-코드-읽기:내려가기-규칙)
5. [Switch 문](#Switch-문)
   - [switch 문과 다형성](#switch-문과-다형성)
6. [서술적인 이름을 사용하라!](#서술적인-이름을-사용하라!)
7. [함수 인수](#함수-인수)
   - [많이 쓰는 단항 형식](#많이-쓰는-단항-형식)

---

## 작게 만들어라

**함수를 만드는 첫째 규칙은 `작게`다. 둘째 규칙은 `더 작게!` 다.**

함수가 작을수록 더 좋다는 증거나 자료를 제시하기 어렵다.
하지만, 함수가 작을 수록 하나의 기능을 수행한다. 이는 논리가 간단하다는 증거이며, 하나의 기능을 수행하는 함수는 언제나 명백하며 읽기 좋다. 

**깨끗한 코드는 한 가지에 집중한다.** 다는 클린 코드의 모토와 같다.

### 블록과 들여쓰기

함수를 작게 만든는 첫 걸음은 `indent`(인덴트, 들여쓰기) `depth`를 2가 넘지 않도록 구현하도록 집중한다.

다시 말해, if 문/else 문/while 문 등에 들어가는 블록은 한줄이어야 한다는 의미다. 이 말은 중첩 구조가 생길만큼 함수가 커져서는 안 된다는 뜻이다. 함수에서 들여쓰기 수준은 1단이나 2단을 넘어서면 안 된다. 

당연한 말이지만, 그래야 함수는 읽고 이해하기 쉬워진다.

## 한 가지만 해라!

다음 함수 규칙은 30여년 동안 여러 가지 다양한 표현으로 개발자들에게 주어진 충고이다.

함수는 **한 가지를 해야 한다.** 그 **한 가지를 잘 해야한다.** 그 **한 가지만을 해야 한다.**

한 가지만 수행한다는 기준은 무엇일까.

함수안에서 다양한 작업을 처리한다고 해서 그 함수는 여러 작업을 수행한다고 단언할 수 없다.
지정된 **함수 이름 아래에서 [추상화](https://ko.wikipedia.org/wiki/%EC%B6%94%EC%83%81%ED%99%94_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99)) 수준이 하나인 단계만 수행한다면** 그 **함수는 한 가지 작업만 한다.**

함수가 `한 가지`만 하는지 판단하는 방법이 하나 더 있다.
단순히 다른 표현이 아니라 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 셈이다.

## 함수 당 추상화 수준은 하나로!

즉 함수가 `한 가지`만 수행한다는 기준은 **함수내 모든 문장의 `추상화 수준`이 동일해야 한다.**

한 함수 내에 추상화 수준을 섞으면, 특정 표현이 `근본 개념`인지 아니면 `세부사항`인지 구분하기 어렵기 때문에 코드를 읽는 사람이 헷갈린다.

### 위에서 아래로 코드 읽기:내려가기 규칙

코드는 위에서 아래로 이야기처럼 읽혀야 좋다.

일반적으로 코드에서 함수 내에서 호출한 함수는 다음 함수가 배치한다.

```java
public class HtmlUtil {
   public static String renderPageWithSetupsAndTeardowns(PageData pageData, 
                                                         boolean isSuite) throws Exception {
      if (isTestPage(pageData)) {
         includeSetupAndteardownPages(pageData, isSuite);
      }
      return pageData.getHtml();
   }

   // renderPageWithSetupsAndTeardowns 메서드 보다 한 단계 낮은 추상화 함수
   public static boolean isTestPage(PageData pageData) {
      return pageData.hasAttribute("Test");
   }
}
```

자연스레 코드의 추상화 수준은 높은 수준부터 한 단계씩 낮아진다. 이를 내려가기 규칙이라 한다.

## Switch 문

switch 문은 작게 만들기 어렵다.

본질적으로 switch 문은 `N가지`를 처리한다. 따라서 switch 문은 구현 코드에선 사용하지 말아야한다. 이는 앞서 `블록과 들여쓰기` 들여쓰기 규약을 위반한다.  

이외에도 switch 문을 구현 코드에서 직접 사용하게 되면 다음과 같은 문제들이 발생한다.

```java
class EmployeeService {
   public Money calculatePay(Employee e) {
      switch (e.type) {
         case COMMISSIONED:
            return calculateCommissionedPay(e);
         case HOURLY:
            return calculateHourlyPay(e);
         default:
            throw new InvalidEmployeeType(e.type);
      }
   }
}
```

1. 함수가 길다.
2. `한 가지` 작업만 수행하지 않는다.
3. 계산에 대한 추상화 수준이 다양하다. ([SRP, Single Responsibility](https://en.wikipedia.org/wiki/Single-responsibility_principle) 위반)
4. 새 유형을 추가할 때마다 코드를 추가해야 한다. ([OCP, Open Closed Principle](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle) 위반)

### switch 문과 다형성

아예 switch 문을 사용하지 말라는 말이 아니다. 일반적으로 switch 문은 단 한번만 참아준다.

[추상 팩토리 디자인 패턴](https://en.wikipedia.org/wiki/Abstract_factory_pattern)을 사용하여 switch 문을 구현 코드에서 숨기면 된다.

```java
public interface Employee {
   Money calculatePay();
}

//----------------
public class CommissionedEmployee implements Employee {
   @Override
   public Money calculatePay() {
     // ...
     return money;
   }
}

public class HourlyEmployee implements Employee {
   @Override
   public Money calculatePay() {
      // ...
      return money;
   }
}

//----------------
public class EmployeeFactory {
  Employee makeEmployee(EmployeeRecord r) {
     switch (r.type) {
        case COMMISSIONED:
           return CommissionedEmployee(e);
        case HOURLY:
           return HourlyEmployee(e);
        default:
           throw new InvalidEmployeeType(e.type);
     }
  }
}
//----------------
public class EmployeeService {
   public Money calculatePay(Employee e) {
      return e.calculatePay();
   }
}
```

다음과 코드 처럼 switch 문과 [다형성(polymorphism)](https://en.wikipedia.org/wiki/Polymorphism_(computer_science))을 이용하여 switch 문을 저차원 클래스에 숨기고 절대로 반복하지 않도록한다.

## 서술적인 이름을 사용하라!

의도를 드러낼 수 있다면 이름이 길어져도 괜찮다.
함수가 작고 단순할수록 서술적인 이름을 고르기도 쉬워진다.

> "코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드라 불러도 되겠다." - 워드 커닝햄(Ward Cunningham)

1. 길고 서술적인 이름이 짧고 어려운 이름보다 좋다. 
2. 이름을 정하느라 시간을 들여도 괜찮다. 이런저런 이름을 넣어 코드를 읽어보면 더 좋다. 
3. 함수 이름을 정할 때는 여러 단어가 쉽게 읽히는 명명법을 사용한다. 
4. 이름을 붙일 때는 일관성이 있어야한다.

|명명법|예시|
|---|---|
|[CamelCase](https://en.wikipedia.org/wiki/Camel_case)|gmoonGithubBlog|
|Constant| DEFAULT_COUNTRY_CODE|
> 참고) [wiki naming convention](https://en.wikipedia.org/wiki/Naming_convention_(programming))

## 함수 인수

함수에서 이상적인 인수 개수는 무항(0개)이다.
특별한 이유가 있더라도 **4개 이상(다항)은 사용하면 안된다.**

함수에 인수 개수가 적을수록 코드를 이해하기 쉽다.
> includeSetupPageInfo(new PageContent()) 보다 includeSetupPage()가 이해하기 더 쉽다.

테스트 관점에서 보면 인수는 더 어렵다.
인수 개수가 3개를 넘어가면, 인수마다 유효한 값으로 모든 조합을 구성해 테스트 케이스를 검증해야 함으로 상당히 부담스럽기 때문이다.

출력 인수는 입력 인수보다 이해하기 어렵다.
> 출력 인수란 함수의 반환 값이 아닌 인수로 결과를 받는 인수다.

```java
public class Member {
   private String name;

   public Member(String name) {
     this.name = name;
   }
   
   private static Member createNew(String name) {
      return new Member(name);
   }
}
```

흔히 우리는 함수에다 `인수`로 `입력`을 넘기고 `반환값`으로 `출력`을 받는다는 개념에 익숙하다.

- 입력 인수: String name
- 출력 인수: Member

```java
public class Member {
   // 출력 인수 사용으로 독자를 햇갈리게 만든다.
   private static Member createNew(Member member, String name) {
      member.name = member;
      return member;
   }
}
```

대개 함수에서 인수로 결과를 받으리라 기대하지 않는다.
이처럼 출력 인수는 독자가 허둥지둥 코드를 재차 확인하게 만든다.

최선은 입력 인수가 없는 경우이며, 차선은 입력 인수가 1개뿐인 경우다.

### 많이 쓰는 단항 형식

