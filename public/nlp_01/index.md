# [NLP] 1.프레임워크와 라이브러리

<!--more-->

# NLP기초
``` 실제 NLP 관련 프로젝트에 들어가기 앞서, NLP 기초를 다지고자 한다```

## 머신러닝 프레임워크와 라이브러리
### Tensorflow
- 머신러닝 오픈소스 라이브러리
- 머신러닝과 딥러닝을 직관적이고 손쉽게 할 수 있도록 설계

### Keras
- 텐서플로우에 대한 추상화된 API 제공
- 백엔드로 텐서플로우 사용, 좀 더 쉽게 딥 러닝을 할 수 있도록 도움

### Gensim
  머신러닝을 사용하여 트픽 모델링과 자연어 처리등을 수행할 수 있게 해주는 오픈소스 라이브러리

### Scikit-learn
- 파이썬 머신러닝 라이브러리
- Naive bayse, SVM등 다양한 머신 러닝 모듈을 불러올 수 있음
- 자체 데이터 제공(아이리스 데이터, 당뇨병 데이터 등등..)
---
### NLTK
  자연어 처리를 위한 파이썬 패키지

### KoNLPy
  한국어 자연어 처리를 위한 형태소 분석기 패키지

---
## 데이터 분석 패키지
### Pandas
- 파이썬 데이터 처리를 위한 라이브러리
- 세 종류의 데이터 구조 사용
    1. Series
        1차원 배열의 값(value)에 각 값에 대응되는 인덱스(index)를 부여할 수 있는 구조
        ```python
        import pandas as pd
        sr = pd.Series([100,200,300,400], index = ["a","b","c","d"])
        print(sr)
        ```
        ```
        a    100
        b    200
        c    300
        d    400
        dtype: int64
        ```
    2. DataFrame
        2차원 리스트를 매개변수로 전달. 행 방향 인덱스와 열 방향 인덱스가 존재 -> 행과 열을 가지는 자료구조
        List, Series, Dict, ndarrays등을 통해 생성할 수 있다.
        ```python
        values = [[1,2,3], [4,5,6], [7,8,9]]
        index = ["I1","I2","I3"]
        columns = ["C1","C2","C3"]

        df = pd.DataFrame(values, index=index, columns=columns)
        print(df)
        ```
        ```
            C1  C2  C3
        I1   1   2   3
        I2   4   5   6
        I3   7   8   9
        ```
         
        ```
        # 리스트로 생성하기
        data = [
            ['1000', 'Steve', 90.72], 
            ['1001', 'James', 78.09], 
            ['1002', 'Doyeon', 98.43], 
            ['1003', 'Jane', 64.19], 
            ['1004', 'Pilwoong', 81.30],
            ['1005', 'Tony', 99.14],
        ]
        df = pd.DataFrame(data)
        print(df)
        ```
        ```
              0         1      2
        0  1000     Steve  90.72
        1  1001     James  78.09
        2  1002    Doyeon  98.43
        3  1003      Jane  64.19
        4  1004  Pilwoong  81.30
        5  1005      Tony  99.14
        ```
         
        ```
        data = { '학번' : ['1000', '1001', '1002', '1003', '1004', '1005'],
        '이름' : [ 'Steve', 'James', 'Doyeon', 'Jane', 'Pilwoong', 'Tony'],
        '점수': [90.72, 78.09, 98.43, 64.19, 81.30, 99.14]}

        df = pd.DataFrame(data)
        print(df)
        ```
        ```
             학번        이름     점수
        0  1000     Steve  90.72
        1  1001     James  78.09
        2  1002    Doyeon  98.43
        3  1003      Jane  64.19
        4  1004  Pilwoong  81.30
        5  1005      Tony  99.14
        ```
    3. Panel

### Numpy
수치 데이터를 다루는 파이썬 패키지
행렬 자료구조인 ndarray를 통해 선형 대수 계산에 많이 사용

1. np.array()
   리스트, 튜플, 배열로부터 ndarray생성
   인덱스가 항상 0으로 시작함
   ```python
   ndarr1 = np.array([1,2,3],[4,5,6])
   print(ndarr1.shape())
   ```
   ```
   (2,3)  # 2 x 3 행렬
   ```
    
    ndarray초기화
    ```
    ndarr2 = np.zeros(2,3) # 모든 값이 0인 2x3 행렬
    ndarr3 = np.full((2,2), 4) # 모든 값이 특정 상수값인 행렬
    ndarr4 = np.eye(3) # 대각선 1, 나머지는 0인 3x3 행렬

    print(ndarr2)
    print(ndarr3)
    ```
    ```
    [[0,0,0]
    [0,0,0]]

    [[4,4]
    [4,4]]

    [[1,0,0]
    [0,1,0]
    [0,0,1]]
    ```
2. np.arange()
   지정해준 범위에 대해서 배열 생성
   `numpy.arange(start, stop, stem, dtype)`
   ```
   a = np.arange(1,10,2) # 1부터 9까지 +2
   ```
   ```
   [1,3,5,7,9]
   ```
3. reshape()
    배열을 다차원으로 변형
    ```
    a = np.array(np.arange(30)).reshape((5,6))
    print(a)
    ```
    ```
    [[ 0  1  2  3  4  5]
    [ 6  7  8  9 10 11]
    [12 13 14 15 16 17]
    [18 19 20 21 22 23]
    [24 25 26 27 28 29]]
    ```
### Matplotlib
데이터를 차트나 플롯으로 시각화하는 패키지


## Machine Learning Workflow
![출처: 딥러닝을 이용한 자연어 처리 입문](https://wikidocs.net/images/page/31947/%EB%A8%B8%EC%8B%A0_%EB%9F%AC%EB%8B%9D_%EC%9B%8C%ED%81%AC%ED%94%8C%EB%A1%9C%EC%9A%B0.PNG "출처: 딥러닝을 이용한 자연어 처리 입문")
(출처: 딥러닝을 이용한 자연어 처리 입문)

### 수집(Acquisition)
머신 러닝을 위한 데이터 수집
자연어 데이터: Corpus(수집된 텍스트의 집합)

### 점검 및 탐색(Inspection and exploration)
수집한 데이터를 점검, 탐색
머신러닝 적용을 위해서 어떻게 데이터를 정제할지 파악
탐색적 데이터 분석(Exploratory Data Analysis, EDA)단계라고도 부름

### 전처리 및 정제(Preprocessing and Cleaning)
토큰화, 정제, 정규화, 불용어 제거 단계

### 모델링 및 훈련(Modeling and Training)
머신러닝에 대한 코드를 작성하는 단계
전처리가 완료된 데이터를 머신러닝 알고리즘을 통해 기계에 학습
대부분의 경우, 모든 데이터를 학습시키지 않고, 훈련용 데이터만 학습시켜 과적합(Overfitting)상황을 피한다.
![출처: 딥러닝을 이용한 자연어 처리 입문](https://wikidocs.net/images/page/31947/%EB%8D%B0%EC%9D%B4%ED%84%B0.PNG "출처: 딥러닝을 이용한 자연어 처리 입문")
(출처: 딥러닝을 이용한 자연어 처리 입문)

### 평가(Evaluation)
테스트용 데이터로 성능을 평가
기계가 예측한 데이터가 테스트용 데이터의 실제 정답과 얼마나 가까운지 측정

### 배포(Deployment)
완성된 모델을 배포


참고) [딥러닝을 이용한 자연어 처리 입문](https://wikidocs.net/book/2155)

