# [NLP] 3.텍스트 전처리 - 정제및 정규화

<!--more-->

# Cleaning and Normalization
Tokenization 전, 텍스트 데이터를 용도에 맞게 정제(cleaning)및 정규화(normalization)하는 일

- 정제(cleaning): 가지고 있는 corpus로부터 noise data제거
- 정규화(normalization): 표현 방법이 다른 단어들을 통합, 같은 단어로 재구성

## 규칙에 기반한 표기가 다른 단어 통합
같은 의미를 갖고 있는 표기가 다른 단어들을 하나의 단어로 정규화
### Lemmatization(표제어 추출)
단어들로부터 표제어를 찾아가는 과정  
```ex) am, are, is -> be```

**형태학적 파싱**  
단어의 어간(stem, 단어의 의미를 가지는 핵심 부분)과 접사(affix, 단어의 추가적인 의미 부여)를 분리하는 작업  
NLTK에서는 WorkNetLemmatizer를 지원  
```python
from nltk.stem import WordNetLemmatizer
n=WordNetLemmatizer()
words=['policy', 'doing', 'organization', 'have', 'going', 'love', 'lives', 'fly', 'dies', 'watched', 'has', 'starting']
print([n.lemmatize(w) for w in words])
```
```python
['policy', 'doing', 'organization', 'have', 'going', 'love', 'life', 'fly', 'dy', 'watched', 'ha', 'starting']
# 'dy', 'ha': 적절하지 못한 단어
# lemmatizer는 단어의 품사 정보를 알아야만 정확한 결과를 도출한다.
# WordNetLemmatizer는 입력으로 단어의 품사 정보 제공 가능 ex) n.lemmatize('has', 'v')
```
-> 단어의 형태가 어느정도 보존되는 특징을 가진다.
### Stemming(어간 추출)
단어에서 어간(stem)을 추출하는 과정  
섬세한 작업이 아니기 때문에, 어간추출후에 나오는 결과는 사전에 없는 단어인 경우도 있다.  
일반적으로 표제어 추출보다 빠른 특성을 가진다.

NLTK에서는 Porter stemmer와 Lancaster Stemmer를 지원한다.  
```python
#Porter Stemmer
from nltk.stem import PorterStemmer
s=PorterStemmer()
words=['policy', 'doing', 'organization', 'have', 'going', 'love', 'lives', 'fly', 'dies', 'watched', 'has', 'starting']
print([s.stem(w) for w in words])
```
```python
['polici', 'do', 'organ', 'have', 'go', 'love', 'live', 'fli', 'die', 'watch', 'ha', 'start']
```

```python
from nltk.stem import LancasterStemmer
l=LancasterStemmer()
words=['policy', 'doing', 'organization', 'have', 'going', 'love', 'lives', 'fly', 'dies', 'watched', 'has', 'starting']
print([l.stem(w) for w in words])
```
```python
['policy', 'doing', 'org', 'hav', 'going', 'lov', 'liv', 'fly', 'die', 'watch', 'has', 'start']
```
-> Stemmer 알고리즘에 따라 동일한 단어에서도 다른 결과를 출력하기 때문에, 어떤 Stemmer가 적합한지 판단하고 사용해야한다.


## 대, 소문자 통합
영어권에서 대문자의 경우, 특정 상황에서만 쓰이기 때문에, 대문자를 소문자로 반환하는 형식으로 단어의 수를 줄일 수 있다.  
## 불필요한 단어 제거
Noise data: 아무 의미를 가지지 않는 글자(특수문자) + 분석하고자 하는 목적에 맞지 않는 불필요한 단어

### 불용어 제거
불용어(Stopword): 문장의 실제 의미 분석을 하는데 도움되지 않는 단어(I, my, me, over ...)  
직접 불용어를 정의해서 사용하거나, 패키지에서 정의된 불용어를 사용  
NLTK에서는 100여개 이상의 단어를 불용어로 패키지 내에서 정의하고 있다.  
```python
from nltk.corpus import stopwords 
from nltk.tokenize import word_tokenize 

example = "Family is not an important thing. It's everything."
stop_words = set(stopwords.words('english')) 

word_tokens = word_tokenize(example)

result = []
for w in word_tokens: 
    if w not in stop_words: # 단어가 불용어가 아니면 추가
        result.append(w) 
```

### 등장 빈도가 적은 단어 제거
텍스트 데이터에서 너무 적게 등장하여 자연어 처리에 도움이 되지 않는 데이터들을 제거.  
### 길이가 짧은 단어
ex) 100000개의 메일 데이터에서 3~4번 밖에 등장하지 않는 데이터 제거
영어권 언어의 경우 길이가 짧은 언어들을 제거하는 것은 문장에서 크게 의미가 없는 단어들을 제거하는 효과를 보여준다.  
길이가 2~3 이하인 단어들을 제거하는 것은 많은 수의 불용어(it, at, on ...)들을 제거하는데 효과적이나, 길이가 짧은 명사들(ox, fox, cat ...) 또한 제거될 가능성이 있다.

## Regular Expression
특정한 규칙을 가진 문자열의 집합을 표현하는데 사용하는 형식 언어.  
문자열을 처리하는 방법 중의 하나로, 정규 표현식을 사용하면 특정한 조건의 문자를 ‘검색’하거나 ‘치환’하는 과정을 매우 간편하게 처리할 수 있다.  
정규표현식을 이용하면 코퍼스 내부에서 반복적으로 등장하는 글자들을 규칙에 기반하여 쉽게 제거할 수 있다.
```python
# 길이가 1~2인 단어들을 정규 표현식을 이용하여 삭제
import re
text = "I was wondering if anyone out there could enlighten me on this car."
shortword = re.compile(r'\W*\b\w{1,2}\b')
print(shortword.sub('', text))
```

```python
 was wondering anyone out there could enlighten this car.
```

참고) [딥러닝을 이용한 자연어 처리 입문](https://wikidocs.net/book/2155)
<!--more-->

