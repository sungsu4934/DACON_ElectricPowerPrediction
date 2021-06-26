# Dacon주관: 전력사용량 예측 AI 경진대회
https://dacon.io/competitions/official/235736/overview/description
 - 대회개요: 건물정보와 시계열 기후정보를 기반으로 60개 건물별 전력사용량 예측
 - Metric: SMAPE
 - Tool: python
 - Rank: Private (15/336, 상위 4.5%), Public (30/336, 상위 8.9%)

### Feature Enginnering
 - 결측치 처리: 선형보간
 - 일자데이터 전처리
    > 1. 일, 시간, 요일 등 추출 
    > 2. 시계열 속성을 알 수 있도록 day_count 및 hour_count 추가
    > 3. 시간 및 요일을 onehotencoding, 요일은 주말과 평일 구분 binary화
    > 4. 공휴일은 제외 (노이즈라고 판단)
    
 - 군집화: 시간별 전력사용량과 요일별 전력사용량을 기반으로 kmean 알고리즘 및 elbow method를 통해 군집화 (주요 건물 구분)
   > 건물별 점심시간, 근무시간 파생변수 도출
  
 - 지수생성
   > 불쾌지수, 냉방도일, 체감온도, 태양광, 코로나 등
  
 - 날씨
   > 일평균 일조량, 누적강수시간, 폭염지수, 일별최저/최고온도 생성
  
 - 기초통계량 파생변수 생성
   > 건물 별 요일, 시간에 따른 전력사용량 표준편차, 중앙값 파생변수 생성 

### Modeling
 - 최종 활용된 모델: lightgbm
 - 절차
   > 1) Bayesian optimization을 통한 건물별 최적 파라미터 searching
   > 2) 건물별 lightgbm 모델을 5-fold 방식으로 생성 (즉 60개 건물을 5개 fold로 적용결과 300개의 모델 생성)
  
### 시행착오
 - 시행착오로 활용된 모델: Lightgbm, xgboost, catboost, randomforest, extratreeforest
 - 시행착오 방법론
  > 1) 각 알고리즘 별 feature importance를 통해 유의미한 변수 50개만 도출 
  > 2) 각 알고리즘 별 50개 변수만으로 bayesian optimization을 통해 파라미터 최적화
  > 3) 건물별 k-fold 모델링
  > 4) 5개 모델을 활용한 스태킹 앙상블
  
### 총평
 - 스태킹 앙상블까지 시도해봤지만 결론적으로 lgbm 단독 모델이 더 우세한 성능
 - 생각과는 달리 유의미하지 않은 변수를 제거하고 submission을 했을 떄, private score가 감소

### 추후 todo
 - Autoencoder 등을 활용한 feature engineering 적용
 - 이후 다른 대회에 참가한다면 시계열 기반의 딥러닝 알고리즘 LSTM, GRU, CONV1D+LSTM 등을 시도
 - 스태킹 앙상블 시 모든 모델을 앙상블 하는 것이 아닌, 유의미한 소수 모델만 앙상블 (상관계수 고려 등)
