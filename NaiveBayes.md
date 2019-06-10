
# Naive Bayes

* 특성들 사이의 독립을 가정하는 베이즈 정리를 적용한 확률 분류기
* 단일 알고리즘을 통한 훈련이 아닌 일반적인 원칙에 근거한 여러 알고리즘들을 이용하여 훈련된다. 
* 텍스트 분류에서 문서를 여러 범주 (예: 스팸, 스포츠, 정치)중 하나로 판단하는 문제에 대한 대중적인 방법
* 자동 의료 진단 분야에서의 응용사례를 보면, 적절한 전처리를 하면 더 진보 된 방법들 (예: 서포트 벡터 머신 (Support Vector Machine))과도 충분한 경쟁력을 보임을 알 수 있다.

장점
1. 지도 학습 (Supervised Learning) 환경에서 매우 효율적으로 훈련 될 수 있다.
  + 많은 실제 응용에서, 나이브 베이즈 모델의 파라미터 추정은 최대우도방법 (Maximum Likelihood Estimation (MLE))을 사용하며, 베이즈 확률론이나 베이지안 방법들은 이용하지 않고도 훈련이 가능하다. 
2. 상당히 적은 데이터로도 파라미터를 추정할 수 있다.
3. 간단한 디자인과 단순한 가정에도 불구하고, 많은 복잡한 실제 상황에서 잘 작동한다. 

성능에 대한 토론

1. 나이브 베이즈 분류는 독립성 가정이 종종 부정확한 결과를 내기도 한다. 하지만 독립성 가정으로 나이브 베이즈 분류를 매우 편리하고 유용하게 사용할 수 있다.
  + 특히 클래스의 조건부 특성 분포를 디커플링 하는 것은 각 분포가 독립적으로 일차원의 분포로 추정 할 수 있게 한다. 
  + 이를 통해서, 특성의 수에 비해 기하 급수적으로 많은 수의 데이터 셋을 필요로 하게되는 차원의 저주의 문제를 완화 할 수 있다.

2. 나이브 베이즈는 종종 올바른 클래스 확률을 도출하는 좋은 추정치를 생성하는데 실패하곤 하지만, 이 추정치가 많은 응용에서 반드시 필요한 요건은 아니다.
  + 예를 들어, 올바른 클래스가 다른 클래스 보다 확률이 높을 가능성이 더 크기 때문에, 나이브 베이즈 분류는 올바른 MAP 결정 규칙에 따른 분류를 할 수 있다. 이는 확률 추정치가 약간 혹은 현저하게 부정확하더라도 항상 마찬가지이다.
  + ==> 나이브 확률 모델의 심각한 결함을 충분히 무시할 만큼 강력하다고 할 수 있다.

### example : 트위터 분류 모델 만들기


#### 1. library 호출, data 불러오기


```R
# 필요한 library
library(tm)
library(NLP)
library(SnowballC)
```

    Warning message:
    "package 'tm' was built under R version 3.5.2"Loading required package: NLP
    Warning message:
    "package 'SnowballC' was built under R version 3.5.2"


```R
# 데이터 불러오기
tweets.mandrill<-read.csv("Mandrill.csv",header=T)
tweets.other<-read.csv('Other.csv',header=T)
tweets.mandrill["class"]<-rep("App",nrow(tweets.mandrill))
tweets.other["class"]<-rep("Other",nrow(tweets.other))
total<-rbind(tweets.mandrill, tweets.other)
```

#### 2. 데이터 구조 살펴보고 수정


```R
str(total)
```

    'data.frame':	300 obs. of  2 variables:
     $ Tweet: Factor w/ 295 levels "#freelance #Jobs Mandrill Template design and strategy implementation by eshuys: Hi there,     I have just had."| __truncated__,..: 81 82 19 20 21 22 23 24 25 26 ...
     $ class: chr  "App" "App" "App" "App" ...
    


```R
total$class<-factor(total$class) #class는 factor화
str(total$class)
```

     Factor w/ 2 levels "App","Other": 1 1 1 1 1 1 1 1 1 1 ...
    

#### 3. Corpus 형태로 반환


```R
total_corpus <-VCorpus(VectorSource(total$Tweet))
as.character(total_corpus[[1]])
```


<span style=white-space:pre-wrap>'[blog] Using Nullmailer and Mandrill for your Ubuntu Linux server outboud mail:  http://bit.ly/ZjHOk7  #plone'</span>


#### 4. Corpus 전체 문서 내 전처리


```R
x<-tm_map(total_corpus, content_transformer(tolower))	#대문자->소문자
x<-tm_map(x, removePunctuation)	#구두점제거
x<-tm_map(x, removeNumbers) #숫자제거
x<-tm_map(x, stripWhitespace)	#여러개의 공백을 하나로
mystopwords<-c(stopwords('english'), "will", "just", "get", "mandrill")	#불용어
x<-tm_map(x, removeWords, mystopwords) #불용어제거
x<-tm_map(x, stemDocument, language="english") #어근추출
```

#### 5. Document Term Matrix 생성


```R
dtm<-DocumentTermMatrix(x)	#dtm
```

#### 6. Sampling-> train, test(7:3)



```R
set.seed(1004)
k<-which(sample(nrow(total))<=ceiling(nrow(total)*0.7))
dtm_train<-dtm[k,]
dtm_test<-dtm[-k,]
train_class<-total[k,]$class
test_class<-total[-k,]$class
```


```R
prop.table(table(train_class))
prop.table(table(test_class))
```


    train_class
          App     Other 
    0.5047619 0.4952381 



    test_class
          App     Other 
    0.4888889 0.5111111 


#### 7. word cloud


```R
library(wordcloud)
wordcloud(x, max.words=100, random.order=F)
```

    Warning message:
    "package 'wordcloud' was built under R version 3.5.3"Loading required package: RColorBrewer
    


![png](output_21_1.png)



```R
app<-subset(total, class=="App")
other<-subset(total, class=="Other")
wordcloud(app$Tweet)
wordcloud(other$Tweet)
```

    Warning message in tm_map.SimpleCorpus(corpus, tm::removePunctuation):
    "transformation drops documents"Warning message in tm_map.SimpleCorpus(corpus, function(x) tm::removeWords(x, tm::stopwords())):
    "transformation drops documents"Warning message in tm_map.SimpleCorpus(corpus, tm::removePunctuation):
    "transformation drops documents"Warning message in tm_map.SimpleCorpus(corpus, function(x) tm::removeWords(x, tm::stopwords())):
    "transformation drops documents"


![png](output_22_1.png)



![png](output_22_2.png)


#### 8. find frequent terms => training


```R
total_freq<-findFreqTerms(dtm_train, 2)
str(total_freq)
```

     chr [1:241] "abl" "acapella" "account" "addit" "adicionei" "alreadi" ...
    


```R
dtm_freq_train<-dtm_train[, total_freq]
dtm_freq_test<-dtm_test[, total_freq]
```


```R
convert_counts<-function(x){
  x<-ifelse(x>0, "Yes", "No")}
```


```R
tweet_train<-apply(dtm_freq_train, 2, convert_counts) #2 :열 단위 연산 
tweet_test<-apply(dtm_freq_test, 2, convert_counts) #2 :열 단위 연산
head(tweet_train)
```


<table>
<thead><tr><th></th><th scope=col>abl</th><th scope=col>acapella</th><th scope=col>account</th><th scope=col>addit</th><th scope=col>adicionei</th><th scope=col>alreadi</th><th scope=col>altern</th><th scope=col>amaz</th><th scope=col>anim</th><th scope=col>anyon</th><th scope=col>...</th><th scope=col>wed</th><th scope=col>week</th><th scope=col>well</th><th scope=col>whistlerbean</th><th scope=col>wordpress</th><th scope=col>worth</th><th scope=col>yeah</th><th scope=col>your</th><th scope=col>youtub</th><th scope=col>youtubeさんから</th></tr></thead>
<tbody>
	<tr><th scope=row>1</th><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>...</td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td></tr>
	<tr><th scope=row>2</th><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>...</td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td></tr>
	<tr><th scope=row>5</th><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>...</td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td></tr>
	<tr><th scope=row>7</th><td>No </td><td>No </td><td>Yes</td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>...</td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td></tr>
	<tr><th scope=row>8</th><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>...</td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>Yes</td><td>No </td><td>No </td><td>No </td><td>No </td></tr>
	<tr><th scope=row>9</th><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>...</td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td><td>No </td></tr>
</tbody>
</table>




```R
library(e1071)
tweet_classifier<-naiveBayes(tweet_train, train_class)
```

    Warning message:
    "package 'e1071' was built under R version 3.5.3"

#### 9. evaluate


```R
tweet_test_pred<-predict(tweet_classifier, tweet_test)
```


```R
# evaluating model performance
library(gmodels)
CrossTable(tweet_test_pred, test_class, prob.chisq=FALSE, prop.t=FALSE,
           dnn=c('predicted', 'actual'))
```

    
     
       Cell Contents
    |-------------------------|
    |                       N |
    | Chi-square contribution |
    |           N / Row Total |
    |           N / Col Total |
    |-------------------------|
    
     
    Total Observations in Table:  90 
    
     
                 | actual 
       predicted |       App |     Other | Row Total | 
    -------------|-----------|-----------|-----------|
             App |        38 |         4 |        42 | 
                 |    14.858 |    14.212 |           | 
                 |     0.905 |     0.095 |     0.467 | 
                 |     0.864 |     0.087 |           | 
    -------------|-----------|-----------|-----------|
           Other |         6 |        42 |        48 | 
                 |    13.001 |    12.436 |           | 
                 |     0.125 |     0.875 |     0.533 | 
                 |     0.136 |     0.913 |           | 
    -------------|-----------|-----------|-----------|
    Column Total |        44 |        46 |        90 | 
                 |     0.489 |     0.511 |           | 
    -------------|-----------|-----------|-----------|
    
     
    

### example : 성별 예측하기

* data : gender.csv
* features
    + favorite color
    + favorite music genre
    + favorite beverage
    + favorite soft drink

#### 데이터 불러오기


```R
gender<-read.csv("gender.csv")
```

#### seed number : 7, train data와 test data를 2:1의 비율로 갖는 train과 test각각의 labeled 데이터 생성


```R
set.seed(7)
gender<-gender[sample(nrow(gender)),]
R<- nrow(gender)*2/3
gender_train<- gender[1:R,-1]
gender_test<- gender[(R+1):nrow(gender),-1]
```


```R
gender$Gender<- factor(gender$Gender)
gender_labels_train<- gender[1:R,'Gender']
gender_labels_test<- gender[(R+1):nrow(gender),'Gender']
```

#### naiveBayes함수로 classifier 모델을 만들고, predict함수로 예측 벡터 생성


```R
classifier<- naiveBayes(gender_train, gender_labels_train)
pred<- predict(classifier, gender_test)
```

#### 예측 결과 확인


```R
library(gmodels)
CrossTable(pred, gender_labels_test,prop.chisq = F, prop.t = F, dnn=c('predicted','actual'))
```

    
     
       Cell Contents
    |-------------------------|
    |                       N |
    |           N / Row Total |
    |           N / Col Total |
    |-------------------------|
    
     
    Total Observations in Table:  22 
    
     
                 | actual 
       predicted |         F |         M | Row Total | 
    -------------|-----------|-----------|-----------|
               F |         6 |         5 |        11 | 
                 |     0.545 |     0.455 |     0.500 | 
                 |     0.600 |     0.417 |           | 
    -------------|-----------|-----------|-----------|
               M |         4 |         7 |        11 | 
                 |     0.364 |     0.636 |     0.500 | 
                 |     0.400 |     0.583 |           | 
    -------------|-----------|-----------|-----------|
    Column Total |        10 |        12 |        22 | 
                 |     0.455 |     0.545 |           | 
    -------------|-----------|-----------|-----------|
    
     
    


```R

```


```R

```
