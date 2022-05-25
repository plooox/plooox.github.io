---
title: "[k8s project] 주제 정하기 ~ 설계"
date: 2022-04-03T20:56:50+09:00
description: "k8s 전문가 양성 과정 중 진행하게된 프로젝트"
draft: false
tags: ["k8s", "AWS"]
categories: ["project"]

hiddenFromHomePage: false
hiddenFromSearch: false
twemoji: true
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

` Goorm kubernetes 전문가 양성 과정` 중 프로젝트 진행 과정을 공유합니다.

<!--more-->

# 프로젝트 시작

교육과정이 전부 끝나고, 팀프로젝트가 시작되었습니다. 어쩌다보니 팀장이 되어버렸는데 이왕 하는거 열심히 해보려고 합니다. 🔥🔥🔥

# 주제 정하기

아무래도 교육과정 동안 배웠던 내용이 k8s가 주가 되었으니, 이것을 적극적으로 활용할만한 주제가 무엇이 있는지 팀원들과 의논해보았습니다.  
k8s를 활용한 다른 사람들의 프로젝트들을 찾아본 결과, k8s의 HPA를 이용하여 서버 부하에 따라 적절하게 Scale in/out하는 프로젝트들을 볼 수 있었습니다. ([참고](https://may9noy.tistory.com/351))  
저의 팀도 Auto Scaling을 통해 서버 부하를 원활하게 대응하는 것을 보여줄 수 있는 프로젝트를 해보면 어떨까라는 이야기가 오갔고, 특정 기간에만 서버 부하가 집중되는 서비스가 어떤게 있을까 고민해본 결과 수강신청 사이트를 k8s를 통해 구현해보면 좋겠다는 생각을 했습니다.

<details>
<summary>아이디어 구체화(초안)</summary>
<div markdown="1">

![프로젝트 개요 drawio](https://user-images.githubusercontent.com/82520143/161428979-1d7cbdb7-979b-4361-8b30-17c7f48fb2f3.png)

</div>
</details>

# 기술스택

교육과정동안 배운 내용을 바탕으로 + $\alpha$ 하여 사용할 기술을 선정하였습니다.
![stack](https://user-images.githubusercontent.com/82520143/161428513-4c9f7b71-31c7-4065-ae1a-5e0446c0a6ce.png)

# 설계 (아키텍쳐 & UI)

4명의 팀원들이 2명씩 나누어서 아키텍쳐 설계와 UI설계를 진행하였습니다. 저는 아키텍쳐 설계를 담당하게 되었습니다.

## 아키텍쳐 설계

DevOps 과정을 이수했다보니 설계에 있어서 AWS 클라우드 환경과 CI/CD 신경을 많이 썼습니다.  
eks를 사용하여 k8s클러스터 환경을 AWS환경에서 구현하고, 여기에 모니터링/로깅 시스템을 덧붙이고, Jenkins/ArgoCD로 빌드, 배포환경을 조성하기로 계획하였습니다.  
설계를 해보는 것이 이번이 처음이라 머리가 빠개질것같았지만, 설계를 하고나니 프로젝트를 어떻게 진행하면 될지 윤곽을 잡을 수 있었습니다.
![Architecture drawio](https://user-images.githubusercontent.com/82520143/161429101-25f59785-3f0c-454e-a225-f8a0cda2fb45.png)

## UI 설계

저랑 다른 팀원이 아키텍쳐 설계를 하는 동안, 다른 두 팀원분들께서 웹 UI설계를 해주셨습니다. ([링크](https://ovenapp.io/project/wMbu8qqTbzosHo4l8pp1VhDw4xDAtgnb#rjie2))

# 회고

팀을 꾸려 개발 프로젝트를 하는것이 처음이고, 개발 프로젝트 팀장을 맡는 것도 처음이라 부담이 많이 되기는 합니다.😂  
그래도 팀장이 되니까 프로젝트에 좀 더 열심히 참여하는거 같고, 배우는것도 더 많이 배우는 것 같아 성취감이 있는거 같습니다.  
이제 다음주부터는 FE설계랑 BE설계가 들어갈 예정입니다. 팀 역할을 정할 때 팀원들끼리 역할을 나눌 때 FE/BE 이런식으로 나눈게 아니라 업무별로 나누어 진행을 하기로 정했습니다. 그래서 React와 Springboot 모두 배우면서 진행을 하게 될 예정인데, 많이 바쁠거 같지만 많이 배울 수 있을거 같아 기대됩니다.
