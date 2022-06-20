# [Clean Code] Chap9. 단위 테스트


<!--more-->

# 단위 테스트

애자일과 TDD → 단위 테스트의 자동화

## TDD 법칙 3가지

{{< admonition >}}
💡 TDD란?

- 테스트 주도 개발
- 반복 테스트를 이용한 소프트웨어 방법론
- 작은 단위의 테스트 케이스를 작성하고 이를 통과하는 코드를 추가하는 단계를 반복하여 구현
- 실패하는 코드 작성 → 테스트 코드를 성공시키는 코드 작성 → 리펙토링 수행 → …
- JUnit은 대표적인 TDD 도구
  {{< /admonition >}}

1. 실패하는 단위 테스트를 통과할 때까지 실제 코드를 작성하지 않는다.
2. 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성
3. 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성

   → 실제 코드를 사실상 전부 테스트하는 테스트 케이스가 생성되지만, 방대한 테스트 코드는 심각한 관리 문제를 유발할 위험이 존재한다.

## 깨끗한 테스트 코드 유지하기

- 테스트 코드를 마구잡이로 작성하는 것을 TDD의 효과를 저하시킨다
- 테스트 코드 또한 실제 코드 못지않게 중요하다.

### 테스트는 유연성, 유지보수성, 재사용성을 제공

- 코드의 유연성, 유지보수성, 재사용성을 제공하는 버팀목 → 단위 테스트

## 깨끗한 테스트 코드

🔥 가독성🔥 이 중요하다 !!

<details>
<summary>리펙토링 전</summary>
<div markdown="1">

```java
public void testGetPageHieratchyAsXml() throws Exception {
  crawler.addPage(root, PathParser.parse("PageOne"));
  crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
  crawler.addPage(root, PathParser.parse("PageTwo"));

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
}

public void testGetPageHieratchyAsXmlDoesntContainSymbolicLinks() throws Exception {
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

  request.setResource("TestPageOne"); request.addInput("type", "data");
  Responder responder = new SerializedPageResponder();
  SimpleResponse response =
    (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  assertSubString("test page", xml);
  assertSubString("<Test", xml);
}
```

- `addPage`와 `assertSubstring`을 부르는 중복된 코드가 너무 많다
  - 자잘구레한 사항들의 테스트코드의 표현력을 저하시킨다
- 테스트와 무관한 코드가 테스트 의도를 흐린다
  - `PathParser` , `Responser`, `response`
- 읽는 사람을 고려하지 않음

</div>
</details>

<details>
<summary>리펙토링 후</summary>
<div markdown="1">

```java
public void testGetPageHierarchyAsXml() throws Exception {
  makePages("PageOne", "PageOne.ChildOne", "PageTwo");

  submitRequest("root", "type:pages");

  assertResponseIsXML();
  assertResponseContains(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
}

public void testSymbolicLinksAreNotInXmlPageHierarchy() throws Exception {
  WikiPage page = makePage("PageOne");
  makePages("PageOne.ChildOne", "PageTwo");

  addLinkTo(page, "PageTwo", "SymPage");

  submitRequest("root", "type:pages");

  assertResponseIsXML();
  assertResponseContains(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
  assertResponseDoesNotContain("SymPage");
}

public void testGetDataAsXml() throws Exception {
  makePageWithContent("TestPageOne", "test page");

  submitRequest("TestPageOne", "type:data");

  assertResponseIsXML();
  assertResponseContains("test page", "<Test");
}
```

- **BUILD-OPERATE-CHECK 패턴**
  - 테스트 자료 생성 (BUILD)
  - 테스트 자료 조작 (OPERATE)
  - 조작 결과 확인 (CHECK)

</div>
</details>

### 도메인에 특화된 테스트 언어

{{< admonition >}}
💡 DSL이란?

- 도메인 특화 언어
- 특정 분야에 최적화된 프로그래밍 언어
  {{< /admonition >}}

- API 위에다 함수와 유틸리티를 구현 → 테스트에서 사용하는 특수 API
- 다른 테스트 코드에서 재활용이 가능하고, 가독성이 높다

### 이중 표준

<details>
<summary>리펙토링 전</summary>
<div markdown="1">

```java
  @Test
  public void turnOnLoTempAlarmAtThreashold() throws Exception {
    hw.setTemp(WAY_TOO_COLD);
    controller.tic();
    assertTrue(hw.heaterState());
    assertTrue(hw.blowerState());
    assertFalse(hw.coolerState());
    assertFalse(hw.hiTempAlarm());
    assertTrue(hw.loTempAlarm());
  }
```

- 세세한 사항이 너무 많음 → 모든 state를 일일이 읽어야하는 번거로움

</div>
</details>

<details>
<summary>리펙토링 후</summary>
<div markdown="1">

```java
  @Test
  public void turnOnLoTempAlarmAtThreshold() throws Exception {
    wayTooCold();
    assertEquals("HBchL", hw.getState());
  }
```

- 대문자는 켜짐, 소문자는 꺼짐을 의미
- 문자는 항상 {heater, blower, cooler, hi-temp-alarm, lo-temp-alarm} 순서

  - 앞에서 말했던 “그릇된 정보를 피하라”라는 규칙 위반
  - 의미만 알고있다면 문자열을 보고 결과를 빠르게 판단 가능

    ```java
    @Test
    public void turnOnCoolerAndBlowerIfTooHot() throws Exception {
      tooHot();
      assertEquals("hBChl", hw.getState());
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
    ```

- `getstate()`

  ```java
  public String getState() {
    String state = "";
    state += heater ? "H" : "h";
    state += blower ? "B" : "b";
    state += cooler ? "C" : "c";
    state += hiTempAlarm ? "H" : "h";
    state += loTempAlarm ? "L" : "l";
    return state;
  }
  ```

  - 효율상 `StringBuffer`가 더 적합하나, `StringBuffer`는 보기 흉하다
  - 위와 같은 코드에서는 `StringBuffer` 를 안쓰는 디메리트가 매우 적다 → 실제 테스트 환경은 자원이 제한적일 가능성이 높다(CPU나 메모리 등등…)

{{< admonition >}}
💡 실제 환경에서는 사용해서는 안되지만, 테스트 환경에서는 문제없는 방식
CPU나 메모리가 제한적인 환경의 경우, 코드의 깨끗함과는 철저히 무관
**이중표준** → 테스트 환경에서 표준을 다르게 두는 것

{{< /admonition >}}

</div>
</details>

## 테스트 당 assert 하나

- assert가 하나 → 결론이 하나 뿐이다 → 코드를 이해하기 쉽다
- assert문을 병합하기 어려운 경우에는 테스트를 쪼개자

  ```java
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
  ```

### 테스트 당 개념 하나

- **테스트 함수마다 한 개념만 테스트하라**

```java
/**
 * addMonth() 메서드를 테스트하는 장황한 코드
 */
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
```

- 여러 개념을 테스트 하고 있음
  - 5월처럼) 31일로 끝나는 달의 마지막 날짜가 주어지는 경우
    1. (6월처럼) 30일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되어서는 안 된다.
    2. 두 달을 더하면 그리고 두 번째 달이 31일로 끝나면 날짜는 31일이 되어야 한다.
  - (6월처럼) 30일로 끝나는 달의 마지막 날짜가 주어지는 경우
    1. 31일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되면 안 된다.
- 개념들을 정리하여 테스트 함수 하나당 하나의 개념만 테스트하도록 구성해야 함

## F.I.R.S.T

- **빠르게(Fast)**
  - 테스트는 빨라야 한다
- **독립적으로(Independent)**
  - 각 테스트는 서로 의존하면 안된다
  - 한 테스트가 다음 테스트가 실행활 환경을 준비하면 안된다
  - 각 테스트는 어떤 순서로 실행해도 괜찮아야 함
- **반복가능하게(Repeatable)**
  - 어떤 환경에서도 반복 가능해야 함
- **자가검증하는(Self-Validating)**
  - 테스트는 `Boolean`값으로 결과를 내야 함
  - 통과 여부를 알고자 로그 파일을 열도록 해선 안된다.
- **적시에(Timely)**
  - 테스트는 적시에 작성해야 한다
  - 테스트하려는 실제 코드를 구현하기 직전에 구현

