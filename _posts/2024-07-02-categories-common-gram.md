---
title: "ES Common Gram"
excerpt: "언어 구별로 색인"

categories:
  - "Trouble Shooting"
tags:
  - [Elasticsearch]

permalink: /categories/troubleshooting/common-gram

toc: true
toc_sticky: true

date: 2024-07-02
last_modified_at: 2024-07-02
---


# Problem
## 분석기 이슈
- 특수문자를 분리 하려다가 실수형에 '.'이 붙어있으면 분리되는 이슈가 발생했다
```
ex) 3.5kg => 3, ., 5, kg
```
- 숫자 '3', '5'가 각각 토큰으로 분리되어 색인되는게 문제!
- 만일 '5kg'이라고 검색 시 리턴이 돼버린다

```
ex) 검색어 '현미쌀 5kg'
문서1: 맛좋고 싱싱한 국내산 현미쌀 5kg
문서2: 값싸고 가성비 좋은 현미쌀 2.5kg

'5kg'에 문서 1만 match가 되어야하는데, 문서 1,2 모두 match!!
```
# How
## "숫자 + 단위" Pattern 처리
- 숫자, 단위가 붙어있을 때 오분석이 되는게 문제였으니 이 pattern이 들어왔을 때 처리할 방법이 있을까 했다
- es에서는 settings에서 regex를 추가하면 간단하게 분석이 가능하다!

```
"단위 Tokenizer": {
    "type": "pattern",
    "pattern": "(\\d+\\.\\d+|\\d+|단위)",
    "group": 0
}

[분석 결과]
3.5kg => 3.5, kg
20.5cm => 20.5, cm 
```
## 정해진 pattern에 매칭되는 것은 Boosting!
- 우선 해당 regex pattern에 걸리면 단위 분석기로 처리한 필드로 query를 날리려고 했다
```
boolean measureMatches(String query) {
    Pattern pattern = Pattern.compile(".*\\d+(\\.\\d+)?\\s*(단위).*");
    Matcher matcher = pattern.matcher(Optional.ofNullable(query).orElse(""));
    return matcher.matches();
}
```
- 원래는 해당 field에 should 조건을 추가하여 'or' query를 날리려고 했는데 성능이 느렸다ㅜ
- 대신에 elasticsearch에는 filter를 걸어서 해당 필드에 match가 된다면 boost 점수를 올릴 수 있다
```
"query": {
    "function_score": {
      "query": {
        "bool": {
          ...
        }
      },
      "functions": [
        ...
        {
          "filter": {
            "match_phrase": {
              "단위필드": {
                "query": "숫자+단위 Query",
                "slop": 0,
                "zero_terms_query": "NONE",
                "boost": 1
              }
            }
          },
          "weight": 가중치
        }
      ],
      ...
    }
  },
  ...
```
- 'match_phrase'를 사용한 이유는 일반적인 'match'를 사용하면 순서가 바뀌어도 나오게 된다!
## common_grams filter
- es에는 pattern이 일치하는 애들은 묶어서 구로 처리가 가능한 filter가 있다
```
ex) 3.5kg => 3.5_kg
```
- 이렇게 되면 성능, 크기에 있어서 더 유리하다고 한다   
[Ref](https://spinscale.de/posts/2021-04-14-using-the-common-grams-filter-for-faster-queries.html)
- 요런식으로 쓰면 된다
```
"단위_common_grams": {
    "type": "common_grams",
    "common_words": [적어야 할 단위들],
    "query_mode": true
}
```
# Review
- 원하던 단위에 해당하는 문서들만 올라와서 이슈가 해결이 됐다
- 어떠한 패턴이 있을 때 es settings에 type: 'custom'으로 해서 분석기 추가하고 색인시켜 처리한 것이 핵심이었다
- 특히 분석기 필터 하나 개발하려면, 여간 쉬운게 아니니 이런 방식도 괜찮은 것 같다

