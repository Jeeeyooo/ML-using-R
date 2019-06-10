
# Regression


```R

```

### example : 자동차 사고가 어떤 경우에 일어나는 지 회귀분석 


```R
acc<-read.csv('car_accident.csv',head=T)
str(acc)
```

    'data.frame':	4119 obs. of  15 variables:
     $ 발생년월일시         : int  2016122320 2016122517 2016122519 2016122610 2016122819 2016111207 2016110919 2016111005 2016110711 2016110818 ...
     $ 발생분               : int  35 48 5 40 40 34 25 0 5 0 ...
     $ 주야                 : Factor w/ 2 levels "야간","주간": 1 2 1 2 1 2 1 1 2 1 ...
     $ 요일                 : Factor w/ 7 levels "금","목","수",..: 1 5 5 4 3 6 3 2 4 7 ...
     $ 사망자수             : int  1 1 1 1 1 1 1 1 1 1 ...
     $ 중상자수             : int  0 0 0 0 0 0 0 0 0 0 ...
     $ 경상자수             : int  0 0 0 0 0 0 0 0 0 0 ...
     $ 발생지시도           : Factor w/ 17 levels "강원","경기",..: 2 9 17 4 3 3 2 12 16 13 ...
     $ 발생지시군구         : Factor w/ 205 levels "가평군","강남구",..: 26 38 172 12 171 9 190 86 132 103 ...
     $ 사고유형_대분류      : Factor w/ 3 levels "차대사람","차대차",..: 1 1 1 1 1 1 1 1 3 1 ...
     $ 사고유형             : Factor w/ 13 levels "공작물충돌","기타",..: 2 13 11 13 13 13 2 13 7 11 ...
     $ 법규위반             : Factor w/ 14 levels "과속","교차로 통행방법 위반",..: 9 9 9 9 4 9 9 4 9 9 ...
     $ 도로형태_대분류      : Factor w/ 5 levels "고가도로위","교차로",..: 2 4 4 4 4 4 4 4 4 4 ...
     $ 당사자종별_1당_대분류: Factor w/ 10 levels "건설기계","농기계",..: 4 4 10 5 4 1 4 4 1 4 ...
     $ 당사자종별_2당_대분류: Factor w/ 12 levels "","건설기계",..: 4 4 4 4 4 4 4 4 1 4 ...
    

#### 아래와 같이 열 이름 변경 
+ 발생년월일시 -> date
+ 발생분 -> 제거 `#2`
+ 주야 -> day_night
+ 요일 -> day
+ 사망자수 -> cnt_dead
+ 중상자수 -> cnt_serious
+ 경상자수 -> cnt_slight
+ 발생지시도 -> province
+ 발생지시군구 -> 제거 `#9`
+ 사고유형_대분류 -> acc_type
+ 사교유형 -> 제거 `#11`
+ 위반 -> law_type
+ 형태_대분류 -> road_type
+ 당사자종별_1당_대분류 -> party1
+ 당사자종별_2당_대분류 -> party2


```R
acc<-acc[,-c(2,9,11)]
colnames(acc)<-c('date','day_night','day','cnt_dead','cnt_serious','cnt_slight','province','acc_type',
                 'law_type','road_type','party1','party2')
```

#### 서울시에서 발생한 교통 사고 사망자/사상자 수를 예측하려고 한다.
#### 따라서, '서울'지역에 대한 데이터만 추출하세요.



```R
library(dplyr)
acc_seoul<-acc%>%filter(province=='서울')%>%select(-'province')
```

    
    Attaching package: 'dplyr'
    
    The following objects are masked from 'package:stats':
    
        filter, lag
    
    The following objects are masked from 'package:base':
    
        intersect, setdiff, setequal, union
    
    

+ 현재 date 데이터는 년(4)월(2)일(2)시(2)로 이루어져있다. 
+ 하지만 우리는 이미 이 데이터가 2016년 교통사고 데이터라는 정보를 알고 있기에 년도에 대한 데이터는 필요하지 않다. 
+ 또한, 교통사고에 영향을 미칠 수 있는 다양한 요소  중에서 날씨와 시간이 있다. 하지만 이미 시간이 늦은지 밝은지에 대한 데이터는 day_night에포함하고 있으므로 month라는 월 데이터만 만들려고 한다. 
+ 또한, 3,4,5는 봄, 6,7,8은 여름 9,10,11은 가을 12,1,2는 겨울을 의미하는 factor로 변형한다. 


```R
library(stringr)
library(dplyr)

acc_seoul<-acc_seoul%>%mutate(month=str_sub(date,5,6))
acc_seoul$month[acc_seoul$month %in% c('03','04','05')]<-'봄'
acc_seoul$month[acc_seoul$month %in% c('06','07','08')]<-'여름'
acc_seoul$month[acc_seoul$month %in% c('09','10','11')]<-'가을'
acc_seoul$month[acc_seoul$month %in% c('12','01','02')]<-'겨울'
acc_seoul$month<-factor(acc_seoul$month)
```

day열에서 '토','일' 열은 weekend 열로 만들어보자. 


```R
acc_seoul<-acc_seoul%>%mutate(weekend=factor(day %in% c('토','일')))
```

각 열에 결측치가 있는지 확인


```R
colSums(is.na(acc_seoul)) # 결측치 없음.
```


<dl class=dl-horizontal>
	<dt>date</dt>
		<dd>0</dd>
	<dt>day_night</dt>
		<dd>0</dd>
	<dt>day</dt>
		<dd>0</dd>
	<dt>cnt_dead</dt>
		<dd>0</dd>
	<dt>cnt_serious</dt>
		<dd>0</dd>
	<dt>cnt_slight</dt>
		<dd>0</dd>
	<dt>acc_type</dt>
		<dd>0</dd>
	<dt>law_type</dt>
		<dd>0</dd>
	<dt>road_type</dt>
		<dd>0</dd>
	<dt>party1</dt>
		<dd>0</dd>
	<dt>party2</dt>
		<dd>0</dd>
	<dt>month</dt>
		<dd>0</dd>
	<dt>weekend</dt>
		<dd>0</dd>
</dl>



acc_type, law_type, road_type, party1, party2 열들을 table 함수를 이용하여 자료 탐색을 했더니 10개 이하인 레벨들이 발견되었다. 깔끔한 분석을 위해 그러한 것들은 제거한다.


```R
law_type_index<-names(which(table(acc_seoul$law_type)<=10))
road_type_index<-names(which(table(acc_seoul$road_type)<=10))
party1_index<-names(which(table(acc_seoul$party1)<=10))
party2_index<-names(which(table(acc_seoul$party2)<=10))

acc_seoul<-acc_seoul%>%
  filter(!law_type %in% law_type_index)%>%
  filter(!road_type %in% road_type_index)%>%
  filter(!party1 %in% party1_index)%>%
  filter(!party2 %in% party2_index)

acc_seoul<-droplevels(acc_seoul)
```

전처리한 데이터를 가지고 cnt_dead, cnt_serious,cnt_slight 각각을 회귀분석을 하여 예측하고자 한다. 

회귀분석을 하기 전에 각각 어떠한 변수들이 유효한지 분산분석을 통해 도출하자.



```R
aov1<-aov(cnt_dead~day_night+acc_type+law_type+road_type+party1+party2+month+weekend,data=acc_seoul)
summary(aov1) # acc_type, party2 변수들이 유효하다.
```


                 Df Sum Sq Mean Sq F value   Pr(>F)    
    day_night     1  0.000 0.00005   0.004 0.951521    
    acc_type      2  0.206 0.10313   7.489 0.000676 ***
    law_type      4  0.067 0.01682   1.222 0.301722    
    road_type     1  0.031 0.03104   2.254 0.134397    
    party1        6  0.074 0.01240   0.901 0.494644    
    party2        5  0.519 0.10375   7.534 1.16e-06 ***
    month         3  0.075 0.02515   1.826 0.142546    
    weekend       1  0.035 0.03479   2.527 0.113046    
    Residuals   284  3.911 0.01377                     
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1



```R
aov2<-aov(cnt_serious~day_night+acc_type+law_type+road_type+party1+party2+month+weekend,data=acc_seoul)
summary(aov2) # acc_type, party2 변수들이 유효하다
```


                 Df Sum Sq Mean Sq F value   Pr(>F)    
    day_night     1   0.07  0.0682   0.337 0.561749    
    acc_type      2   5.76  2.8813  14.253 1.26e-06 ***
    law_type      4   1.84  0.4598   2.274 0.061434 .  
    road_type     1   0.61  0.6052   2.994 0.084664 .  
    party1        6   1.32  0.2205   1.091 0.367985    
    party2        5   5.32  1.0630   5.258 0.000123 ***
    month         3   0.16  0.0549   0.272 0.845776    
    weekend       1   0.06  0.0641   0.317 0.573772    
    Residuals   284  57.41  0.2022                     
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1



```R
aov3<-aov(cnt_slight~day_night+acc_type+law_type+road_type+party1+party2+month+weekend,data=acc_seoul)
summary(aov3) #acc_type, road_type, party2, month,weekend 변수들이 유효하다.
```


                 Df Sum Sq Mean Sq F value   Pr(>F)    
    day_night     1   0.84   0.838   3.223  0.07369 .  
    acc_type      2   8.13   4.065  15.628 3.64e-07 ***
    law_type      4   1.31   0.327   1.256  0.28739    
    road_type     1   2.24   2.245   8.631  0.00358 ** 
    party1        6   2.70   0.450   1.731  0.11369    
    party2        5   9.40   1.881   7.230 2.17e-06 ***
    month         3   2.23   0.744   2.862  0.03716 *  
    weekend       1   1.10   1.099   4.225  0.04076 *  
    Residuals   284  73.86   0.260                     
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1


위에서 구한 의미 있는 변수들을 가지고 회귀분석을 실시하여 3개의 회귀식을 도출



```R
lm1<-glm(cnt_dead~acc_type+party2,data=acc_seoul,family='poisson')
lm2<-glm(cnt_serious~acc_type+party2,data=acc_seoul,family='poisson')
lm3<-glm(cnt_slight~acc_type+road_type+party2+month+weekend,data=acc_seoul,family='poisson')
```


```R
summary(lm1)
```


    
    Call:
    glm(formula = cnt_dead ~ acc_type + party2, family = "poisson", 
        data = acc_seoul)
    
    Deviance Residuals: 
        Min       1Q   Median       3Q      Max  
    -0.2318   0.0000   0.0000   0.0000   0.8134  
    
    Coefficients: (1 not defined because of singularities)
                     Estimate Std. Error z value Pr(>|z|)
    (Intercept)       0.16599    1.04616   0.159    0.874
    acc_type차대차    0.05716    1.01379   0.056    0.955
    acc_type차량단독 -0.16599    1.06251  -0.156    0.876
    party2보행자     -0.16599    1.04867  -0.158    0.874
    party2승용차     -0.16599    0.30732  -0.540    0.589
    party2승합차     -0.22314    0.38730  -0.576    0.565
    party2이륜차     -0.22314    0.34156  -0.653    0.514
    party2자전거     -0.22314    0.40825  -0.547    0.585
    party2화물차           NA         NA      NA       NA
    
    (Dispersion parameter for poisson family taken to be 1)
    
        Null deviance: 3.7822  on 307  degrees of freedom
    Residual deviance: 3.0532  on 300  degrees of freedom
    AIC: 638.12
    
    Number of Fisher Scoring iterations: 4
    



```R
summary(lm2)
```


    
    Call:
    glm(formula = cnt_serious ~ acc_type + party2, family = "poisson", 
        data = acc_seoul)
    
    Deviance Residuals: 
        Min       1Q   Median       3Q      Max  
    -1.0801  -0.4082  -0.2294  -0.2294   3.5568  
    
    Coefficients: (1 not defined because of singularities)
                       Estimate Std. Error z value Pr(>|z|)  
    (Intercept)       -17.20559 3467.85860  -0.005   0.9960  
    acc_type차대차     16.66660 3467.85858   0.005   0.9962  
    acc_type차량단독   14.93691 3467.85865   0.004   0.9966  
    party2보행자       13.56801 3467.85863   0.004   0.9969  
    party2승용차       -0.09699    0.44544  -0.218   0.8276  
    party2승합차       -1.94591    1.06904  -1.820   0.0687 .
    party2이륜차       -1.35812    0.69007  -1.968   0.0491 *
    party2자전거      -16.76359 1096.63322  -0.015   0.9878  
    party2화물차             NA         NA      NA       NA  
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    
    (Dispersion parameter for poisson family taken to be 1)
    
        Null deviance: 197.19  on 307  degrees of freedom
    Residual deviance: 137.15  on 300  degrees of freedom
    AIC: 209.07
    
    Number of Fisher Scoring iterations: 15
    



```R
summary(lm3)
```


    
    Call:
    glm(formula = cnt_slight ~ acc_type + road_type + party2 + month + 
        weekend, family = "poisson", data = acc_seoul)
    
    Deviance Residuals: 
        Min       1Q   Median       3Q      Max  
    -2.0775  -0.5719  -0.3605  -0.1859   2.9625  
    
    Coefficients: (1 not defined because of singularities)
                       Estimate Std. Error z value Pr(>|z|)   
    (Intercept)      -2.018e+01  5.718e+03  -0.004  0.99718   
    acc_type차대차    1.881e+01  5.718e+03   0.003  0.99738   
    acc_type차량단독  1.714e+00  5.802e+03   0.000  0.99976   
    road_type단일로   9.893e-01  3.233e-01   3.060  0.00221 **
    party2보행자      1.745e+01  5.718e+03   0.003  0.99757   
    party2승용차      8.888e-01  6.553e-01   1.356  0.17499   
    party2승합차      1.149e+00  6.698e-01   1.716  0.08620 . 
    party2이륜차     -4.401e-01  8.545e-01  -0.515  0.60652   
    party2자전거     -1.190e+00  1.175e+00  -1.013  0.31110   
    party2화물차             NA         NA      NA       NA   
    month겨울        -1.324e+00  4.333e-01  -3.056  0.00224 **
    month봄           6.314e-03  3.314e-01   0.019  0.98480   
    month여름        -4.812e-01  4.469e-01  -1.077  0.28158   
    weekendTRUE      -7.138e-01  3.807e-01  -1.875  0.06081 . 
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    
    (Dispersion parameter for poisson family taken to be 1)
    
        Null deviance: 249.66  on 307  degrees of freedom
    Residual deviance: 157.10  on 295  degrees of freedom
    AIC: 268.37
    
    Number of Fisher Scoring iterations: 16
    


#### 예측에 불필요한 인자들을 제거, 다항식 추가 => 개선.


```R
summary(step(lm1))
```

    Start:  AIC=638.12
    cnt_dead ~ acc_type + party2
    
               Df Deviance    AIC
    - party2    5   3.5841 628.65
    - acc_type  1   3.0565 636.12
    <none>          3.0532 638.12
    
    Step:  AIC=628.65
    cnt_dead ~ acc_type
    
               Df Deviance    AIC
    - acc_type  2   3.7822 624.85
    <none>          3.5841 628.65
    
    Step:  AIC=624.85
    cnt_dead ~ 1
    
    


    
    Call:
    glm(formula = cnt_dead ~ 1, family = "poisson", data = acc_seoul)
    
    Deviance Residuals: 
         Min        1Q    Median        3Q       Max  
    -0.01615  -0.01615  -0.01615  -0.01615   0.86061  
    
    Coefficients:
                Estimate Std. Error z value Pr(>|z|)
    (Intercept)  0.01610    0.05652   0.285    0.776
    
    (Dispersion parameter for poisson family taken to be 1)
    
        Null deviance: 3.7822  on 307  degrees of freedom
    Residual deviance: 3.7822  on 307  degrees of freedom
    AIC: 624.85
    
    Number of Fisher Scoring iterations: 4
    



```R
summary(step(lm2))
```

    Start:  AIC=209.07
    cnt_serious ~ acc_type + party2
    
               Df Deviance    AIC
    - acc_type  1   138.19 208.11
    <none>          137.15 209.07
    - party2    5   154.79 216.71
    
    Step:  AIC=208.11
    cnt_serious ~ party2
    
             Df Deviance    AIC
    <none>        138.19 208.11
    - party2  6   197.19 255.11
    


    
    Call:
    glm(formula = cnt_serious ~ party2, family = "poisson", data = acc_seoul)
    
    Deviance Residuals: 
        Min       1Q   Median       3Q      Max  
    -1.0801  -0.4199  -0.2294  -0.2294   3.5568  
    
    Coefficients:
                  Estimate Std. Error z value Pr(>|z|)    
    (Intercept)    -2.2687     0.5774  -3.929 8.51e-05 ***
    party2보행자   -1.3689     0.7303  -1.874   0.0609 .  
    party2승용차    1.6037     0.6236   2.572   0.0101 *  
    party2승합차   -0.2162     1.1547  -0.187   0.8515    
    party2이륜차    0.3716     0.8165   0.455   0.6491    
    party2자전거  -15.0339  1096.6333  -0.014   0.9891    
    party2화물차    1.7297     0.6901   2.507   0.0122 *  
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    
    (Dispersion parameter for poisson family taken to be 1)
    
        Null deviance: 197.19  on 307  degrees of freedom
    Residual deviance: 138.19  on 301  degrees of freedom
    AIC: 208.11
    
    Number of Fisher Scoring iterations: 15
    



```R
summary(step(lm3))
```

    Start:  AIC=268.37
    cnt_slight ~ acc_type + road_type + party2 + month + weekend
    
                Df Deviance    AIC
    <none>           157.10 268.37
    - acc_type   1   160.19 269.45
    - weekend    1   161.16 270.42
    - party2     5   173.31 274.57
    - month      3   170.81 276.07
    - road_type  1   167.42 276.68
    


    
    Call:
    glm(formula = cnt_slight ~ acc_type + road_type + party2 + month + 
        weekend, family = "poisson", data = acc_seoul)
    
    Deviance Residuals: 
        Min       1Q   Median       3Q      Max  
    -2.0775  -0.5719  -0.3605  -0.1859   2.9625  
    
    Coefficients: (1 not defined because of singularities)
                       Estimate Std. Error z value Pr(>|z|)   
    (Intercept)      -2.018e+01  5.718e+03  -0.004  0.99718   
    acc_type차대차    1.881e+01  5.718e+03   0.003  0.99738   
    acc_type차량단독  1.714e+00  5.802e+03   0.000  0.99976   
    road_type단일로   9.893e-01  3.233e-01   3.060  0.00221 **
    party2보행자      1.745e+01  5.718e+03   0.003  0.99757   
    party2승용차      8.888e-01  6.553e-01   1.356  0.17499   
    party2승합차      1.149e+00  6.698e-01   1.716  0.08620 . 
    party2이륜차     -4.401e-01  8.545e-01  -0.515  0.60652   
    party2자전거     -1.190e+00  1.175e+00  -1.013  0.31110   
    party2화물차             NA         NA      NA       NA   
    month겨울        -1.324e+00  4.333e-01  -3.056  0.00224 **
    month봄           6.314e-03  3.314e-01   0.019  0.98480   
    month여름        -4.812e-01  4.469e-01  -1.077  0.28158   
    weekendTRUE      -7.138e-01  3.807e-01  -1.875  0.06081 . 
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    
    (Dispersion parameter for poisson family taken to be 1)
    
        Null deviance: 249.66  on 307  degrees of freedom
    Residual deviance: 157.10  on 295  degrees of freedom
    AIC: 268.37
    
    Number of Fisher Scoring iterations: 16
    


#### ==> lm1빼고는 AIC값이 모두 줄어들었지만 너무나도 미미한 수치.



```R

```


```R
summary(lm_dead<-lm(cnt_dead~day+acc_type+party2,data=acc_seoul))

```


    
    Call:
    lm(formula = cnt_dead ~ day + acc_type + party2, data = acc_seoul)
    
    Residuals:
         Min       1Q   Median       3Q      Max 
    -0.29833 -0.01074  0.00713  0.01214  0.93285 
    
    Coefficients: (1 not defined because of singularities)
                      Estimate Std. Error t value Pr(>|t|)    
    (Intercept)       1.175395   0.123398   9.525  < 2e-16 ***
    day목             0.003018   0.023851   0.127  0.89938    
    day수            -0.002912   0.021817  -0.133  0.89391    
    day월             0.017875   0.024798   0.721  0.47158    
    day일             0.076579   0.024137   3.173  0.00167 ** 
    day토            -0.013861   0.022624  -0.613  0.54057    
    day화            -0.010129   0.024694  -0.410  0.68198    
    acc_type차대차    0.046361   0.117779   0.394  0.69414    
    acc_type차량단독 -0.185798   0.124190  -1.496  0.13571    
    party2보행자     -0.182527   0.122584  -1.489  0.13756    
    party2승용차     -0.172483   0.039614  -4.354 1.85e-05 ***
    party2승합차     -0.224066   0.047929  -4.675 4.48e-06 ***
    party2이륜차     -0.223765   0.043037  -5.199 3.75e-07 ***
    party2자전거     -0.231524   0.050618  -4.574 7.06e-06 ***
    party2화물차            NA         NA      NA       NA    
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    
    Residual standard error: 0.1154 on 294 degrees of freedom
    Multiple R-squared:  0.2047,	Adjusted R-squared:  0.1695 
    F-statistic: 5.821 on 13 and 294 DF,  p-value: 1.474e-09
    


결과가 좋지 않은 이유
+ 각 집단에 대한 충분한 데이터가 부족했다. 그래서 회귀식을 사용할 때  몇몇 변수들이 결측이 되었던 것으로 보이고 올바른 추정을 할 수 없었다.
+ 추정하고자하는 변수(y)가 cnt_dead,cnt_serious,cnt_slight로 아주 작은 범위를 갖는 이산형변수였다. 또한 cnt_dead는 1, cnt_serious와 cnt_slight는 0에 몰려있는 데이터였다.





```R

```


```R

```


```R

```


```R

```
