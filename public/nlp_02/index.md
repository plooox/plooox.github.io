# [NLP] 2.텍스트 전처리 - 토큰화

<!--more-->

# Text Preprocessing
# Tokenization
주어진 corpus에서 토큰(token)이라 불리는 단위로 나누는 작업을 지칭한다
## Word Tokenization
토큰의 기준을 단어(word)로 하는 경우, 단어 토른화라고 한다.  
    ex) 구두점(. , ? ; !)을 제외, 띄어쓰기를 기준으로 토큰화  
    ```Time is an illusion. Lunchtime double so!```
    ```> "Time", "is", "an", "illustion", "Lunchtime", "double", "so"```
### 고려 사항
1. 구두점, 특수 문자 제외 여부 선택  
   ex) 2021/08/15: '/'가 날짜를 구분해 주는 역할
2. 줄임말, 단어 내 띄어쓰기
    ex) New York: "New"와 "York"로 분리시 본래 의미가 없어질 수 있음

### Penn Treebank Tokenization
표준으로 쓰이는 토큰화 방법 중 하나  
Rule)  
1. 하이픈('-')으로 구성된 단어는 하나로 유지
2. 아포스트로피('`')로 "접어"가 함께하는 단어는 분리

```python
from nltk.tokenize import TreebankWordTokenizer

tokenizer = TreebankWordTokenizer()
txt = "Starting a home-based restaurant may be an ideal. it doesn't have a food chain or restaurant of their own."

print(tokenizer.tokenize(txt))
```  
```
['Starting', 'a', 'home-based', 'restaurant', 'may', 'be', 'an', 'ideal.', 'it', 'does', "n't", 'have', 'a', 'food', 'chain', 'or', 'restaurant', 'of', 'their', 'own', '.']
```

## Sentence Tokenization
토큰의 단위가 문장(sentence)인 경우  
corpus를 문장 단위로 분류하기 위해서는, corpus가 어떤 국적의 언어인지, 해당 corpus에서 특수문자들이 어떻게 사용되고 있는지에 따라 직접 규칙을 정의할 필요가 있다.
```python
# NLTK에서 지원하는 영어문장 토큰화 - sent_tokenize
import nltk
nltk.download('punkt')
from nltk.tokenize import sent_tokenize

sent_tokenize(text)
text="His barber kept his word. But keeping such a huge secret to himself was driving him crazy. Finally, the barber went up a mountain and almost to the edge of a cliff. He dug a hole in the midst of some reeds. He looked about, to make sure no one was near."
```
```
['His barber kept his word.',
 'But keeping such a huge secret to himself was driving him crazy.',
 'Finally, the barber went up a mountain and almost to the edge of a cliff.',
 'He dug a hole in the midst of some reeds.',
 'He looked about, to make sure no one was near.']
```
 
```python
# 한국어 문장 토큰화 도구
import kss

txt = '딥 러닝 자연어 처리가 재미있기는 합니다. 그런데 문제는 영어보다 한국어로 할 때 너무 어려워요. 농담아니에요. 이제 해보면 알걸요?'
print(kss.split_sentences(txt))
```
```
['딥 러닝 자연어 처리가 재미있기는 합니다.', '그런데 문제는 영어보다 한국어로 할 때 너무 어려워요.', '농담아니에요.', '이제 해보면 알걸요?']

```

## Binary Classifier
문장 토큰화의 예외 사항을 발생시키는 마침표의 처리를 위해, 입력에 따라 두 개의 클래스로 분류하는 이진 분류기(binary classifier)를 사용하기도 한다.
 
두 개의 클래스
1. 마침표(.)가 단어의 일부분(약어)인 경우
2. 마침표(.)가 문장의 구분자인 경우

## 한국어 토큰화의 어려움
1. 교착어
   - 어근과 접사에 의해 단어의 기능이 결정.
   - 형태소(morpheme)의 개념 이해가 필수적: **형태소 토큰화**
        -> 형태소: 뜻을 가진 가장 작은 말의 단위  
        - 자립 형태소: 접사, 어미, 조사와 상관없이 사용할 수 있는 형태소  
        - 의존 형태소: 다른 형태소와 결합하여 사용되는 형태소
2. 띄어쓰기가 지켜지지 않는 corpus가 비교적 많음

## 품사 태깅(Part-of-speech tagging)
단어의 표기는 같으나, 품사에 따라서 단어의 의미가 달라지기도 함.  
단어의 의미를 제대로 파악하기 위해서는 단어가 어떤 품사로 쓰였는지 구분하는 것이 주요 지표가 될 수 있음.

### nltk pos_tag
```python
from nltk.tokenize import word_tokenize
from nltk.tag import pos_tag

text="I am actively looking for Ph.D. students. and you are a Ph.D. student."
a = word_tokenize(text)
pos_tag(a)
```
```
[('I', 'PRP'),          #인칭대명사
 ('am', 'VBP'),         #동사
 ('actively', 'RB'),    #부사
 ('looking', 'VBG'),    #현재부사
 ('for', 'IN'),         #전치사
 ('Ph.D.', 'NNP'),      #고유명사
 ('students', 'NNS'),   #복수형명사
 ('.', '.'),            
 ('and', 'CC'),         #접속사
 ('you', 'PRP'),        
 ('are', 'VBP'),
 ('a', 'DT'),           #관사
 ('Ph.D.', 'NNP'),
 ('student', 'NN'),
 ('.', '.')]
```

### KoNLPy
한국어 자연어 처리를 위한 파이선 패키지  
형태소 분석기로  Okt(Open Korea Text), 메캅(Mecab), 코모란(Komoran), 한나눔(Hannanum), 꼬꼬마(Kkma) 지원  
형태소 분석기 마다 성능과 결과가 다르게 나오기 때문에, 필요 용도에 따라 어던 형태소 분석기가 적절한지 판단하고 사용.
```python
#okt
from konlpy.tag import Okt  
okt=Okt()  
print(okt.morphs("열심히 코딩한 당신, 연휴에는 여행을 가봐요"))
print(okt.pos("열심히 코딩한 당신, 연휴에는 여행을 가봐요"))  
print(okt.nouns("열심히 코딩한 당신, 연휴에는 여행을 가봐요"))  
```
```
['열심히', '코딩', '한', '당신', ',', '연휴', '에는', '여행', '을', '가봐요']
[('열심히', 'Adverb'), ('코딩', 'Noun'), ('한', 'Josa'), ('당신', 'Noun'), (',', 'Punctuation'), ('연휴', 'Noun'), ('에는', 'Josa'), ('여행', 'Noun'), ('을', 'Josa'), ('가봐요', 'Verb')]
['코딩', '당신', '연휴', '여행']
```
```python
#kkma
from konlpy.tag import Kkma  
kkma=Kkma()  
print(kkma.morphs("열심히 코딩한 당신, 연휴에는 여행을 가봐요"))
print(kkma.pos("열심히 코딩한 당신, 연휴에는 여행을 가봐요"))  
print(kkma.nouns("열심히 코딩한 당신, 연휴에는 여행을 가봐요"))  
```
```
['열심히', '코딩', '하', 'ㄴ', '당신', ',', '연휴', '에', '는', '여행', '을', '가보', '아요']
[('열심히', 'MAG'), ('코딩', 'NNG'), ('하', 'XSV'), ('ㄴ', 'ETD'), ('당신', 'NP'), (',', 'SP'), ('연휴', 'NNG'), ('에', 'JKM'), ('는', 'JX'), ('여행', 'NNG'), ('을', 'JKO'), ('가보', 'VV'), ('아요', 'EFN')]
['코딩', '당신', '연휴', '여행']
```

참고) [딥러닝을 이용한 자연어 처리 입문](https://wikidocs.net/book/2155)
<!--more-->

