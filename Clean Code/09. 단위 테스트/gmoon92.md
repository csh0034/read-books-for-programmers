# 9장 단위 테스트

TDD(Test Driven Development)라는 개념이 없을 때, 우리들 대다수에게 단위 테스트란 자기 프로그램이 '돌아간다'는 사실만
확인하는 일회성 코드에 불과했다.

우리 분야는 지금까지 눈부신 성장을 이뤘지만 앞으로 갈 길은 여전히 멀다. 애자일과 TDD 덕택에 단위 테스트를 자동화하는 프로그래머들이 이미
많아졌으며 점점 더 늘어가는 추세다.

하지만 우리 분야에 테스트를 추가하려고 급하게 서두르는 와중에 많은 개발자들이 제대로 된 테스트 케이스를 작성해야 한다는 좀 더 미묘한 (그리고
더욱 중요한) 사실을 놓쳐 버렸다.

## 목차

1. [TDD 법칙 세 가지](#TDD-법칙-세-가지)
2. [깨끗한 테스트 코드 유지하기](#깨끗한-테스트-코드-유지하기)
   - [테스트는 유연성 유지보수성 재사용성을 제공한다](#테스트는-유연성-유지보수성-재사용성을-제공한다)
3. [깨끗한 테스트 코드](#깨끗한-테스트-코드)
   - [도메인에 특화된 테스트 언어](#도메인에-특화된-테스트-언어)
   - [이중 표준](#이중-표준)
4. [테스트 당 assert 하나](#테스트-당-assert-하나)
   - [테스트 당 개념 하나](#테스트-당-개념-하나)
5. [FIRST](#FIRST)
6. [결론](#결론)

## TDD 법칙 세 가지

TDD는 실제 코드를 짜기 전에 단위 테스트부터 짜라고 요구한다.

1. 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
2. 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
3. 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.

위 세가지 규칙을 따르면 개발과 테스트가 대략 30초 주기로 묶인다. 이렇게 일하면 실제 코드를 사실상 전부 테스트하는 테스트 케이스가 나온다.

하지만 실제 코드와 맞먹을 정도로 **방대한 테스트 코드는 심각한 관리 문제**를 유발하기도 한다.

## 깨끗한 테스트 코드 유지하기

시스템이 커지면서 방대한 테스트 코드로 인해 기능 하나 추가하기 어렵게 만든다.

"테스트 코드에 실제 코드와 동일한 품질 기준을 적용하지 않아야 한다"고 명시적으로 결정한 팀을 코치해달라는 요청을 받았다. 테스트 코드는 "
지저분해도 빨리"가 주가 되고 단위 테스트에서 규칙을 깨도 좋다는 허가장을 주었다.

핵심은 테스트 코드도 깔끔하게 관리해야 한다는 것이다.

1. 테스트 코드가 지저분할수록 변경하기 어려워진다.
2. 테스트 코드가 복잡할수록 실제 코드를 짜는 시간보다 테스트 케이스를 추가하는 시간이 더 걸린다.
3. 실제 코드를 변경해 기존 테스트 케이스가 실패하기 시작하면, 지저분한 코드로 인해, 실패하는 테스트 케이스를 점점 더 통과시키기
   어려워진다.
4. 계속해서 늘어나는 지저분한 테스트 코드는 부담이 된다.

결론적으로 지저분한 테스트 코드로 인해 테스트 슈트를 폐기하는 상황까지 이르게 된다. 테스트 슈트가 없으면 기능 수정에 대한 검증을 할 수
없기에 결함율이 높아지기 시작한다.

**결함 수가 많아지면 개발자는 변경을 주저한다.**

테스트 코드는 실제 코드 못지 않게 중요하다. 테스트 코드는 이류 시민이 아니다. 실제 코드 못지 않게 깨끗하게 짜야한다.

### 테스트는 유연성 유지보수성 재사용성을 제공한다

테스트 코드를 깨끗하게 유지하지 않으면 결국은 잃어버린다. 테스트 코드가 사라지면 실제 코드를 유연하게 만드는 버팀목이 사라진다.

`단위 테스트`는 코드에 유연성, 유지보수성, 재사용성을 제공하는 버팀목이다.

- 테스트 케이스가 있으면 변경에 두렵지 않다.
- 테스트 케이스가 없다면 모든 변경이 잠정적인 버그다.

테스트 케이스가 있다면 변경이 쉬워진다. 테스트 코드가 지저분하면 코드를 변경하는 능력이 떨어지며 코드 구조를 개선하는 능력도 떨어진다.

## 깨끗한 테스트 코드

깨끗한 테스트 코드를 만드는 세가지 조건

가독성, 가독성, 가독성!!!

어쩌면 가독성은 실제 코드보다 테스트 코드에 더더욱 중요하다.

가독성을 높이려면 여느 코드와 마찬가지다. `명료성`, `단수성`, `풍부한 표현력`이 필요하다.

```java
// 목록 9-1
public class SerializedPageResponderTest {
  public void testGetPageHierarchyAsXml() throws Exception {
    crawler.addPage(root, PathParser.parse("PageOne"));
    crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
    crawler.addPage(root, PathParser.parse("PageTwo"));

    request.setResource("root");
    request.addInput("type", pages);

    Responseder response =
            (SimpleResponse) responder.makeResponse(
                    new FitNesseContext(root), request);
    String xml = response.getContent();

    assertEqauls("text/xml", response.getContentType());
    assertSubString("<name>PageOne</name>", xml);
    assertSubString("<name>PageTwo</name>", xml);
    assertSubString("<name>ChildOne</name>", xml);
  }

  public void testGetPageHierarchyAsXmlDoesntContainSymbolicLinks() throws Exception {
    WikiPage pageOne = crawler.addPage(root, PathParser.parse("PageOne"));
    crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
    crawler.addPage(root, PathParser.parse("PageTwo"));

    PageData data = pageOne.getData();
    WikiPageProperties properties = data.getProperties();
    WikiPageProperty symLinks = properties.set(SymbolicPage.PROPERTY_NAME);
    symLinks.set("SymPage", "PageTwo");
    pageOne.commit(data);

    request.setResource("root");
    request.addInput("type", "pages");
    Responder responder = new SerializedPageResponder();
    SimpleResponse response =
            (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
    String xml = response.getContent();

    assertEquals("text/xml", response.getContentType());
    assertSubString("<name>PageOne</name>", xml);
    assertSubString("<name>PageTwo</name>", xml);
    assertSubString("<name>ChildOne</name>", xml);
    assertNotSubString("SymPage", xml);
  }

  public void testGetDataAsHtml() throws Exception {
    crawler.addPage(root, PathParser.parse("TestPageOne"), "test page");

    request.setResource("TestPageOne");
    request.addInput("type", "data");
    Responder responder = new SerializedPageResponder();
    SimpleResponse response =
            (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
    String xml = response.getContent();

    assertEquals("text/xml", response.getContentType());
    assertSubString("test page", xml);
    assertSubString("<Test", xml);
  }
}
```

위 코드는 읽는 사람을 고려하지 않는다.

```java
// 목록 9-2
public class SerializedPageResponderTest {
   public void testGetPageHierarchyAsXml() throws Exception {
      makePages("PageOne", "PageOne.ChildOne", "PageTwo");

      submitRequest("root", "type:pages");

      assertResponseIsXml();
      assertResponseContains(
              "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
      );
   }
}
```

위 코드는 리펙토링한 테스트 코드다.

[build-operate-check 패턴](https://medium.com/swlh/usual-production-patterns-applied-to-integration-tests-50a941f0b04a) 이 위와 같은 테스트 구조에 적합하다. 각 테스트는 명확히 세 부분으로 나눠진다.

1. 테스트 자료를 만든다.
2. 테스트 자료를 조작한다.
3. 조작한 결과가 올바른지 확인한다.

테스트 코드는 본론에 돌입해 진짜 필요한 자료 유형과 함수만 사용한다. 그러므로 코드를 읽는 사람은 온갖 잡다하고 세세한 코드에 주눅들고 헷갈릴 필요 없이 코드가 수행하는 기능을 재빨리 이해한다.

### 도메인에 특화된 테스트 언어

`목록 9-1` 테스트 코드는 도메인에 특화된 언어(DSL)로 테스트 코드를 구현하는 기법을 보여준다.

흔히 쓰는 시스템 조작 API를 사용하는 대신 API위에다 함수와 유틸리티를 구현한 후 그 함수와 유틸리티를 사용하므로 테스트 코드를 짜기도 읽기도 쉬워진다.

이렇게 구현한 함수와 유틸리티는 테스트 코드에서 사용하는 특수 API가 된다. 테스트를 구현하는 당사자와 나중에 테스트를 읽어볼 독자를 도와주는 `테스트 언어`다.

숙련된 개발자라면 자기 코드를 좀 더 간결하고 표현력이 풍부한 코드로 리펙터링해야 마땅하다.

### 이중 표준

`테스트 API 코드`에 적용하는 표준은 실제 코드에 적용하는 표준과 확실히 다르다.

단순하고, 간결하고, 표현력이 풍부해야 하지만, **실제 코드만큼 효율적일 필요는 없다.**

실제 환경이 아니라 테스트 환경에서 돌아가는 코드이기 때문이다. 실제 환경과 테스트 환경은 요구사항이 판이하게 다르다.

다음 테스트 코드는 온도가 '급격하게 떨어지면' 경보, 온풍기, 송풍기가 모두 가동되는지 확인하는 테스트 코드다.

```java
// 목록 9-3
public class EnvironmentControllerTest {
   @Test
   public void turnOnLoTempAlarmAtThreshold() throws Exception {
      hw.setTemp(WAY_TOO_COLD);
      controller.tic();
      assertTrue(hw.heaterState());
      assertTrue(hw.blowerState());
      assertFalse(hw.coolerState());
      assertFalse(hw.hiTempAlarm());
      assertTrue(hw.loTempAlarm());
   }
}
```

`목록 9-3`은 세세한 사항이 아주 많다. 상태 이름과 상태 값을 확인하느라 눈길이 이리저리 흩어진다. 

따분하고 미덥잖다. 테스트 코드를 읽기가 어렵다.


```java
// 목록 9-4
public class EnvironmentControllerTest {
   @Test
   public void turnOnLoTempAlarmAtThreshold() throws Exception {
      wayTooCold();
      assertEquals("HBchL", hw.getState());
   }
}
```

기존의 따분한 코드들은 `wayTooCold`라는 함수에 숨겼다.

비록 위 방식이 `그릇된 정보를 피하라`는 규칙의 위반에 가깝지만 여기서는 적절해보인다.

```java
// 목록 9-5 더 복잡한 선택
public class EnvironmentControllerTest {
   @Test
   public void turnOnCoolerAndBlowerIfTooHot() throws Exception {
      tooHot();
      assertEquals("hBChL", hw.getState());
   }
   
   @Test
   public void turnOnHeaterAndBlowerIfTooCold() throws Exception {
      tooCold();
      assertEquals("HBchl", hw.getState());
   }

   @Test
   public void turnOnHiTempAlarmAtThreshold() throws Exception {
      wayTooHot();
      assertEquals("hBCHl", hw.getState());
   }

   @Test
   public void turnOnLoTempAlarmAtThreshold() throws Exception {
      wayTooCold();
      assertEquals("HBchL", hw.getState());
   }
}
```

`목록 9-6`은 getState 함수를 보여준다. 코드가 그리 효율적이지 못하다는 사실에 주목하자. 효율을 높이려면 StringBuffer가 더 적합하다.

```java
// 목록 9-6
public class MockControlHardware {
   public String getState() {
      String state = "";
      state += heater ? "H" : "h";
      state += blower ? "B" : "b";
      state += cooler ? "C" : "c";
      state += hiTempAlarm ? "H" : "h";
      state += loTempAlarm ? "L" : "l";
      return state;
   }
}
```

StringBuffer는 보기에 흉하다. 실제 코드에서도 크게 무리가 아니라면 StringBuffer를 피한다. StringBuffer를 안써서 치르는 대가가 미미하다. 실제 운영환경에선 메모리가 제한적일 가능성이 높다. 하지만, 테스트 환경에선 자원이 제한적일 가능성이 낮다.

이것이 이중 표준의 본질이다. 

실제 환경에서는 절대로 안 되지만 테스트 환경에서는 전혀 문제 없는 방식이 있다. 대게 메모리나 CPU 효율과 관련 있는 경우다. **코드의 깨끗함과는 철저히 무관하다.**

## 테스트 당 assert 하나

JUnit으로 테스트 코드를 짤 때는 함수마다 assert 문을 단 하나만 사용해야 한다고 주장하는 학파가 있다.

assert 문이 단 하나인 함수는 결론이 하나라서 코드를 이해하기 쉽고 빠르다.

```java
// 목록 9-7 단일 Assert
public class SerializedPageResponderTest {
   public void testGetPageHierarchyAsXml() throws Exception {
      givenPages("PageOne", "PageOne.ChildOne", "PageTwo");

      whenRequestIsIssued("root", "type:pages");

      thenResponseShouldBeXML();
   }

   public void testGetPageHierarchyHasRightTags() throws Exception {
      givenPages("PageOne", "PageOne.ChildOne", "PageTwo");

      whenRequestIsIssued("root", "type:pages");

      thenResponseShouldContain(
              "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
      );
   }
}
```

위에서 함수 이름을 바꿔 `given-when-then`이라는 관례를 사용하여 가독성을 높였다.

불행하게도, 위에서 보듯이 테스트를 분리하면 중복되는 코드가 많아진다. 

이 경우 [Template Method 패턴](https://refactoring.guru/design-patterns/template-method) 을 사용하면 중복을 제거할 수 있다. given/when 부분을 부코 클래스에 두고 then 부분을 자식 클래스에 두면 된다.

다른 방법으론 `@Before` 함수에 given/when 부분을 넣고 `@Test` 함수에서 then 을 구성하면 된다.

`단일 assert 문` 규칙은 훌륭한 지침이다.

### 테스트 당 개념 하나

**테스트 함수마다 한 개념만 테스트하라.**

이것저것 잡다한 개념을 연속으로 테스트하는 긴 함수는 피한다. 

새 개념을 한 함수로 몰아넣으면 독자가 각 절이 거기에 존재하는 이유와 각 절이 테스트하는 개념을 모두 이해야한다.

```java
// 목록 9-8 addMonth() 메서드를 테스트하는 장황한 코드
public class CalenderTest {
   public void testAddMonths() {
      SerialDate d1 = SerialDate.createInstance(31, 5, 2004);

      SerialDate d2 = SerialDate.addMonths(1, d1);
      assertEquals(30, d2.getDayOfMonth());
      assertEquals(6, d2.getMonth());
      assertEquals(2004, d2.getYYYY());

      SerialDate d3 = SerialDate.addMonths(2, d1);
      assertEquals(31, d3.getDayOfMonth());
      assertEquals(7, d3.getMonth());
      assertEquals(2004, d3.getYYYY());

      SerialDate d4 = SerialDate.addMonths(1, SerialDate.addMonths(1, d1));
      assertEquals(30, d4.getDayOfMonth());
      assertEquals(7, d4.getMonth());
      assertEquals(2004, d4.getYYYY());
   }
}
```

1. 기존 테스트 코드에서 일반적인 규칙을 찾는다.
2. 개념이 분리되었다면, 하나의 개념 당 하나의 테스트 함수로 분리한다. 

테스트 코드의 가장 좋은 규칙은 다음과 같다.

- **"개념 당 assert 문 수를 최소로 줄여라"**
- **"테스트 함수 하나는 개념 하나만 테스트하라"**

## FIRST

깨끗한 테스트는 다음 다섯 규칙을 따른다.

1. Fast: 테스트는 빨라야한다. 테스트가 느리면 자주 돌릴 엄두를 못 낸다.
2. Independent: 각 테스트는 서로 의존하면 안된다. 각 테스트 코드는 독립적으로 어떤 순서로 실행해도 괜찮아야 한다. 멱등성을 보장해야 한다.
3. Repeatable: 테스트는 어떤 환경에서도 반복 가능해야 한다. 네트워크에 연결되지 않는 환경에서도 실행할 수 있어야 한다.
4. Self-Validating: 테스트는 부울(bool) 값으로 결과를 내야 한다. 성공 아니면 실패다. 테스트가 스스로 성공과 실패를 판단해야한다.
5. Timely: 테스트는 적시에 작성해야 한다. 단위 테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다. 실제 코드를 구현한 다음에 테스트 코드를 만들면 실제 코드가 테스트하기 어렵다는 사실을 발견할지도 모른다.

## 결론

사실상 깨끗한 테스트 코드라는 주제는 책 한 권을 할애해도 모자랄 주제다.

테스트 코드는 실제 코드만큼이나 **프로젝트 건강**에 중요하다. 어쩌면 실제 코드보다 더 중요할지도 모르겠다. 테스트 코드는 실제 코드의 유연성, 유지보수성, 재사용성을 보존하고 강화하기 때문이다.

테스트 코드는 지속적으로 깨끗하게 관리하자. 표현력을 높이고 간겨라게 정리하자. 테스트 API를 구현해 도메인 특화 언어(DSL, Domain Specific Language)를 만들자. 그러면 그만큼 테스트 코드를 짜기가 쉬워진다.

테스트 코드가 방치되어 망가지면 실제 코드도 망가진다. 테스트 코드를 깨끗하게 유지하자.
