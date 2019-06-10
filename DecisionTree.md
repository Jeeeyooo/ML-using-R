
# Decision Tree 

분류 / 회귀 모두에 사용할 수 있다.

장점

* 결과를 해석하고 이해하기 쉽다.
* 자료를 가공할 필요가 거의 없다.
    * 다른 기법들의 경우 자료를 정규화하거나 임의의 변수를 생성하거나 값이 없는 변수를 제거해야 하는 경우가 있다.
* 수치 자료와 범주 자료 모두에 적용할 수 있다. 
* 화이트박스 모델. 모델에서 주어진 상황이 관측 가능하다면 불 논리를 이용하여 조건에 대해 쉽게 설명할 수 있다.
    * (인공신경망은 대표적인 블랙 박스 모델 (결과에 대한 설명을 이해하기 어렵기 때문에))
* 안정적이다. 해당 모델 추리의 기반이 되는 명제가 다소 손상되었더라도 잘 동작한다.
* 대규모의 데이터 셋에서도 잘 동작한다. 방대한 분량의 데이터를 일반적인 컴퓨터 환경에서 합리적인 시간 안에 분석할 수 있다.


한계
* 최적의 결정 트리를 학습하는 문제는 NP-완전 문제.
  * 결과적으로, 실질적인 결정 트리 학습 알고리즘은 각 노드에서의 부분 최적값을 찾아내는 탐욕 알고리즘 같은 휴리스틱 기법을 기반으로 하고 있다.
  * 이런 알고리즘들은 최적 결정 트리를 알아낸다고 보장할 수는 없다.
  * 부분 최적화에 의한 영향을 줄이기 위하여 이중 정보 거리(dual information distance, DID)와 같은 방법을 사용하기도 한다.[4]
* 결정 트리 학습자가 훈련 데이터를 제대로 일반화하지 못할 경우 너무 복잡한 결정 트리를 만들 수 있다. (과적합 문제)
  * 이 문제를 해결하기 위해서 가지치기 같은 방법을 사용하여야 한다.
* 결정 트리로는 배타적 논리합이나 패리티, 멀티플렉서와 같은 문제를 학습하기 어렵다.
  * 이런 문제를 학습하기 위해서는 결정 트리가 엄청나게 커지기 때문에 문제의 표현 방법을 바꾸거나 통계 관련 학습법이나 귀납 논리 프로그래밍처럼 더 많은 것을 표현할 수 있는 학습 알고리즘을 사용하여야 한다.
* 각각 서로 다른 수의 단계로 분류가 가능한 변수를 포함하는 데이터에 대하여 더 많은 단계를 가지는 속성 쪽으로 정보 획득량이 편향되는 문제가 있다.
    * 하지만 이 문제는 조건부 추론을 통해 해결이 가능하다.
* 데이터의 특성이 특정 변수에 수직/수평적으로 구분되지 못할 때 분류율이 떨어지고, 트리가 복잡해지는 문제가 발생한다.
    * 신경망 등의 알고리즘이 여러 변수를 동시에 고려하지만 결정트리는 한 개의 변수만을 선택하기 때문에 발생하는 당연한 문제이다.
* 약간의 차이에 따라 (레코드의 개수의 약간의 차이) 트리의 모양이 많이 달라질 수 있다.
    * 두 변수가 비슷한 수준의 정보력을 갖는다고 했을 때, 약간의 차이에 의해 다른 변수가 선택되면 이 후의 트리 구성이 크게 달라질 수 있다.

### example : 타이타닉에서 탑승자의 신원과 생존여부

* Embarked : 어디에서 탔는지
 * C = Cherbourg; Q = Queenstown; S = Southampton

#### 데이터 불러온 뒤 feature를 확인하여 생존자의 신원 파악에 필요 없는 열 삭제


```R
titanic<-read.csv('titanic.csv',header =T)
str(titanic) # PassengerId, Ticket, Name을 삭제
```

    'data.frame':	891 obs. of  10 variables:
     $ Survived   : Factor w/ 2 levels "No","Yes": 1 2 2 2 1 1 1 1 2 2 ...
     $ PassengerId: int  1 2 3 4 5 6 7 8 9 10 ...
     $ Pclass     : Factor w/ 3 levels "1st","2nd","3rd": 3 1 3 1 3 3 1 3 3 2 ...
     $ Name       : Factor w/ 891 levels "Abbing, Mr. Anthony",..: 109 191 358 277 16 559 520 629 417 581 ...
     $ Gender     : Factor w/ 2 levels "female","male": 2 1 1 1 2 2 2 2 1 1 ...
     $ SibSp      : int  1 1 0 1 0 0 0 3 0 1 ...
     $ Parch      : int  0 0 0 0 0 0 0 1 2 0 ...
     $ Ticket     : Factor w/ 681 levels "110152","110413",..: 524 597 670 50 473 276 86 396 345 133 ...
     $ Fare       : num  7.25 71.28 7.92 53.1 8.05 ...
     $ Embarked   : Factor w/ 3 levels "C","Q","S": 3 1 3 3 3 2 3 3 3 1 ...
    


```R
titanic<-titanic[c(-2,-4,-8)]
```

#### RIPPER algorithm을 이용하여 규칙을 만들고 해석


```R
library(RWeka)
titanic_JR<-JRip(Survived ~ ., data = titanic)
titanic_JR
```


    JRIP rules:
    ===========
    
    (Gender = female) and (Pclass = 1st) => Survived=Yes (92.0/3.0)
    (Gender = female) and (Pclass = 2nd) => Survived=Yes (76.0/6.0)
    (Gender = female) and (Fare <= 22.3583) => Survived=Yes (116.0/48.0)
     => Survived=No (605.0/113.0)
    
    Number of Rules : 4
    


1. 여성이고 1등석에 앉았으면, 살아남았다. (92개 사례 중 오류 3개 발생)
2. 여성이고 2등석에 앉았으면, 살아남았이다. (76개 사례 중 오류 6개 발생)
3. 여성이고 운임을 22.3583 이하 냈으면, 살아남았다. (116개 사례 중 오류 48개 발생)
4. 나머지는 살아남지 못했다. (605개 사례 중 오류 113개 발생)

#### C5.0 algorithm을 이용하여 규칙을 만들고 해석


```R
library(C50)
titanic_C50 <- C5.0(Survived ~ ., data = titanic, rules = T)
titanic_C50
```

    Warning message:
    "package 'C50' was built under R version 3.5.3"


    
    Call:
    C5.0.formula(formula = Survived ~ ., data = titanic, rules = T)
    
    Rule-Based Model
    Number of samples: 891 
    Number of predictors: 6 
    
    Number of Rules: 8 
    
    Non-standard options: attempt to group attributes
    



```R
summary(titanic_C50)
```


    
    Call:
    C5.0.formula(formula = Survived ~ ., data = titanic, rules = T)
    
    
    C5.0 [Release 2.07 GPL Edition]  	Sat Jun 08 15:06:56 2019
    -------------------------------
    
    Class specified by attribute `outcome'
    
    Read 891 cases (7 attributes) from undefined.data
    
    Rules:
    
    Rule 1: (577/109, lift 1.3)
    	Gender = male
    	->  class No  [0.810]
    
    Rule 2: (491/119, lift 1.2)
    	Pclass = 3rd
    	->  class No  [0.757]
    
    Rule 3: (54/1, lift 2.5)
    	Gender = female
    	Fare > 15.2458
    	Embarked = C
    	->  class Yes  [0.964]
    
    Rule 4: (170/9, lift 2.5)
    	Pclass in {1st, 2nd}
    	Gender = female
    	->  class Yes  [0.942]
    
    Rule 5: (9, lift 2.4)
    	Gender = female
    	Fare <= 13.8625
    	Embarked = C
    	->  class Yes  [0.909]
    
    Rule 6: (7, lift 2.3)
    	SibSp <= 0
    	Fare > 26
    	Fare <= 26.3875
    	->  class Yes  [0.889]
    
    Rule 7: (33/6, lift 2.1)
    	Gender = female
    	Parch <= 0
    	Embarked = Q
    	->  class Yes  [0.800]
    
    Rule 8: (19/4, lift 2.0)
    	Gender = female
    	Parch > 0
    	Fare <= 20.575
    	Embarked = S
    	->  class Yes  [0.762]
    
    Default class: No
    
    
    Evaluation on training data (891 cases):
    
    	        Rules     
    	  ----------------
    	    No      Errors
    
    	     8  149(16.7%)   <<
    
    
    	   (a)   (b)    <-classified as
    	  ----  ----
    	   530    19    (a): class No
    	   130   212    (b): class Yes
    
    
    	Attribute usage:
    
    	 90.24%	Gender
    	 74.19%	Pclass
    	 12.91%	Embarked
    	  9.99%	Fare
    	  5.84%	Parch
    	  0.79%	SibSp
    
    
    Time: 0.0 secs
    


+ 남자면 살아남지 못했다. (0.810)
+ 3등석이면 살아남지 못했다. (0.757)
+ 여자이고 요금이 15.2458보다 많으며 C = Cherbourg 에서 승선했으면 살아남았다. (0.964)
+ 1등석, 2등석이고 여자이면 살아남았다 (0.942)
+ 여자이고 요금이 13.8625 이하이며 C = Cherbourg 에서 승선했으면 살아남았다. (0.909)

...

#### party 패키지에서 ctree를 사용해 시각화를 진행하고 싶었으나, jupyter 상에서는 돌아가지 않음
library(party)

tree <- ctree(Class ~ Max.L.Ra + Comp, data = titanic)

plot(tree)


![image.png](attachment:image.png)

![png](decisintreeoutput.png)

![png](/decisintreeoutput.png)

### example : 서비스 만족도를 예측하는 회귀트리


```R
ashopping<-read.csv("Ashopping.csv")
str(ashopping)
library(rpart)
shop<-rpart(서비스.만족도~.,data = ashopping)
shop
```

    'data.frame':	1000 obs. of  42 variables:
     $ 고객ID                       : int  1 2 3 4 5 6 7 8 9 10 ...
     $ 이탈여부                     : int  0 1 0 0 0 0 0 0 0 0 ...
     $ 총.매출액                    : int  4007080 3168400 2680780 5946600 13745950 3323610 2369340 12717240 6899100 4461040 ...
     $ 방문빈도                     : int  17 14 18 17 73 26 6 109 17 19 ...
     $ X1회.평균매출액              : int  235711 226314 148932 349800 188301 127831 394890 116672 405829 234792 ...
     $ 할인권.사용.횟수             : int  1 22 6 1 9 20 30 4 17 27 ...
     $ 총.할인.금액                 : int  5445 350995 186045 5195 246350 348145 380945 354735 364895 357645 ...
     $ 고객등급                     : int  1 2 1 1 1 1 1 1 1 1 ...
     $ 구매유형                     : int  4 4 4 4 2 4 1 2 4 4 ...
     $ 클레임접수여부               : int  0 0 1 1 0 0 0 1 0 0 ...
     $ 구매.카테고리.수             : int  6 4 6 5 6 5 6 6 6 5 ...
     $ 거주지역                     : int  6 4 6 5 6 5 6 6 6 5 ...
     $ 성별                         : int  1 1 1 1 0 1 1 1 1 1 ...
     $ 고객.나이대                  : int  4 1 6 6 6 5 6 4 3 6 ...
     $ 거래기간                     : int  1079 537 1080 1019 1086 1089 874 1093 1027 986 ...
     $ 할인민감여부                 : int  0 0 0 0 0 0 0 0 0 0 ...
     $ 멤버쉽.프로그램.가입전.만족도: int  5 2 6 3 5 7 5 3 2 6 ...
     $ 멤버쉽.프로그램.가입후.만족도: int  7 3 6 5 6 4 5 3 4 7 ...
     $ Recency                      : int  7 2 7 7 7 7 7 7 7 6 ...
     $ Frequency                    : int  3 3 3 3 6 4 2 7 3 3 ...
     $ Monetary                     : int  4 3 2 5 7 3 1 6 5 4 ...
     $ 상품.만족도                  : int  6 2 4 3 5 7 5 6 4 6 ...
     $ 매장.만족도                  : int  5 5 6 5 6 7 2 7 3 6 ...
     $ 서비스.만족도                : int  6 4 7 5 6 7 3 7 6 7 ...
     $ 상품.품질                    : int  7 6 6 6 5 7 5 6 6 7 ...
     $ 상품.다양성                  : int  7 7 7 6 6 7 4 6 6 7 ...
     $ 가격.적절성                  : int  6 6 6 6 6 7 5 6 7 6 ...
     $ 상품.진열.위치               : Factor w/ 5 levels ".","4","5","6",..: 5 4 5 3 3 5 3 4 4 5 ...
     $ 상품.설명.표시               : Factor w/ 5 levels ".","4","5","6",..: 4 1 1 4 4 5 3 4 4 4 ...
     $ 매장.청결성                  : int  6 7 6 6 5 6 4 6 6 6 ...
     $ 공간.편의성                  : int  7 7 6 6 6 6 5 6 6 7 ...
     $ 시야.확보성                  : int  6 6 6 5 6 6 6 5 5 6 ...
     $ 음향.적절성                  : int  6 6 6 6 6 6 4 7 6 6 ...
     $ 안내.표지판.설명             : int  6 6 6 6 5 5 6 6 6 6 ...
     $ 친절성                       : int  6 5 7 6 5 5 5 6 6 6 ...
     $ 신속성                       : int  6 3 7 6 6 5 4 5 6 6 ...
     $ 책임성                       : int  6 6 6 6 6 6 5 6 6 6 ...
     $ 정확성                       : int  6 6 6 5 5 6 5 6 5 6 ...
     $ 전문성                       : int  6 6 7 6 6 5 4 6 6 6 ...
     $ D1                           : int  0 0 0 0 1 0 0 1 0 0 ...
     $ D2                           : int  0 0 0 0 0 0 0 0 0 0 ...
     $ D3                           : int  1 1 1 1 0 1 0 0 1 1 ...
    

    Warning message:
    "package 'rpart' was built under R version 3.5.3"


    n= 1000 
    
    node), split, n, deviance, yval
          * denotes terminal node
    
      1) root 1000 1424.0640000 5.544000  
        2) 방문빈도< 9.5 127  282.2205000 4.291339  
          4) 상품.만족도< 2.5 23   32.8695700 3.304348 *
          5) 상품.만족도>=2.5 104  221.9904000 4.509615  
           10) 매장.만족도< 3.5 29   40.2069000 3.689655 *
           11) 매장.만족도>=3.5 75  154.7467000 4.826667  
             22) 고객ID>=881 8   26.0000000 3.500000 *
             23) 고객ID< 881 67  112.9851000 4.985075  
               46) 전문성< 5.5 25   65.4400000 4.320000  
                 92) 고객.나이대>=4.5 11   24.7272700 3.454545 *
                 93) 고객.나이대< 4.5 14   26.0000000 5.000000 *
               47) 전문성>=5.5 42   29.9047600 5.380952 *
        3) 방문빈도>=9.5 873  913.5693000 5.726231  
          6) 상품.만족도< 5.5 565  606.0991000 5.454867  
           12) 매장.만족도< 3.5 52   70.5192300 4.403846  
             24) 방문빈도< 13.5 17   18.1176500 3.588235 *
             25) 방문빈도>=13.5 35   35.6000000 4.800000 *
           13) 매장.만족도>=3.5 513  472.3158000 5.561404  
             26) 멤버쉽.프로그램.가입전.만족도< 2.5 93   67.1182800 4.795699 *
             27) 멤버쉽.프로그램.가입전.만족도>=2.5 420  338.5976000 5.730952  
               54) X1회.평균매출액>=824227 7   20.0000000 4.000000 *
               55) X1회.평균매출액< 824227 413  297.2688000 5.760291  
                110) 매장.만족도< 5.5 302  229.0894000 5.645695 *
                111) 매장.만족도>=5.5 111   53.4234200 6.072072 *
          7) 상품.만족도>=5.5 308  189.5422000 6.224026  
           14) 멤버쉽.프로그램.가입전.만족도< 6.5 280  171.7000000 6.150000 *
           15) 멤버쉽.프로그램.가입전.만족도>=6.5 28    0.9642857 6.964286 *


#### cp 값을 이용하여 사전가지치기는 어떤 수준에서 이루어져야할지 고려해보고 rpartXse 함수를 이용해서 1-se rule을 이용한 사후 가지치기를 진행해보세요.



```R
printcp(shop)
```

    
    Regression tree:
    rpart(formula = form, data = data, cp = cp, minsplit = minsplit)
    
    Variables actually used in tree construction:
    [1] 매장.만족도                   멤버쉽.프로그램.가입전.만족도
    [3] 방문빈도                      상품.만족도                  
    
    Root node error: 1424.1/1000 = 1.4241
    
    n= 1000 
    
            CP nsplit rel error  xerror     xstd
    1 0.160298      0   1.00000 1.00165 0.056049
    2 0.082811      1   0.83970 0.88023 0.045577
    3 0.045596      2   0.75689 0.82148 0.044302
    4 0.019307      4   0.66570 0.73571 0.042177
    

사전가지치기: 1개-2개 사이에서 cp가 비교적 크게 감소하는 것을 확인


```R
library("DMwR")
shop<-rpartXse(서비스.만족도~.,data=ashopping)
shop
```


    n= 1000 
    
    node), split, n, deviance, yval
          * denotes terminal node
    
     1) root 1000 1424.06400 5.544000  
       2) 방문빈도< 9.5 127  282.22050 4.291339 *
       3) 방문빈도>=9.5 873  913.56930 5.726231  
         6) 상품.만족도< 5.5 565  606.09910 5.454867  
          12) 매장.만족도< 3.5 52   70.51923 4.403846 *
          13) 매장.만족도>=3.5 513  472.31580 5.561404  
            26) 멤버쉽.프로그램.가입전.만족도< 2.5 93   67.11828 4.795699 *
            27) 멤버쉽.프로그램.가입전.만족도>=2.5 420  338.59760 5.730952 *
         7) 상품.만족도>=5.5 308  189.54220 6.224026 *


#### k-fold 기법을 이용해 여러 트리를 실험해보고 가장 예측력이 좋은 모델을 bestscore 함수를 이용해서 찾으세요. 또한, rpart의 se값은 0,0.5,1를 각각 가지는 세 가지 모델을 만들고, cvsettings는 4번 반복, 10개의 폴더 생성, set.seed는 1234로 설정해주세요.



```R
cv.rpart <- function(form,train,test,...) {
  m <- rpartXse(form,train,...)
  p <- predict(m,test)
  mse <- mean((p-resp(form,test))^2)
  c(nmse=mse/mean((mean(resp(form,train))-resp(form,test))^2))
}


cv.lm <- function(form,train,test,...) {
  m <- lm(form,train,...)
  p <- predict(m,test)
  p <- ifelse(p < 0,0,p)
  mse <- mean((p-resp(form,test))^2)
  c(nmse=mse/mean((mean(resp(form,train))-resp(form,test))^2))
}

res <- experimentalComparison(
  c(dataset(서비스.만족도 ~ .,ashopping,'서비스.만족도')),
  c(variants('cv.lm'),
    variants('cv.rpart',se=c(0,0.5,1))),
  cvSettings(3,10,1234))



```

    
    
    #####  CROSS VALIDATION  EXPERIMENTAL COMPARISON #####
    
    ** DATASET :: 서비스.만족도
    
    ++ LEARNER :: cv.lm  variant ->  cv.lm.v1 
    
     3 x 10 - Fold Cross Validation run with seed =  1234 
    Repetition  1 
    Fold:  1

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      2

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      3

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      4

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      5

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      6

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      7

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      8

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      9

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      10

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

    
    Repetition  2 
    Fold:  1

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      2

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      3

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      4

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      5

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      6

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      7

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      8

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      9

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      10

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

    
    Repetition  3 
    Fold:  1

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      2

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      3

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      4

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      5

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      6

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      7

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      8

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      9

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

      10

    Warning message in predict.lm(m, test):
    "prediction from a rank-deficient fit may be misleading"

    
    
    
    ++ LEARNER :: cv.rpart  variant ->  cv.rpart.v1 
    
     3 x 10 - Fold Cross Validation run with seed =  1234 
    Repetition  1 
    Fold:  1  2  3  4  5  6  7  8  9  10
    Repetition  2 
    Fold:  1  2  3  4  5  6  7  8  9  10
    Repetition  3 
    Fold:  1  2  3  4  5  6  7  8  9  10
    
    
    ++ LEARNER :: cv.rpart  variant ->  cv.rpart.v2 
    
     3 x 10 - Fold Cross Validation run with seed =  1234 
    Repetition  1 
    Fold:  1  2  3  4  5  6  7  8  9  10
    Repetition  2 
    Fold:  1  2  3  4  5  6  7  8  9  10
    Repetition  3 
    Fold:  1  2  3  4  5  6  7  8  9  10
    
    
    ++ LEARNER :: cv.rpart  variant ->  cv.rpart.v3 
    
     3 x 10 - Fold Cross Validation run with seed =  1234 
    Repetition  1 
    Fold:  1  2  3  4  5  6  7  8  9  10
    Repetition  2 
    Fold:  1  2  3  4  5  6  7  8  9  10
    Repetition  3 
    Fold:  1  2  3  4  5  6  7  8  9  10
    


```R
bestScores(res)
res
summary(res)
```


<strong>$서비스.만족도</strong> = <table>
<thead><tr><th></th><th scope=col>system</th><th scope=col>score</th></tr></thead>
<tbody>
	<tr><th scope=row>nmse</th><td>cv.lm.v1 </td><td>0.6638608</td></tr>
</tbody>
</table>




    
    ==  Cross Validation  Experiment ==
    
     3 x 10 - Fold Cross Validation run with seed =  1234 
    4  learning systems
    tested on  1  data sets


    
    == Summary of a  Cross Validation  Experiment ==
    
     3 x 10 - Fold Cross Validation run with seed =  1234 
    
    * Data sets ::  서비스.만족도
    * Learners  ::  cv.lm.v1, cv.rpart.v1, cv.rpart.v2, cv.rpart.v3
    
    * Summary of Experiment Results:
    
    
    -> Datataset:  서비스.만족도 
    
    	*Learner: cv.lm.v1 
                  nmse
    avg     0.66386076
    std     0.09128129
    min     0.51052289
    max     0.87033983
    invalid 0.00000000
    
    	*Learner: cv.rpart.v1 
                 nmse
    avg     0.7329059
    std     0.0947100
    min     0.6021429
    max     0.9709654
    invalid 0.0000000
    
    	*Learner: cv.rpart.v2 
                  nmse
    avg     0.74404172
    std     0.07883506
    min     0.59938409
    max     0.89441552
    invalid 0.00000000
    
    	*Learner: cv.rpart.v3 
                  nmse
    avg     0.75513963
    std     0.08018485
    min     0.61141727
    max     0.96553109
    invalid 0.00000000
    
    

#### mse가 가장 낮은 모델인 맨 첫번째 모델이 선택된다


```R

```


```R

```


```R

```


```R

```


```R

```


```R

```
