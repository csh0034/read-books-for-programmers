# 10장 클래스

이 장에서는 깨끗한 클래스를 다룬다.

코드의 표현력과 그 코드로 이루어진 함수에 아무리 신경 쓸지라도 **좀 더 차원 높은 단계까지 신경 쓰지 않으면 깨끗한 코드를 얻기는 어렵다.**

## 목록

1. [클래스 체계](#클래스-체계)
   - [캡슐화](#캡슐화)
2. [클래스는 작아야 한다](#클래스는-작아야-한다)
   - [클래스의 크기의 척도](#클래스의-크기의-척도)
   - [단일 책임 원칙](#단일-책임-원칙)
   - [응집도](#응집도)
   - [응집도를 유지하면 작은 클래스 여럿이 나온다](#응집도를-유지하면-작은-클래스-여럿이-나온다)
3. [변경하기 쉬운 클래스](#변경하기-쉬운-클래스)
   - [변경으로부터 격리](#변경으로부터-격리)

## 클래스 체계

클래스를 정의하는 표준 자바 관례

1. 변수
   a. static
   b. public
   c. protected
   d. private
2. 생성자
   a. 부 생성자
   b. 주 생성자
3. 함수
   a. 기본적으로 static -> public 접근 제한자 순으로 배치한다.
   b. 비공개(private) 함수는 자신을 호출하는 공개 함수 직후에 넣는다. 추상화 단계가 순차적으로 내려가도록 구성하여 신문 기사처럼 읽히도록 한다.

### 캡슐화

변수와 유틸리티 함수는 가능한 공개하지 않는 편이 낫지만 반드시 숨겨야 한다는 법칙도 없다.

때론, **유틸리티 함수를 protected로 선언해 테스트 코드에 접근을 허용하기도 한다.** 우리에게는 테스트는 아주 중요하다. 캡슐화가 깨진다. 

최우선적으로 그 전에 비공개 상태를 유지할 온갖 방법을 강구한다. 캡슐화를 풀어주는 결정은 언제나 최후의 수단이다.

## 클래스는 작아야 한다

클래스를 만들 때 첫 번째 규칙은 크기다.

클래스는 작아야 한다. 두 번째 규칙도 더 작아야 한다. 함수와 마찬가지로, '작게'가 기본 규칙이라는 의미다.

### 클래스의 크기의 척도

- 함수의 크기의 척도는 행 수로 측정한다.
- 클래스의 크기의 척도는 `책임의 수`로 측정한다.

책임의 수는 메서드의 수와 무관하다. 메서드 수가 작음에도 불구하고 책임이 너무 많으면 클래스는 크다고 할 수 있다.

```java
// 목록 10-2 책임이 많은 클래스
public class SuperDashboard {
   public Component getLastFocusedComponent(){};
   public void setLastFocused(Component lastFocused){};
   public int getMajorVersionNumber(){};
   public int getMinorVersionNumber(){};
   public int getBuildNumber(){};
}
```

**클래스 이름은** 해당 **클래스 책임을 기술**해야 한다.

- 실제로 작명은 클래스 크기를 줄이는 첫 번째 관문이다.
- 간결한 이름이 떠오르지 않는다면 필경 클래스 크기가 너무 커서 그렇다.
- 클래스 이름이 모호하다면 필경 클래스 책임이 너무 많아서다.

### 단일 책임 원칙

단일 책임 원칙(SRP, Single Responsibility Principle)은 **클래스나 모듈을 변경할 이유가 단 하나뿐이어야 한다는 원칙**이다.

SRP는 "책임"이라는 개념을 정의하며 적절한 클래스 크기를 제시한다. 클래스는 책임, 즉 변경할 이유가 하나여야 한다는 의미다.

```java
// 목록 10-3 단일 책임 클래스
public class Version {
   public int getMajorVersionNumber(){};
   public int getMinorVersionNumber(){};
   public int getBuildNumber(){};
}

public class SuperDashboard {
  private Version version;

  public Component getLastFocusedComponent(){};
  public void setLastFocused(Component lastFocused){};
}
```

책임, 즉 변경할 이유를 파악하려 애쓰다 보면 코드를 추상화하기도 쉬워진다.

SRP는 객체 지향 설계에서 더욱 중요한 개념이다. 또한 이해하고 지키기 수월한 개념이기도 하다.

하지만 이상하게도 SRP는 클래스 설계자가 가장 무시하는 규칙 중 하나다.

소프트웨어를 돌아가게 만드는 활동과 소프트웨어를 깨끗하게 만드는 활동은 완전히 별개다. 관심사를 분리하는 작업은 프로그램만이 아니라 프로그래밍 활동에서도 마찬가지로 중요하다.

다수 개발자는 자잘한 단일 책임 클래스가 많아지면 큰 그림을 이해하기 어려워진다고 우려한다. 큰 그림을 이해하려면 이 클래스 저 클래스를 수 없이 넘나들어야 한다고 걱정한다.

하지만 작은 클래스가 많은 시스템이든 큰 클래스가 몇 개뿐인 시스템이든 돌아가는 부품은 그 수가 비슷하다. 어느 시스템이든 익힐 내용은 그 양이 비슷하다.

규모가 어느 수준에 이르는 시스템은 논리가 많고 복잡하다. 이런 복잡성을 다루려면 체계적인 정리가 필수다.

큰 클래스 몇개가 아니라 작은 클래스 여럿으로 이뤄진 시스템이 더 바람직하다. 작은 클래스는 각자 맡은 책임이 하나며, 변경할 이유가 하나며, 다른 작은 클래스와 협력해 시스템에 필요한 동작을 수행한다.

### 응집도

클래스는 인스턴스 변수 수가 작아야 한다.

각 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야 한다. 일반적으로 메서드가 변수를 더 많이 사용할 수록 메서드와 클래스는 응집도가 더 높다.

일반적으로 응집도가 가장 높은 클래스는 가능하지도 바람직하지도 않다. 그렇지만 우리는 응집도가 높은 클래스를 선호한다. **응집도가 높다**는 말은 **클래스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는 의미**기 때문이다.

```java
// 목록 10-4 응집도가 높은 Stack 클래스
public class Stack {
   private int topOfStack = 0;
   List<Integer> elements = new LinkedList<Integer>();

   public int size() {
      return topOfStack;
   }

   public void push(int element) {
      topOfStack++;
      elements.add(element);
   }

   public int pop() throws PoppedWhenEmpty {
      if (topOfStack == 0)
         throw new PoppedWhenEmpty();
      int element = elements.get(--topOfStack);
      elements.remove(topOfStack);
      return element;
   }
}
```

다음 Stack 클래스를 보면 메서드 인수 수가 적고, 대신 인스턴스 변수를 사용한다. 응집도가 높아 클래스에 속한 메서드와 인스턴스 변수가 서로 의존적이며 논리적인 단위로 묶였다. 

이처럼 "함수를 작게, 매개변수 목록을 짧게"라는 규칙을 따르다 보면 인스턴스 변수가 아주 많아진다. 

**인스턴스 변수가 많아졌다**는 의미는 **새로운 클래스로 쪼개야 한다**는 신호다. 응집도가 높아지도록 변수와 메서드를 적절히 분리해 새로운 클래스 두 세개로 쪼개야 한다.

### 응집도를 유지하면 작은 클래스 여럿이 나온다

큰 함수를 작은 함수 여럿으로 나누기만 해도 클래스 수가 많아진다.

1. 큰 함수 일부를 작은 함수로 분리한다.
2. 함수의 인수는 인스턴스 변수로 승격한다.
3. 함수를 쪼개기 용이하다.
4. 점차 인스턴스 변수 수가 많아진다.
5. 인스턴스 변수가 많아진다. 즉 클래스의 책임이 많아진다는 신호다.
6. 클래스의 책임이 많아지면 응집력을 잃는다.
7. 응집력을 잃는다면, 새로운 클래스로 쪼개라

큰 함수부터 작은 함수로, 작은 함수를 구성하다 보면 인스턴스 수가 많아지며 클래스의 응집력을 잃는다면 새로운 클래스로 쪼개면 된다. 그러면서 프로그램에 점점 더 체계가 잡히고 구조가 투명해진다.

> 특히 클래스 설계에 있어 컴포넌트는 매우 중요하다. 잘 설계된 컴포넌트는 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 얼마나 잘 숨겼는냐가 중요하다. 또한, 상속보다는 컴포지션(Composition)을 사용하여 설계하는게 중요하다.

리펙토링 규칙은 간단하다. 

1. 기존 테스트 코드를 작성한다.
2. 테스트 코드를 기반으로 하나의 책임을 가지도록 클래스를 분리한다.
3. 좀 더 길고 서술적인 변수 이름을 사용한다.
4. 리펙터링한 프로그램은 코드에 주석을 추가하는 수단으로 함수 선언과 클래스 선언을 활용한다.
5. 가독성을 높이고자 공백을 추가하고 형식을 맞춘다.

## 변경하기 쉬운 클래스

소프트웨어는 늘 변한다. 그리고 뭔가 변경할 때마다 시스템이 의도대로 동작하지 않을 위험이 따른다. 깨끗한 시스템은 클래스를 체계적으로 정리해 변경에 수반하는 위험을 낮춘다.

`목록 10-9`는 update 문과 같은 일부 SQL 기능을 지원하지 않는다.

1. 언젠가 update 문을 지원할 시점이 오면 클래스에 '손대어' 고쳐야 한다.
2. 어떤 변경이든 코드에 '손대면' 다른 코드를 망가뜨릴 잠정적인 위험이 따른다.
3. 테스트도 완전히 다시 작성해야 한다.

```java
// 목록 10-9 변경이 필요해 '손대야' 하는 클래스
public class Sql {
  public Sql(String table, Column[] columns)
  public String create()
  public String insert(Object[] fields)
  public String selectAll()
  public String findByKey(String keyColumn, String keyValue)
  public String select(Column column, String pattern)
  public String select(Criteria criteria)
  public String preparedInsert()
  private String columnList(Column[] columns)
  private String valuesList(Object[] fields, final Column[] columns)
  private String selectWithCriteria(String criteria)
  private String placeholderList(Column[] columns)
}
```

1. 새로운 SQL 문을 지원하려면 코드를 수정해야 한다.
2. 기존 SQL 문을 수정할 때도 코드를 수정해야 한다.

기존 Sql 클래스는 변경할 이유가 두 가지이므로 Sql 클래스는 SRP를 위반한다.

비공개 메서드는 코드를 개선할 잠재적인 여지를 시사한다. 하지만 실제로 개선에 뛰어드는 계기는 시스템이 변해서라야 한다. 논리적으로 완성으로 여긴다면 책임을 분리하려 시도할 필요가 없다. 가까운 장래에 update 문이 필요하지 않다면 Sql 클래스를 내버려두는 편이 좋다.

하지만 클래스에 손대는 순간, 설계를 개선하려는 고민과 시도가 필요하다.

```java
// 목록 10-10 닫힌 클래스 집합
public abstract class Sql {
  public Sql(String table, Column[] columns) {}
  public abstract String generate();
}

public class CreateSql extends Sql {
  public CreateSql(String table, Column[] columns)
  @Override public String generate()
}

public class SelectSql extends Sql {
  public SelectSql(String table, Column[] columns)
  @Override public String generate()
}

public class InsertSql extends Sql {
  public InsertSql(String table, Column[] columns)
  @Override public String generate()
}


public class SelectWithCriteriaSql extends Sql {
   public SelectWithCriteriaSql(String table, Column[] columns, Criteria criteria)
   @Override public String generate()
}

public class SelectWithMatchSql extends Sql {
   public SelectWithMatchSql(String table, Column[] columns, Column column, String pattern)
   @Override public String generate()
}

public class FindByKeySql extends Sql {
  public FindByKeySql(String table, Column[] columns, String keyColumn, String keyValue)
  @Override public String generate()
}

public class PreparedInsertSql extends Sql {
   public PreparedInsertSql(String table, Column[] columns)
   @Override public String generate()
   private String placeholderList (Column[]columns)
}

public class Where {
   public Where(String criteria)
   public String generate()
}

public class ColumnList {
   public ColumnList(Column[] columns)
   public String generate()
}
```

- 공통된 책임은 추상화 클래스로 구현했다.
- 하나의 책임만 지니도록 클래스를 분리했다.
- 목록 10-10 처럼 재구성한 Sql 클래스는 SRP, OCP(Open-Closed Principle) 원칙을 따른다.
  - OCP란 클래스는 확장에 개방적이고 수정에 폐쇄적이여야 한다는 원칙이다.
  - 새로운 UpdateSql 클래스를 구현해야 한다면, Sql 추상화 클래스를 구현하면 그만이다.

새 기능을 수정하거나 기존 기능을 변경할 때 건드릴 코드가 최소인 시스템 구조가 바람직하다. 이상적인 시스템이라면 **새 기능을 추가할 때 시스템을 확장할 뿐 기존 코드를 변경하지 않는다.**

### 변경으로부터 격리

요구사항은 변하기 마련이다. 코드도 마찬가지다.

상세한 구현에 의존하는 클라이언트 클래스는 구현이 바뀌면 위험에 빠진다. 그래서 우리는 인터페이스와 추상 클래스를 사용해 구현이 미치는 영향을 격리한다.

> 구체적인 클래스는 상세한 구현을 포함하며 추상 클래스는 개념만 포함한다.

상세한 구현에 의존하는 코드는 테스트가 어렵다.

예를 들어 Portfolio 클래스를 만든다고 가정하자.

1. 외부 TokyoStockExchange API를 사용해 포트폴리오 값을 계산한다.
2. TokyoStockExchange 값은 5 분마다 달라진다.
3. 값이 매번 달라지기 때문에 테스트하기 쉽지 않다.

```java
// 테스트가 어려운 부분을 인터페이스로 추상화
public interface StockExchange {
  Money currentPrice(String symbol);
} 
```

```java
public class PortfolioTest {
   private FixedStockExchangeStub exchange;
   private Portfolio portfolio;

   @Before
   protected void setUp() throws Exception {
      // 5분 마다 바뀌는 값을 사용하는 대신, 테스트를 하기 위해 사용
      exchange = new FixedStockExchangeStub();
      exchange.fix("MSFT", 100);
      portfolio = new Portfolio(exchange);
   }

   @Test
   public void GivenFiveMSFTTotalShouldBe500() throws Exception {
      portfolio.add(5, "MSFT");
      Assert.assertEquals(500, portfolio.value());
   }
}
```

이처럼 테스트가 가능할 정도로 시스템의 결합도를 낮추면 유연성과 재사용성도 더욱 높아진다.

결합도가 낮다는 소리는 각 시스템 요소가 다른 요소로부터 그리고 변경으로부터 잘 격리되어 있다는 의미다. 시스템 요소가 서로 잘 격리되어 있으면 각 요소를 이해하기도 더 쉬워진다.

이렇게 결합도를 최소로 줄이면 자연스럽게 또 다른 클래스 설계 원칙인 DIP(Dependency Inversion Principle) 를 따르는 클래스가 나온다. 본질적으로 DIP는 클래스가 상세 구현이 아니라 추상화에 의존해야 한다는 원칙이다.