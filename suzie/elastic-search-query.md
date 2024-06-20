## [Elastic Search] Query 검색 

### 넣고 싶은 내용
- match query 에서 자르는 방법
```
사례 /path1/path2/path3
- match query(and) option 으로 검색할 때 /path/path3 도 검색됨
ㄴ> match_phrase_prefix 사용(순서보장)
```


### 1. Match
- analyzing
- operator(default: or)

- match
- match_phrase
- match_phrase_prefix

### 2. Term
- Normalizer

- term *
- terms *
- range
- exists
- prefix
- wildcard * -> 느림, 느린 이유까지
- regexp
- fuzzy
- type
- ids

#### Match vs Term
```
Analyzing?

```

### 3. Multi Match
ㄴ fields
- type
```
case 공유
- ES 버전과 Client 버전이 달라서 match_phrase_prefix가 안되는 경우 -> multi_match 사용
ㄴ multi_match에서 type으로 match_phrase_prefix를 사용 가능
```