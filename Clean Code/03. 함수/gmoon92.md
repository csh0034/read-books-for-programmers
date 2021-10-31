# 3장 함수

어떤 프로그램이든 가장 기본적인 단위가 함수다. 

1. 함수가 읽기 쉽고 이해하기 쉬운 이유는 무엇일까?
2. 의도를 분명히 표현하는 함수는 어떻게 구현할 수 있을까?
3. 함수에 어떤 속성을 부여해야 처음 읽는 사람이 프로그램 내부를 직관적으로 파악할 수 있을까?

이 장은 함수를 잘 만드는 법을 소개한다.

## 목차

1. [작게 만들어라](#작게-만들어라)
    - [블록과 들여쓰기](#블록과-들여쓰기)
2. [한 가지만 해라](#한-가지만-해라)
3. [함수 당 추상화 수준은 하나로](#함수-당-추상화-수준은-하나로)
4. [위에서 아래로 코드 읽기](#위에서-아래로-코드-읽기)
5. [Switch 문](#switch-문)
   - [switch 문과 다형성](#switch-문과-다형성)
6. [서술적인 이름을 사용하라](#서술적인-이름을-사용하라)
7. [함수 인수](#함수-인수)
   - [많이 쓰는 단항 형식](#많이-쓰는-단항-형식)
   - [플래그 인수](#플래그-인수)
   - [이항 함수](#이항-함수)
   - [상함 함수](#상함-함수)
   - [인수 객체](#인수-객체)
   - [인수 목록](#인수-목록)
   - [동사와 키워드](#동사와-키워드)
8. [부수 효과를 일으키지 마라](#부수-효과를-일으키지-마라)
   - [출력 인수](#출력-인수)
9. [명령과 조회를 분리하라](#명령과-조회를-분리하라)
10. [오류 코드보다 예외를 사용하라](#오류-코드보다-예외를-사용하라)
    - [Try Catch 블록 뽑아내기](#try-catch-블록-뽑아내기)
    - [오류 처리도 한 가지 작업이다](#오류-처리도-한-가지-작업이다)
    - [의존성 자석](#의존성-자석)
11. [반복하지 마라](#반복하지-마라)
12. [구조적 프로그래밍](#구조적-프로그래밍)
13. [함수를 어떻게 짜죠](#함수를-어떻게-짜죠)
14. [결론](#결론)

---

## 작게 만들어라

**함수를 만드는 첫째 규칙은 `작게`다. 둘째 규칙은 `더 작게` 다.**

함수가 작을수록 더 좋다는 증거나 자료를 제시하기 어렵다.
하지만, 함수가 작을 수록 하나의 기능을 수행한다. 이는 논리가 간단하다는 증거이며, 하나의 기능을 수행하는 함수는 언제나 명백하며 읽기 좋다. 

**깨끗한 코드는 한 가지에 집중한다.** 다는 클린 코드의 모토와 같다.

### 블록과 들여쓰기

함수를 작게 만든는 첫 걸음은 `indent`(인덴트, 들여쓰기) `depth`를 2가 넘지 않도록 구현하도록 집중한다.

다시 말해, if 문/else 문/while 문 등에 들어가는 블록은 한줄이어야 한다는 의미다. 이 말은 중첩 구조가 생길만큼 함수가 커져서는 안 된다는 뜻이다. 함수에서 들여쓰기 수준은 1단이나 2단을 넘어서면 안 된다. 

당연한 말이지만, 그래야 함수는 읽고 이해하기 쉬워진다.

## 한 가지만 해라

다음 함수 규칙은 30여년 동안 여러 가지 다양한 표현으로 개발자들에게 주어진 충고이다.

함수는 **한 가지를 해야 한다.** 그 **한 가지를 잘 해야한다.** 그 **한 가지만을 해야 한다.**

한 가지만 수행한다는 기준은 무엇일까.

함수안에서 다양한 작업을 처리한다고 해서 그 함수는 여러 작업을 수행한다고 단언할 수 없다.
지정된 **함수 이름 아래에서 [추상화](https://ko.wikipedia.org/wiki/%EC%B6%94%EC%83%81%ED%99%94_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99)) 수준이 하나인 단계만 수행한다면** 그 **함수는 한 가지 작업만 한다.**

함수가 `한 가지`만 하는지 판단하는 방법이 하나 더 있다.
단순히 다른 표현이 아니라 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 셈이다.

## 함수 당 추상화 수준은 하나로

즉 함수가 `한 가지`만 수행한다는 기준은 **함수내 모든 문장의 `추상화 수준`이 동일해야 한다.**

한 함수 내에 추상화 수준을 섞으면, 특정 표현이 `근본 개념`인지 아니면 `세부사항`인지 구분하기 어렵기 때문에 코드를 읽는 사람이 헷갈린다.

### 위에서 아래로 코드 읽기
### 내려가기 규칙

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

## 서술적인 이름을 사용하라

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

흔히 함수에 인수 1개를 넘기는 단항 형식의 함수들은 다음과 같다. 

1. 조회 함수
   - 인수에 질문을 던지는 경우
   - **`boolean`** fileExists(`"MyFile"`)
2. 명령 함수
   - 인수를 뭔가로 변환해 결과를 반환하는 경우
   - **`InputStream`** fileOpen(`"MyFile"`)
3. 이벤트 함수
   - 이벤트 함수란 입력 인수로 시스템의 상태를 변경하는 경우
   - 이벤트 함수는 입력 인수만 있다. 출력 인수는 없다.

함수 이름을 지을 때는 두 경우를 분명히 구분하여 짓는다.
또한 언제나 일괄적인 방식으로 두 형식을 사용한다. (명령과 조회를 분리하라 참조)

```java
public class Member {
   // 회원의 접근 상태를 변경
   public void passwordAttemptFailedNTimes(int attempts) {
      failCount++;
      plusRetryAccessTime();
   }
}
```

이외 다른 유형의 단항 함수는 가급적 피한다.

예를들어 void includeSetupPageInfo(StringBuffer pageText) 같은 함수를 사용하지 말아라.

1. 리턴 타입을 보니 `void`다. 상태를 변경하는 이벤트 함수인가?
2. 아니면 메서드 명을 보니 변환 함수 같은데, 리턴 타입이 없다.
3. 무슨 혼종인가. 다음 함수는 나쁜 코드다.

이처럼 변환 함수에서 출력 인수를 사용하면 혼란을 가중시킨다.

- `StringBuffer includeSetupPageInfo(StringBuffer pageText)`

- 입력 인수를 변환하는 함수라면 변환 결과는 반환값으로 돌려준다.
- 입력 인수를 그대로 돌려주는 함수일지라도 변환 함수 형식을 따르는 편이 좋다. (적어도 변환 형태는 유지하기 때문.)

> 애초에 출력 인수에 대한 이슈가 논쟁되지 않도록, 클래스를 구성하는게 제일 좋다. 출력에 필요한 부분은 인스턴스 변수를 통해 구성해도 좋은 생각이다. <br/>
> - `void getHtmlPage()`

### 플래그 인수

플래그 인수는 추하다.
함수로 부울 값을 넘기는 관례는 정말로 끔찍하다.

`String getServerPublicUrl(boolean isSecureRequest)`

boolean 값을 넘기는 순간부터 여러 가지 작업을 수행하고 있다는 증거다.

### 이항 함수

단항 함수보다 이항 함수가 더 어렵다.
물론 이항 함수가 적절한 경우가 있다.

1. 관례적으로 직교 좌표계처럼 두 가지 값을 하나의 값으로 인지하고 있는 경우
   - _new Point(`1`, `2`)_
2. 두 인자에는 자연적인 순서가 존재한다.

 위 조건을 부합하지 않는 이항 함수는 깨끗한 코드가 아니다.
JUnit4의 `assertEquals(expected, actual)`가 대표적이다. 
두 인수엔 자연적인 순서가 존재하지 않기에 개발자의 실수를 유발 시킨다.

 이항 함수가 무조건 나쁘다는 소리는 아니다.
프로그램을 짜다보면 불가피한 경우도 생긴다. 그만큼 위험이 따른다는 사실을 인지하고 가능하면 단항 함수로 바꾸도록 애써야 한다.

### 상함 함수

함수의 인수가 늘어가면 `순서`, `주춤`, `무시`로 야기되는 문제가 두 배 이상 늘어난다.
특히 삼항 함수는 신중히 고려해야한다.

### 인수 객체

함수에 인수 개수가 2개 이상이 존재할 가능성이 있다면, 인수 객체를 고려해야 하다.

```java
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius);
```

혹여 객체를 생성해 인수를 줄이는 방법이 눈속임이라 여기는 개발자가 있다면 그렇지 않다고 말할 수 있다.
결국은 인수 객체를 구성하는 행위 역시 개념을 표현하는 수단이다.

### 인수 목록

때로는 인수 개수가 가변적인 함수(가변 인자, varargs)도 필요하다.
대표적인 예로 String.format 메서드가 있다.

```java
public String format(String format, Object... args);
```

실제로 String format 선언부를 살펴보면 이항 함수로 불 수 있다.

앞서 함수에서 이상적인 인수 개수는 무항이다.
단항 함수는 쉽고, 이항 함수는 어렵고, 삼항 함수는 더 어렵다.
이와 마찬가지로 함수에 가변 인자를 구성 시, 이미 인수 개수가 3개 이상이라면 문제가 된다.

```java
void triad(String name, int count, Integer... ars);
```

### 동사와 키워드

함수의 의도나 인수의 순서와 의도를 제대로 표현하려면 좋은 함수 이름이 필수다.

- 단항 함수는 함수와 인수가 동사/명사 쌍으로 이뤄야 한다.
  - `이름(name)`이 `무엇이든` `쓴다(write)`
  - write(name)
  - `이름(name)` `필드(field)`를 `쓴다(write)`
  - write`Field`(name)
- 인수 개수가 2개 이상인 함수는 이름에 키워드를 추가한다.
  - 인수 순서를 기억할 필요가 없어진다.
  - assert`Expected`Equals`Actual`(expected, actual)

## 부수 효과를 일으키지 마라

부수 효과는 거짓말이다.

본래 함수는 한 가지 일을 수행해야 한다.
함수 이름과 별개로 함수 내부에서 다른 작업을 수행한다면, 결국 추후에 예상치 못한 결과를 초래한다.

- 시간적인 결합(temporal coupling)
  - [wiki - Coupling (computer programming)](https://en.wikipedia.org/wiki/Coupling_(computer_programming))
  - [How to avoid temporal coupling in C#](https://www.infoworld.com/article/3239347/how-to-avoid-temporal-coupling-in-c-sharp.html)
  - [Forms of Temporal Coupling](https://www.pluralsight.com/tech-blog/forms-of-temporal-coupling/)
  - [Design Smell: Temporal Coupling](https://blog.ploeh.dk/2011/05/24/DesignSmellTemporalCoupling/)
- 순서 종속성(order dependency)
  - [Argument-Order Dependency](https://gist.github.com/evagabriela/6237082)
  - [How To Remove Argument Order Dependency In Ruby](https://mixandgo.com/learn/how-to-remove-argument-order-dependency-in-ruby)

```java
public class UserValidator {
   public void checkPassword(String username, String password) {
      User user = userRepository.findByUsername(username);
      if (user == null) {
         Session.initialize();
         // ^-- 부수적인 효과
         throw new NotFoundUserException();
      }
      // check user password logic...
   }
}
```

다음 코드에서 `Session.initialize();` 메서드 호출은 `부수 효과`다.

이런 부수 효과는 `시간적인 결합`을 초래한다. 
시간적인 결합은 혼란을 야기한다. 특히 이러한 부수 효과 코드가 숨겨질 경우 더더욱 혼란을 증폭시킨다.

만약 시간적인 결합이 필요하다면 함수 이름에 분명히 명시한다.

물론 함수가 `한 가지`만 한다는 규칙을 위반하지만, `checkPasswordIfUserNullAndInitializeSession`라는 이름이 훨씬 좋다.

### 출력 인수

일반적으로 인수를 함수 입력으로 해석한다.

- 보통 함수에 메시지를 보낸다.
- 인수 객체의 상태를 변경하지 않는다.

```java
// 오용 사례 - 출력 인수
public void appendFooter(StringBuffer report);
```

출력 인수는 저자를 주춤하게 한다. 인지적으로 거슬린다는 뜻이다.

출력 인수로 사용하라고 설계한 키워드가 바로 **`this`**다.
객체 지향 언어에서는 출력 인수를 사용할 필요가 거의 없다.

따라서 객체 인수의 상태를 변경하려면, 다음과 같이 호출 방식을 변경해야한다.

`report.appendFooter();`

## 명령과 조회를 분리하라

함수는 하나만 해야 한다.

- 객체 상태 변경
- 객체 정보 반환

```java
// 오용
if (set("username", "gmoon")) { ... }
```

다음 함수는 "set"이라는 단어가 동사인지 형용사인지 분간하기 어려워 코드만 봐선 의미가 모호하다.



## 오류 코드보다 예외를 사용하라

명령 함수에서 오류 코드를 반환하는 방식은 명령/조회 분리 규칙을 미묘하게 위반한다.

1. 오류 코드는 여러 중첩되는 코드를 생성되게 된다.
2. 코드가 도메인 로직과 오류 처리 코드로 뒤섞여 양분된다. 

```java
if (deletePage(page) == E_OK) { ... }
```

오류 코드 대신 예외를 사용하여 정상 동작과 요류 처리 동작을 분리해야한다.

```java
try {
  // Domain logic...
} catch (Exception e) {
  // Error logic...
  log.error(e.getMessage());
}
```

### Try Catch 블록 뽑아내기

`try/catch` 블록은 원래 추하다.

코드 구조에 혼란을 일으키며, 정상 동작과 오류 처리 동작을 뒤섞는다.
그러므로 **try/catch 블록을 별도 함수로 뽑아내는 편이 좋다.**

```java
public class HtmlUtil {
  public void delete(Page page) {
    try {
      deletePageAndAllReferences(page);
    } catch (Exception e) {
      logError(e);
    }
  }
  
  // 정상 동작, 도메인 함수 분리
  private void deletePageAndAllReferences(Page page) throws Exception {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
  }

  // 오류 처리, 에러 함수 분리
  private void logError(Exception e) {
    log.error(e.getMessage());
  }
}
```

### 오류 처리도 한 가지 작업이다

함수는 `한 가지` 작업만 해야한다. 
오류 처리도 `한 가지 오류만` 처리하는 함수로 구성한다.

함수에 키워드 try가 있다면 함수는 try문으로 시작해 catch/finally 문으로 끝나야 한다는 의미다.

### 의존성 자석

오류 코드를 정의하지 마라. 예외 클래스를 사용해야 한다.

오류 코드를 반환한다는 이야기는, 클래스든 열거형 변수든, 어디선가 오류 코드를 정의한다는 뜻이다.

```java
public enum Error {
  OK,
  INVALID,
  NO_SUCH,
  LOCKED,
  OUT_OF_RESOURCES,
  WAITING_FOR_EVENT;
}
```

위와 같은 클래스는 의존성 자석(dependency magnet)이다.

Error enum이 변한다면, Error enum을 사용하는 클래스 전부를 다시 컴파일하고 다시 배치해야한다. Error 클래스 변경이 어려워진다.

> - [What is a striking example of an unnecessary dependency magnet you have encountered as a programmer?](https://www.quora.com/What-is-a-striking-example-of-an-unnecessary-dependency-magnet-you-have-encountered-as-a-programmer)
> - [stackoverflow - How do you prevent classes becoming 'dependency magnets' and God classes?](https://stackoverflow.com/questions/900105/how-do-you-prevent-classes-becoming-dependency-magnets-and-god-classes)


결과적으로 의존성 자석들은 OCP 위반하게 된다.

오류 코드 대신 예외를 사용하면 새 예외는 Exception 클래스에서 파생되기 때문에, 재컴파일/재배치 없이 새 예외 클래스를 추가할 수 있다.

## 반복하지 마라

중복은 소프트웨어에서 모든 악의 근원이다.

- 코드 길이가 늘어난다.
- OCP 위반한다. 변경에 취약하다.

프로그래밍의 많은 원칙과 기법은 중복을 제거하거나 제어할 목적으로 나왔다. 

- DRY(Don't Repeat Yourself) 원칙
- 관계형 데이터베이스의 정규 형식
  - 자료에서 중복을 제거할 목적 
- 객체 지향 프로그래밍: 부모 클래스로 물아 중복을 없애거나, [디자인 패턴](https://ko.wikipedia.org/wiki/%EB%94%94%EC%9E%90%EC%9D%B8_%ED%8C%A8%ED%84%B4_(%EC%B1%85)) 유형 중 생성 패턴(Creational Patterns)이 중복 코드를 배제하기 위한 디자인 패턴이다.
- 구조적 프로그래밍
- AOP(Aspect Oriented Programming)
- COP(Component Oriented Programming)

## 구조적 프로그래밍

어떤 개발자는 에츠허르 데이크스트라 구조적 프로그래밍 원칙을 따른다.

> [데이크스트라 구조적 프로그래밍 원칙](https://ko.wikipedia.org/wiki/%EA%B5%AC%EC%A1%B0%EC%A0%81_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D) - 프로그램의 논리 구조는 제한된 몇 가지 방법만을 이용하여 비슷한 서브 프로그램들로 구성된다. 프로그램에 있는 각각의 구조와 그 사이의 관계를 이해하면 프로그램 전체를 이해해야 하는 수고를 덜 수 있어, SoC에 유리하다.

모든 함수와 함수 내 모든 블록에 입구(entry)와 출구(exit)가 하나만 존재해야 한다고 말한다.

1. 함수의 return 문은 하나여야 한다.
2. 루프 안에서 `break`/`continue`를 사용해선 안되며 `goto`는 절대로, 절대로 안 된다.

구조적 프로그래밍의 목표와 규율은 공감한다.
구조적 프로그래밍은 함수가 작다면 위 규칙은 별 이익을 제공하지 못한다. **함수가 아주 클 때만 상당한 이익을 제공한다.**

그러므로 간혹 return, break, continue를 여러 차례 사용해도 괜찮다.
때론 단일 입/출구 규칙(single entry-exit rule)보다 의도를 표현하기 쉬워진다.

goto 문은 큰 함수에서만 의미가 있으므로, 작은 함수에서는 피해야만 한다.

## 함수를 어떻게 짜죠

소프트웨어를 짜는 행위는 여느 글짓기와 비슷하다.

초안은 대개 서투르고 어수선하므로 원하는 대로 읽힐 때까지 다듬고, 고치고 문단을 정리한다.

코드도 마찬가지로 코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거한다. **이 모든 행위는 단위 테스트 기반에서 진행되어야 한다.**

## 결론

프로그래밍의 기술은 언제나 언어 설계의 기술이다.
함수는 프로그래밍 언어의 동사며, 클래스는 명사다.

대가(master) 개발자는 시스템을 (구현할) 프로그램이 아니라 풀어갈 이야기로 여긴다. 프로그래밍 언어라는 수단을 사용해 좀 더 풍부하고 좀 더 표현력이 강한 언어를 만들어 이야기를 풀어가야 한다.

진짜 목표는 시스템이라는 이야기를 풀어가는 데 있다는 사실을 명심하자.


