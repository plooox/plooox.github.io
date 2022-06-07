---
title: "[Clean Code] Chap4. 주석"
date: 2022-06-07T17:20:51+09:00
description: "Clean Code 스터디에서 담당했던 부분 포스트 입니다."
draft: false
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

# 주석

> 나쁜 코드에 주석을 달지 마라. 새로 짜라.

## Intro

- 잘 달린 주석은 유용하나, 경솔하고 근거없는 주석은 코드를 이해하기 어렵게 만든다.
- 주석은 필요악이다.
- 코드로 의도를 표현하지 못해서, 실패를 만회하기 위해 주석을 사용
- 주석은 코드가 오래될 수록 거짓말을 하게 될 가능성이 높다.
  - 주석을 유지 보수하는 것은 현실적으로 어렵기 때문

## 주석은 나쁜 코드를 보완하지 못한다.

- 코드에 주석을 추가하는 이유 → 코드 품질이 나쁘기 때문!
- 표현력이 풍부하고 깔금하며 주석이 없는 코드 >>>> 복잡하고 어수선하며 주석이 많이 달린 코드

## 코드로 의도를 표현하라!

주석 없이도 코드를 통해 충분히 의도를 전달할 수 있다.

```java
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다.
if ((emplotee.flags & HOURLY_FLAG) && (employee.age > 65)
```

```java
if (employee.isEligibleForFullBenefits())
```

## 좋은 주석

### 법적인 주석

- \***\*저작권 정보와 소유권 정보\*\***
  ```java
  // Copyright (C) 2003, 2004, 2005 by Object Montor, Inc. All right reserved.
  // GNU General Public License
  ```

### 정보를 제공하는 주석

- 기본적인 정보를 주석으로 제공
  ```java
  // 테스트 중인 Responder 인스턴스를 반환
  protected abstract Responder responderInstance();
  ```
- 가능하다면 함수이름에 정보를 담는 것이 더 좋음
  ```java
  protected abstract Responder responderBeingTested();
  ```

### 의도를 설명하는 주석

- 결정에 깔린 의도를 설명
  ```java
  public int compareTo(Object o){
       if(o instanceof WikiPagePath){
           ....
       }
       // ~~한 경우에는 우선순위가 높다.
       return 1;
   }
  ```
  ```java
  //  더 나은 예
  public void testConcurrentAddWidgets() {
       ...
       // 스레드를 대량 생성하는 방법으로 경쟁 조건을 만든다.
       for (int i = 0; i > 2500; i++) {
  		    WidgetBuilderThread widgetBuilderThread =
  		        new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
  		    Thread thread = new Thread(widgetBuilderThread);
  		    thread.start();
  		}
       ...
   }
  ```

### 의미를 명료하게 밝히는 주석

- 모호한 인수나 반환값에 대해 표현
- 표준 라이브러리나 변경 불가능한 코드를 사용할 시 유용

```java
public void testCompareTo() throws Exception {
		...
		assertTrue(a.compareTo(a) == 0); // a == a
		assertTrue(a.compareTo(b) != 0); // a != b
		...
}
```

### 결과를 경고하는 주석

- 다른 프로그래머에게 결과를 경고할 목적으로 사용
  ```java
  // 여유 시간이 충분하지 않다면 실행하지 마십시오.
  public void _testWithReallyBigFile() {
  		...
  }
  ```
  ```java
  public static SimpleDateFormat makeStanardHttpDateFormat() {
       // SimpleDateFormat은 스레드에 안전하지 못하기 때문에 각 인스턴스를 독립적으로 생성해야 한다.
       SimpleDateFormat df = new SimpleDateFormat(".......");
       return df;
   }
  ```

### TODO 주석

- ‘앞으로 할 일’을 `//TODO` 주석으로 남겨두면 편하다
  ```java
  // TODO-MdM 현재 필요하지 않다.
  // 체크아웃 모델을 도입하면 함수가 필요 없다.
  protected VersionInfo makeVersion() throws Exception {
      return null;
  }
  ```
  - 프로그래머가 필요하다 여기지만 당장 구현하기 어려운 업무를 기술
    - 더 이상 필요없는 기능 삭제
    - 누군가에게 문제를 봐달라는 요청
    - 더 좋은 이름을 떠올려 달라는 부탁
    - 앞으로 발생할 이벤트에 맞춰 코드를 코치라는 주의

### 중요성을 강조하는 주석

```java
String listItemContent = match.group(3).trim();
// 여기서 trim은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
// 문자열에 시작 공백이 있으면 다른 문자열로 인식되기 때문이다.
new ListItemWidget(this, listItemContent, this.level + 1);
return buildList(text.substring(match.end()));
```

### \***\*공개 API에서 Javadocs\*\***

- 공개 API를 구현한다면 반드시 훌륭한 Javadocs를 작성한다
- Javadocs역시 그릇된 정보를 전달할 수 있음.

## 나쁜주석

- 대다수의 주석이 해당
  - 허술한 코드를 지탱
  - 엉성한 코드를 변명
  - 미숙한 결정을 합리화
  - 프로그래머가 주절거리는 독백

### 주절거리는 주석

- 특별한 이유 없이 의무감으로 다는 주석은 전적으로 시간낭비
  ```java
  public void loadProperties() {
      try {
          String propertiesPath = propertiesLocation + "/" + PROPERTIES_FILE;
          FileInputStream propertiesStream = new FileInputStream(propertiesPath);
          loadedProperties.load(propertiesStream);
      } catch (IOException e) {
          // 속성 파일이 없다면 기본값을 모두 메모리로 읽어 들였다는 의미다.
      }
  }
  ```
  - 저자에게는 의미가 있지만, 다른 사람에게는 전해지지 않음
  - 주석의 의미를 알아내려면 다른 코드를 뒤져보는 수밖에 없음 → 이해가 안되어 다른 모듈까지 뒤져야 하는 주석은 제대로 된 주석이 아니다.

### 같은 이야기를 중복하는 주석 & 오해할 여지가 있는 주석

- 주석은 읽는데 더 시간이 오래 걸린다
  ```java
   // this.closed가 true일 때 반환되는 유틸리티 메서드다.
   // 타임아웃에 도달하면 예외를 던진다.
   public synchronized void waitForClose(final long timeoutMillis)
       throws Exception{
       if(!closed) {
           wait(timeoutMillis);
           if(!closed)
               throw new Exception("~~~~");
       }
   }
  ```
  - `this.closed` 가 `true`여야 메서드가 반환됨 → `true`로 변하는 순간에 메서드가 반환된다고 오해할 수 있음

### 의무적으로 다는 주석

- 모든 함수에 Javadocs를 달거나 모든 변수에 주석을 달아야 할 필요가 없다.
  ```java
  /**
   *
   * @param title CD 제목
   * @param author CD 저자
   * @param tracks CD 트랙 숫자
   * @param durationInMinutes CD 길이(단위: 분)
   */
  public void addCD(String title, String author, int tracks, int durationInMinutes) {
      CD cd = new CD();
      cd.title = title;
      cd.author = author;
      cd.tracks = tracks;
      cd.duration = durationInMinutes;
      cdList.add(cd);
  }
  ```
  - Javadocs를 넣으라는 규칙 때문에 생긴 아무 의미 없는 주석

### 이력을 기록하는 주석

- 소스 코드 관리 시스템(git, bitbucket)이 있으니 필요가 없다.
  ```java
  * 변경 이력 (11-Oct-2001부터)
  * ------------------------------------------------
  * 11-Oct-2001 : 클래스를 다시 정리하고 새로운 패키징
  * 05-Nov-2001: getDescription() 메소드 추가
  ...
  ```

### 있으나 마나 한 주석

- 너무나 당연한 사실을 언급하며 새로운 정보를 제공하지 않는 주석
  ```java
  /*
   * 기본 생성자
   */
  protected AnnualDateRule() {
  	...
  }
  ```

### 무서운 잡음

- 오픈 소스, 웹 사이트에서 가져온 코드 중, 복붙을 잘못해서 생긴 주석들이 해당

### 함수나 변수로 표현 가능한 경우

```java
// 주석 있는 ver
// 전역 목록 <smodule>에 속하는 모듈이 우리가 속한 하위 시스템에 의존하는가?
if (module.getDependSubsystems().contains(subSysMod.getSubSystem()))

// 주석 제거 ver
ArrayList moduleDependencies = smodule.getDependSubSystems();
String ourSubSystem = subSysMod.getSubSystem();
if (moduleDependees.contains(ourSubSystem))
```

### 위치를 표시하는 주석

- 소스 파일에서 특정 위치를 표시하려 주석을 사용하는 경우
  ```java
  // Actions /////////////////////////////////////////////
  ```
  - 가독성을 낮출 수 있음 (특히 뒷부분 슬래시)
  - 반드시 필요한 경우에는 사용해도 됨

### 닫는 괄호에 다는 주석

- 중첩이 심하고 장황한 함수에서는 유용할지도 모르나, 작고 캡슐화된 함수에서는 잡음이다.

### 공로를 돌리거나 저자를 표시하는 주석

- 이게 필요함?
- 소스 코드 관리 시스템이 알아서 기억해주기 때문에 필요가 없다
  ```java
  /* 릭이 추가함 */
  ```

### 주석으로 처리한 코드

- 소스 코드 관리 시스템에 저장하는 편이 좋다.
  ```java
  this.bytePos = writeBytes(pngIdBytes, 0);
  //hdrPos = bytePos;
  writeHeader();
  writeResolution();
  //dataPos = bytePos;
  if (writeImageData()) {
      wirteEnd();
      this.pngBytes = resizeByteArray(this.pngBytes, this.maxPos);
  } else {
      this.pngBytes = null;
  }
  return this.pngBytes;
  ```

### HTML 주석

- 혐오스럽다

### 전역 정보

- 주석은 근처에 있는 코드에 대해서만 기술할 것.
  ```java
  /**
   * 적합성 테스트가 동작하는 포트: 기본값은 <b>8082</b>.
   *
   * @param fitnessePort
   */
  public void setFitnessePort(int fitnessePort) {
      this.fitnewssePort = fitnessePort;
  }
  ```
  - 주석이 기본 포트 정보를 기술하고 있음 → 시스템 어딘가에 있는 다른 함수에 관한 내용임
  - 다른 함수가 바뀌어도 이 주석이 쉽게 바뀔까?

### 너무 많은 정보

- 흥미로운 역사나 관련 없는 정보는 피곤하다.

### 모호한 관계

- 주석 자체가 다시 설명을 필요로하는 경우
  ```java
  /*
   * 모든 픽셀을 담을 만큼 충분한 배열로 시작(여기에 필터 바이트를 더한다).
   * 그리고 헤더 정보를 위해 200바이트를 더함
   */
  this.pngBytes = new byte[((this.width+1) * this.height * 3) + 200];
  ```
  - 필터 바이트는 뭘까?
  - +1, \*3은 왜하지?
  - 한 픽셀이 바이트인가?
  - 200은 뭐지?

### 함수 헤더

- 짧고 한 가지만 수행하며 이름을 잘 붙인 함수 >>> 주석으로 헤더를 추가한 함수

### **비공개 코드에서 Javadocs**

- 공개하지 않을 코드라면 Javadocs는 사용하지 않는게 좋다.
