# [NLP] 4.텍스트 전처리 - 인코딩 & 패딩

<!--more-->

# 정수 인코딩
컴퓨터는 텍스트보다 숫자를 더 잘 처리할 수 있다.  
이 특징을 살리기 위해 NLP에서는 텍스트를 숫자로 바꾸는 기법들을 제안한다.  
이러한 기법들은 각 단어를 고유한 정수값에 Mapping시키는 전처리 작업이 요구되기도 한다.
### Dictionary
```python
from nltk.tokenize import sent_tokenize
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords

text = "A barber is a person. a barber is good person. a barber is huge person. he Knew A Secret! The Secret He Kept is huge secret. Huge secret. His barber kept his word. a barber kept his word. His barber kept his secret. But keeping and keeping such a huge secret to himself was driving the barber crazy. the barber went up a huge mountain."

text = sent_tokenize(text)  #문장 토큰화
vocab = {} 
sentence = []
stop_words = set(stopwords.words('english'))

for i in text:
    s = word_tokenize(i) # 문장에 대해 단어 토큰화
    res = []

    for w in s:
        w = w.lower() #대문자 -> 소문자
        if w not in stop_words:  #불용어 제거
            if len(w) > 2:  # 단어 길이가 2이하인 경우 제거
                res.append(w)
                if w not in vocab:
                    vocab[w] = 0
                vocab[w] += 1
    sentence.append(res)
print("Word Tokenize")
print(sentence) # 단어토큰화
print("Word Frequency")
print(vocab)    # 중복을 제거한 단어의 빈도수

vocab_sorted = sorted(vocab.items(), key = lambda x:x[1], reverse= True) #빈도수 순으로 정렬
print(vocab_sorted)

word_to_idx = {}    #빈도수 높은 순서대로 고유번호 부여
idx = 0
for (word, freq) in vocab_sorted:
    if freq > 1:
        idx += 1
        word_to_idx[word] = idx
print("Encoded Word")
print(word_to_idx)
```
```python
Word Tokenize
[['barber', 'person'], ['barber', 'good', 'person'], ['barber', 'huge', 'person'], ['knew', 'secret'], ['secret', 'kept', 'huge', 'secret'], ['huge', 'secret'], ['barber', 'kept', 'word'], ['barber', 'kept', 'word'], ['barber', 'kept', 'secret'], ['keeping', 'keeping', 'huge', 'secret', 'driving', 'barber', 'crazy'], ['barber', 'went', 'huge', 'mountain']]

Word Frequency
{'barber': 8, 'person': 3, 'good': 1, 'huge': 5, 'knew': 1, 'secret': 6, 'kept': 4, 'word': 2, 'keeping': 2, 'driving': 1, 'crazy': 1, 'went': 1, 'mountain': 1}

[('barber', 8), ('secret', 6), ('huge', 5), ('kept', 4), ('person', 3), ('word', 2), ('keeping', 2), ('good', 1), ('knew', 1), ('driving', 1), ('crazy', 1), ('went', 1), ('mountain', 1)]

Encoded Word
{'barber': 1, 'secret': 2, 'huge': 3, 'kept': 4, 'person': 5, 'word': 6, 'keeping': 7}
```
```word_to_idx```를 이용하여 토큰화된 sentence의 각 단어를 정수로 변환시킬 수 있다.
> ex) ['barber', 'person'] -> [1,5]
> 
단어 집합에 존재하지 않는 단어들은 **Out-Of-Voca**라고 하며, 새로운 index르 인코딩한다.
```python
word_to_idx['OOV'] = len(word_to_idx) + 1
```
### Counter
```python
from collections import Counter

words = sum(sentence, [])       #단어집합을 만들기 위해 문장의 경계인 []제거
vocab = Counter(words)          #중복을 제거하고 단어의 빈도수 기록
vocab = vocab.most_common(5)    #등장 빈도수가 높은 상위 5개의 단어만 저장
print(vocab)
```
```python
[('barber', 8), ('secret', 6), ('huge', 5), ('kept', 4), ('person', 3)]
```
### NLTK - FreqDist
```Counter()```와 사용방법이 동일
```python
from nltk import FreqDist
import numpy as np

vocab = FreqDist(np.hstack(sentence))   #문장 구분 제거 == sum(sentence, [])
print(vocab.most_common(5))
```
```python
[('barber', 8), ('secret', 6), ('huge', 5), ('kept', 4), ('person', 3)]
```
### enumerate
enumerate: 순서가 있는 자료형(list, set, tuple,dictionary, string)을 입력받아 인덱스를 순차적으로 리턴
```python
word_to_index = {word[0]: index+1 for index, word in enumerate(vocab)}
print(word_to_idx)
```
```python
{'barber': 1, 'secret': 2, 'huge': 3, 'kept': 4, 'person': 5, 'word': 6, 'keeping': 7}
```
## Keras
Keras는 기본적인 전처리 도구를 제공
정수 인코딩의 경우, Keras의 Tokenizer를 사용하기도 한다.
```python
from tensorflow.keras.preprocessing.text import Tokenizer

tokenizer = Tokenizer()
tokenizer.fit_on_texts(sentences) #sentences: 단어 토큰화된 데이터

print("[Word index]")   # 인덱스 출력
print(tokenizer.word_index)
print("[Word count]")   # 카운트 출력
print(tokenizer.word_counts)
print("[sequences]")    # corpus를 인덱스로 변환
print(tokenizer.texts_to_sequences(sentence))
```
```
[Word index]
{'barber': 1, 'secret': 2, 'huge': 3, 'kept': 4, 'person': 5, 'word': 6, 'keeping': 7, 'good': 8, 'knew': 9, 'driving': 10, 'crazy': 11, 'went': 12, 'mountain': 13}
[Word count]
OrderedDict([('barber', 8), ('person', 3), ('good', 1), ('huge', 5), ('knew', 1), ('secret', 6), ('kept', 4), ('word', 2), ('keeping', 2), ('driving', 1), ('crazy', 1), ('went', 1), ('mountain', 1)])
[sequences]
[[1, 5], [1, 8, 5], [1, 3, 5], [9, 2], [2, 4, 3, 2], [3, 2], [1, 4, 6], [1, 4, 6], [1, 4, 2], [7, 7, 3, 2, 10, 1, 11], [1, 12, 3, 13]]
```
Keras Tokenizer는 기본적으로 OOV에 대해 정수 인코딩 과정에서 단어를 제거한다는 특징을 가진다.  
따라서 OOV를 보존하고 싶으면 인자 ```oov_token```을 사용한다.
```python
voca_size = 5
# 빈도수 상위 5개 단어만 사용, OOV와 숫자 0 고려하여 크기는 +2
tokenizer = Tokenizer(num_words= voca_size + 2, oov_token='OOV')
tokenizer.fit_on_texts(sentences)
print(tokenizer.texts_to_sequences(sentences))
```
```python
[[2, 6], [2, 1, 6], [2, 4, 6], [1, 3], [3, 5, 4, 3], [4, 3], [2, 5, 1], [2, 5, 1], [2, 5, 3], [1, 1, 4, 3, 1, 2, 1], [2, 1, 4, 1]]
```
# Padding
기계는 길이가 동일한 문서에 대해 행렬 연산이 가능  
병렬 연산을 위해서 여러 문장의 길이를 동일하게 맞춰주는 작업  
**데이터에 특정 값을 채워서 데이터의 크기(shape)를 조정하는 작업**

## Numpy 사용
```python
import numpy as np

max_len = max(len(item) for item in encoded) #문장의 최대 길이 구하기
for item in encoded:
    while len(item) < max_len:
        item.append(0)  # 숫자 0 사용 -> Zero padding

padded_np = np.array(encoded)
print(padded_np)
```
```python
[[ 1  5  0  0  0  0  0]
 [ 1  8  5  0  0  0  0]
 [ 1  3  5  0  0  0  0]
 [ 9  2  0  0  0  0  0]
 [ 2  4  3  2  0  0  0]
 [ 3  2  0  0  0  0  0]
 [ 1  4  6  0  0  0  0]
 [ 1  4  6  0  0  0  0]
 [ 1  4  2  0  0  0  0]
 [ 7  7  3  2 10  1 11]
 [ 1 12  3 13  0  0  0]]
```
## keras 사용
```python
from tensorflow.keras.preprocessing.sequence import pad_sequences

encoded = tokenizer.texts_to_sequences(sentences)
pad_seq = pad_sequences(encoded)
print(pad_seq)
```
```python
[[ 1  5  0  0  0  0  0]
 [ 1  8  5  0  0  0  0]
 [ 1  3  5  0  0  0  0]
 [ 9  2  0  0  0  0  0]
 [ 2  4  3  2  0  0  0]
 [ 3  2  0  0  0  0  0]
 [ 1  4  6  0  0  0  0]
 [ 1  4  6  0  0  0  0]
 [ 1  4  2  0  0  0  0]
 [ 7  7  3  2 10  1 11]
 [ 1 12  3 13  0  0  0]]
```
### 길이 제한, 값 수정
```python
# 길이제한: maxlen = 5
# value 수정: 사용된 정수와 겹치지 않게 단어 집합 크기보다 1 큰 수 사용 
padded = pad_sequences(encoded, maxlen=5, padding='post', value = len(tokenizer.word_index)+1)
print(padded)
```
```python
# 길이가 maxlen(= 5)보다 작으면 0으로 패딩, 크면 데이터 소멸
[[ 1  5 14 14 14]
 [ 1  8  5 14 14]
 [ 1  3  5 14 14]
 [ 9  2 14 14 14]
 [ 2  4  3  2 14]
 [ 3  2 14 14 14]
 [ 1  4  6 14 14]
 [ 1  4  6 14 14]
 [ 1  4  2 14 14]
 [ 3  2 10  1 11]
 [ 1 12  3 13 14]]
```
# One-Hot Encoding
## Vocabulary(단어집합)
서로 다른 단어들의 집합  
텍스트 데이터의 모든 단어를 중복을 허용하지 않고 모아놓을 것을 지칭  
One-Hot encoding은 단어 집합을 만들고 각 단어에 정수 인덱스를 부여하는 것에서부터 시작된다.
## One-Hot Encoding
단어 집합의 크기를 백터의 차원으로 하고, 표현하고자 하는 단어의 인덱스에 1의 값을 부여하는 단어의 벡터 표현 방식.  
이렇게 표현된 벡터를 One-Hot vector라고 한다.
## Keras 활용
```python
from tensorflow.keras.preprocessing.text import Tokenizer
from tensorflow.keras.utils import to_categorical

text = "나랑 점심 먹으러 갈래 점심 메뉴는 햄버거 갈래 갈래 햄버거 최고야"
sub_text="점심 먹으러 갈래 메뉴는 햄버거 최고야"

# 1. 정수 인코딩
t = Tokenizer()
t.fit_on_texts([text])  # 각 단어에 대한 인코딩
print(t.word_index)

encoded = t.texts_to_sequences([sub_text])[0] #[[1,2,3,4]] -> [1,2,3,4]
print(encoded)

# 2. One-Hot vector 생성
one_hot = to_categorical(encoded)
print(one_hot)
```
```python
{'갈래': 1, '점심': 2, '햄버거': 3, '나랑': 4, '먹으러': 5, '메뉴는': 6, '최고야': 7} #단어 인덱스
[2, 5, 1, 6, 3, 7]  # 정수 인코딩
[[0. 0. 1. 0. 0. 0. 0. 0.]  # 원-핫 인코딩
 [0. 0. 0. 0. 0. 1. 0. 0.]
 [0. 1. 0. 0. 0. 0. 0. 0.]
 [0. 0. 0. 0. 0. 0. 1. 0.]
 [0. 0. 0. 1. 0. 0. 0. 0.]
 [0. 0. 0. 0. 0. 0. 0. 1.]]
```
## 한계점
1. 단어의 개수가 늘어날 수록, 벡터를 저장하기 위한 공간이 늘어난다.
2. 단어의 유사도를 반영할 수 없음  
   ex) book과 books를 일반적으로 다른 단어라고 취급

참고) [딥러닝을 이용한 자연어 처리 입문](https://wikidocs.net/book/2155)
<!--more-->

