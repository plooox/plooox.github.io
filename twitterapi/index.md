# Colab 환경에서 트윗 데이터 수집하기

<!--more-->

# Colab환경에서 Twitter 크롤링 
## Twitter API 생성
1. 트위터 가입
2. 개발자 계정 등록하기
3. API생성
참고) [트위터 API 승인받기](https://specificthinking.tistory.com/entry/%ED%8A%B8%EC%9C%84%ED%84%B0-API-%EC%8A%B9%EC%9D%B8%EB%B0%9B%EA%B8%B0-%EA%B0%9C%EB%B0%9C%EC%9E%90-%EA%B3%84%EB%B0%9C%EC%9E%90-%EA%B3%84%EC%A0%95-%EB%B0%8F-Consumer-Key-Access-Token-%EB%B0%9C%EA%B8%89-%EB%93%B1)

## 트윗 검색 데이터 수집 + Pororo를 활용한 감정 분석
1. twitter_python
```python
twitter_consumer_key = ""
twitter_consumer_secret = ""  
twitter_access_token = ""
twitter_access_secret = ""

import twitter

twitter_api = twitter.Api(consumer_key=twitter_consumer_key,
                          consumer_secret=twitter_consumer_secret, 
                          access_token_key=twitter_access_token, 
                          access_token_secret=twitter_access_secret)
```
```python
query = " "
elements = twitter_api.GetSearch(term = query, count = 10000)

pos = 0
pos_list = []
neg = 0
neg_list = []

senti_Analysis = Pororo(task = "sentiment", model = "brainbert.base.ko.nsmc", lang = "ko")

for status in statuses:
    #print(senti_Analysis(status.text))
    if (senti_Analysis(status.text) == "Negative"):
        neg += 1
        neg_list.append(status.text)
    if (senti_Analysis(status.text) == "Positive"):
        pos += 1
        pos_list.append(status.text)
```

2. tweepy
```python
import tweepy
from tweepy import Stream
from tweepy import OAuthHandler
from tweepy.streaming import StreamListener

auth = tweepy.OAuthHandler(consumer_key = twitter_consumer_key, consumer_secret = twitter_consumer_secret)
api = tweepy.API(auth)
```
```python
from pororo import Pororo

query = "KeyWord"

pos = 0
pos_list = []
neg = 0
neg_list = []

sa = Pororo(task = "sentiment", model = "brainbert.base.ko.nsmc", lang = "ko")

res = []

for tw in tweepy.Cursor(api.search, q = query, since = "2021-07-14").items():
    tw_txt = tw.text
    if sa(tw_txt) == "Positive":
        pos += 1
        pos_list.append(tw_txt)
    if sa(tw_txt) == "Negative":
        neg += 1
        neg_list.append(tw_txt)
    res.append(tw_txt)

print(pos)
print(neg)
```
### Twitter API의 한계
- 1주전 트윗까지만 접근 가능
- 쿼리 속도 제한 : 5분당 180쿼리

## twint
```python
import twint

c = twint.Config()


c.Search = '페이커'
c.Limit = 5
c.Since = '2021-09-13'
c.Until = '2021-09-14'
c.Output = "test.json"
c.Popular_tweets = False
c.Store_json = True

twint.run.Search(c)
```
- 익명으로 사용 가능
- 거의 모든 트윗 수집 가능
- 속도 제한 없음
  
> 테스트 결과 성능이 매우 좋지 못하였음


참고) [파이썬과 트위터 API를 활용한 트위터 크롤링
](https://hleecaster.com/python-twitter-api/)
    [트위터 파헤치기 시리즈 첫번째 - 수집하기
](https://jeongwookie.github.io/2020/04/23/200423-analyze-tweet-series-1-collect/)

