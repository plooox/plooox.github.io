# [Web] 브라우저 렌더링과 HTML 파싱


<!--more-->

# 브라우저 주요기능

> 사용자가 선택한 자원을 서버에 요청하고 브라우저에 표시

- URI(Uniform Resource Identifier)에 의해 결졍된 자원(주로 HTML)에 의해 저장
- HTML과 CSS명세에 따라, HTML 파일을 해석해서 표시

# 기본 구조

![https://d2.naver.com/content/images/2015/06/helloworld-59361-1.png](https://d2.naver.com/content/images/2015/06/helloworld-59361-1.png)

- 사용자 인터페이스
  - 주소 표시줄, 이전/다음 버튼, 북마크 메뉴 등. 요청 페이지를 보여주는 창을 제외한 나머지 모든 부분
- 브라우저 엔진
  - 사용자 인터페이스와 렌더링 엔진 사이의 동작을 제어
- 렌더링 엔진
  - 요청한 콘텐츠를 표시
  - ex) HTML을 요청하면 HTML과 CSS를 파싱하여 화면에 표시
- 통신
  - HTTP 요청과 같은 네트워크 호출에 사용
  - 플랫폼 독립적인 인터페이스이고 각 플랫폼 하부에서 실행됨
- UI 백엔드
  - 콤보 박스와 창 같은 기본적인 장치를 그림
  - 플랫폼에서 명시하지 않은 일반적인 인터페이스로서, OS 사용자 인터페이스 체계를 사용
- 자바스크립트 해석기
  - 자바스크립트 코드를 해석하고 실행
- 자료 저장소
  - 자료를 저장하는 계층

# 렌더링 엔진

- **요청 받은 내용을 브라우저 화면에 표시**
- HTML 및 XML 문서와 이미지를 표시
- 파이어폭스는 게코(Gecko) 엔진, 사파리와 크롬은 웹킷(Webkit) 엔진 사용
- Webkit
  - 최초 리눅스 플랫폼에서 동작하기 위해 제작된 오픈소스 엔진
  - 애플이 맥과 윈도우즈에서 사파리 브라우저를 지원하기 위해 수정

## 동작 과정

![https://d2.naver.com/content/images/2015/06/helloworld-59361-2.png](https://d2.naver.com/content/images/2015/06/helloworld-59361-2.png)

1. **HTML 문서를 파싱하고, 태그를 DOM 노드로 변환**
2. CSS파일과 함께 스타일 요소 파싱
3. 스타일 정보 & HTML 표시 규칙 → **Render Tree 생성**
4. 각 노드가 화면의 정확한 위치에 표시되도록 Render Tree 배치
5. UI 벡엔드에서 Render Tree의 각 노드를 경유하며 그리기과정 수행

## Parsing

- 문서 파싱: 브라우저가 코드를 이해하고, 사용할 수 있는 구조로 변환
- 파싱결과는 문서구조를 나타내는 노드트리(= 파싱트리, 문법트리)
- **어휘 분석**과 **구문 분석**
  - **어휘 분석**
    - 자료 → 토큰으로 분해
    - **토큰**: 유효하게 구성된 단위의 집합체
  - **구문 분석**
    - 언어의 구문 규칙을 적용
    - 규칙에 맞으면 노드가 파싱트리에 추가

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8dbc435a-6a28-4ce8-8e2d-287db76c4f5e/Untitled.png)

- **하향식 파서**와 **상향식 파서** 두 종류가 존재
  - **하향식 파서**
    - 표현식에 해당하는 높은 수준의 규칙을 탐색
    - 점진적으로 다른 규칙을 탐색
    - `2+3-1` → `+3-1` → `3-1` → `-1` → `1` → ``
  - **상향식 파서**
    - 입력값이 오른쪽으로 이동하면서 구문 규칙으로 남는것이 점차 감소
    - 이동-감소 파서

## HTML 파서

- HTML마크업 → 파싱트리
- 문맥 자유 문법이 아님 → 전통적 파서 적용 불가
  - 언어가 너그러움
  - HTML에 대한 브라우저의 관용

### 파싱 알고리즘

- 토큰화
  - 어휘 분석
  - 입력값 → 토큰으로 파싱
  - HTML토큰: 시작태그, 종료태그, 속성 이름, 속성 값
  - 트리 구축
    - `초기상태(자료상태)` - `태그 열림 상태` - `태그 이름 상태` 가 재귀적으로 작동
      ![https://d2.naver.com/content/images/2015/06/helloworld-59361-10.png](https://d2.naver.com/content/images/2015/06/helloworld-59361-10.png)
      ![https://d2.naver.com/content/images/2015/06/helloworld-59361-11.png](https://d2.naver.com/content/images/2015/06/helloworld-59361-11.png)

### 오류처리

- 규칙이 위반된 HTML 태그들이 있어도 브라우저의 파서는 이러한 오류를 찾아서 처리해줌
- HTML에서는 이러한 요구사항 일부를 정의함
- HTML파서는 오류에 대한 관용이 있어야한다
  1. _어떤 태그의 안쪽에 추가하려는 태그가 금지된 것일 때 일단 허용된 태그를 먼저 닫고 금지된 태그는 외부에 추가한다._
  2. _파서가 직접 요소를 추가해서는 안된다. 문서 제작자에 의해 뒤늦게 요소가 추가될 수 있고 생략 가능한 경우도 있다. HTML, HEAD, BODY, TBODY, TR, TD, LI 태그가 이런 경우에 해당한다._
  3. _인라인 요소 안쪽에 블록 요소가 있는 경우 부모 블록 요소를 만날 때까지 모든 인라인 태그를 닫는다._
  4. _이런 방법이 도움이 되지 않으면 태그를 추가하거나 무시할 수 있는 상태가 될 때까지 요소를 닫는다._

---

# 참고) Tokenizer, Lexer, Parser

## Tokenizer

- 어떤 구문에서 의미있는 요소들을 토큰으로 쪼개는 역할

### Token

- 어휘 분석의 단위 → “의미있는 단위”
- HTML에서 토큰은 시작 태그, 종료 태그, 속성 이름과 속성 값

## Lexer

- Tokenizer에 의해 쪼개진 Token의 의미를 분석
- Tokenizer + Lexer = Lexical Analyzer
  - 의미있는 조각을 검출하여 Token생성

## Parser

- Lexical Analyze된 데이터를 가지고 구조화
- AST의 형태를 가짐

### AST

- Abstract Syntax Tree
- 데이터를 의미별로 분리하여 컴퓨터가 이해할 수 있는 구조로 변경시킨 트리

---

[참고자료 - 브라우저는 어떻게 동작하는가?](https://d2.naver.com/helloworld/59361))  
[컴파일러 이론에서 토크나이저(Tokenizer), 렉서(Lexer), 파서(Parse) 의 역할](https://velog.io/@mu1616/%EC%BB%B4%ED%8C%8C%EC%9D%BC%EB%9F%AC-%EC%9D%B4%EB%A1%A0%EC%97%90%EC%84%9C-%ED%86%A0%ED%81%AC%EB%82%98%EC%9D%B4%EC%A0%80Tokenizer-%EB%A0%89%EC%84%9CLexer-%ED%8C%8C%EC%84%9CParse-%EC%9D%98-%EC%97%AD%ED%95%A0)  
[[컴파일러] 토크나이저, 렉서, 파서 (Tokenizer, Lexer, Parser)](https://gobae.tistory.com/94#%EC%25B-%25--%EC%25--%25--%25--%EA%25B-%AC%EB%AC%25B-%25--%ED%25-A%25B-%EB%25A-%AC-AST-)

