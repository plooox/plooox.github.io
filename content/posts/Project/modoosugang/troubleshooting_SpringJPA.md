---
title: "[Trouble Shooting] JPA에서 Like, Contains구문 사용시 에러"
date: 2022-04-19T16:05:37+09:00
description: "Springboot 프로젝트 중 발생한 Trouble 해결 과정"
draft: false
tags: ['springboot']
categories: ['troubleshooting']

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
React + Springboot + Mysql을 사용한 프로젝트를 진행하던 중 발생한 문제를 다룹니다.
<!--more-->
# 문제상황 
```React.useEffect()```를 사용해 웹 페이지에 접속과 동시에, api 서버 요청을 통해 특정 문자열이 포함된 데이터를 DB에서 가져오는 기능을 구현하고 있었습니다.  JpaRepository를 사용하여 Mysql DB서버에 특정 문자열이 포함된 DB를 사용하고자 다음과 같은 Service 코드를 작성하였습니다.   
```java
// Repository
package com.example.modoosugang_be.Repository;

import com.example.modoosugang_be.Domain.Lecture;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.util.List;

public interface LectureRepository extends JpaRepository<Lecture, Long> {
    List<Lecture> findAllByProfessorContains(String professor);
}
```
```java
// Service
package com.example.modoosugang_be.Service;

import com.example.modoosugang_be.Domain.Lecture;
import com.example.modoosugang_be.Repository.LectureRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
@RequiredArgsConstructor
public class LectureService {

    private final LectureRepository lectureRepository;

    public List<Lecture>callUnivLecture(String univ) {
        List<Lecture> lectures = lectureRepository.findAllByProfessorContains(univ);

        return lectures;
    }
}

```
원래대로라면 Jpa를 통해 명명된 Method들이 알아서 잘 작동되어야 하는데, 웬걸? 이상한 문제상황에 직면했다.  
![image](https://user-images.githubusercontent.com/82520143/163941583-2e3bfe6c-dc12-4d97-ab25-c16f143d05f8.png)  
찾아보니 JPA사용 중 ```java.lang.IllegalArgumentException``` 은 Entity 작성을 잘못하거나 하면 종종 발생하는 에러라고 한다.  그런데 내 경우에는 특이하게도 서버를 돌리는 첫 시행에는 돌아가다가, 반복 시행시 에러가 나는 기이한(?) 현상이 발생하였다.   
 

# 원인분석 + 해결
에러를 해결하기 위해 여러 가지 시도를 해봤었다.
1. Entity & DB 매칭 재확인
2. Jpa에서 contains말고 startWith, like 사용해보기
3. DB에 Column을 추가해서 contains말고 findAllBy만 사용하기
시도해보니 3번 방법말고는 해결이되지 않았었다... 진짜로 DB를 엎어야되나 생각하던 중, [이런 걸](https://github.com/spring-projects/spring-data-jpa/issues/2472) 확인해볼 수 있었다.   

![image](https://user-images.githubusercontent.com/82520143/163948201-ea81a8f4-f26a-4929-b325-690aec9a4f53.png)  
내 경우와는 똑같은 것은 아니었지만 위 Issue를 보니, Springboot 최신버전과 Hibernate를 같이 사용시 startingWith, contains, startsWith, Like와 같은 구문을 사용시 동일한 에러를 발생시킨다는 걸 알 수 있었다.  
Participants들의 코멘트들을 보니, Hibernate버전을 다운그레이드 하거나, @Query를 이용해 직접 쿼리문을 작성하면 해결되는 것을 확인할 수 있었다.  팀 단위로 굴러가는 프로젝트다보니, 버전을 수정하는 것보다 직접 쿼리문을 작성하는 방법을 선택했다.
```java
// ~중략~

public interface LectureRepository extends JpaRepository<Lecture, Long> {

    @Query(value = "SELECT v FROM Lecture v WHERE v.professor Like :univ%")
    List<Lecture> findLecture(@Param("univ")String univ);
}
```
평소에 에러가 발생하면 구글이나 Stackoverflow만 검색했었는데, Github도 유심히 봐야겠다.