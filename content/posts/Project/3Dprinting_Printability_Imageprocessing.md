---
title: "[Project] 머신러닝을 이용한 3D프린팅의 printability 최적조건 - 이미지 처리"
date: 2022-01-13T17:16:04+09:00
description: "졸업논문 프로젝트(1)"
draft: false
tags: ["3D printing", "ImageProcessing"]
categories: ["project"]

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

#  개요
[https://github.com/plooox/ML_3Dprinter_Scaffold](https://github.com/plooox/ML_3Dprinter_Scaffold)  

졸업논문의 일환으로 팀원들과 “머신러닝을 이용한 3D 프린팅의 printability 최적조건”이라는 주제로 논문을 작성하였습니다. 저는 몇몇 팀원들과 이미지처리부분과 머신러닝 파트를 맡아 진행하였는데, 그 중에서 이미지 처리 과정을 기술하고자 합니다.

논문의 핵심은 3D 프린팅을 할 때 사용되는 여러가지 파라미터를 설정해서 나오는 출력물이 사용자가 원하는 만큼의 품질을 낼 수 있도록 하는 여러 파라미터들의 설정값 조합을 추정하는 과정을 머신러닝을 통해 구할 수 있는지 검증하고, 분석하는 것입니다. 일정 수의 테스트 데이터를 직접 출력하여 머신러닝모델에 학습시키고, 학습된 모델이 추정한 값이 유효한지 검증하였습니다.

이 페이지에서는 아래 빨간 박스인 이미지 처리 부분을 포스트하려고 합니다.
![Workflow](https://user-images.githubusercontent.com/82520143/149294476-24cf9dd4-9f8b-46a0-8c98-7e64c24ddd3d.png)

# 3D printing
## 도면
도면의 경우, AUTOCAD Inventor프로그램을 사용할 수 있는 팀원을 통해 얇은 1layer로 구성된 500μm 반지름을 가진 작은 원형 pit이 100개 나열된 micro-pit 모델을 설계하였고, 3D printer를 이용해 이 도면을 프린팅 했습니다.   
![Micor-pit](https://user-images.githubusercontent.com/82520143/149294814-febdcfc0-c128-4f82-85cb-5903dd69a240.png)

## Parameter selection
실험 설계과정에서 조절하는 파라미터는 출력 속도, 노즐 온도, 베드 온도 총 3가지 파라미터를 선정해 랜덤으로 파라미터 조합을 생성하고 직접 프린트하여 총 278개의 입력 데이터셋을 형성하였습니다. 파라미터 범위는 전체적인 출력 형태를 확인하기 위해 본 실험에서 사용한 3D 프린트의 출력이 불가한 범위를 파악하고 출력이 가능한 범위로 한정하였습니다.

|Parameter|Range|
|------|---|
|출력 온도|180℃ ~ 240℃|
|베드 온도|55℃ ~ 85℃|
|출력 속도|30mm/s ~ 150mm/s|

![Parameter](https://user-images.githubusercontent.com/82520143/149295431-99ece7be-04c9-46e8-914d-35c8572553c4.png)

# Image processing
출력한 출력물을 이용해 논문에서 언급한 “좋은 출력물”을 판단할 수 있도록 데이터화 시키는 과정을 거쳤습니다. "좋은 출력물"은 도면과 유사한 크기의 균일한 공극을 가지는 출력물을 의미합니다. 이를 판단하는 기준인 **Printablility**는 출력물의 각 공극의 픽셀 합과 핏 간 표준편차를 통해 이루어 졌습니다. 공극의 픽셀 합을 통해 도면과 출력물의 유사성을 판단하고, 표준편차를 통해 균일도를 확인할 수 있기 때문입니다. 따라서 픽셀 합이 크고, 표준편차 값이 낮을수록 Printability가 높다고 정의하였습니다. Printability를 구하기 위해 프린터에서 출력한 이미지를 이미지 처리하는 과정이 이루어졌습니다. 이미지 처리는 Python과 Python 라이브러리인 OpenCV(cv2)를 이용하였습니다.  
![스크린샷 2022-01-13 오후 5 59 49](https://user-images.githubusercontent.com/82520143/149298519-8a35f32e-feb8-4819-aa2a-71a46dfcc5b7.png)

## Preprocessing

이미지처리 과정을 자동화 하기 위해, 출력물 사진의 바깥쪽 바탕을 제거하고, 흑/백으로 이진화 하는 전처리 과정을 거쳤습니다.  
  
우선, 이미지에서 micro-pits부분만 뽑아내기 위해, Thresh값을 바꾸어가며 이미지를 이진화 해보았습니다. Thresh값이 높아질수록 외부 노이즈는 줄어들지만, 내부 이미지가 잘 나오지 않았고, Thresh값을 낮추면 이미지 자체의 퀄리티는 높지만 외부에 노이즈가 생기는 것을 확인 할 수 있었습니다.  
![스크린샷 2022-01-13 오후 6 06 27](https://user-images.githubusercontent.com/82520143/149299453-808f782d-2255-46ed-be9b-11e16bd6b292.png) 

따라서 Thresh값을 높게 잡아 BoundingRect값을 잡았을 때 출력물 테두리가 명확히 잡히도록 하고, 테두리를 기준으로 이미지를 Crop하여 출력물만 나타나는 이미지를 새로 구성한 뒤, 이 이미지를 낮은 Thresh값에서 이진화 하는 방식을 취하였습니다.  
![스크린샷 2022-01-13 오후 6 07 08](https://user-images.githubusercontent.com/82520143/149299562-ae4429bc-72f6-4ba0-baa2-0258e744f502.png)

## Pixel Counting
이후 Crop된 이미지를 10 x 10으로 구획을 나누어 구획별 검정 픽셀의 수를 구하였고, 이를 통해 전체 이미지에서 검정 픽셀 수, 표준편차를 구하여 데이터셋으로 구축하였습니다.  
![스크린샷 2022-01-13 오후 6 07 47](https://user-images.githubusercontent.com/82520143/149299654-d799fdbe-da24-40fe-bc86-37350823c951.png)  
