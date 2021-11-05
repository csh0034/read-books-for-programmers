# 6장 객체와 자료 구조

변수를 비공개(private)로 정의한 이유는 남들이 변수에 의존하지 않게 만들고 싶어서다.

충동이든 변덕이든, 변수 타입이나 구현을 맘대로 바꾸고 싶어서다.

그렇다면 어째서 수많은 개발자가 조회(getter) 함수와 설정(setter) 함수를 당연하게 공개(public)해 비공개 변수를 외부에 노출할까?

```java
public class Member {
  private String id;
  private String username;
  
  // public getter/setter properties
} 
```

## 목차

1. [자료 추상화](#자료-추상화)
2. [자료와 객체 비대칭](#자료와-객체-비대칭)
    - [절차적인 클래스 자료구조](#절차적인-클래스-자료구조)
    - [다형적인 도형 객체 클래스](#다형적인-도형-객체-클래스)
3. [디미터 법칙](#디미터-법칙)
    - [기차 충돌](#기차-충돌)
    - [잡종 구조](#잡종-구조)
    - [구조체 감추기](#구조체-감추기)
4. [자료 전달 객체](#자료-전달-객체)
    - [활성 레코드](#활성-레코드)
5. [결론](#결론)

## 자료 추상화

자료를 세세하게 공개하기보다는 추상적인 개념으로 표현하는 편이 좋다.

개발자는 객체가 포함하는 자료를 표현할 가장 좋은 방법을 심각하게 고민해야 한다.
아무 생각 없이 조회/설정 함수를 추가하는 방법이 가장 나쁘다.

추상화 관점에 따라, 혹은 외부로 노출시키느냐 마느냐는 개발자의 구현 방식에 따라 달라진다.

```java
// 목록 6-1 구체적인 Point 클래스
public class Point { // 직교좌표계 사용
  public double x; 
  public double y;
}

// 목록 6-2 추상적인 Point 클래스
public interface Point {
  // [1] 직교좌표계 사용
  double getX();
  double getY();
  void setCartesian(double x, double y);
  // [2] 극좌표계 사용
  double getR();
  double getTheta();
  void setPolar(double r, double theta);
}
```

관례적으로 외부로부터 구현을 감추기 위해 추상화 클래스를 활용한다. 인터페이스는 자료 구조를 명백하게 표현한다.

|구체적인 클래스|추상적인 클래스|
|-----------|-----------|
|자료 구조 표현|자료 구조 이상을 표현|
|외부로부터 구현을 노출|외부로부터 구현을 숨김|
|어떤 구현체를 사용하는지 직관적이다.|어떤 구현체를 사용하지 몰라도 인터페이스는 자료 구조를 명백하게 표현|
|- 조회 함수와 설정 함수를 개별적으로 설정하게 강제한다.|- 조회 함수를 통해 개별적으로 읽어야한다.<br/>`getX()`, `getY()`<br/>- 설정 함수는 한꺼번에 설정해야한다<br/>`void setCartesian(double x, double y)`|

- 구현을 감추려면 추상화가 필요하다.
  - 변수 사이에 함수라는 계층을 넣는다고 구현이 저절로 감춰지지 않는다.
- 멤버 변수를 `private`로 선언하더라도 공개 getter/setter 메서드를 제공하면 구현을 외부로 노출하는 셈이다.
  - 형식 논리에 치우쳐 조회/설정 함수로 변수를 다룬다고 클래스가 되지 않는다.
- 추상 인터페이스를 제공해 **사용자가 구현을 모른 채** `자료의 핵심`을 **조작**할 수 있어야 `진정한 의미의 클래스`다.

## 자료와 객체 비대칭

객체와 자료 구조의 정의는 본질적으로 상반된다.

- `객체`: 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개
- `자료 구조`: 자료를 그대로 공개하며, 별다른 함수는 제공하지 않는다.

객체와 자료 구조는 근본적으로 양분된다.

- 새로운 함수 추가할 경우엔 절차적인 코드가 적합하다.
- 새로운 클래스(자료 구조)를 추가할 경우엔 객체 지향적인 코드가 적합하다.

분별 있는 프로그래머는 모든 것이 객체라는 생각이 미신임을 잘 안다. 때로는 단순한 자료 구조와 절차적인 코드가 가장 적합한 상황도 있다.

### 절차적인 클래스 자료구조 

```java
public class Square {
  public Point topLeft;
  public double side;
}

public class Rectangle {
  public Point topLeft;
  public double height;
  public double width;
}

public class Geometry {
  public final double PI = 3.1415;
  
  public double area(Object shape) throws NoSuchShapeException {
    if (shape instanceof Square) {
      Square s = (Square) shape;
      return s.side * s.side;
    } else if (shape instanceof Rectangle) {
      Rectangle r = (Rectangle) shape;
      return r.height * r.width;
    }

    throw new NoSuchShapeException();
  }
}
```

객체 지향 프로그래머가 위 코드에 대해 클래스가 절차적이라 비판한다면 맞는 말이다. 하지만. 그런 비웃음이 100% 옳다고 말하기는 어렵다.

- 장점: 함수 추가 용이하다.
  - Geometry 클래스에 둘레 길이를 구하는 perimeter() 함수를 추가하고 싶다면 도형 클래스는 아무 영향도 받지 않는다.
- 단점: 새로운 클래스 추가가 어렵다.
  - 새 도형을 추가하고 싶다면? Geometry 클래스에 속한 함수를 모두 고쳐야 한다.

### 다형적인 도형 객체 클래스

다형성(Polymorphism)을 활용한 객체 지향적인 도형 클래스다.

```java
public class Square implements Shape {
  private Point topLeft;
  private double side;

  @Override
  public double area() {
    return side*side;
  }
}

public class Rectangle implements Shape {
  private Point topLeft;
  private double height;
  private double width;

  @Override
  public double area() {
    return height*width;
  }
} 
```

객체 지향적 클래스는 다형성(Polymorphism)을 활용해 새로운 도형 클래스를 추가하는데 용이하지만, 도형에 공통적인 함수를 추가가 어렵다는 단점이 존재한다.

전문적인 개발자는 상속 없이 클래스에 메서드를 효과적으로 추가하기 위해 [Visitor 패턴의 Dual-Patch](https://refactoring.guru/design-patterns/visitor) 을 사용하여 문제를 해결한다.

> 참고: [refactoring guru - Visitor and Double Dispatch](https://refactoring.guru/design-patterns/visitor-double-dispatch)

## 디미터 법칙

디미터 법칙은 잘 알려진 휴리스틱으로, 모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙이다.

흔히 `낯선이에게 말을 걸지 말아라`(Don't talk to strange) 법칙으로 알려져 있다. 객체 지향 생활 체조 원칙 중 **한줄에 점을 하나만 찍는다.**라고 요약되기도 한다.

> 디미터라는 프로젝트를 진행하던 중, OOP에서 객체들의 협력 경로를 제한하면 객체간의 결합도를 낮추는 사실을 발견했다. 이러한 배경으로 _객체간 협력 경로를 제한하는 법칙_ 을 "디미터 법칙"이라 한다.

객체 구조의 경로를 따라 멀리 떨어져 있는 낯선 객체에 메시지를 보내는 설계를 피하라는 것이다. Principle of least knowledge(최소 지식 원칙)

객체는 조회 함수로 내부 구조를 공개하면 안 된다는 의미다. 디미터 법칙은 "클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다"고 주장한다.

- 클래스 C
- f가 생성한 객체
- f 인수로 넘어온 객체
- C 인스턴스 변수에 저장된 객체

```java
public class Member {
  private String username;
  private Role role;
  private Team team;
  private MemberOption memberOption;

  // 클래스 C, this 객체, 객체 자신을 호출한다.
  public Member buildAdminRole() {
    this.role = Role.ADMIN;
    return this;
  }

  // f가 생성한 객체, 메서드 내에서 생성된 지역 객체
  // 인수로부터 반환되는 객체가 아닌, 함수 로직에서 새로 생성한 객체를 의미.
  public Member newAssignAdminUser() {
    Member admin = new Member(this);
    admin.role = Role.ADMIN;
    return admin;
  }

  // f 인수로 넘어온 객체
  public Team assignNewTeam(Team team) {
    this.team.removeMember(this);
    
    team.addMember(this);
    this.team = team;
    return this.team;
  }
  
  // C 인스턴스 변수에 저장된 객체
  public MemberOption disableAccountOfPasswordFail() {
    this.memberOption.lockOfPasswordFail();
    return memberOption;
  }
}
```

간단히 정리하자면 디미터 법칙에서 허용되는 메서드의 반환 객체는 다음과 같다.

1. this
2. 인스턴스 변수
3. 메서드의 인수
4. 메서드 내에서 새로 생성해서 반환하는 객체

이외의 메서드가 반환하는 객체의 메서드는 호출하면 안된다.

이해가 어려울 수 있다. 다시 강조하자면 `낯선이에게 말을 걸지 말아라`(Don't talk to strange) 법칙으로, **한줄에 점을 하나만 찍는다.** 라는 규칙을 지키면 된다.

### 기차 충돌

흔히 다음과 같은 코드를 기차 충돌이라 부른다.

```java
final String outputDir = ctxt.getOptions()
                            .getScratchDir()
                            .getAbsolutionPath();
```

여러 객체가 한 줄로 이어진 기차처럼 보이기 때문이다.
조잡하다 여겨지는 방식이므로 피하는 편이 좋다.

```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutionPath();
```

다음 코드는 디미터 법칙을 위반할까?

함수 하나가 아는 지식이 굉장히 많다. 위 코드를 사용하는 함수는 많은 객체를 탐색할 줄 안다는 말이다.

위 예제가 디미터 법칙을 위반하는지 여부는 ctx, Options. ScratchDir 이 객체인지 아니면 자료 구조인지에 달렸다.

- 객체라면 내부 구조를 숨겨야 하므로 디미터 법칙을 위반한다.
- 자료구조라면 당연히 내부 구조를 노출하므로 디미터 법칙이 적용되지 않는다.

디미터 법칙은 객체 영역에서 논리적으로 적용되는 법칙이기 때문이다.

### 잡종 구조

하지만 단순 자료구조더라도 조회함수와 설정함수를 정의하라 요구하는 프레임워크와 표준(JavaBeans)이 존재한다.

이런 혼란을 틈타, `절반은 객체`, `절반은 자료 구조`인 `잡종 구조`가 탄생한다.

이런 구조는 새로운 함수, 새로운 클래스(자료 구조)를 추가하기 어렵다. 단점만 모아놓은 구조다.

- 중요한 기능을 수행하는 함수 포함 (객체)
- 공개 변수나 공개 조회/설정 함수도 포함 (자료 구조)

잡종 구조는 다른 함수가 절차적인 프로그래밍의 자료 구조 접근 방식처럼 비공개 변수를 사용하고 싶은 유혹에 빠지기 십상이다.

> 때로는 기능 욕심(`Feature Envy`)이라고 부른다.

### 구조체 감추기

만약 다음 `ctxt`, `options`, `scratchDir`이 객체라면 줄줄이 역어선 안된다.

```java
final String outputDir = ctxt.getOptions()
        .getScratchDir()
        .getAbsolutionPath();
```

객체라면 내부 구조를 감춰, 외부에서 어떤 자료 구조를 사용하고 있는지 노출하면 안된다.

```java
// Bad
ctxt.getAbsolutePathOfScratchDirectoryOption();
ctxt.getScratchDirectoryOption().getAbsolutionPath();
```

그렇다하여 다음과 같은 방법은 옳지 않다. ctxt가 **객체라면 뭔가를 하라고 말을해야지**, ~~속을 드러내라고 말하면 안된다.~~

```java
String outFile = outputDir + "/" + className.replace('.', '/') + ".class";
FileOutputStream fout = new FileOutputStream(outFile);
BufferedOutputStream bos = new BufferedOutputStream(fout);
```

1. 추상화 수준을 뒤섞어 놓아 다소 불편하다.
2. 어찌 됐든 위 코드를 분석하여, 함수의 목적을 파악한다.
3. 절대 경로를 얻으려는 이유는 임시 파일을 생성하기 위함이었다.
4. 이를 근거로 함수 이름을 다시 작성한다.
 
```java
// 생성해라 파일 스트림을 !!
ctxt.createScratchFileStream(classFileName);

// Bad
ctxt.getAbsolutePathOfScratchDirectoryOption();
ctxt.getScratchDirectoryOption().getAbsolutionPath();
```

`createScratchFileStream` 메서드명은 다음과 같은 장점이 있다.

- ctxt의 내부 구조를 드러내지 않는다.
- 모듈에서 해당 함수는 자신이 몰라야 하는 여러 객체를 탐색할 필요가 없다.
- 결론적으로 디미터 법칙을 위반하지 않는다.

## 자료 전달 객체

**자료 구조체**의 **전형적인 형태는 공개 변수만 있고 함수가 없는 클래스**다.

이런 자료 구조체를 때로는 자료 전달 객체(DTO, Data Transfer Object)라 한다.

```java
public class Address {
  public String street;
  public String streetExtra;
  public String city;
  public String state;
  public String zip;
} 
```

좀 더 일반적인 형태는 `빈(bean)` 구조다.

```java
public class Address {
  private String street;
  private String streetExtra;
  private String city;
  private String state;
  private String zip;
  
  // constructor, getter/setter properties
} 
```

빈은 비공개 변수를 조회/설정 함수로 조작한다. 일종의 사이비 캡슐화로, 일부 OO 순수주의자나 만족시킬 뿐 별다른 이익을 제공하지 않는다.

### 활성 레코드

활성 레코드는 DTO의 특수한 형태다.

- 자료 구조 특징 
  - getter/setter
- `탐색 함수` 제공
  - save
  - find

공개 변수가 있거나 비공개 변수에 조회/설정 함수가 있는 자료 구조지만, 대게 save나 find와 같은 탐색 함수도 제공한다.

활성 레코드는 데이터베이스 테이블이나 다른 소스에서 자료를 직접 변환한 결과다.

불행히도 활성 레코드에 비즈니스 규칙 메서드를 추가해 이런 자료 구조를 객체로 취급하는 개발자가 흔한다. 하지만 이는 바람직하지 않다. 그러면 자료 구조도 아니고 객체도 아닌 잡종 구조가 나오기 때문이다.

해결책은 간단하다. 활성 레코드는 자료 구조로 취급한다.

비즈니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성한다.
> 여기서 내부 자료는 활성 레코드의 인스턴스일 가능성이 높다.

## 결론

객체는 동작을 공개하고 자료를 숨긴다.

- 기존 동작을 변경하지 않으면 새 객체 타입을 추가하기 쉽다. 
- 반면 기존 객체에 새 동작을 추가하기는 어렵다. 

자료 구조는 별다른 동작 없이 자료를 노출한다.

- 기존 자료 구조에 새 동작을 추가하기 쉽다.
- 기존 함수에 새 자료 구조를 추가하기는 어렵다.

따라서 시스템을 구현할 때, 새로운 자료 타입을 추가하는 유연성이 필요하면 객체가 더 적합하다. 다른 경우로 새로운 동작을 추가하는 유연성이 필요하면 자료구조와 절차적인 코드가 더 적합하다.

우수한 소프트웨어 개발자는 편견없이 이 사실을 이해해 직면한 문제에 최적인 해결책을 선택한다.