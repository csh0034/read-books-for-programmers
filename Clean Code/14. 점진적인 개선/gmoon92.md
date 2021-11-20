# 14장 점진적인 개선

이 장은 점진적인 개선을 보여주는 사례 연구다.

출발은 좋았으나 **`확장성`** 이 부족했던 모듈을 소개한다.

**`명령행 인수`** 와 **`구문`** 을 분석하기 위해 직접 구현한 **`Args`** 유틸 클래스 사례를 통해, 일반적으로 리팩토링할 시 발생되는 문제점과 해결 방법에 대해 소개한다.

## 목차

1. [Args 구현](#args-구현)
    - [어떻게 짰느냐고?](#어떻게-짰느냐고)
2. [Args: 1차 초안](#1차-초안)
    - [그래서 멈췄다.](#그래서-멈췄다)
    - [점진적으로 개선한다.](#점진적으로-개선한다)
3. [String 인수](#string-인수)
4. [결론](#결론)

## Args 구현

우선 코드 컨벤션과 구조에 신경을 쓰며 코드를 작성한다.

- 이름을 붙인 방법
- 함수 크기
- 코드 형식

1. 위에서 아래로 코드가 읽히도록 코드를 작성한다.
2. 의미 있는 이름으로 의사를 표현하도록 노력한다.

### 어떻게 짰느냐고

프로그래밍은 과학보다 **`공예(craft)`** 에 가깝다는 사실이다. 깨끗한 코드를 짜려면 먼저 지저분한 코드를 짠 뒤에 정리해야 한다는 의미다.

잘 짠 코드는 이야기처럼 읽힌다. 따라서 프로그래밍은 글쓰기처럼 작성하는게 중요하다.

맨 처음 글 작문할 때, 초안부터 작성한다. 먼저 1차 초안을 쓰고, 그 초안을 고쳐 2차 초안을 만들고, 계속 고쳐 최종안을 만든다. **깔끔한 작품을 내놓으려면 단계적으로 개선해야 한다.**

대다수 신참 프로그래머는 이 충고를 충실히 따르지 않는다. 

그들은 무조건 돌아가는 프로그램을 목표로 잡는다. 일단 프로그램이 '돌아가면' 다음 업무로 넘어간다. '돌아가는' 프로그램은 그 상태가 어떻든 그대로 버려둔다.

반면 경험이 풍부한 전문 프로그래머라면 이런 행동이 전문가로서 자살 행위라는 사실을 잘 안다.

## 1차 초안

사실 **`1차 초안`** 은 낯 뜨거운 표현이다. 명백히 미완성이다.

- 인스턴스 변수 개수가 많으면 안된다.
  - 인스턴수 변수가 많다는 의미는 클래스의 책임이 많은건 아닌지 생각해 볼만하다.
- try-catch-catch 블록은 지저분한 코드다.
  - 비즈니스 논리와 오류 처리 로직은 분리한다.
- 함수 이름, 변수 이름을 선택하는 방식이 어설프다.

위 처럼 아쉬운 부분도 있지만, 대부분의 1차 초안 코드는 간결하고 단순하며 이해하기도 쉽다. 

그렇다하여 미완성된 코드를 그대로 두면 계속해서 그 위에 코드가 얹혀지게 되며 시간이 갈수록 코드가 점차 지저분해지는 이유다. 유지보수가 적당히 수월했던 코드가 버그와 결함이 숨어있을지도 모른다는 상당히 의심스러운 코드로 뒤바뀐다.

시간이 흘러 코드는 통제를 벗어나기 시작한다. 

여기저기 눈에 거슬리기 시작하며 끔찍한 수준으로 코드는 완전히 엉망이 되어버린다.

### 그래서 멈췄다

기능 요구사항을 충족하기 위해 대부분 개발자는 잘못된 방식을 취한다.

- if 문을 추가한다.
  - 함수의 추상화 수준을 다양해지게 만든다. 
  - 함수가 한 가지만 한다는 규칙을 위반한다.
- 함수의 인수를 추가한다.
  - 이상적인 인수 개수는 무항이다.
  - 인수 개수가 적을수록 함수 코드를 이해하기 쉽다.
  - 테스트를 어렵게 만든다. 모든 인수의 조합의 테스트 케이스를 작성해야 된다.
- 인스턴스를 추가한다.
  - 클래스의 크기가 커진다.
  - 클래스의 SRP을 위반할 가능성이 높다.

이처럼 코드 수정/추가 시 더 나빠지리라는 사실이 자명하다면, 기능을 더 이상 추가하지 않기로 결정하고 리펙터링을 시작한다.

먼저 기존에 작성된 코드에서 공통된 특성을 지닌 부분을 파악하는 추상화 단계를 거친다. 공통된 특성들은 모듈화하고 기존 작성된 로직과 분리한다. 

### 점진적으로 개선한다

**`프로그램을 망치는 가장 좋은 방법`** 중 하나는 **`개선`** 이라는 이름 아래 구조를 크게 뒤집는 행위다.

어떤 프로그램은 그저 그런 **`개선`** 에서 결코 회복하지 못한다.

자칫 `개선`된 코드로 인해, `돌아가는` 프로그램이라는 원초적인 개발 목적마저 망치는 경우가 있기 때문이다. `개선` 전과 똑같이 프로그램을 돌리기가 아주 어렵다.

코드 변경을 가한 후에도 시스템이 변경 전과 똑같이 돌아가야 한다. 테스트 주도 개발(TDD, Test-Driven Development)이라는 기법을 사용한다.

1. 리팩토링할 클래스의 모든 시나리오에 대해 테스트 케이스를 작성한다.
   - 예외 테스트
   - 기능 테스트
   - 인수 테스트
2. 자잘한 변경을 시도한다.
   - 코드를 최소로 건드리는, 가장 단순한 변경부터 시도한다. 
   - 내부에서 사용하고 있는 자료는 클래스로 감싼다.
     - 일급 컬렉션: Collection Type 
     - 래퍼 클래스: Primitive Type
   - NullPointException 을 대비하라.
     - `null`을 반환/전달 하지 마라.
3. 코드를 변경할 때마다 테스트가 통과되는지 확인한다.

## String 인수

아래 코드는 **`if-else`** 로 연쇄적으로 이어져 **`SRP`** , **`OCP`** 원칙들을 위배한다.

```java
// if-else 로 연쇄적으로 이어지는 구문
public class Args {
  // ...
  private void setArgument(char argChar) {
    if (isBooleanArg(argChar)) {
      setBooleanArg(argChar, true);
    } else if (isStringArg(argChar)) {
      setStringArg(argChar);
    } else if (isIntArg(argChar)) {
      setIntArg(argChar);
    } else {
      return;
    }
    return;
  }
}
```

다음 **`if-else`** 구문을 완전히 제거하기 위해선 추상화 단계를 거처야 한다.

1. 비슷한 논리를 띈 코드들을 하나의 클래스로 몰아 넣는다.
2. 테스트를 수행하고, 테스트가 통과되는지 확인한다. 
   - 프로그램 구조를 조금씩 변경하는 동안에도 시스템의 정상 동작을 유지하기 쉬워지기 때문이다.
3. 파생 클래스를 구성하여 기능을 분산한다.
   - 비슷한 논리가 하나의 클래스에 배치 됐다면, 이제 파생 클래스를 만들어 기능을 분산할 차례다. 
   - 이 역시 추상화 단계를 거친다. 
   - 비슷한 논리들의 공통된 부분을 파악하고 **`기존 클래스를 추상화한다.`**
   - 모든 책임은 각 파생 클래스로 전가한다.
     - 실제 구현은 파생 클래스에 숨긴다. 
     - 각기 다른 로직(비즈니스 논리와 예외 로직)은 파생된 클래스로 내려 구체화한다.
     - 즉 모든 책임을 각 파생 클래스로 전가한다.

이 단계를 거치게 되면 전체 시스템이 훨씬 더 일반적으로 변한다.

```java
// 추상화 단계를 거친 코드
public class Args {
  // ...
  // if-else 구문을 제거할 수 있게 된다.
  private void setArgument(char argChar) {
    ArgumentMarshaller m = marshallers.get(argChar);
    if (m == null) {
      return false;
    }
    try {
      m.set(currentArgument);
      return true;
    } catch (ArgsException e) {
      e.setErrorArgumentId(argChar);
      throw e;
    }
  }
}
```

리펙터링은 루빅 큐브 맞추기와 비슷하다.

큰 목표 하나를 이루기 위해 자잘한 단계를 수없이 거친다. 각 단계를 거쳐야 다음 단계가 가능하다. 물론 리팩토링하기 전에 작성된 모든 테스트는 통과되어야 한다. 

소프트웨어 설계는 분할만 잘해도 품질이 크게 높아진다. 적절한 장소를 만들어 코드만 분리해도 설계가 좋아진다.

특별히 눈여겨볼 코드는 `ArgsException` 의 `errorMessage 메서드`다.

`ArgsException` 클래스가 오류 메시지 형식까지 책임져야 맞는걸까? <br/>
솔직하게 말해, 이것은 절충안이다. 

- 맡겨서는 안 된다고 생각하면 새로운 클래스가 필요하다. 
- 하지만 미리 깔끔하게 만들어진 오류 메시지로 얻는 장점을 무시하기 어렵다.

```java
public class ArgsException extends RuntimeException {
  // constructor, getter/setter properties
  
  public String errorMessage() {
    switch (errorCode) {
      case OK:
        return "TILT: Should not get here.";
      case UNEXPECTED_ARGUMENT:
        return String.format("Argument -%c unexpected.", errorArgumentId);
      case MISSING_STRING:
        return String.format("Could not find string parameter for -%c.", errorArgumentId);
      case INVALID_INTEGER:
        return String.format("Argument -%c expects an integer but was '%s'.", errorArgumentId, errorParameter);
      case MISSING_INTEGER:
        return String.format("Could not find integer parameter for -%c.", errorArgumentId);
      case INVALID_DOUBLE:
        return String.format("Argument -%c expects a double but was '%s'.", errorArgumentId, errorParameter);
      case MISSING_DOUBLE:
        return String.format("Could not find double parameter for -%c.", errorArgumentId);
      case INVALID_ARGUMENT_NAME:
        return String.format("'%c' is not a valid argument name.", errorArgumentId);
      case INVALID_ARGUMENT_FORMAT:
        return String.format("'%s' is not a valid argument format.", errorParameter);
    }
    return "";
  }

  public enum ErrorCode {
    OK, INVALID_ARGUMENT_FORMAT, UNEXPECTED_ARGUMENT, INVALID_ARGUMENT_NAME,
    MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, MISSING_DOUBLE, INVALID_DOUBLE
  }
}
```

## 결론

그저 돌아가는 코드만으로는 부족하다.

- 돌아가는 코드가 심하게 망가지는 사례는 흔하다.
- 단순히 돌아가는 코드에 만족하는 프로그래머는 전문가 정신이 부족하다.
- 설계와 구조를 개선할 시간이 없다고 변명하지마라. 동의할 수 없다.
- **나쁜 코드**는 아주 오랫동안, 심각하게 개발 프로젝트에 **악영향을 미친다.**
  - 나쁜 요구사항은 다시 정하면 된다.
  - 나쁜 팀 역학은 복구하면 된다.
  - 나쁜 일정은 다시 짜면 된다.
  - **`하지만 나쁜 코드는 썩어 문드러진다.`**

물론 나쁜 코드는 깨끗한 코드로 **`개선할 수 있지만, 비용이 엄청나게 많이 든다.`**

- 코드가 썩어가며 모듈은 서로 얽히고설켜 뒤엉키고 숨겨진 의존성이 수도 없이 생긴다.
- 오래된 의존성을 찾아내 깨려면 **상당한 시간과 인내심이 필요하다.**

**`반면 처음부터 코드를 깨끗하게 유지하기란 상대적으로 쉽다.`**

- 아침에 엉망으로 만든 코드를 오후에 정리하기는 어렵지 않다.
- 5분 전에 엉망으로 만든 코드는 지금 당장 정리하기 아주 쉽다.

**코드는 `언제나`, 최대한 `깔끔하고` `단순하게` 정리하자. 절대로 썩어가게 방치하면 안 된다.**