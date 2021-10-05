# 2021 산업·데이터공학전공 데이터분석대회
2021 홍익대학교 산업·데이터공학전공 데이터분석대회

: 감성분석을 통한 배달 APP 리뷰 시스템 개선

개발기간 : 2021-06 ~ 2021-09

## 개선의 필요성
한국소비자연맹이 실시한 2021년 배달앱 소비자 인식도 조사에 따르면 배달 APP 이용자의 51.8%가 리뷰 페이지 개선을 원한다는 조사 결과가 있다.

- 리뷰 요약 정보를 보면 영업점에 대해 단순 평균 별점 기반의 평가는 신뢰성과 정보제공이 부족하여 영업점 선별의 어려움
- 이용자가 영업점의 장점 및 이슈를 한 눈에 알기 어려워 리뷰를 일일이 찾아야 하는 노력과 시간이 필요

![그림1](https://user-images.githubusercontent.com/87981867/135284871-418a874e-53a3-4d3c-b6f7-ee29d1651e42.png)

![image](https://user-images.githubusercontent.com/87981867/135284912-93e77a54-dba5-4543-af39-4be0c9118825.png)

![image](https://user-images.githubusercontent.com/87981867/135284937-17134c25-9f30-424c-9035-3d622bef8a6d.png)

영업점에 대해 텍스트 기반 평가 및 정보 제공 서비스를 통해 이용자들이 직관적이고 쉽게 영업점 선별 가능하도록 감성분석을 통해 리뷰 시스템을 개선하도록 한다.

## 한국어 텍스트 처리
### 사용 데이터(Raw Data)
- 요기요 APP과 연동이 되는 요기요 웹 홈페이지에서 무작위로 1개의 영업점(웅이네오돌뼈닭발도) 선정 후, 크롤링하여 리뷰 데이터 2,292건 수집

category : 음식 종류

store : 영업점

id : APP 이용자 ID

review : 이용자가 평가한 텍스트 내용

star : 이용자가 평가한 별점(1점~5점)

![image](https://user-images.githubusercontent.com/87981867/135285624-831e0d34-e7a1-468d-9708-7e25b3478205.png)

### 데이터 전처리
- 리뷰 데이터에 대해 결측치 & 중복, \n(개행문자), 특수 문자 및 이모티콘, 의미 없는 자음 & 모음 제거

- Hanspell 한글 맞춤법 검사 라이브러리 사용하여 띄어 쓰기 및 맞춤법 검사

#### 리뷰 데이터 전처리 전

![image](https://user-images.githubusercontent.com/87981867/135286193-8913f9cc-8045-46f7-a50b-95d89bf3ba66.png)

#### 리뷰 데이터 전처리 후

![image](https://user-images.githubusercontent.com/87981867/135286216-64a3d930-e617-425f-a35d-81d405297fe1.png)

### 형태소 분석 - 명사 단위
- 텍스트 리뷰에서 명사 단위로 형태소 분석을 하기 위해 KoNLPy 라이브러리의 Kkma, Hannanum, Okt 형태소 분석기 비교
- Okt 분석기를 사용하여 명사(Noun) 단위로 추출 

![image](https://user-images.githubusercontent.com/87981867/135286394-9a9cdbd4-00fe-4a0a-b8af-96f148f8f0bc.png)

![image](https://user-images.githubusercontent.com/87981867/135286403-e4db7aaa-ccab-4e43-95ee-fac90305d290.png)

### 불용어(Stopwords) 제거

#### STEP1
[적당히 매운 양념 맛이 최고였어요 너무 맛있게잘 먹었습니다 다만 배달이 조금 더 빨랐으면]

#### STEP2
[‘양념’, ‘맛’, ‘최고’, ‘다만’, ‘배달’ , ‘조금’, ‘더’] 

#### STEP3
[‘양념’, ‘맛’, ‘최고’, ‘배달’] 

위와 같이 기본 불용어들을 1차 제거 후, 분석에 의미 없는 단어들을 추가로 불용어 지정 후 제거한다.

### BoW(Bag of Words)
- 텍스트에서 사용된 단어의 종류와 빈도만을 바탕으로 분석하여 전체 문장 구조를 보지 않고 사용된 단어만 보더라도 대략의 의미 파악이 가능

- 추출된 각 단어에 고유한 인덱스(index) 부여하고 각 인덱스의 위치에 단어의 빈도 수를 기록한 벡터(Vector) 생성

#### Example(예시)

리뷰1.적당히 매운 양념 맛이 최고였어요 너무 맛있게 잘 먹었습니다 다만 배달이 조금 더 빨랐으면

#### 인덱스(index) 부여
(‘양념’ : 0, ‘맛’ : 1, ‘최고’ : 2, ‘배달’ : 3)

#### 빈도 벡터 생성
[1, 2, 1, 1]

### TF-IDF
- 비정형 데이터인 텍스트를 분석하려면 정형 데이터인 숫자로 변환하는 과정이 필요하여, 각 리뷰에 등장하는 단어의 빈도를 행렬로 나타내고 단어에 가중치를 부여 

- 여러 리뷰에서 자주 등장하는 단어는 중요도가 낮다고 판단하며, 특정 리뷰에서만 자주 등장하는 단어는 중요도가 높다고 판단

![그림2](https://user-images.githubusercontent.com/87981867/135288779-5c01522b-de32-4142-864a-eee140aa3965.png)

## Modeling

### 모델 학습
- 독립변수(X)는 텍스트 리뷰 데이터, 종속변수(Y)는 별점에 따른 감성(긍정&부정) 데이터로 설정  

- 텍스트 리뷰를 기반하여 배달 APP 이용자들이 영업점에 대해 긍정과 부정의 여부를 모델 학습을 통해 감성 분류 

X : 배달 APP 이용자의 평가내용(텍스트 리뷰)

Y : 배달 APP 이용자의 긍정 & 부정 감성(긍정 : 1, 부정 : 0)

### 감성 라벨링(Sentiment Labeling)
- 종속변수(Y) 설정을 위해 별점 기반으로  별점 1~3개는 부정(0), 별점 4~5개는 긍정(1)으로 텍스트 리뷰에 대해 감성 라벨링

- 실제 배달 APP 텍스트 리뷰들을 살펴보면 별점 3개까지 영업점에 대해 부정적인 평가를 남기고 있다

![image](https://user-images.githubusercontent.com/87981867/135289659-50aa646e-e213-4f09-a3bf-f846d8a0406a.png)

### Scikit Learn - Logistic Regression
- 분류 모델링을 위해 머신러닝 대표 모델을 적합한 결과 분류 결과의 차이는 크게 나지 않는다
- 하지만 Logistic Regression 모델은 긍/부정(1또는0)의 감성분류가 가능하고 회귀계수를 이용해 긍/부정 키워드 추출이 가능하다는 점에서 프로젝트 목적에 적합하여 선택

#### Logistic Regression
![image](https://user-images.githubusercontent.com/87981867/135290166-4bf58356-71d8-4c13-936f-77debdc305e8.png)

*accuracy: 0.90

*precision: 0.90

*recall: 1.00

*F1: 0.95

#### Support Vector Machine
![image](https://user-images.githubusercontent.com/87981867/135290254-cb74d5e7-126f-4cf5-a7b3-85947584cd61.png)

*accuracy: 0.91

*precision: 0.91

*recall: 1.00

*F1: 0.95

#### Random Forest
![image](https://user-images.githubusercontent.com/87981867/135290290-480d8f70-d9f8-435e-b6c9-6ce2991609c4.png)

*accuracy: 0.90

*precision: 0.91

*recall: 0.99

*F1: 0.95


- 그러나 모델이 리뷰를 모두 긍정(1)으로 분류하는 것을 보아 소수의 부정(0) 리뷰도 긍정(1) 리뷰로 분류하는 (Overfitting) 문제가 발생

과적합을 해결하기 위해서는 데이터 양을 늘리거나 target data 샘플링 조정을 해주어 어느정도 해결이 가능하다. 이미 데이터 양은 충분하기 때문에 target data의 샘플링을 조정하기로 한다.

#### SVMSMOTE
- Oversampling 기법 중 SVMSMOTE을 사용하여 긍/부정 데이터의 비율을 동일하게 맞추어 train data를 재학습

Before Sampling:  Counter({1: 1379, 0: 161})

After oversampling(SVMSMOTE):  Counter({1: 1379, 0: 1379})

![image](https://user-images.githubusercontent.com/87981867/135291764-6412a200-beb7-4e47-b0d5-2ca76a11f9d4.png)

*accuracy: 0.86

*precision: 0.92

*recall: 0.94

*F1: 0.93

모델 재학습을 통해 test data를 분류한 결과, 분류 오류가 발생하였지만 부정 리뷰를 부정(0)으로 분류할 수 있어 과적합이 어느 정도 해결되며 우수한 정확도를 가짐  

### 교차검증(Cross Validation)
- 교차 검증(Cross Validation)을 통해 생성된 모델이 일관성 있는 정확도로 감성 분류를 하였는지 모델 검증

- 모델 재학습을 해도 과적합이 어느정도 있는 상태이기 때문에 Stratified K-Fold 검증 실시(K=4) 

![image](https://user-images.githubusercontent.com/87981867/135292140-94cfc408-a25f-4716-bb87-a258370df16a.png)

K = 1

*accuracy: 0.85942029

K = 2

*accuracy: 0.84347826

K = 3

*accuracy: 0.85050798

K = 4

*accuracy: 0.86357039

모델 평균 정확도 : 0.8542

## Sentiment Classification

### 긍정 & 부정 리뷰 분류
- 학습된 모델을 이용해 ‘웅이네오돌뼈닭발도’ 영업점의 전체 리뷰 데이터에 대해 감성분류 실시

- ‘실제 감성’은 별점에 의해 라벨링된 감성, ‘예측 감성’은 생성된 모델에 의해 분류된 감성, ‘예측 확률’은 모델에 의해 분류된 감성의 예측 확률을 나타냄

![image](https://user-images.githubusercontent.com/87981867/135292614-34e39a60-7fbe-463c-a9a1-8cc1f215c670.png)

![image](https://user-images.githubusercontent.com/87981867/135292679-d52b9d45-f58f-412a-8b88-c7af5883a2ea.png)

![image](https://user-images.githubusercontent.com/87981867/135292687-b99d2190-638e-4030-893c-aaaa5b9766df.png)

예측 값과 실제 값의 비율 차이가 거의 없어 구축한 분류 모델이 감성 분류를 잘 했다고 볼 수 있다.

## Sentiment Keyword

### 긍정 & 부정 키워드 추출
- 회귀계수가 양인 경우는 단어가 긍정적인 영향을 미쳤다고 볼 수 있고, 반면에 음인 경우는 부정적인 영향을 미쳤다고 볼 수 있음

- 회귀계수들을 크기순으로 정렬하면 긍정 & 부정 키워드를 출력하는 지표

![image](https://user-images.githubusercontent.com/87981867/135292965-23d23ddf-fbb8-41aa-bc59-14e67dea865d.png)

![image](https://user-images.githubusercontent.com/87981867/135292988-01d5ed2b-4876-484a-867c-d48280578dff.png)

![image](https://user-images.githubusercontent.com/87981867/135293003-8967d29c-6599-4cb6-b686-c6ce30ede5e8.png)

해당 영업점에 대해서 계란찜, 닭발, 치즈의 맛을 좋게 표현하였고 배달에 대해서 매우 부정적으로 생각함

## 결론
- 텍스트 리뷰 기반의 감성분류(좋아요 & 별로에요) 수치와 감성 키워드를 나타내어 기존의 리뷰 시스템 보다 객관적이며 많은 정보를 제공

- 추출된 키워드를 통해서 이용자가 느끼는 영업점의 장단점을 파악할 수 있고, 이를 기반으로 앞으로 유지해야 할 좋은 서비스와 개선이 필요한 아쉬운 서비스에 대해서도 어느정도 판단 가능

- 직관적인 지표들을 통해 리뷰를 일일이 읽지 않아도 영업점들을 서로 비교하기 원할 
