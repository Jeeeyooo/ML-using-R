
# kNN

k-최근접 이웃 알고리즘

k-Nearest Neighbor algorithm


* non-parameter 방식. (비모수 방식)
* k : k의 역할은 몇 번째로 가까운 데이터까지 살펴볼 것인가를 정하는 수.
  + ex) k=21이면 가장 가까운 21개의 데이터를 전부 살펴보게 됨
* 입력 : 특징 공간 내 k개의 가장 가까운 훈련 데이터로 구성되어 있음
* 출력 : kNN이 분류로 사용되었는지, 회귀로 사용되었는지에 따라 다름
  + kNN 분류) 어느 class에 소속되어 있는지.
  + kNN 회귀) k개의 최근접 이웃이 가진 값의 평균.
* instance기반 학습, lazy learning의 일종
* 가장 간단한 Machine Learning 알고리즘
* 단점: 데이터의 지역 구조에 민감. 

## Example : Classifying Cancer Samples

### Step 1 : Collecting Data

* data : breast cancer
* http://archive.ics.uci.edu/ml/datasets/Breast+Cancer+Wisconsin+%28Diagnostic%29

#### 데이터 구성
id + cancer diagnosis + 30 numeric measurements (mean, std err, worst(largest) value) of digited cell nuclei (디지털화된 세포핵)

##### diagnosis
* M : malignant 악성
* B : benign 양성

##### numeric measurements 항목
* Radius          반지름
* Texture         질감
* Perimeter       둘레
* Area            면적
* Smoothness      매끄러움
* Compactness     빽빽함
* Concavity       오목함
* Concave points  오목한 점 개수
* Symmetry        대칭
* Fractal dimension

### Step 2: Exploring and preparing the data


```R
# import the CSV file

# dir = "[your path]"
# setwd(dir)
wbcd <- read.csv("wisc_bc_data.csv", stringsAsFactors = FALSE)
```


```R
# examine the structure of the wbcd data frame
str(wbcd)
```

    'data.frame':	569 obs. of  32 variables:
     $ id               : int  87139402 8910251 905520 868871 9012568 906539 925291 87880 862989 89827 ...
     $ diagnosis        : chr  "B" "B" "B" "B" ...
     $ radius_mean      : num  12.3 10.6 11 11.3 15.2 ...
     $ texture_mean     : num  12.4 18.9 16.8 13.4 13.2 ...
     $ perimeter_mean   : num  78.8 69.3 70.9 73 97.7 ...
     $ area_mean        : num  464 346 373 385 712 ...
     $ smoothness_mean  : num  0.1028 0.0969 0.1077 0.1164 0.0796 ...
     $ compactness_mean : num  0.0698 0.1147 0.078 0.1136 0.0693 ...
     $ concavity_mean   : num  0.0399 0.0639 0.0305 0.0464 0.0339 ...
     $ points_mean      : num  0.037 0.0264 0.0248 0.048 0.0266 ...
     $ symmetry_mean    : num  0.196 0.192 0.171 0.177 0.172 ...
     $ dimension_mean   : num  0.0595 0.0649 0.0634 0.0607 0.0554 ...
     $ radius_se        : num  0.236 0.451 0.197 0.338 0.178 ...
     $ texture_se       : num  0.666 1.197 1.387 1.343 0.412 ...
     $ perimeter_se     : num  1.67 3.43 1.34 1.85 1.34 ...
     $ area_se          : num  17.4 27.1 13.5 26.3 17.7 ...
     $ smoothness_se    : num  0.00805 0.00747 0.00516 0.01127 0.00501 ...
     $ compactness_se   : num  0.0118 0.03581 0.00936 0.03498 0.01485 ...
     $ concavity_se     : num  0.0168 0.0335 0.0106 0.0219 0.0155 ...
     $ points_se        : num  0.01241 0.01365 0.00748 0.01965 0.00915 ...
     $ symmetry_se      : num  0.0192 0.035 0.0172 0.0158 0.0165 ...
     $ dimension_se     : num  0.00225 0.00332 0.0022 0.00344 0.00177 ...
     $ radius_worst     : num  13.5 11.9 12.4 11.9 16.2 ...
     $ texture_worst    : num  15.6 22.9 26.4 15.8 15.7 ...
     $ perimeter_worst  : num  87 78.3 79.9 76.5 104.5 ...
     $ area_worst       : num  549 425 471 434 819 ...
     $ smoothness_worst : num  0.139 0.121 0.137 0.137 0.113 ...
     $ compactness_worst: num  0.127 0.252 0.148 0.182 0.174 ...
     $ concavity_worst  : num  0.1242 0.1916 0.1067 0.0867 0.1362 ...
     $ points_worst     : num  0.0939 0.0793 0.0743 0.0861 0.0818 ...
     $ symmetry_worst   : num  0.283 0.294 0.3 0.21 0.249 ...
     $ dimension_worst  : num  0.0677 0.0759 0.0788 0.0678 0.0677 ...
    


```R
# drop the id feature
wbcd <- wbcd[-1]
```


```R
# table of diagnosis
table(wbcd$diagnosis)
```


    
      B   M 
    357 212 



```R
# recode diagnosis as a factor
wbcd$diagnosis <- factor(wbcd$diagnosis, levels = c("B", "M"),
                         labels = c("Benign", "Malignant"))
```


```R
# table or proportions with more informative labels
round(prop.table(table(wbcd$diagnosis)) * 100, digits = 1)
```


    
       Benign Malignant 
         62.7      37.3 



```R
# summarize three numeric features
summary(wbcd[c("radius_mean", "area_mean", "smoothness_mean")])
```


      radius_mean       area_mean      smoothness_mean  
     Min.   : 6.981   Min.   : 143.5   Min.   :0.05263  
     1st Qu.:11.700   1st Qu.: 420.3   1st Qu.:0.08637  
     Median :13.370   Median : 551.1   Median :0.09587  
     Mean   :14.127   Mean   : 654.9   Mean   :0.09636  
     3rd Qu.:15.780   3rd Qu.: 782.7   3rd Qu.:0.10530  
     Max.   :28.110   Max.   :2501.0   Max.   :0.16340  



```R
# create normalization function
normalize <- function(x) {
  return ((x - min(x)) / (max(x) - min(x)))
}
```


```R
# test normalization function - result should be identical
# 그 데이터 범위에서의 상대적 크기를 통해 0과 1사이의 값으로 변환
normalize(c(1, 2, 3, 4, 5))
normalize(c(10, 20, 30, 40, 50))
normalize(c(10, 20, 30, 40, 100))

```


<ol class=list-inline>
	<li>0</li>
	<li>0.25</li>
	<li>0.5</li>
	<li>0.75</li>
	<li>1</li>
</ol>




<ol class=list-inline>
	<li>0</li>
	<li>0.25</li>
	<li>0.5</li>
	<li>0.75</li>
	<li>1</li>
</ol>




<ol class=list-inline>
	<li>0</li>
	<li>0.111111111111111</li>
	<li>0.222222222222222</li>
	<li>0.333333333333333</li>
	<li>1</li>
</ol>




```R
# normalize the wbcd data
wbcd_n <- as.data.frame(lapply(wbcd[2:31], normalize))
```


```R
# confirm that normalization worked
summary(wbcd_n$area_mean)
```


       Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
     0.0000  0.1174  0.1729  0.2169  0.2711  1.0000 



```R
# create training and test data
wbcd_train <- wbcd_n[1:469, ]   #469개
wbcd_test <- wbcd_n[470:569, ]  #100개
```


```R
# create labels for training and test data
wbcd_train_labels <- wbcd[1:469, 1]
wbcd_test_labels <- wbcd[470:569, 1]
# 첫번째 열: diagnoise (B or M)
# diagnoise 값만 따로 저장 (label값이므로)
```

### Step 3: Training a model on the data ----


```R
# load the "class" library
library(class)

wbcd_test_pred <- knn(train = wbcd_train, test = wbcd_test,
                      cl = wbcd_train_labels, k = 21)
```

    Warning message:
    "package 'class' was built under R version 3.5.3"


```R
# class library의 knn함수 사용

# ??knn

summary(wbcd_test_pred)
round(prop.table(table(wbcd$diagnosis)) * 100, digits = 1)
```




<dl class=dl-horizontal>
	<dt>Benign</dt>
		<dd>63</dd>
	<dt>Malignant</dt>
		<dd>37</dd>
</dl>




    
       Benign Malignant 
         62.7      37.3 


### Step 4: Evaluating model performance ----


```R
# load the "gmodels" library
library(gmodels)
```

    Warning message:
    "package 'gmodels' was built under R version 3.5.3"


```R
# Create the cross tabulation of predicted vs. actual
CrossTable(x = wbcd_test_labels, y = wbcd_test_pred,
           prop.chisq = T)
```

    
     
       Cell Contents
    |-------------------------|
    |                       N |
    | Chi-square contribution |
    |           N / Row Total |
    |           N / Col Total |
    |         N / Table Total |
    |-------------------------|
    
     
    Total Observations in Table:  100 
    
     
                     | wbcd_test_pred 
    wbcd_test_labels |    Benign | Malignant | Row Total | 
    -----------------|-----------|-----------|-----------|
              Benign |        61 |         0 |        61 | 
                     |    13.255 |    22.570 |           | 
                     |     1.000 |     0.000 |     0.610 | 
                     |     0.968 |     0.000 |           | 
                     |     0.610 |     0.000 |           | 
    -----------------|-----------|-----------|-----------|
           Malignant |         2 |        37 |        39 | 
                     |    20.733 |    35.302 |           | 
                     |     0.051 |     0.949 |     0.390 | 
                     |     0.032 |     1.000 |           | 
                     |     0.020 |     0.370 |           | 
    -----------------|-----------|-----------|-----------|
        Column Total |        63 |        37 |       100 | 
                     |     0.630 |     0.370 |           | 
    -----------------|-----------|-----------|-----------|
    
     
    

### Step 5: Improving model performance ----


```R
# use the scale() function to z-score standardize a data frame
wbcd_z <- as.data.frame(scale(wbcd[-1]))
```


```R

# confirm that the transformation was applied correctly
summary(wbcd_z$area_mean)
# mean = 0
```


       Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    -1.4532 -0.6666 -0.2949  0.0000  0.3632  5.2459 



```R
# create training and test datasets
wbcd_train <- wbcd_z[1:469, ]
wbcd_test <- wbcd_z[470:569, ]
```


```R
# re-classify test cases
wbcd_test_pred <- knn(train = wbcd_train, test = wbcd_test,
                      cl = wbcd_train_labels, k = 21)
```


```R
# Create the cross tabulation of predicted vs. actual
CrossTable(x = wbcd_test_labels, y = wbcd_test_pred,
           prop.chisq = FALSE)
```

    
     
       Cell Contents
    |-------------------------|
    |                       N |
    |           N / Row Total |
    |           N / Col Total |
    |         N / Table Total |
    |-------------------------|
    
     
    Total Observations in Table:  100 
    
     
                     | wbcd_test_pred 
    wbcd_test_labels |    Benign | Malignant | Row Total | 
    -----------------|-----------|-----------|-----------|
              Benign |        61 |         0 |        61 | 
                     |     1.000 |     0.000 |     0.610 | 
                     |     0.924 |     0.000 |           | 
                     |     0.610 |     0.000 |           | 
    -----------------|-----------|-----------|-----------|
           Malignant |         5 |        34 |        39 | 
                     |     0.128 |     0.872 |     0.390 | 
                     |     0.076 |     1.000 |           | 
                     |     0.050 |     0.340 |           | 
    -----------------|-----------|-----------|-----------|
        Column Total |        66 |        34 |       100 | 
                     |     0.660 |     0.340 |           | 
    -----------------|-----------|-----------|-----------|
    
     
    


```R
# try several different values of k
wbcd_train <- wbcd_n[1:469, ]
wbcd_test <- wbcd_n[470:569, ]
```


```R
wbcd_test_pred <- knn(train = wbcd_train, test = wbcd_test, cl = wbcd_train_labels, k=1)
CrossTable(x = wbcd_test_labels, y = wbcd_test_pred, prop.chisq=FALSE)
```

    
     
       Cell Contents
    |-------------------------|
    |                       N |
    |           N / Row Total |
    |           N / Col Total |
    |         N / Table Total |
    |-------------------------|
    
     
    Total Observations in Table:  100 
    
     
                     | wbcd_test_pred 
    wbcd_test_labels |    Benign | Malignant | Row Total | 
    -----------------|-----------|-----------|-----------|
              Benign |        58 |         3 |        61 | 
                     |     0.951 |     0.049 |     0.610 | 
                     |     0.983 |     0.073 |           | 
                     |     0.580 |     0.030 |           | 
    -----------------|-----------|-----------|-----------|
           Malignant |         1 |        38 |        39 | 
                     |     0.026 |     0.974 |     0.390 | 
                     |     0.017 |     0.927 |           | 
                     |     0.010 |     0.380 |           | 
    -----------------|-----------|-----------|-----------|
        Column Total |        59 |        41 |       100 | 
                     |     0.590 |     0.410 |           | 
    -----------------|-----------|-----------|-----------|
    
     
    


```R
wbcd_test_pred <- knn(train = wbcd_train, test = wbcd_test, cl = wbcd_train_labels, k=5)
CrossTable(x = wbcd_test_labels, y = wbcd_test_pred, prop.chisq=FALSE)
```

    
     
       Cell Contents
    |-------------------------|
    |                       N |
    |           N / Row Total |
    |           N / Col Total |
    |         N / Table Total |
    |-------------------------|
    
     
    Total Observations in Table:  100 
    
     
                     | wbcd_test_pred 
    wbcd_test_labels |    Benign | Malignant | Row Total | 
    -----------------|-----------|-----------|-----------|
              Benign |        61 |         0 |        61 | 
                     |     1.000 |     0.000 |     0.610 | 
                     |     0.968 |     0.000 |           | 
                     |     0.610 |     0.000 |           | 
    -----------------|-----------|-----------|-----------|
           Malignant |         2 |        37 |        39 | 
                     |     0.051 |     0.949 |     0.390 | 
                     |     0.032 |     1.000 |           | 
                     |     0.020 |     0.370 |           | 
    -----------------|-----------|-----------|-----------|
        Column Total |        63 |        37 |       100 | 
                     |     0.630 |     0.370 |           | 
    -----------------|-----------|-----------|-----------|
    
     
    


```R
wbcd_test_pred <- knn(train = wbcd_train, test = wbcd_test, cl = wbcd_train_labels, k=11)
CrossTable(x = wbcd_test_labels, y = wbcd_test_pred, prop.chisq=FALSE)
```

    
     
       Cell Contents
    |-------------------------|
    |                       N |
    |           N / Row Total |
    |           N / Col Total |
    |         N / Table Total |
    |-------------------------|
    
     
    Total Observations in Table:  100 
    
     
                     | wbcd_test_pred 
    wbcd_test_labels |    Benign | Malignant | Row Total | 
    -----------------|-----------|-----------|-----------|
              Benign |        61 |         0 |        61 | 
                     |     1.000 |     0.000 |     0.610 | 
                     |     0.953 |     0.000 |           | 
                     |     0.610 |     0.000 |           | 
    -----------------|-----------|-----------|-----------|
           Malignant |         3 |        36 |        39 | 
                     |     0.077 |     0.923 |     0.390 | 
                     |     0.047 |     1.000 |           | 
                     |     0.030 |     0.360 |           | 
    -----------------|-----------|-----------|-----------|
        Column Total |        64 |        36 |       100 | 
                     |     0.640 |     0.360 |           | 
    -----------------|-----------|-----------|-----------|
    
     
    


```R
wbcd_test_pred <- knn(train = wbcd_train, test = wbcd_test, cl = wbcd_train_labels, k=15)
CrossTable(x = wbcd_test_labels, y = wbcd_test_pred, prop.chisq=FALSE)
```

    
     
       Cell Contents
    |-------------------------|
    |                       N |
    |           N / Row Total |
    |           N / Col Total |
    |         N / Table Total |
    |-------------------------|
    
     
    Total Observations in Table:  100 
    
     
                     | wbcd_test_pred 
    wbcd_test_labels |    Benign | Malignant | Row Total | 
    -----------------|-----------|-----------|-----------|
              Benign |        61 |         0 |        61 | 
                     |     1.000 |     0.000 |     0.610 | 
                     |     0.953 |     0.000 |           | 
                     |     0.610 |     0.000 |           | 
    -----------------|-----------|-----------|-----------|
           Malignant |         3 |        36 |        39 | 
                     |     0.077 |     0.923 |     0.390 | 
                     |     0.047 |     1.000 |           | 
                     |     0.030 |     0.360 |           | 
    -----------------|-----------|-----------|-----------|
        Column Total |        64 |        36 |       100 | 
                     |     0.640 |     0.360 |           | 
    -----------------|-----------|-----------|-----------|
    
     
    


```R
wbcd_test_pred <- knn(train = wbcd_train, test = wbcd_test, cl = wbcd_train_labels, k=21)
CrossTable(x = wbcd_test_labels, y = wbcd_test_pred, prop.chisq=FALSE)
```

    
     
       Cell Contents
    |-------------------------|
    |                       N |
    |           N / Row Total |
    |           N / Col Total |
    |         N / Table Total |
    |-------------------------|
    
     
    Total Observations in Table:  100 
    
     
                     | wbcd_test_pred 
    wbcd_test_labels |    Benign | Malignant | Row Total | 
    -----------------|-----------|-----------|-----------|
              Benign |        61 |         0 |        61 | 
                     |     1.000 |     0.000 |     0.610 | 
                     |     0.968 |     0.000 |           | 
                     |     0.610 |     0.000 |           | 
    -----------------|-----------|-----------|-----------|
           Malignant |         2 |        37 |        39 | 
                     |     0.051 |     0.949 |     0.390 | 
                     |     0.032 |     1.000 |           | 
                     |     0.020 |     0.370 |           | 
    -----------------|-----------|-----------|-----------|
        Column Total |        63 |        37 |       100 | 
                     |     0.630 |     0.370 |           | 
    -----------------|-----------|-----------|-----------|
    
     
    


```R
wbcd_test_pred <- knn(train = wbcd_train, test = wbcd_test, cl = wbcd_train_labels, k=27)
CrossTable(x = wbcd_test_labels, y = wbcd_test_pred, prop.chisq=FALSE)
```

    
     
       Cell Contents
    |-------------------------|
    |                       N |
    |           N / Row Total |
    |           N / Col Total |
    |         N / Table Total |
    |-------------------------|
    
     
    Total Observations in Table:  100 
    
     
                     | wbcd_test_pred 
    wbcd_test_labels |    Benign | Malignant | Row Total | 
    -----------------|-----------|-----------|-----------|
              Benign |        61 |         0 |        61 | 
                     |     1.000 |     0.000 |     0.610 | 
                     |     0.938 |     0.000 |           | 
                     |     0.610 |     0.000 |           | 
    -----------------|-----------|-----------|-----------|
           Malignant |         4 |        35 |        39 | 
                     |     0.103 |     0.897 |     0.390 | 
                     |     0.062 |     1.000 |           | 
                     |     0.040 |     0.350 |           | 
    -----------------|-----------|-----------|-----------|
        Column Total |        65 |        35 |       100 | 
                     |     0.650 |     0.350 |           | 
    -----------------|-----------|-----------|-----------|
    
     
    

### k = 5 , k = 21일 때 가장 나은 성능을 나타낸다.
