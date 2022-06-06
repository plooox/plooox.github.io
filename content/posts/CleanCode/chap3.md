---
title: "[Clean Code] Chap3. 함수"
date: 2022-06-06T22:04:21+09:00
description: "Clean Code 스터디에서 담당했던 부분 포스트 입니다."
draft: true
tags: ["CleanCode"]
categories: ["CleanCode"]

hiddenFromHomePage: false
hiddenFromSearch: false
twemoji: false
lightgallery: true
fraction: true

toc:
  enable: true
  auto: true
code:
  copy: true
  # ...
math:
  enable: true
  # ...
comment:
  enable: true
---

<!--more-->

# 함수

## Intro

- 함수 → 프로그램의 가장 기본적인 단위
- 처음 읽는 사람이 프로그램 내부를 직관적으로 파악하기 위해서 읽기 쉽고 이해하기 쉬운 함수를 만들어야 한다.
- 함수 - 절망편
  ```java
  public static String testableHtml(PageData pageData, boolean includeSuiteSetup)
      throws Exception {
    Wikipage wikipage = pageData.getWikiPage();
    StringBuffer buffer = new StringBuffer();
    if (pageData.hasAttribute("Test")) {
      if (includeSuiteSetup) {
        WikiPage suiteSetup = PageCrawlerlmpl.getlnheritedPage(
          SuiteResponder.SUITE_SETUP_NAME, wikiPage);
          if (suiteSetup != null) {
            wikiPagePath pagePath =
              suiteSetup.getPageCrawler().getFullPath(suiteSetup);
            String pagePathName = PathParser.render(pagePath);
            buffer.append("include -setup .")
                  .append(pagePathName)
                  .append("\n");
          }
      }
      WikiPage setup =
        PageCrawlerlmpl.getInheritedPage("SetUp", wikiPage);
      if (setup != null) {
        WikiPagePath setupPath =
          wikiPage.getPageCrawler().getFullPath(setup);
        String setupPathName = PathParser.render(setupPath);
        buffer.append("!include -setup .")
              .append(setupPathName)
              .append("\n");
      }
    }
    buffer.append(pageData.getContent());
    if (pageData.hasAttribute("Test")) {
      WikiPage teardown =
        pageCrawlerlmpl.getInheritedPage("TearDown", wikiPage);
      if (teardown != null) {
        WikiPagePath tearDownPath = wikiPage.getPageCrawler().getFullPath(teardown);
        String tearDownPathName = PathParser.render(tearDownPath);
        buffer.append("\n")
              .append("!include -teardown .")
              .append(tearDownPathName)
              .append("\n");
      }
      if (includeSuiteSetup) {
        WikiPage suiteTeardown = PageCrawlerlmpl.getlnheritedPage(
                        SuiteResponder.SUITE_TEARDOWN_NAME,
                        wikiPage
        );
        if (suiteTeardown != null) {
          Wikipagepath pagePath =
            suiteTeardown.getPageCrawler().getFullPath (suiteTeardown);
            String pagePathName = PathParser.render(pagePath);
            buffer.append("!include -teardown .")
                  .append(pagePathName)
                  .append("\n");
        }
      }
    }
    pageData.setContent(buffer.toString());
    return pageData.getHtml();
  }
  ```
- 1차 리펙토링
  ```java
  public static String renderPageWithSetupsAndTeardowns( PageData pageData, boolean isSuite) throws Exception {
  	boolean isTestPage = pageData.hasAttribute("Test");
  	if (isTestPage) {
  		WikiPage testPage = pageData.getWikiPage();
  		StringBuffer newPageContent = new StringBuffer();
  		includeSetupPages(testPage, newPageContent, isSuite);
  		newPageContent.append(pageData.getContent());
  		includeTeardownPages(testPage, newPageContent, isSuite);
  		pageData.setContent(newPageContent.toString());
  	}
  	return pageData.getHtml();
  }
  ```

## 작게 만들어라!

- 함수를 만드는 첫번째 규칙 → **작게!**
- **2 ~ 4줄 길이의 함수**를 만들 것을 권장
  - 1차 리펙토링( 7 ~ 8 줄 )
    ```java
    public static String renderPageWithSetupsAndTeardowns( PageData pageData, boolean isSuite) throws Exception {
    	boolean isTestPage = pageData.hasAttribute("Test");
    	if (isTestPage) {
    		WikiPage testPage = pageData.getWikiPage();
    		StringBuffer newPageContent = new StringBuffer();
    		includeSetupPages(testPage, newPageContent, isSuite);
    		newPageContent.append(pageData.getContent());
    		includeTeardownPages(testPage, newPageContent, isSuite);
    		pageData.setContent(newPageContent.toString());
    	}
    	return pageData.getHtml();
    }
    ```
  - 2차 리펙토링( 3 ~ 4 줄 )
    ```java
    public static String renderPageWithSetupsAndTeardowns( PageData pageData, boolean isSuite) throws Exception {
       if (isTestPage(pageData))
       	includeSetupAndTeardownPages(pageData, isSuite);
       return pageData.getHtml();
    }
    ```

### 블록과 들여쓰기

- `**if문`, `if/else문`, `while문` 등에 들어가는 블록은 한 줄\*\* 이어야 한다
  - 바깥을 감싸는 함수가 작아지고, 코드를 이해하기도 쉬워진다
- 중첩 구조가 생길만큼 함수가 켜져서는 안된다
  - 함수의 들여쓰기 수준은 2단을 넘지 않도록 한다.

## 한가지만 해라

<aside>
💡 함수는 한 가지를 해야 한다. 그 한가지를 잘 해야 한다. 그 한 가지만을 해야 한다.

</aside>

### 한 가지의 기준은?

- 지정된 함수 이름 아래에서 **추상화 수준이 하나**!
- 의미를 유지하면서 코드를 줄이기 불가능한 정도
  - 의미 있는 이름으로 다른 함수를 추출할 수 있다면, 그 함수는여러 작업을 하는 것

## 함수 당 추상화 수준은 하나로!

- **모든 문장의 추상화 수준이 동일**
  [참고](https://onestone-dev.tistory.com/3)

### 내려가기 규칙

- 위에서 아래로 코드 읽기
- 한 함수 다음에는 추상화 수준이 한 단계 낮은 함수가 온다

```
TO 설정 페이지와 해제 페이지를 포함하려면, 설정 페이지를 포함하고, 테스트 페이지 내용을 포함하고,
해제 페이지를 포함한다.
	TO 설정 페이지를 포함하려면, 슈트이면 슈트 설정 페이지를 포함한 후 일반 설정 페이지를 포함한다.
    	TO 슈트 설정 페이지를 포함하려면, 부모 계층에서 `SuiteSetUp` 페이지를 찾아 include 문과
    	페이지 경로를 추가한다.
	TO 부모 계층을 검색하려면, ......
```

## Switch 문

- switch문 특) 작게 만들기 어렵고, 한가지 작업만 시키기도 어렵다
  ```java
  public Money calculatePay(Employee e) throws InvalidEmployeeType {
  	switch (e.type) {
  		case COMMISSIONED:
  			return calculateCommissionedPay(e);
  		case HOURLY:
  			return calculateHourlyPay(e);
  		case SALARIED:
  			return calculateSalariedPay(e);
  		default:
  			throw new InvalidEmployeeType(e.type);
  	}
  }
  ```
  문제점)
  - 함수가 길다. 유형을 추가하게 되면 더욱 길어진다.
  - 한 가지 작업만 수행하지 않음
  - Single Responsibility Principle(SRP) 위배 → 코드를 변경해야할 이유가 다양함
    - SRP(단일 책임 원칙): 클래스는 단 한 개의 책임을 가져야 한다
  - Open Closed Principle(OCP) 위배 → 새 직원 유형 추가할 때마다 코드를 변경
    - OCP(개방-폐쇄 원칙): 확장에 대해 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다
  - 동일한 구조의 함수가 무한정 존재할 수 있음
    - `isPayday`, `deliverPay` 같은 함수가 있다면 똑같이 Switch문이 생기고, 코드가 너저분해진다.
- 다형성을 이용하여 switch문을 ABSTRACT FACTORY에 숨겨 다형적 객체를 생성하는 코드 안에서만 switch를 사용하도록 구성
  - Factory는 Switch문을 사용해 적절한 Employee 파생 클래스 생성
  - 다형성(Polymorphism)을 통해 Switch문 재사용
  ```java
  public abstract class Employee {
  	public abstract boolean isPayday();
  	public abstract Money calculatePay();
  	public abstract void deliverPay(Money pay);
  }
  -----------------
  public interface EmployeeFactory {
  	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
  }
  -----------------
  public class EmployeeFactoryImpl implements EmployeeFactory {
  	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
  		switch (r.type) {
  			case COMMISSIONED:
  				return new CommissionedEmployee(r) ;
  			case HOURLY:
  				return new HourlyEmployee(r);
  			case SALARIED:
  				return new SalariedEmploye(r);
  			default:
  				throw new InvalidEmployeeType(r.type);
  		}
  	}
  }
  ```

## 서술적인 이름을 사용

- 함수가 하는 일을 좀 더 잘 표현할 수 있도록 서술적인 이름을 사용
  - 깨끗한 코드 → 코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행
- 짧고 어려운 이름 <<< 길고 서술적인 이름
- 일관성이 있어야 한다.
  - 모듈 내에서 함수 이름은 같은 문구, 명사, 동사를 사용

## 함수 인수

- 함수에서 이상적인 인수 → 0개(무항) > 1개(단항) > 2개(이항)
  - 3개 이상은 지양
- 인수는 코드 이해에 방해가 되는 요소이므로, 최소화 해야함

### 많이 쓰는 단항 형식

- 인수에 질문을 던지는 경우
  `boolean fileExists(“MyFile”)`
- 인수를 뭔가로 변환해 결과를 변환하는 경우
  `InputStream fileOpen(“MyFile”)`
- 이벤트 함수: 입력 인수만 있음.
  - 이벤트라는 사실이 코드에 명확히 드러나도록 이름과 문맥을 주의해서 선택
  `passwordAttemptFailedNtimes(int attempts)`

### 바람직하지 않은 단항 형식

- `void transform(StringBuffer in)` <<< `StringBuffer transform(StringBuffer out)`
  - 입력 인수를 변환하는 함수라면 변환 결과는 반환해주는 것이 혼동을 방지한다.

### 플래그 인수

- 추하다.
- 함수로 boolean 값을 넘기는 건 끔찍하다.

### 이항 함수

- 일반적으로 이항 함수보다 단항 함수가 이해하기 쉽다
  - 예외) 좌표계 관련 함수
- 일반적으로 2개 인수간의 자연적인 순서가 있어야 함.

### 삼항 함수

- 말해 뭐해…

### 인수 객체

- 인수가 2~3개 이상 필요할 경우, 일부를 독자적인 클라스로 묶어서 사용하자.

`Circle makeCircle(double x, double y, double radius)`

→ `Circle makeCircle(Point center, double radius)`

### 동사와 키워드

- **단항 함수**는 함수와 인수가 **동사/명사 쌍**을 이뤄아 한다
  `writeField(name)`
- 함수 이름에 키워드(인수 이름)을 넣기
  `assertExpectedEqualsActual(expected, actual)` : 인수 순서를 기억할 필요가 없어짐

## 부수효과를 일으키지 마라!

### 부수효과?

- 함수 내의 실행으로 인해 **함수 외부가 영향을 받는 것**

```python
dump = 10
def addWithDump(a, b):
    global dump
    dump += 10
    return a+b+dump

print(addWithDump(1, 2)) # 23
print(addWithDump(1, 2)) # 33
```

### 부수효과는 거짓말이다

한 함수에서는 딱 한가지만 수행

```java
public class UserValidator {
	private Cryptographer cryptographer;
	public boolean checkPassword(String userName, String password) {
		User user = UserGateway.findByName(userName);
		if (user != User.NULL) {
			String codedPhrase = user.getPhraseEncodedByPassword();
			String phrase = cryptographer.decrypt(codedPhrase, password);
			if ("Valid Password".equals(phrase)) {
				**Session.initialize(); // <---- 부수 효과**
				return true;
			}
		}
		return false;
	}
}
```

- `checkPassword` : 암호를 확인
  - 이름만으로 세션을 초기화 한다는 걸 알 수 없음
- 부수 효과는 시간적인 결합을 초래한다.
  - 함수가 특정 상황에서만 호출하도록 강제함 → 혼란 유발
  - 시간적인 결합이 필요하다면 명시해주어야함
    `checkPassword` → `checkPasswordAndInitializeSession`

### 출력인수

[참고자료](https://velog.io/@sansam202/Output-Parameter에-대한-고찰)

- method의 값 반환 목적으로 method로 전달된 parameter를 사용하는 것
- 일반적으로 인수는 함수 **입력**으로 인지
  - appendFooter(s) ← 딱 보면 헷갈린다
    - s를 Footer에 첨부?
    - s에 Footer를 첨부?
- OOP 환경에서 출력 인수를 사용해야할 이유가 없다
  - `this.` 로 전부 커버가 되기 때문

## 명령과 조회를 분리하라

- 함수는 **뭔가를 수행**하거나 **뭔가에 답하거나** 둘 중 하나만 해야한다
- `public boolean set(String attribute, String value);` → `if(set(“username”, “unclebob”))…`
  - 의미가 모호함(동사? 형용사?)
  - 명령과 조회를 분리하여 혼란을 제거
    ```java
    if (attributeExists("username")) {
      setAttribute("username", "unclebob");
    }
    ```

## 오류 코드 보다 예외를 사용하라!

- 오류 코드 사용시, 명령/조회 분리 규칙을 위반할 가능성이 높음
- try/catch를 사용하면 오류 처리 코드가 원래 코드에서 분리되므로 코드가 깔끔해 진다.\*\*\*\*
- 오류 코드 사용
  ```java
  if (deletePage(page) == E_OK) {
  	if (registry.deleteReference(page.name) == E_OK) {
  		if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
  			logger.log("page deleted");
  		} else {
  			logger.log("configKey not deleted");
  		}
  	} else {
  		logger.log("deleteReference from registry failed");
  	}
  } else {
  	logger.log("delete failed"); return E_ERROR;
  }
  ```
- try/catch 사용
  ```java
  try {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
  } catch (Exception e) {
    logger.log(e.getMessage());
  }
  ```
- try/catch 블록도 별도 함수로 분리하는 것이 가독성에 좋다
  ```java
  public void delete(Page page) {
  	try {
  		deletePageAndAllReferences(page);
    	} catch (Exception e) {
    		logError(e);
    	}
  }

  private void deletePageAndAllReferences(Page page) throws Exception {
  	deletePage(page);
  	registry.deleteReference(page.name);
  	configKeys.deleteKey(page.name.makeKey());
  }

  private void logError(Exception e) {
  	logger.log(e.getMessage());
  }
  ```

## 반복하지 마라

- 중복은 소프트웨어에서 모든 악의 근원
- 소스 코드에서 중복을 제거하려는 지속적인 노력 필요

## 구조적 프로그래밍

- 다익스트라 왈, **모든 함수와 함수 내 모든 블록에 입구와 출구가 하나여야 된다.**
- return 문이 하나만 존재해야 한다
- break나 continue를 사용해선 안되며, goto는 쳐다도 보지마라
- 큰 함수에 대해 효과적인 솔루션
  - 작은 함수에서는 break나 continue를 요령껏 써도 된다
  - goto는 큰 함수에서나 쓸모 있는 거니 작은 함수에서는 쓰면 안된다. (결론: 쓰지마라)

## 함수를 어떻게 짜죠?

- 처음에는 길고 복잡하고, 들여쓰기 단계나 중복된 루프도 많다. 인수목록도 길다.
- 처음 작성한 코드들을 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거하고, 메서드를 줄이고, 순서를 바꾼다. + 동시에 코드는 항상 단위 테스트를 통과한다.
