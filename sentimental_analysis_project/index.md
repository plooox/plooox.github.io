# SNS 감정분석을 통한 여론 조사

<!--more-->

<!--more-->
# 개요

대표적인 소셜 네트워크 서비스인 트위터에서 특정 키워드의 검색 결과에 대한 트윗 데이터를 수집하고, 수집한 트윗 내용을 바탕으로 감정 분석을 실시하여 특정 키워드에 대한 여론이 긍정적인지, 부정적인지를 보여주는 프로그램 입니다.

![](https://user-images.githubusercontent.com/82520143/148382820-6027e725-7d02-406b-bcae-6aeb3b422b5c.png)

# 데이터 수집

트위터의 경우, [트위터api](https://developer.twitter.com/en/docs/twitter-api)를 통해 트위터의 데이터를 받을 수 있습니다.  
하지만 트위터 api의 경우, 무료로 사용하게 되면 7일 이내의 데이터만 수집이 가능하다는 단점을 가지고 있습니다. 7일이라는 기간은 내가 필요로 하는 기간(6개월 이상)에 비하면 터무니 없이 부족했기 때문에, 외부 프로그램을 통해 데이터를 크롤링 해야만 했습니다. 오픈소스로 존재하는 수많은 크롤러들 중 대부분은 크롤링이 잘 되지 않았습니다(2021년 8월 기준). 그 중 [snscrape](https://github.com/JustAnotherArchivist/snscrape)라는 파이썬 패키지가 원활하게 데이터 수집이 되는 것을 확인하고, 이를 통해 데이터 수집을 하였습니다.



```python
import numpy as np
import pandas as pd
import datetime as dt
import re
import pickle
import csv
import time
import snscrape.modules.twitter as sntwitter

start = "2021-01-01"
end = "2021-08-31"

dateTimeList = []
posNegList = []
contentList = []
tweetList = []

querys = [ #Keywords.... ]

for query in querys:
    params = query+"lang:ko "+"since:"+start+" until:"+end

    csvRoute = "./"+query+".csv"

    csvFile = open(csvRoute, "a", newline='', encoding="utf-8")
    csvWrite.writerow(["Date","Pos/Neg","Tweet"])
    csvFile.close()

    # for each keywords...
    for i,tweet in enumerate(sntwitter.TwitterSearchScraper(params).get_items()):
        # Twitter data -> Date, Content
        # Save csv file
        tw_content = tweet.content 
        tw_date = tweet.date
        tw_posNeg = -1      #Pos/Neg setting run another code!

        csvFile = open(csvRoute, "a", newline='', encoding="utf-8")
        csvWrite = csv.writer(csvFile)

        csvWrite.writerow([tw_date, tw_posNeg, tw_content])
        csvFile.close()

f = open(csvRoute, "a", newline='', encoding="utf-8")
r = csv.reader(f)

cf = open('./Keyword1.csv', 'a', newline='', encoding='utf-8')
wf = csv.writer(cf)
wf.writerow(["Date","Pos/Neg","Tweet"])    
f.close()
cf.close()
```

# 감정 분석

수집한 트위터 데이터들을 가지고 감정 분석을 하여 트윗 텍스트가 긍정적인 의미를 내포하고 있는지, 부정적인 의미를 내포하고 있는지를 분석합니다.  
이를 위해 kakao brain에서 배포한 [PORORO(Platform Of neuRal mOdels for natuRal language prOcessing)](https://github.com/kakaobrain/pororo)를 이용하였습니다.  
PORORO에서는 다양한 자연어 처리 Task를 지원하는데, 이중에서 “Sentimental Analysis”를 이용하면 입력한 텍스트 데이터가 ‘긍정’인지, ‘부정’인지 분류할 수 있습니다.

```python
import pandas as pd
import numpy as np
import csv
from pororo import Pororo

sa = Pororo(task = "sentiment", model = "brainbert.base.ko.nsmc", lang = "ko")

fileList = [#Keywords]

# for each keywords..
for keyword in fileList:
    csvName = keyword+'_sa.csv'
    csvFile = open(csvName, 'a', newline='', encoding='utf-8')
    csvWrite = csv.writer(csvFile)
    csvWrite.writerow(['Date','Pos/Neg','Tweet'])
    csvFile.close()
    
    refCsv = pd.read_csv(keyword+'.csv')
    for idx, rowData in refCsv.iterrows():
        tw_date = rowData['Date']
        tw_content = rowData['Tweet']
        tw_posneg = -1
        try:
            if sa(tw_content) == "Positive":
                tw_posneg = 1
            elif sa(tw_content) == "Negative":
                tw_posneg = 0

            # Save csv file with Pos/Neg
            # Add for each row
            csvFile = open(csvName, 'a', newline='', encoding='utf-8')
            csvWrite = csv.writer(csvFile)
            csvWrite.writerow([tw_date, tw_posneg, tw_content])
            csvFile.close()

            if (idx % 10000 == 0):  # Check running
                print("Now: "+ keyword+ " - "+str(idx))
        except:
            print("error")

```

# 키워드 추출

이왕 데이터를 수집한 겸, 찾고자 하는 키워드와 연관도가 높은 키워드를 보여주는 것도 좋을 것 같다고 생각해 키워드 추출 부분을 중간에 추가하였습니다.

## Noun - Corpus 생성

우선 수집한 트윗 데이터에서, 키워드가 될 수 있는 명사 데이터만 추출하였습니다. 아무래도 텍스트 데이터에는 키워드라고 부르기 어려운 단어들이 많아서, 이부분을 걸러내기 위함입니다. 형태소 분석기 중 kkma에서 텍스트 데이터에서 명사를 추출하는 기능이 있어서 이를 활용하였습니다.

```python
import pandas as pd
import numpy as np
import pickle
from konlpy.tag import Kkma 

pName = [#Keywords]

for name in pName:
  df = pd.read_csv('#Path'+name+'.csv')
  tw = df['Tweet']
  i = 0
  twList = []
  original = []
  for twData in tw:
    noun = okt.nouns(twData)
    original.append(twData)
    string = ""
    for idx, v in enumerate(noun):
      if len(v) < 2:
        noun.pop(idx)
      string = ' '.join(noun)
    twList.append(string)
    i += 1
    if (i % 1000 == 0):
      print("Running....")
      with open('#Path'+name+'.pkl', 'wb') as file:
        pickle.dump(twList, file)
  print("End: "+name)
  with open('#Path'+name+'.pkl', 'wb') as file:
    pickle.dump(twList, file)
```

## 키워드 추출

[KR-WordRank](https://github.com/lovit/KR-WordRank)는 비지도학습을 기반으로 한국어 키워드를 추출합니다. 이를 활용해 생성한 Noun - Corpus에서 상위 7개의 키워드를 추출했습니다.


```python
from krwordrank.word import KRWordRank
from krwordrank.sentence import summarize_with_sentences

stopwords = {} # Stopwords you want to except
penalty = lambda x:0 if (25 <= len(x) <= 80) else 1
keywords, sents = summarize_with_sentences(
    twList,
    penalty=penalty,   # penalty of word-length: remove too short and too long
    stopwords = stopwords,
    diversity=0.5,     # Diversity of keywords
    num_keywords=100,
    num_keysents=10,
    verbose=False
)
pName = [#Keywords]

for name in pName:
  df = pd.read_csv('#Path'+name+'.csv')
  tw = df['Tweet']
  twList = []
  nounStr = ""
  for twData in tw:
    noun = okt.nouns(twData)
    nounStr = ' '.join(noun)
    nounPos = kkma.pos(nounStr)
    string = ""
    onlyNNP = []
    for (text, pos) in nounPos:
      if pos == 'NNP':
        onlyNNP.append(text)
    string = ' '.join(onlyNNP)
    twList.append(string)
  with open('#Path'+name+'_NNP.pkl', 'wb') as file:
    pickle.dump(twList, file)
```

# 어플리케이션

처리한 데이터를 사용자에게 보여주기 위해 웹 페이지를 구현하기로 하였습니다.  
웹페이지를 구성한다면 Django, Flask등을 활용하여 웹 어플리케이션을 만들게 되는데, 저는  Streamlit이라는 파이썬 웹 어플리케이션 툴을 사용하기로 하였습니다.  

[Streamlit](https://streamlit.io/)은 기계학습과 데이터 과학을 위한 웹 앱을 편리하게 만들수 있도록 합니다.
Streamlit을 통해 로컬 환경에서 웹 어플리케이션을 만들어 시연해보았습니다.

 | 
:--------------------------------------------------------------------------------------------------------:|:--------------------------------------------------------------------------------------------------------:
![](https://user-images.githubusercontent.com/82520143/148383439-9ab30f79-7013-47e8-a6fe-81aefc63d7f1.png) | ![](https://user-images.githubusercontent.com/82520143/148383499-aadb125e-087d-4023-bba6-3108476201a5.png)

# 후기

작년 하반기에 만들었던 것을 문서화 하였습니다.
과제나 학습이 아닌 프로젝트를 처음으로 해보게 되었는데, 감회가 새로웠습니다.  
프로그램을 설계하고, 구현하는 과정에서 내가 모르는 것들을 찾아가는 것이 좋았습니다.  
다만 너무 외부 오픈소스에 의존해서 만들었다는 점이 아쉽게 느껴지고, 만들다보니 개선할 점도 많이 나온 것 같아 좀 아쉽다는 느낌이 듭니다.
