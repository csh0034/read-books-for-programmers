# 15장 JUnit 들여다보기

**`JUnit`** 은 자바 프레임워크 중에서 가장 유명하다.

개념은 단순하며 정의는 정밀하고 구현은 우아하다. 하지만 실제 코드는 어떨까?

이 장에서는 JUnit 프레임워크에서 가져온 코드를 평가한다.

## 목차

1. [JUnit 프레임워크](#junit-프레임워크)
   - [접두어 제거](#접두어-제거)
   - [조건문을 캡슐화하자](#조건문은-캡슐화하자)
   - [의미있는 지역 변수](#의미있는-지역-변수)
   - [부정 조건을 피하라](#부정-조건을-피하라)
   - [의미있는 함수 이름](#의미있는-함수-이름)
   - [함수를 작게 유지하라](#함수를-작게-유지하라)
   - [함수의 사용 방식은 일관성 있게 하라](#함수의-사용-방식은-일관성-있게-하라)
   - [숨겨진 시간적인 결합](#숨겨진-시간적인-결합)
   - [함수 호출 순서](#함수-호출-순서)
   - [경계 조건은 캡슐화 하라](#경계-조건은-캡슐화-하라)
2. [결론](#결론)

## JUnit 프레임워크

초기 Junit은 켄트 벡과 에릭 감마(Erich Gamma) 두 사람이 함께 아틀란타 행 비행기를 타고 가다 Junit 기반을 다졌다.

아래 Junit 의 ComparisonCompactor 클래스는 문자열 비교 오류를 파악할 때 유용한 코드다. 비록 저자들이 모듈을 아주 좋은 상태로 남겨두었지만, 몇 가지 개선해야할 부분이 보인다.

**`보이스카우트 규칙`** 에 따라 우리는 처음 왔을 때보다 더 깨끗하게 해놓고 떠나보자.

- [접두어 제거](#접두어-제거)
- [조건문을 캡슐화하자](#조건문은-캡슐화하자)
- [의미있는 지역 변수](#의미있는-지역-변수)
- [부정 조건을 피하라](#부정-조건을-피하라)
- [의미있는 함수 이름](#의미있는-함수-이름)
- [함수를 작게 유지하라](#함수를-작게-유지하라)
- [함수의 사용 방식은 일관성 있게 하라](#함수의-사용-방식은-일관성-있게-하라)
- [숨겨진 시간적인 결합](#숨겨진-시간적인-결합)
- [함수 호출 순서](#함수-호출-순서)
- [경계 조건은 캡슐화 하라](#경계-조건은-캡슐화-하라)

```java
package junit.framework;

public class ComparisonCompactor {
  private static final String ELLIPSIS = "...";
  private static final String DELTA_END = "]";
  private static final String DELTA_START = "[";
  private int fContextLength;
  private String fExpected;
  private String fActual;
  private int fPrefix;
  private int fSuffix;

  public ComparisonCompactor(int contextLength, String expected, String actual) {
    fContextLength = contextLength;
    fExpected = expected;
    fActual = actual;
  }

  public String compact(String message) {
    if (fExpected == null || fActual == null || areStringsEqual()) {
      return Assert.format(message, fExpected, fActual);
    }
    findCommonPrefix();
    findCommonSuffix();
    String expected = compactString(fExpected);
    String actual = compactString(fActual);
    return Assert.format(message, expected, actual);
  }

  private String compactString(String source) {
    String result = DELTA_START + source.substring(fPrefix, source.length() - fSuffix + 1) + DELTA_END;
    if (fPrefix > 0) {
      result = computeCommonPrefix() + result;
    }
    if (fSuffix > 0) {
      result = result + computeCommonSuffix();
    }
    return result;
  }

  private void findCommonPrefix() {
    fPrefix = 0;
    int end = Math.min(fExpected.length(), fActual.length());
    for (; fPrefix < end; fPrefix++) {
      if (fExpected.charAt(fPrefix) != fActual.charAt(fPrefix)) {
        break;
      }
    }
  }

  private void findCommonSuffix() {
    int expectedSuffix = fExpected.length() - 1;
    int actualSuffix = fActual.length() - 1;
    for (; actualSuffix >= fPrefix && expectedSuffix >= fPrefix; actualSuffix--, expectedSuffix--) {
      if (fExpected.charAt(expectedSuffix) != fActual.charAt(actualSuffix)) {
        break;
      }
    }
    fSuffix = fExpected.length() - expectedSuffix;
  }

  private String computeCommonPrefix() {
    return ( fPrefix > fContextLength ? ELLIPSIS : "" ) + fExpected.substring(Math.max(0, fPrefix - fContextLength), fPrefix);
  }

  private String computeCommonSuffix() {
    int end = Math.min(fExpected.length() - fSuffix + 1 + fContextLength, fExpected.length());
    return fExpected.substring(fExpected.length() - fSuffix + 1, end) + ( fExpected.length() - fSuffix + 1 < fExpected.length() - fContextLength ? ELLIPSIS : "" );
  }

  private boolean areStringsEqual() {
    return fExpected.equals(fActual);
  }
}
```

### 접두어 제거

가장 먼저 눈에 거슬리는 부분은 멤버 변수 앞에 붙인 접두어 f 다.

- 접두어를 포함한 네이밍은 구닥다리 코드라는 징표다.
- 오늘날 사용하는 개발 환경에서는 이처럼 변수 이름에 범위를 명시할 필요가 없다.

접두어 f는 중복되는 정보다. 그러므로 접두어 f를 모두 제거하자.

```java
public class ComparisonCompactor {
   private int contextLength;
   private String expected;
   private String actual;
   private int prefix;
   private int suffix;
   // ...
}
```

## 조건문은 캡슐화하자

다음으로 compact 함수 시작부에 **`캡슐화 되지 않은 조건문`** 이 보인다.

```java
public class ComparisonCompactor {
   public String compact(String message) {
      // 캡슐화 되지 않은 조건문
      if (expected == null || actual == null || areStringEqual()) {
         return Assert.format(message, expected, actual);
      }
      
      // ...
   }
}
```

조건문을 캡슐화하여 의도를 명확히 표현해야 한다. 

즉, **`조건문을 메서드로 뽑아내 적절한 이름을 붙인다.`**

```java
public class ComparisonCompactor {
   public String compact(String message) {
      if (shouldNotCompact()) {
         return Assert.format(message, expected, actual);
      }

      // ...
   }

   // 메서드로 캡슐화한 조건문
   private boolean shouldNotCompact() {
      return expected == null || actual == null || areStringEqual();
   }
}
```

### 의미있는 지역 변수

함수의 지역 변수 이름은 맴버 변수 이름 그대로 사용하지마라.

compact 함수에서 사용하는 지역 변수 `expected`, `actual` 이름도 눈에 거슬린다.

이름을 명확하게 붙여 표현한다.

```java
public class ComparisonCompactor {
   public String compact(String message) {
      // ...
//      String expected = compactString(this.expected);
//      String actual = compactString(this.actual);
      String compactExpected = compactString(this.expected);
      String compactActual = compactString(this.actual);
      return Assert.format(message, compactExpected, compactActual);
   }
}
```

### 부정 조건을 피하라

부정문은 긍정문 보다 이해하기 더 어렵다. 긍정 조건으로 표현하라.

```java
public class ComparisonCompactor {
   public String compact(String message) {
      if (shouldNotCompact()) {
         return Assert.format(message, expected, actual);
      }

      findCommonPrefix();
      findCommonSuffix();
      String compactExpected = compactString(this.expected);
      String compactActual = compactString(this.actual);
      return Assert.format(message, compactExpected, compactActual);
   }

   private boolean shouldNotCompact() {
      return expected == null || actual == null || areStringEqual();
   }
}
```

그러므로 첫 문장 if를 긍정으로 만들어 조건문을 반전한다.

```java
public class ComparisonCompactor {
   public String compact(String message) {
      if (canBeCompact()) {
         findCommonPrefix();
         findCommonSuffix();
         String compactExpected = compactString(this.expected);
         String compactActual = compactString(this.actual);
         return Assert.format(message, compactExpected, compactActual);
      }
      return Assert.format(message, expected, actual);
   }

   private boolean canBeCompact() {
      return expected != null  && actual != null && !areStringEqual();
   }
}
```

### 의미있는 함수 이름

함수 이름이 이상하다. 

- 문자열을 압축하는 함수라지만 실제로 `canBeCompacted` 가 `false`이면 압축하지 않는다.
- 함수에 `compact` 라는 이름을 붙이면 오류 점검이라는 부가 단계가 숨겨진다.

따라서 실제로는 `formatCompactedComparison` 이름이 적합하다. 이제 if 문 안에서는 예상 문자열과 실제 문자열을 진짜로 압축한다.

### 함수를 작게 유지하라

함수를 짜다 보면 한 함수 안에다 여러 단락을 이어서 일련의 작업을 수행하고픈 유혹에 빠진다.

`canBeCompact` 조건문안에 코드 맥락은 명시적으로 들어나지 않는다. 이런 함수는 한가지만 수행 하는 함수가 아니다. 한가지만 수행하는 작은 함수로 나눠야 된다.

따라서 별도의 함수로 추출하여 의미 있는 이름으로 함수를 작게 유지한다.

```java
public class ComparisonCompactor {
   public String formatCompactedComparison(String message) {
      if (canBeCompact()) {
         findCommonPrefix();
         findCommonSuffix();
         String compactExpected = compactString(this.expected);
         String compactActual = compactString(this.actual);
         return Assert.format(message, compactExpected, compactActual);
      }
      return Assert.format(message, expected, actual);
   }
}
```

- if 문 안에서는 예상 문자열과 실제 문자열을 진짜로 압축한다.
- 이 부분을 빼내어 `compactExpectedAndActual` 이라는 메서드로 만든다.
- 하지만 형식을 맞추는 작업은 `formatCompactedComparison` 메서드로 만든다.
- `compactExpectedAndActual`은 압축만 수행한다.

```java
public class ComparisonCompactor {
   public String formatCompactedComparison(String message) {
      if (canBeCompact()) {
         compatExpectedAndActual();
         return Assert.format(message, compactExpected, compactActual);
      }
      return Assert.format(message, expected, actual);
   }

   private void compatExpectedAndActual() {
      findCommonPrefix();
      findCommonSuffix();
      this.compactExpected = compactString(this.expected);
      this.compactActual = compactString(this.actual);
   }
}
```

위에서 `compactExpect`와 `compactActual`을 멤버 변수로 승격했다는 사실에 주의한다.

### 함수의 사용 방식은 일관성 있게 하라 

`compatExpectedAndActual()` 함수 사용 방식이 일관적이지 못하다.

- 새 함수에서 마지막 두줄은 변수를 반환하지만
- 첫째 줄과 둘째 줄은 반환값이 없다.

```java
private void compatExpectedAndActual() {
   findCommonPrefix();
   findCommonSuffix();
   this.compactExpected = compactString(this.expected);
   this.compactActual = compactString(this.actual);
}
```

다음과 같이 함수의 사용 방식을 일관성 있게 변경하자.

```java
private void compatExpectedAndActual() {
   this.prefixIndex = findCommonPrefix();
   this.suffixIndex = findCommonSuffix();
   this.compactExpected = compactString(this.expected);
   this.compactActual = compactString(this.actual);
}
```

- 멤버 변수 이름도 좀 더 정확하게 바꿨다.
  - `prefix` -> **`prefixIndex`**
  - `suffix` -> **`suffixIndex`**

### 숨겨진 시간적인 결합

`findCommonSuffix`를 주의 깊게 살펴보면 숨겨진 **`시간적인 결합(hidden temporal coupling)`** 이 존재한다.

- `findCommonSuffix` 는 `findCommonPrefix` 계산에 의존한다.
- 다시 말해, `findCommonSuffix` 는 `prefixIndex`를 계산한다는 사실에 의존한다.
- 따라서 반드시 `findCommonSuffix` 호출은 `findCommonPrefix` 메서드를 먼저 호출한 뒤 호출해야 한다.

```java
private int findCommonSuffix() {
  int expectedSuffix = expected.length() - 1;
  int actualSuffix = actual.length() - 1;
  for (; actualSuffix >= prefixIndex && expectedSuffix >= prefix; actualSuffix--, expectedSuffix--) {
     if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix)) {
        break;
     }
  }
  return expected.length() - expectedSuffix;
}
```

그래서 시간 결합을 외부에 노출하고자 `findCommonSuffix`를 고쳐 `prefixIndex`를 인수로 넘겼다.

```java
private void compatExpectedAndActual() {
    this.prefixIndex = findCommonPrefix();
    this.suffixIndex = findCommonSuffix(this.prefixIndex);
    this.compactExpected = compactString(this.expected);
    this.compactActual = compactString(this.actual);
}
```

### 함수 호출 순서

사실 이 방법은 썩 내키진 않는다.

- `prefixIndex`를 인수로 전달하는 방식은 다소 자의적이다.
- 함수 호술 순서는 확실히 정해지지만, 인수가 필요한 이유는 설명하지 못한다.
- 코드 맥락 상 `prefixIndex`가 필요한 이유가 분명히 드러나지 않는다.

코드를 원래대로 돌려 놓고, findCommonPrefixAndSuffix()를 추가하자.

두 함수를 호출하는 순서대로 배치한다. 앞서 고친 코드보다 훨씬 더 분명해진다.

```java
public class ComparisonCompactor {
   private void compatExpectedAndActual() {
      findCommonPrefixAndSuffix();
      this.compactExpected = compactString(this.expected);
      this.compactActual = compactString(this.actual);
   }

   private void findCommonPrefixAndSuffix() {
     findCommonPrefix();
     
     // findCommonSuffix() logic...
   }
}
```

코드가 훨씬 나아졌다.

### 경계 조건은 캡슐화 하라

경계 조건은 빼먹거나 놓치기 십상이다. 경계 조건은 한곳에서 별도로 처리한다.

예를 들어 아래 코드 `suffixIndex +1` 은 경계 조건이다.

```java
public class ComparisonCompactor {
   private String compactString(String source) {
      String result = DELTA_START
              // 경계 조건 = suffixIndex +1
              + source.substring(prefixIndex, source.length() - suffixIndex +1)
              + DELTA_END;
      // ...
      return result;
   }
}
```

- `+1`이라는 경계 조건은 여러 코드에서 남발된다.
- `suffixIndex` 이름도 `suffixLength` 가 더 적합함으로 수정한다.

다음과 같이 경계 조건은 한 곳에서 처리할 수 있도록 코드를 변경한다.

- **`private int suffixLength = 1;`**

```java
public class ComparisonCompactor {
   // 경계 조건은 멤버 변수로 선언하여 한 곳에서 처리하도록 한다.
   private int suffixLength = 1;
   
   private String compactString(String source) {
      String result = DELTA_START
              + source.substring(prefixIndex, source.length() - suffixLength)
              + DELTA_END;
      // ...
      return result;
   }
}
```

리팩토링한 모듈은 일련의 분석 함수와 일련의 조합 함수로 나뉜다.

함수내 모든 추상화는 수준이 동일하며, 함수는 추상화 수준 한 단계만 내려가도록 배치했다. <br/>
결과적으로 전체 함수는 위상적으로 정렬했으므로 각 함수가 사용된 직후에 정의된다.

리팩토링하는 과정에서 초반에 내렸던 결정 일부를 번복했다는 사실을 눈치채라. 

흔히 생기는 일이다. 코드를 리팩토링 하다 보면 원래 했던 변경을 되돌리는 경우가 흔하다. <br/> 
리팩토링은 코드가 어느 수준에 이를 때까지 수많은 **시행착오를 반복하는 작업이기 때문이다.**

## 결론

- 보이스 스카우트 규칙을 적용해 모듈은 처음보다 조금 더 깨끗해졌다. 
- 원래 깨끗하지 못했다는 말은 아니다.
- **`하지만 세상에 개선이 불필요한 모듈은 없다.`**
- 코드를 처음보다 조금 더 깨끗하게 만드는 책임은 우리 모두에게 있다.