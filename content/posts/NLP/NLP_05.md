---
title: "[NLP] 5.텍스트 전처리 - Splitting Data"
date: 2021-08-24T15:01:45+09:00
description: "프로젝트를 위한 NLP 공부"
draft: false
tags: ["NLP"]
categories: ["NLP"]

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
# Splitting Data
## Superivised Learning(지도학습)
```정답이 있는 데이터```를 활용하여 학습  
입력값(X_Label)과 결과값(Y_Label)이 있는 테스트 데이터를 학습시킨다.  

## X_Label과 Y_Label 분리하기
### zip함수
```python
sequence = [['a', 1], ['b', 2], ['c', 3]]
x,y = zip(*sequence)    #데이터 unpack위해 *사용
print(x)
print(y)
```
```python
('a', 'b', 'c')
(1, 2, 3)
```
### DataFrame
```py
import pandas as pd

values = [['당신에게 드리는 마지막 혜택!', 1],
['내일 뵐 수 있을지 확인 부탁드...', 0],
['도연씨. 잘 지내시죠? 오랜만입...', 0],
['(광고) AI로 주가를 예측할 수 있다!', 1]]
columns = ['메일 본문', '스팸 메일 유무']

df = pd.DataFrame(values, columns=columns)

x_label = df['메일 본문']
y_label = df['스팸 메일 유무']
print("[X_Label]")
print(x_label)
print("\n[Y_Label]")
print(y_label)
```
```py
[X_Label]
0          당신에게 드리는 마지막 혜택!
1      내일 뵐 수 있을지 확인 부탁드...
2      도연씨. 잘 지내시죠? 오랜만입...
3    (광고) AI로 주가를 예측할 수 있다!
Name: 메일 본문, dtype: object

[Y_Label]
0    1
1    0
2    0
3    1
Name: 스팸 메일 유무, dtype: int64
```
## 테스트 데이터 분리
x와 y가 이미 분리된 데이터에 대한 테스트 데이터 분리 과정
### sklearn
```train_test_split```을 활용해 테스트 데이터 분리  
```py
x_train, x_test, y_train, y_test = train_test_split(x, y, test_size= 0.2, random_state=42)
```
- x: 독립 변수 데이터(배열, 데이터 프레임)
- y: 종속 변수 데이터, 레이블 데이터
- test_size: 테스트용 데이터 개수, 1보다 작을 시 비율 의미
- train_size: 학습용 데이터의 개수, 1바도 작을 시 비율 의미
- random_state: 난수 시드
```py
import numpy as np
from sklearn.model_selection import train_test_split
x,y = np.arange(10).reshape((5,2)), range(5) #분리된 x,y 데이터 생성
y = list(y)
print(x)
print(y)
print("\n\n")
x_train, x_test, y_train, y_test = train_test_split(x, y, test_size = 0.33, random_state = 1234)
# 전체 데이터 중 1/3을 테스트 데이터로 지정
# random_state의 seed_value = 42

print("[X_train]")
print(x_train)
print("[X_test]")
print(x_test)

print("\n[Y_train]")
print(y_train)
print("[Y_test]")
print(y_test)
```
```py
[[0 1]
 [2 3]
 [4 5]
 [6 7]
 [8 9]]
[0, 1, 2, 3, 4]



[X_train]
[[2 3]
 [4 5]
 [6 7]]
[X_test]
[[8 9]
 [0 1]]

[Y_train]
[1, 2, 3]
[Y_test]
[4, 0]
```

참고) [딥러닝을 이용한 자연어 처리 입문](https://wikidocs.net/book/2155)
<!--more-->
