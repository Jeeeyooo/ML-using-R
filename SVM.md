
# SVM

### 선형 분류 모델
선형 분류 모델은 선형을 이용해서 데이터를 분리하는 것으로, 가까운 데이터를 기반으로 분류하는 nearest neighborlearning과 X변수와 Y변수간의 선형관계를 보여주는 linear regressionmodeling 개념이 기반이 되었다고 할 수 있다.

### 거리 기반 머신러닝 기법들

#### kNN 알고리즘
학습 데이터 중에서 k개의 가장 가까운 사례를 사용하여 수치 예측 및 분류를 하는 방식으로 거리에 따라서 영향력을 달리 주고 싶을 때 사용한다. 하지만 과적합화의 문제가 있을 수 있고 이상치에 예민하다.

#### SVM 알고리즘
과적합화 문제를 벌칙(penalty)항을 이용하여 이 문제를 최소화시킬 수 있고 일반화에 있어서 이상치에 둔감하기 때문에 높은 일반화 성능을 가질 수 있다. 비확률적 이진 선형 분류 모델을 만들고 선형, 비선형 분류에 사용될 수 있다. 또한 고차원 공간에서는 효율적인 분류를 위해 커널트릭을 사용하기도 한다.

#### k-means 알고리즘
대용량 데이터에 대한 탐색적인 기법으로 사전정보 없이 의미있는 자료구조를 찾아낼 수 있는 방법이다. 또한 관찰치 간의 거리를 데이터형태에 맞게만 정의하면 거의 모든 형태의 데이터에 적용이 가능하다. (기본적으로 관찰치 간의 거리를 데이터형태에 맞게만 정의하면, 거의 모든 형태의 데이터에 대하여 적용이 가능하다.) 그러나 군집 분석의 결과는 관찰치 사이의 비유사성인 거리 또는 유사성을 어떻게 정의하느냐에 따라 크게 좌우되고, 각 변수에 대한 가중치를 결정하는 것도 매우 어렵다.


### SVM의 원리


* 최대의 마진을 갖는 초평면(hyperplane)을 이용한다.
    * 초평면 h(x)는 d-차원의 가중치 벡터를 나타내는 w와, 편향(bias)을 나타내는상수 b로 이루어져 있다. 
* 자료 집합이 선형으로 분리가 가능한 경우에는
    * h(x)<0인 모든 점들은 -1의 군집으로
    * h(x)>0인 모든 점들은 +1의 군집으로 분류할 수 있다.
* 선형 분류기로 분류가 불가능한 비선형 구조를 가진 데이터에 대해 사용할 수 있다.
    * 이에 kernel SVM은 원공간(Input Space)의 데이터를 선형분류가 가능한 고차원 공간(Feature Space)으로 매핑한 뒤 두 범주를 분류하는 초평면을 찾는다.
    



### SVM 모델의 종류


> 분류기준 1 : 에러의 허용 여부

1) Hard Margin SVM : Error case가 하나도 없어서 매우 엄격하게 두 개의 클라스를 분류하는 초평면을 구하는 방법이다. 

2) Soft Margin SVM : Error case를 허용하여 에러를 어느정도 인정하고 손실함수를 이용하여 에러를 최소화시키는 방법이다.

> 분류기준2 : 결정 경계의 형태

1) linear SVM : 선형적인 데이터 셋을 다루는 접근 방법

2) kernel SVM : 비선형적인 데이터 셋을 다루는 접근 방법




### SVM 커널 기법의 종류

* 선형 커널(linear)
    * 문서 분류 등에서 자주 발생한느 대량의 희박 자료벡터를 다룰 때 유용

* 다항식 커널(polynomial) : 원래 특성의 가능한 조합을 지정된 차수까지 모두 계산한다.
    * 이미지 처리에 주로 사용
    
* 가우시안 커널(Gaussian) : 차원이 무한한 특성 공간에 매핑한다.

* RBF (Radial Basis Function)

* 시그모이드 커널(sigmoid)
    * 신경망에 대한 프록시로 주로 사용됨
    
* 스플라인, ANOVARBF
    * 전형적으로 회귀 문제에 잘 수행됨
    
* 하이퍼볼릭 탄젠트 커널

* 일반적으로 널리 사용되는 것은 RBF 커널 SVM
    * 좋은 성능을 얻으려면 매개변수인 C와 gamma를 잘 조정해줘야 한다.
        * C는 데이터 샘플들이 다른 클래스에 놓이는 것을 허용하는 정도를 결정
        * gamma는 결정 경계의 곡률을 결정
        * 두 값 모두 커질수록 알고리즘의 복잡도는 증가하고, 작아질수록 복잡도는 낮아진다. 


### example : 목소리 파일에서 추출한 feature들로 어떤 성별의 목소리인지 예측하는 모델


```R
voice=read.csv("voice.csv")
str(voice) # 3168 obs, 21 vars, categorical.
```

    'data.frame':	3168 obs. of  21 variables:
     $ meanfreq: num  0.0598 0.066 0.0773 0.1512 0.1351 ...
     $ sd      : num  0.0642 0.0673 0.0838 0.0721 0.0791 ...
     $ median  : num  0.032 0.0402 0.0367 0.158 0.1247 ...
     $ Q25     : num  0.0151 0.0194 0.0087 0.0966 0.0787 ...
     $ Q75     : num  0.0902 0.0927 0.1319 0.208 0.206 ...
     $ IQR     : num  0.0751 0.0733 0.1232 0.1114 0.1273 ...
     $ skew    : num  12.86 22.42 30.76 1.23 1.1 ...
     $ kurt    : num  274.4 634.61 1024.93 4.18 4.33 ...
     $ sp.ent  : num  0.893 0.892 0.846 0.963 0.972 ...
     $ sfm     : num  0.492 0.514 0.479 0.727 0.784 ...
     $ mode    : num  0 0 0 0.0839 0.1043 ...
     $ centroid: num  0.0598 0.066 0.0773 0.1512 0.1351 ...
     $ meanfun : num  0.0843 0.1079 0.0987 0.089 0.1064 ...
     $ minfun  : num  0.0157 0.0158 0.0157 0.0178 0.0169 ...
     $ maxfun  : num  0.276 0.25 0.271 0.25 0.267 ...
     $ meandom : num  0.00781 0.00901 0.00799 0.2015 0.71281 ...
     $ mindom  : num  0.00781 0.00781 0.00781 0.00781 0.00781 ...
     $ maxdom  : num  0.00781 0.05469 0.01562 0.5625 5.48438 ...
     $ dfrange : num  0 0.04688 0.00781 0.55469 5.47656 ...
     $ modindx : num  0 0.0526 0.0465 0.2471 0.2083 ...
     $ label   : Factor w/ 2 levels "female","male": 2 2 2 2 2 2 2 2 2 2 ...
    


```R
library(e1071)
library(klaR)
library(caret)
library(kernlab)

```

    
    Attaching package: 'kernlab'
    
    The following object is masked from 'package:ggplot2':
    
        alpha
    
    


```R
which(voice=='NA') # there is none. 

```






```R
N=nrow(voice) ; N
set.seed(123)
sampling=sample(N, N*0.75)
voice_train=voice[sampling, ]
voice_test=voice[-sampling, ]
```


3168



```R
voice_classifier=ksvm(label ~ . , data=voice_train, kernel="vanilladot" ) # linear
voice_classifier
```

     Setting default kernel parameters  
    


    Support Vector Machine object of class "ksvm" 
    
    SV type: C-svc  (classification) 
     parameter : cost C = 1 
    
    Linear (vanilla) kernel function. 
    
    Number of Support Vectors : 194 
    
    Objective Function Value : -180.2347 
    Training error : 0.025253 



```R
voice_predictions=predict(voice_classifier, voice_test)
head(voice_predictions)
```


<ol class=list-inline>
	<li>male</li>
	<li>male</li>
	<li>male</li>
	<li>male</li>
	<li>male</li>
	<li>male</li>
</ol>

<details>
	<summary style=display:list-item;cursor:pointer>
		<strong>Levels</strong>:
	</summary>
	<ol class=list-inline>
		<li>'female'</li>
		<li>'male'</li>
	</ol>
</details>



```R
table(voice_predictions, voice_test$label)
agreement= voice_predictions==voice_test$label
table(agreement)
prop.table(table(agreement)) # ~98.10% accurate
```


                     
    voice_predictions female male
               female    378    6
               male        9  399



    agreement
    FALSE  TRUE 
       15   777 



    agreement
         FALSE       TRUE 
    0.01893939 0.98106061 



```R
voice_classifier_rbf=ksvm(label~., data=voice_train, kernel="rbfdot")
```


```R
voice_predictions_rbf=predict(voice_classifier_rbf, voice_test) 
agreement_rbf= voice_predictions_rbf==voice_test$label
table(agreement_rbf)
```


    agreement_rbf
    FALSE  TRUE 
       14   778 



```R
prop.table(table(agreement_rbf)) #~99.23% accurate

```


    agreement_rbf
         FALSE       TRUE 
    0.01767677 0.98232323 



```R
prop.table(table(agreement))
```


    agreement
         FALSE       TRUE 
    0.01893939 0.98106061 



```R
prop.table(table(agreement_rbf)) # better result
```


    agreement_rbf
         FALSE       TRUE 
    0.01767677 0.98232323 



```R


```
