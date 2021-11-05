# 7장 오류 처리

오류 처리는 프로그램에 반드시 필요한 요소다.

간단히 말해, 뭔가 잘못될 가능성은 늘 존재한다. 뭔가 잘못되면 바로 잡을 책임은 바로 우리 개발자에게 있다.

깨끗한 코드와 오류 처리는 확실히 연관성 있다.

여기저기 흩어진 오류 처리 코드 때문에 실제 코드가 하는 일을 파악하기가 거의 불가능하다는 의미다. 오류 처리는 중요하다. 하지만 오류 처리 코드로 인해 `프로그램 논리`를 이해하기 어려워진다면 깨끗한 코드라 부르기 어렵다.

## 목차

1. [오류 코드보다 예외를 사용하라](#오류-코드보다-예외를-사용하라)
2. [Try-Catch-Finally 문부터 작성하라](#try-catch-finally-문부터-작성하라)
3. [미확인 예외를 사용하라](#미확인-예외를-사용하라)
4. [예외에 의미를 제공하라](#예외에-의미를-제공하라)
5. [호출자를 고려해 예외 클래스를 정의하라](#호출자를-고려해-예외-클래스를-정의하라)
6. [정상 흐름을 정의하라](#정상-흐름을-정의하라)
7. [null을 반환하지 마라](#null을-반환하지-마라)
8. [null을 전달하지 마라](#null을-전달하지-마라)
9. [결론](#결론)

## 오류 코드보다 예외를 사용하라

예전엔 예외를 지원하지 않는 프로그래밍 언어가 많았다.

예외를 지원하지 않는 언어는 오류를 처리하고 보고하는 방법이 제한적이었다.

- 오류 플래그 설정
- 호출자에게 오류 코드 반환

```java
public class DeviceController {
  // ...
  public void sendShutDown() {
    DeviceHandle handle = getHandle(DEV1);
    // 디바이스 상태 검증
    if (handle != DeviceHandle.INVALID) {
      // 레코드 필드에 디바이스 상태 저장
      retriveDeviceRecord(handle);

      // 디바이스 상태가 일시정지가 아니면 종료
      if (record.getStatus() != DEVICD_SUSPENDED) {
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        clsoeDevice(handle);
      } else {
        logger.log("Device suspended. Unable to shut down");
      }
    } else {
      logger.log("Invalid handle for: " + DEV1.toString());
    }
  }
}
```

함수를 호출한 즉시 오류를 확인해야 하기 때문에 호출자 코드가 복잡해진다. 비즈니스 로직과 오류를 처리하는 로직을 섞으면 복잡해진다.

오류가 발생하면 예외를 던지는 편이 낫다. 비즈니스 로직과 오류를 처리하는 로직을 분리한다.

```java
public class DeviceController {
  // ...
  public void sendShutDown() {
    try {
      tryToShutDown();
    } catch (DeviceShutDownError e) {
      logger.log(e);
    }
  }
  
  private void tryToShutDown() throws DeviceShutDownError {
    DeviceHandle handle = getHandle(DEV1);
    DeviceRecord record = retrieveDeviceRecord(handle);

    pauseDevice(handle);
    clearDeviceWorkQueue(handle);
    clsoeDevice(handle);
  }

  private DeviceHandle DeviceHandle(DeviceID id) {
    // find DeviceHandle logic...
    throw new DeviceShutDownError("Invalid handle for: " + id.toString());
  }
}
```

## Try-Catch-Finally 문부터 작성하라

예외에서 프로그램 안에다 범위를 정의한다.

어떤 면에서 try 블록은 트랜잭션과 비슷하다. try 블록에서 무슨일이 생기든지 catch 블록은 프로그램 상태를 일관성 있게 유지해야 한다.

try 블록에서 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기 쉬워진다.

```java
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
  sectionStor.retrieveSection("invalid - file");
}

public List<RecordedGrip> retrieveSection(String sectionName) {
  // 실제로 구현할 때까지 비어 있는 더미를 반환한다.
  return new ArrayList<RecordedGrip>();
}
```

`retrieveSection`에서 예외를 던지지 않음으로 단위 테스트 결과는 실패한다.

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
  try {
    FileInputStream stream = new FileInputStream(sectionName);
  } catch (Exception e) {
    throw new StorageException("retrieval error", e);
  }
  return new ArrayList<RecordedGrip>();
}
```

테스트 코드가 통과된다.

테스트가 통과되면 예외 유형을 좁혀 실제로 FileInputStream 생성자가 던지는 `FileNotFoundException`을 잡아낸다.

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
  try {
    FileInputStream stream = new FileInputStream(sectionName);
  } catch (FileNotFoundException e) {
    throw new StorageException("retrieval error", e);
  }
  return new ArrayList<RecordedGrip>();
}
```

먼저 강제로 예외를 일으키는 테스트 케이스를 작성한 후 테스트를 통과하게 코드를 작성하는 방법을 권장한다. 

그러면 자연스럽게 try 블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션 본질을 유지하기 쉬워진다.

## 미확인 예외를 사용하라

여러 해 동안 자바 프로그래머들은 확인된(checked) 예외의 장단점을 놓고 논쟁을 벌여왔다. 자바 첫 버전이 확인된 예외를 선보였던 당시는 확인된 예외가 멋진 아이디어로 여겨졌다.

- Checked Exception
- UnChecked Exception

확인된 예외는 몇 가지 장점을 제공하지만, 반드시 필요하지 않다는 사실이 분명해졌다.

> C#, C++, 파이썬, 루비에선 Checked Exception를 지원하지 않는다. 그럼에도 안정적인 소프트웨어를 구현하기에 무리가 없다.

확인된 예외는 책임을 호출하는 메서드에게 전가한다.

- OCP(Open-Close Principle)를 위반한다.
  - 하위 단계에서 코드를 변경하면 사우이 단계 메서드 선언부를 전부 고쳐야 한다.
  - 메서드에서 확인된 예외를 던지있고, catch 블록은 세 단계 위에 있다면 그 사이 메서드 모두가 선언부에 해당 예외를 정의해야 한다.

결과적으로 최하위 단계에서 최상위 단계까지 연쇄적인 수정이 일어난다. 모든 함수가 최 하위 함수에서 던지는 예외를 알아야 하므로 캡슐화가 깨진다.

때론, 확인된 예외도 유용하다. 

아주 중요한 라이브러리르 작성한다면 모든 예외를 잡아야 한다. 하지만 일반적인 애플리케이션은 의존성이라는 비용이 이익보다 크다.

## 예외에 의미를 제공하라

예외를 던질 때는 전후 상황을 충분히 덧붙인다.

실패한 코드의 의도를 파악하려면 호출 스택만으로 부족하다.

오류 메시지에 정보를 담아 예외와 함께 던져, 예외에 대한 충분한 정보를 넘겨주도록 한다.

## 호출자를 고려해 예외 클래스를 정의하라

오류를 분류하는 방법은 수없이 많다. 오류가 발생한 위치로 분류가 가능하다.

- 컴포넌트
- 오류 유형

애플리케이션에서 오류를 정의할 때 프로그래머에게 가장 중요한 관심사는 **오류를 잡아내는 방법**이 되어야 한다.

```java
ACMEPort port = new ACMEPort(12);

try {
  port.open();
} catch (DeviceResponseException e) {
  reportPortError(e);
  logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {
  reportPortError(e);
  logger.log("Unlock exception", e);
} catch (GMXError e) {
  reportPortError(e);
  logger.log("Device reponse exception", e);
} finally {
  // ...
}
```

우리가 오류를 처리하는 방식은 오류를 일으킨 원인과 무관하게 비교적 일정하다.

1. 오류를 기록한다.
2. 프로그램을 계속 수행해도 좋은지 확인한다.

위 경우는 예외에 대응하는 방식이 예외 유형과 무관하게 거의 동일하다. 호출하는 라이브러리 API를 감싸면서 예외 유형 하나를 반환하면 된다.

```java
LocalPort port = new LocalPort(12);
try {
  port.open();
} catch (PortDeviceFailure e) {
  reportError(e);
  looger.log(e.getMessage(), e);
} finally {
  // ...
}
```

여러 공통된 오류를 관리하는 예외 Wrapper 클래스 구성한다.

예외에 대응하는 방식이 예외 유형과 무관하게 거의 동일하면, 호출하는 라이브러리 API를 감싸면서 예외 유형 하나를 반환하는 Wrapper 클래스 구성한다.

1. 외부 API를 감싸면 외부 라이브러리와 프로그램 사이에서 의존성이 크게 줄어든다.
2. 다른 라이브러리 교체 비용이 적다.
3. 테스트가 용이하다.
    - Wrapper 클래스에서 외부 API를 호출하는 대신 테스트 코드를 넣어주는 방법으로 프로그램을 테스트하기도 쉬워진다.
4. 프로그램이 사용하기 편리한 API를 정의할 수 있다.
    - 특정 업체가 API를 설계한 방식에 발목 잡히지 않는다.

흔히 예외 클래스가 하나만 있어도 충분한 코드가 많다.

## 정상 흐름을 정의하라

비즈니스 논리와 오류 처리가 잘 분리된 코드가 나온다. 그러다 보면 오류 감지가 프로그램 언저리로 밀려난다.

외부 API를 감싸 독자적인 예외를 던지고, 코드 위에 처리기를 정의해 중단되 계산을 처리한다. 대개는 멋진 처리 방식이지만, **때로는 중단이 적합하지 않은 때도 있다.**

```java
try {
  MealExpenses expenses = expenseReportDAO.getMeals(employee.getId());
  this.total += expenses.getTotal();
} catch (MealExpensesNotFound e) {
  this.total += getMealPerDiem();
}
```

예외가 논리를 따라가기 어렵게 만든다. 

특수 상황을 처리할 필요가 없다면 코드가 간결해진다.

```java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getId());
this.total += expenses.getTotal();

public class ExpenseReportDAO {
  public MealExpenses getMeals(EmployeeId id) {
    try {
      return getMealExpenses();
    } catch (MealExpensesNotFound e) {
      return new PerDiemMealExpenses();
    }
  }
}

public class PerDiemMealExpenses implements MealExpenses {
  public int getTotal() {
    // 기본값으로 일일 기본 식비를 반환한다.
  }
}
```

이를 특수 사례 패턴(special case pattern)이라 부른다. 클래스를 만들거나 객체를 조작해 특수 사례를 처리하는 방식이다. 

객체가 예외적인 상황을 캡슐화해서 처리하므로 클라이언트는 코드가 예외적인 상황을 처리할 필요가 없어진다.

## null을 반환하지 마라

null을 반환하는 습관은 오류를 유발하는 행위다.

null을 반환하는 코드는 일거리를 느릴뿐만 아니라 **호출자에게 문제를 떠넘긴다.** 

```java
List<Employee> employees = getEmployees();
if (employees != null) {
  for (Employee e : employees) {
    totalPay += e.getPay();
  }
}
```

NullPointException으로 예외를 처리하는 코드든 null 확인하는 코드든 어느 쪽이든 바람직하지 않다.

사용하려는 외부 API가 null을 반환한다면, 감싸기(Wrapper) 메서드를 구현해 예외를 던지거나 특수 사례 객체를 반환하는 방식을 고려한다.

```java
List<Employee> employees = getEmployees();
  for (Employee e : employees) {
  totalPay += e.getPay();
}

// NPE 방지 Wrapper 메서드 구현
List<Employee> getEmployees() {
  return employees == null ? Collections.emptyList() : Employees;
}
```

## null을 전달하지 마라

null을 반환하는 메서드도 나쁘지만, **메서드에 null을 전달하는 방식은 더 나쁘다.**

정상적인 인수로 null을 기대하는 API가 아니라면 메서드로 null을 전달하는 코드는 최대한 피한다.

흔히 null 인수에 대한 방어적 코드 기법은 다음과 같다.

1. Unchecked Exception 활용
2. assert 문 활용

```java
import com.sun.tools.corba.se.idl.InvalidArgument;

public class Calculator {
  // Unchecked Exception 활용
  public double xProjection(Point p1, Point p2) {
    if (p1 == null || p2 == null) {
      throw new InvalidArgumentException("Invalid argument for Calculator.xProjection");
    }
    return ( p2.x - p1.x ) * 1.5;
  }

  // 2. assert 문 활용
  public double xProjection(Point p1, Point p2) {
    assert p1 != null : "p1 should not be null";
    assert p2 != null : "p2 should not be null";
    return ( p2.x - p1.x ) * 1.5;
  }
}
```


대다수 프로그래밍 언어는 호출자가 실수로 넘기는 null을 적절히 처리하는 방법이 없다. 애초에 null을 넘기지 못하도록 금지하는 정책이 합리적이다.

인수로 null이 넘어오면 코드에 문제가 있다는 말이다.

## 결론

깨끗한 코드는 읽기도 좋아야 하지만 **안정성도 높아야 한다.**

이 둘은 상충하는 목표가 아니다. 오류 처리를 비즈니스 논리와 분리해 독자적인 사안으로 고려하면 튼튼하고 깨끗한 코드를 작성할 수 있다.

오류 처리를 프로그램 논리와 분리하면 독립적인 추론이 가능해지며 코드 유지보수성도 크게 높아진다.