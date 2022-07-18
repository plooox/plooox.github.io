# Git 개념 이해하기


<!--more-->

# Git & Github

## Git

  <details>
  <summary>분산 버전 관리 도구(DVCS)</summary>
  <div markdown="1">

      - 하나의 중앙 서버가 존재하나, 여러 클라이언트가 각자의 pc저장소에 중앙 서버의 전체 사본을 가지고 작업
      - 중앙 서버에서 일부 파트만 가져오는 SVN과 차이를 보인다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile6.uf.tistory.com%2Fimage%2F237B984B58CE95E90BD98B)
_출처) https://git-scm.com/book/ko/v2/시작하기-버전-관리란%3F_

  </div>
  </details>
    
-> 소스코드를 효과적으로 관리

## Github

- git으로 관리되는 프로젝트를 공유할 수 있는 서비스
- 오픈소스의 경우 무료로 서버 제공

# Git 구조

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile6.uf.tistory.com%2Fimage%2F237B984B58CE95E90BD98B)
_출처) https://sjh836.tistory.com/37_

## Git 단계

- Workspace(working Directory): 현재 작업중인 git 프로젝트 파일이 존재하는 내 pc 디렉토리
- Index(Staging Area): 커밋할 변경 내역들의 대기 장소
- Local Repository: 커밋들이 스냅삿으로 기록된 곳
- Remote Repository: 원격 서버에서 관리되는 저장소

## .git 초기 구성

```bash
HEAD
config
description
/branches
/hooks
/objects
/refs
출처: https://sjh836.tistory.com/37 [빨간색코딩:티스토리]
```

## git add

- index에 object이름과 실제 파일 이름이 추가, /objects에 blob타입으로 파일 내용이 추가
- 같은 파일이라도 파일 내용이 달라지면 새로운 object가 생성
  - 파일 내용이 동일하면 새로운 object가 생성
- object명은 SHA1으로 hasing된다

⇒ Workspace의 변경내역을 Index에 올리는 과정

  <details>
  <summary>cf) 옵션</summary>
  <div markdown="1">

- `git add <파일/디렉토리 경로>`: 디렉토리의 일부 변경내용만 Index에 추가
- `git add .`: 현재 디렉토리 내 모든 내용 추가
- `git add -A`: 디렉토리의 모든 내용 추가

  </div>
  </details>

## git commit

- object에 commit 객체와 tree객체가 추가
  - tree객체: commit시 index의 스냅샷 기록
  - commit객체: tree객체명 + Author + Date + Message 기록
    - 2번째 commit부터는 이전 commit객체명을 저장(parent)
  -

### Git 객체

- object: blob과 tree정보
- tree: 디렉토리와 blob정보
- blob: 파일 내용
- tag: commit객체를 가리키며 태그명과 만든이, 주석을 포함

# Git 명령어

- git status
  - Working Directory와 Staging Area의 상태를 확인
  - **Changes to be committed**: index와 blob이 가리키는 객체가 서로 같음(add를 했음)
  - **Changes not staged for commit**: index와 blob이 가리키는 객체가 서로 다름(add를 안함)
- git push
  - 로컬에서 commit한 내용을 remote repo에 업로드
- git branch
  - `HEAD`: 현재 브랜치를 가리킴
  - `.git/refs/heads/{brach name}` 경로에 생성 → 커밋 객체를 가리킴
- git merge
  - 현재 브랜치가 가리키는 커밋 객체를 다른 객체와 병합(HEAD 브랜치가 기준)
- git checkout
  - branch 변경 → HEAD가 바뀜

---

### 참고자료

[git의 원리 (git object를 중심으로)](https://sjh836.tistory.com/37)

[기본적인 Git 사용법과 Pull Request를 통해 프로젝트 기여하기](https://www.secmem.org/blog/2019/04/10/git_pr/)

[Git 이란 ? Git 구조와 용어 간단하게 살펴보자](https://haenny.tistory.com/338)

