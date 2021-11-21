# 16장 SerialDate 리팩터링

[JCommon 라이브러리](https://www.jfree.org/jcommon/) 를 뒤져보면 **`org.jfree.date.SerialDate`** 라는 클래스가 있다.

이 장에서는 바로 이 `SerialDate`라는 클래스를 탐험한다.

`SerialDate`는 날짜를 표현하는 자바 클래스다. 
자바의 시간 기반 날짜 클래스보다 순수 날짜 클래스의 필요성을 느껴 `SerialDate`를 구현했다.

> `SerialDate` 를 구현한 사람은 데이비드 길버트(David Gilbert)다. <br/> 
> 데이비드는 확실히 숙련된 프로그래머로 엄청난 절제력과 전문가 정신을 보여준다. <br/>
> 의도와 목적을 고려한건대, `SerialDate`는 분명히 `우수한` 코드다.

- [GitHub - JCommon SerialDate](https://github.com/jfree/jcommon/blob/master/src/main/java/org/jfree/date/SerialDate.java)
- [GitHub - JCommon SerialDateTest](https://github.com/jfree/jcommon/blob/master/src/test/java/org/jfree/date/SerialDateTest.java)

## 목차

1. [첫째, 돌려보자](#돌려보자)
    - [코드 커버리지 분석 도구 활용](#코드-커버리지-분석-도구-활용)
    - [코드 커버리지를 위한 테스트 케이스](#코드-커버리지를-위한-테스트-케이스)
    - [경계 조건 오류](#경계-조건-오류)
    - [영원히 실행되지 않는 코드](#영원히-실행되지-않는-코드)
    - [예외를 사용하라](#예외를-사용하라)
2. [둘째, 고쳐보자](#고쳐보자)
    - [IDE에서 제공하는 리팩토링 도구를 활용해라](#ide에서-제공하는-리팩토링-도구를-활용해라)
    - [불필요한 주석 제거](#불필요한-주석-제거)
    - [javadoc 주석에서 사용되는 언어를 통일하자](#javadoc-주석에서-사용되는-언어를-통일하자)
    - [주석대신 의미있는 이름으로 표현하라](#주석대신-의미있는-이름으로-표현하라)
    - [static import 에만 와일드 카드 허용한다](#static-import-에만-와일드-카드-허용한다)
    - [serialVersionUID 자동 생성을 활용하자](#serialversionuid-자동-생성을-활용하자)
    - [추상화 수준을 고려한 클래스 이름](#추상화-수준을-고려한-클래스-이름)
    - [수학적 개념을 이용한 클래스 이름](#수학적-개념을-이용한-클래스-이름)
    - [Enum 클래스를 활용해라](#enum-클래스를-활용해라)
    - [클래스의 책임과 함수와 변수 이동](#클래스의-책임과-함수와-변수-이동)
    - [추상화 클래스에선 구현 정보를 다루지 마라](#추상화-클래스에선-구현-정보를-다루지-마라)
    - [접근제한자를 신경쓰자](#접근제한자를-신경쓰자)
    - [사용하지 않는 코드는 과감히 제거해라](#사용하지-않는-코드는-과감히-제거해라)
    - [final 키워드는 절재하자](#final-키워드는-절재하자)
    - [의미없는 중첩된 조건문은 통합한다](#의미없는-중첩된-조건문은-통합한다)
    - [불필요한 오버로딩 제거](#불필요한-오버로딩-제거)
    - [기능 욕심을 부린다면 분리해라](#기능-욕심을-부린다면-분리해라)
    - [실제로 테스트 코드에서만 사용되는 테스트 코드는 제거해라](#실제로-테스트-코드에서만-사용되는-테스트-코드는-제거해라)
    - [물리적 의존성과 논리적 의존성](#물리적-의존성과-논리적-의존성)
    - [정리해보자](#정리해보자)
3. [결론](#결론)

## 돌려보자

리팩토링에 앞서 **`코드 커버리지 분석 도구`** 를 활용하여, 테스트 코드가 대상 클래스의 기능들을 어느 정도 커버하는지 확인해봐야 한다.

### 코드 커버리지 분석 도구 활용

우선 `SerialDateTests` 테스트 케이스를 실행해보자.

모든 테이스 케이스는 실패하지 않는다고 하여, 모든 기능을 대변하고 있다는건 아니다.
`SerialDateTests`의 테스트 커버리지는 대략 50% 정도였다. 단위 테스트가 실행되지 않은 코드가 클래스 여기저기에 흩어져 있었다.

**`코드 커버리지 분석 도구를 통해 실행하는 코드와 실행하지 않는 코드를 파악한다.`**

클래스를 철저히 이해하고 리팩토링하려면, 훨씬 높은 테스트 커버리지가 필요하다.

> 코드 커버리지는 소프트웨어의 테스트를 논할 때, 얼마나 테스트 케이스를 충족하고 있는가를 나타내는 지표중 하나다. 

### 코드 커버리지를 위한 테스트 케이스

테스트의 코드 커버리지가 낮다면, 독자적으로 단위 테스트 케이스를 구현한다.

- 우선 실패한 테스트 케이스를 주석 처리한다.
  - 단. 리팩토링 후엔 모든 테이스 케이스가 통과해야 한다.
- 테스트 케이스를 추가하기 전에, 기존 함수의 기능을 파악한다.
  - 함수 이름을 추이하여 기능 구현의 구멍들을 보안한다.
  - 만약, 기존 코드를 수정하거나 부가 기능을 추가 하기에 애매하다면 주석으로 남겨둔다.
- 테스트 케이스를 추가한다.
  - 파악된 기능 시나리오를 테스트 케이스에 온전히 담으려 노력한다.

### 경계 조건 오류

흔히 **`경계 조건 코드`** 는 버그를 발생할 확률이 높다. 

- 경계 조건 코드는 한 곳에서 관리할 수 있도록 수정한다.
- 따라서 인수 테스트 시에 인수 값은 **경계 값으로 테스트한다.**

> 경계 조건이란 에러가 발생할 수 있는 기준점 혹은 경계가 되는 조건을 뜻한다.

```java
// Bad 경계 조건 -> "-1"
if (dayOfWeek -1 > targetWeekday) {}

// Best
int baseDayOfWeek = 1;
if (baseDayOfWeek >= targetWeekday) {}
```

### 영원히 실행되지 않는 코드

코드 커버리지 분석 도구를 활용하면 **`실행되지 않는 코드`** 를 파악하기 수월하다.

- 구현된 코드 중에, 절대적으로 특정 조건문이 타지 않아 영원히 코드가 실행되지 않는 경우가 발생한다. 
- 이러한 코드들은 기능을 구현한 뒤 시간이 흘러야 알 수 있다.

### 예외를 사용하라

**`에러 코드 또는 메시지 보단 예외를 사용해라`**

예전에 예외를 지원하지 않는 System call 형태의 함수들의 성공 여부 반환 값은 다음과 같이 정의한다.

- **`1`** : 성공
- **`0`** : 정상 종료
- **`-1`** : 예외 발생

하지만 호출한 함수쪽에선 반환된 에러 코드나, 메서지를 확인하는 코드가 들어가게 됨으로 오류가 발생하면 예외를 던지는 편이 났다.

```java
public abstract class SerialDate {
  public static String weekInMonthToString(final int count) {
    switch (count) {
      case SerialDate.FIRST_WEEK_IN_MONTH : return "First";
      case SerialDate.SECOND_WEEK_IN_MONTH : return "Second";
      case SerialDate.THIRD_WEEK_IN_MONTH : return "Third";
      case SerialDate.FOURTH_WEEK_IN_MONTH : return "Fourth";
      case SerialDate.LAST_WEEK_IN_MONTH : return "Last";
      default :
        throw new IllegalArgumentException("SerialDate.weekInMonthToString(): invalid code.");
    }
  }
}
```

> 참고로 UNIX 시스템 함수의 반환 값을 성공/실패만 다룰 경우, 0을 성공, -1 에러를 뜻한다. 이외에도 일반적으로 -1 은 다룰 수 없는 값을 뜻하기도 하다.

## 고쳐보자

별도로 언급은 하지 않았지만, 앞서 코드를 고칠때마다 단위 테스트를 실행하며 진행했다.

### IDE에서 제공하는 리팩토링 도구를 활용해라

가급적이면 IDE에서 제공하는 리팩토링 도구를 활용하여 이름을 변경한다거나, 함수 이동, 클래스 추출 등 리팩토링 기능을 적극 활용한다.

### 불필요한 주석 제거

불필요한 주석은 제거한다.

- **`좋은 주석`**
  - 법적인 정보, 라이센스 정보, 저작권을 다루는 주석은 보존한다.
- **`나쁜 주석`**
  - 반면, 소스 변경 이력 정보를 다루는 주석은 제거한다.
    - 주석보단 소스 코드 제어 도구를 활용해라
  - 설명이 부실한 주석은 코드만 복잡하게 만든다. 제거한다.

### javadoc 주석에서 사용되는 언어를 통일하자

javadoc 주석에서 사용되는 언어가 다양하면 모양새를 제대로 맞추기 어렵다.

- 굳이 javadoc 주석을 사용한다면 `<pre>` 태그로 감싸는 편이 좋다.
- 그러면 소스 코드에 보이는 형식이 javadoc에 그대로 유지된다.

### 주석대신 의미있는 이름으로 표현하라

주석은 불필요하다.

의도는 좋았으나 개발자가 딱 맞을 정도로 엄밀하게 주석을 달지 못하기도 한다. 불필요한 주석은 거짓말과 잘못된 정보가 쌓이기 좋은 곳이다. **주석과 기타 유사한 주석은 제거한다.**

주석을 작성하기 보다는 의미있는 함수, 변수 이름을 작명한다.

```java
public static final int EARLIEST_DATE_ORDINAL = 2;
public static final int LATEST_DATE_ORDINAL = 2958465;
```

### static import 에만 와일드 카드 허용한다

**`static import`** 에만 와일드 카드 허용한다.

```java
// Bad
import java.util.*;

// Best
import java.util.List;
import java.util.ArrayList;

import static org.assertj.core.api.Assertions.*;
```

### serialVersionUID 자동 생성을 활용하자

`serialVersionUID` 변수는 직렬화(serializer)를 제어한다.

- 이 변수가 변경되면 이전 소프트웨어 버전에서 직렬화한 객체를 더 이상 인식하지 못한다.
- 즉, 이전 버전에서 직렬화한 클래스를 복원하려 시도하면 `InvalidClassException` 예외가 발생한다.

`serialVersionUID` 변수를 선언하지 않으면 컴파일러가 자동으로 생성한다.<br/>
리팩토링 단계에선, 잠시 컴파일러에게 생성을 맡겨 자동 제어할 수 있도록 한다.

### 추상화 수준을 고려한 클래스 이름

클래스 이름은 서술적이고 추상화 수준을 고려하여 작성한다.

SerialDate 클래스 이름은 오해의 소지가 다분하다. Serializable에서 파생된 클래스인가? 아니다. 일련번호(serial number)를 사용해 클래스를 구현했기 때문이다.

- 서술적인 이름을 선택하자.
  - `일련번호` 라는 용어는 정확하지 못하다.
  - 좀 더 서술적인 용어를 선택하여 표현하라.
- 추상화 수준을 고려하여 작명한다.
  - SerialDate 클래스는 추상화 클래스다.
  - Serial 이라는 구현을 암시하는 용어를 선택했다.
  - 이름에 구현을 암시하는 용어를 선택하여 독자를 햇갈리게 만든다.
  - 추상화 클래스는 구현을 숨긴다. 즉 구현을 암시하는 용어를 선택하지 않는다.
- 되도록 표준으로 사용하고 있는 클래스 이름은 피한다.
  - JDK에 포함된 클래스 명이나, 유명한 라이브러리의 클래스 명은 되도록 피하자.
  - 독자가 햇갈릴 수 있는 오해의 소지가 있다.

### 수학적 개념을 이용한 클래스 이름

- 클래스 이름은 수학적 개념을 사용하면 좋다.
    - 때론, 수학적 명칭이 의도를 더 분명하게 표현한다.
    - 예를 들어 범위 끝 날짜를 범위에 포함할지 여부 책임의 클래스 이름을 짓는다고 가정하자.
    - `개구간(open interval`, `반개구간(half-open interval)`, `폐구간(closed interval)` 수학적 개념을 착안해 `DateInterval`로 결정했다.

### Enum 클래스를 활용해라

상수 집합을 다루는 클래스는 피한다.

옛날 자바 프로그래머가 많이 쓰던 기교다. 바람직하다고 보기 어렵다.

```java
// 상수 집합 클래스를 구현한 SerialDate
public abstract class SerialDate implements Comparable,
        Serializable,
        MonthConstants {
  // ...
}

// 상수 집합 클래스
public interface MonthConstants {
  public static final int JANUARY = 1;
  public static final int FEBRUARY = 2;
  public static final int MARCH = 3;
  // ...
  public static final int OCTOBER = 10;
  public static final int NOVEMBER = 11;
  public static final int DECEMBER = 12;
}
```

`MonthConstants`를 enum으로 변경하면 오류 검사 코드를 제거할 수 있다. 유효한지 값이 체크하던 `isValidMonthCode` 함수를 제거할 수 있다.

```java
public abstract class SerialDate implements Comparable,
        Serializable {
  
  public static enum Month {
    JANUARY(1),
    FEBRUARY(2),
    MARCH(3),
    // ...
    OCTOBER(10),
    NOVEMBER(11),
    DECEMBER(12);
    
    private final int value;
    // ...
  }
}
```

### 클래스의 책임과 함수와 변수 이동

- 함수와 변수는 실제 사용하는 클래스로 이동시킨다.
- 만약 공통적으로 사용될 여지가 있다면?
  - 미리 공통적으로 사용할지 여부 고려하지 않는다.
  - 현재 사용하는 클래스가 하나라면, 직접 사용하고 있는 클래스로 이동시킨다.

사실 SerialDate 클래스에서 `EARLIEST_DATE_ORDINAL` 상수는 사용하지 않는다.

```java
public abstract class SerialDate {
  public static final int EARLIEST_DATE_ORDINAL = 2;
  public static final int LATEST_DATE_ORDINAL = 2958465;
}
```

SerialDate에서 파생된 `SpreadsheetDate`에서 사용함으로 클래스로 옮긴다.

```java
public class SpreadsheetDate extends SerialDate {
  public static final int EARLIEST_DATE_ORDINAL = 2;
  public static final int LATEST_DATE_ORDINAL = 2958465;
}

public abstract class SerialDate { }
```

### 추상화 클래스에선 구현 정보를 다루지 마라

추상화 클래스에서 실제 구현 대상 클래스를 명시하지 않는다.

- 추상 클래스는 구체적인 구현 정보를 포함할 필요가 없다.
- 기반 클래스(base class, 부모 클래스)는 파생 클래스(derivative, 자식 클래스)를 몰라야 바람직하다.

[Abstract factory 패턴](https://refactoring.guru/design-patterns/abstract-factory) 을 적용해 구체적인 팩토리 클래스를 통해 파생 클래스를 생성할 수 있도록 설계한다. 

```java
public abstract class DayDateFactory {
  private static DayDateFactory factory = new SpreadsheetDateFactory();
  public static void setInstance(DayDateFactory factory) {
    DayDateFactory.factory = factory;
  }
  
  // override...
  protected abstract DayDate makeDate(int ordinal);
  protected abstract DayDate makeDate(int day, int month, int year);
  
  public static DayDate makeDate(int ordinal) {
    return factory.makeDate(ordinal);
  }
  
  public static DayDate makeDate(int day, int month, int year) {
    return factory.makeDate(day, month, year);
  }
}
```

### 접근제한자를 신경쓰자

다른 곳에서 사용하지 않는 함수, 변수의 접근제한자는 공개하지 마라.

```java
// Bad
public static final int LAST_DAY_OF_MONTH = 12;

// Best
private static final int LAST_DAY_OF_MONTH = 12;
```

### 사용하지 않는 코드는 과감히 제거해라

사용하지 않는 함수, 변수는 소스 코드에서 과감히 제거한다.

독자의 오해할 소지가 다분하다.

### final 키워드는 절재하자

인수와 변수 선언에서 `final` 키워드는 모두 제거했다. 

- 몇 군데를 제하면, 실질적인 가치는 없으면서 코드만 복잡하게 만든다.
- final 키워드를 남발한 소스 코드는 오히려 악효과다.

> 반면, 로버트 시몬스는 "코드 전체에 `final`을 사용하라..."고 강력히 권장한다.

### 의미없는 중첩된 조건문은 통합한다

의미없이 중첩된 조건문은 `&&` 또는 `||` 연산자를 이용해 통합한다.

```java
// Bad
public class LadderGame {
    public boolean isDrewLine(){
      if (isFirstLine()){
        if (emptyPreviousLine()) {
          return true;
        }
      }
      
      return false;
    } 
}

// Best
public class LadderGame {
  public boolean isDrewLine(){
    if (isFirstLine() && emptyPreviousLine()){
      return true;
    }
    return false;
  }
}
```

### 불필요한 오버로딩 제거

클래스 내부에서 불필요한 오버로딩은 제거하여 단순화한다.

아래 `getMonths(final boolean shortened)`를 호출하는 코드는 getMonths() 함수 뿐이다.

따라서 두 메서드를 합쳐 코드를 단순화한다.

```java
public abstract class SerialDate {
  public static String[] getMonths() {
    return getMonths(false);
  }

  // 해당 함수를 호출하는 코드는 위 `getMonths()` 함수 뿐이다.
  public static String[] getMonths(final boolean shortened) {
    if (shortened) {
      return DATE_FORMAT_SYMBOLS.getShortMonths();
    }
    
    return DATE_FORMAT_SYMBOLS.getMonths();
  }
}

public abstract class SerialDate {
  public static String[] getMonths() {
    boolean shortened = false;
    if (shortened) {
      return DATE_FORMAT_SYMBOLS.getShortMonths();
    }

    return DATE_FORMAT_SYMBOLS.getMonths();
  }
}
```

### 기능 욕심을 부린다면 분리해라

리팩토링을 진행하다 보면 특정 메서드가 **`기능 욕심(feature envy)`** 을 부리는 경우가 생긴다.

기능 욕심이란 메소드가 자신이 속한 클래스보다 다른 클래스에 관심을 가지고 있는 경우를 뜻한다.

가장 흔한 욕심이 데이터에 대한 욕심이다. 그 메소드는 분명히 다른 곳에 있고 싶은 것이고, 따라서 **`Move Method`** 를 사용한다.

때로는 메소드의 특정 부분만 이런 욕심으로 고통 받는데, 이럴 때는 욕심이 많은 부분에 대해서 **`Extract Method`** 를 사용한 다음 적절한 위치로 옮겨주기 위해 **`Move Method`** 를 사용한다.

만약 내부 클래스에 대해 관심을 부린다면, 내부 클래스가 커졌다고 의심해 볼만하다. 내부 클래스가 커진다면 분리한다. 내부 클래스가 커진다면 일관성을 유지하도록 분리해 독자적인 원시 파일로 구성한다.

### 실제로 테스트 코드에서만 사용되는 테스트 코드는 제거해라

구현하다 보면 특정 메서드가 테스트 코드에서만 사용될 경우가 있다.

이런 테스트 코드는 제거한다.

### 물리적 의존성과 논리적 의존성

뭔가가 구현에 논리적으로 의존한다면 물리적으로도 의존해야 마땅하다.

예를 들어 아래 `getDayOfWeek` 메서드는 DayDate 추상화 클래스와 물리적으로 의존성은 없다. 하지만 논리적 의존성이 존재한다.

```java
public class SpreadsheetDate extends DayDate {
  // 물리적 의존성은 없지만 논리적 의존성이 존재한다.
  public Day getDayOfWeek() {
    // ... logic
  }
}
```

이런 논리적 의존성은 불편하다. 즉 getDayOfWeek 메서드는 논리적임으로 DayDate 클래스로 올려야 한다.

```java
public class SpreadsheetDate extends DayDate {
  // 추상화 메서드 구현
  @Override
  public Day getDayOfForOrdinalZero() {
    return Day.SUNDAY;
  }
}

public abstract class DayDate {
  public Day getDayOfWeek() {
    Day startingDay = getDayOfForOrdinalZero();
    int startingOffset = startingDay.index = Day.SUNDAY.index;
    return Day.make(( getOrdinalDay() + startingOffset ) % 7 + 1);
  }
  
  public abstract Day getDayOfForOrdinalZero();
}
```

### 정리해보자

마지막으로 리팩토링 과정을 정리해보자.

1. 너무 오래된 주석은 제거하거나 개선한다.
2. enum을 모두 독자적인 소스 파일로 옮긴다.
3. 정적 변수와 정적 메서드를 DateUtils이라는 새 클래스로 옮겼다.
4. 일부 추상 메서드를 DayDate 클래스로 끌어 올렸다.
5. enum type 을 적극 활용했다.
6. 구현 중복은 제거했다.
   - 기존 `plusYears`, `plushMonths` 메서드는 구현 중복이 존재했다
   - `correctLastDayOfMonth` 라는 새 메서드를 생성해 중복을 없앴다.
7. 경계 조건을 한곳에서 관리했다.
   - 팔방미인으로 사용하던 숫자 1을 enum을 활용하여 없앴다.

흥미롭게도 코드 커버리지는 84.9%로 감소했다.

테스트하는 코드가 줄어서가 아니라 클래스 크기가 작아지는 바람에 테스트하지 않는 코드의 비중이 커졌기 때문이다. 테스트하지 않은 코드는 너무 사소해 테스트할 필요도 없다.

## 결론

다시 한 번 우리는 보이스카우트 규칙을 따랐다.

- 체크아웃한 코드보다 좀 더 깨끗한 코드를 체크인하게 되었다.
- 시간은 걸렸지만 가치 이쓴ㄴ 작업이었다.
- 테스트 커버리지가 증가했으며, 버그 몇 개를 고쳤으며, 코드 크기가 줄었고, 코드가 명확해졌다.

다른 사람은 우리보다 코드를 좀 더 쉽게 이해하리라. 그래서 우리보다 코드를 좀 더 쉽게 개선하리라.
