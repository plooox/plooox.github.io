# [Project] 머신러닝을 이용한 3D프린팅의 printability 최적조건 - 머신러닝

<!--more-->

[https://github.com/plooox/ML_3Dprinter_Scaffold](https://github.com/plooox/ML_3Dprinter_Scaffold)  

저번 포스트에서 작성했던 이미지 처리부분에 이어서, 머신러닝 모델 선정, 학습, 분석 과정을 기술합니다.  
![스크린샷 2022-01-13 오후 10 03 13](https://user-images.githubusercontent.com/82520143/149335249-fee1706c-11a4-4b07-9a6a-30489d028309.png)

# Machine Learning

## 분류 기반 방법론

작성한 논문에서는 Printability가 높은 파라미터 조합을 얻을 수 있는것을 목표로 하였습니다. 따라서 파라미터 값을 지정해주는 회귀 모델보다 최적 파라미터 조합과 파라미터 범위를 추정할 수 있는 분류 모델이 적합하고 판단하였습니다. 분류 모델은 Support Vector Machine(SVM)과 Random Forest(RF)를 기반으로 하여 두개의 머신러닝 모델을 비교하였습니다.  
다음은 표준편차(Std), 전체 픽셀 수(Sum)에 따른 데이터 셋의 분포입니다.  
![스크린샷 2022-01-13 오후 6 33 08](https://user-images.githubusercontent.com/82520143/149303766-e6a24e5c-f1d6-4d4f-99a7-088174856691.png) 

이를 토대로 적절한 임계값을 지정하여 Printability의 “낮음”과 “높음”을 라벨링 하였습니다. 데이터셋의 분포를 “좋음”, “보통”, “나쁨”으로 나누어 3x3 분할된 9개의 Class를 생성하였습니다.  
![스크린샷 2022-01-13 오후 6 33 16](https://user-images.githubusercontent.com/82520143/149303786-53ad6a7b-c67c-47d2-8900-de06c6deb5c6.png) 

##  Train Validation Test
데이터 신뢰성을 위해 **train validation test**에 기반하여 모델을 평가하였습니다. 학습 세트(train set), 검증 세트(validation set), 평가 세트(test set)에 각각 60:20:20 비율로 데이터를 분할하여 학습시켰습니다. Validation set을 training set으로 hold-out 기반으로 반복 학습시켰고, 가장 잘 학습된 모델을 선정하였습니다. 그리고  학습시킨 model에 test set으로 test해서 최종적인 성능을 측정하고, 학습이 잘 되었는지 분석하는 과정을 거쳤습니다.  

![스크린샷 2022-01-13 오후 6 35 06](https://user-images.githubusercontent.com/82520143/149304147-c3f386b6-3505-4b90-be64-09d5aed1f1af.png)  

본 논문에서는 RF의 하이퍼 파라미터 중 tree개수를 정하는 n_estimators, SVM_RBF의 C, gamma값을 알고리즘의 매개변수로 하여 Test를 통해 최적의 값을 찾아 테스트 하였습니다. 테스트 결과 RF에서는 n_estimator가 25개 이상이 되었을 때, 성능이 크게 증가하는 것을 확인 할 수 있었다. 이를 기반으로 n_estimator값을 61로 설정하고, accuracy_score와 f1_score를 측정해보았습니다.

![스크린샷 2022-01-13 오후 6 35 15](https://user-images.githubusercontent.com/82520143/149304178-2ba5bcea-8c5c-47c1-8852-bb48f69e8238.png)  

RBF kernel을 사용하는 SVM 알고리즘에서는 정규화 파라미터인 C(cost)와 곡률 파라미터인 Gamma값을 조정하여 최적의 결정 경계를 출력하는 값을 찾고자 하였습니다.  
![스크린샷 2022-01-13 오후 6 35 26](https://user-images.githubusercontent.com/82520143/149304202-215a6421-8e45-4a25-8413-360239debfdc.png)  

튜닝 결과 gamma = 100, C = 35를 파라미터 값으로 하여 accuracy_score를 측정하였습니다.  
![스크린샷 2022-01-13 오후 6 35 34](https://user-images.githubusercontent.com/82520143/149304223-7178d35f-393e-4b06-84cd-0d475a085d51.png) 

# 결과 및 분석

## Accuracy score
RF와 SVM의 Accuracy_score를 비교한 표입니다.
![스크린샷 2022-01-13 오후 6 38 48](https://user-images.githubusercontent.com/82520143/149304816-023f36d4-a40f-4303-bb66-87a112726847.png)  
두 모델 모두 Validation Set과  Test  Set의 차이가 크게 나지 않은 것으로 보아, 학습이 어느정도 일관되게 잘 이루어졌다는 것을 알 수 있었습니다. 그리고 해당 실험에서는 RF모델로 학습을 시켰을 때, 좀 더 높은 정확도가 보이는 것을 알 수 있었습니다.

## Parameter Set  
이를 기반으로 RF모델을 통해, 전체 파라미터 범위의 데이터셋을 넣고, 분류해보는 과정을 수행해 보았습니다. 학습된 모델이 데이터셋을 분류하는 것을 확인할 수 있었습니다. 분류한 데이터 셋에서 “좋음”을 나타내는 Class 1~3에 속한 임의의 데이터와 “나쁨을 나타내는 Class 7~9에 속한 임의에 데이터를 추출하여 직접 다시 프린팅 해본 결과, 실제로도 “좋음”과 “나쁨”을 구별할 수 있다는 것을 확인할 수 있었습니다.  
![스크린샷 2022-01-13 오후 6 39 04](https://user-images.githubusercontent.com/82520143/149304869-4ce8480e-10dc-4737-9019-5603a9572f65.png)  
![스크린샷 2022-01-13 오후 6 39 13](https://user-images.githubusercontent.com/82520143/149304899-57833881-3a90-4b5c-8411-f2901d8bee0a.png)  

## Feature Importance
RF모델의 경우, ```feature_importance_```를 통해 매개변수 중요도를 파악할 수 있습니다.   
![스크린샷 2022-01-13 오후 6 44 13](https://user-images.githubusercontent.com/82520143/149305765-7c44a980-d5a3-48e7-8b8e-1b995e6e6720.png)

뿐만 아니라 SVM모델에서도 ```coef_```를 통해 중요도를 확인 할 수 있습니다.
![스크린샷 2022-01-13 오후 10 18 53](https://user-images.githubusercontent.com/82520143/149337449-be4e1bac-c256-4052-a831-8b64feabee46.png)
  
노즐 온도, 출력 속도, 베드 온도 순으로 중요도가 높다는 결과값이 나오게 되었는데, 이처럼 RF모델을 사용하게 된다면 3D 프린터 사용자가 파라미터를 조정시 어떤 파라미터를 주의깊게 조정해야하는지 알 수 있다.  


