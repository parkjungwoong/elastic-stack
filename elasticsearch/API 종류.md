# API 종류
인덱스(색인된 데이터를 담을 그릇) 생성를 생성하고 document(색인 데이터)를 생성한다.\
이 후 색인된 데이터를 검색한다. 이러한 행위를 도와주는 REST API 제공한다. 

## 목차
- [인덱스 관리](#인덱스-관리)
- [문서 관리](#문서-관리)
- [검색](#검색)
- [집계](#집계)

## 인덱스 관리
- 인덱스 생성\
매핑 설정을 통해 document에 포함된 field와 field의 Data Type을 지정한다. 한번 설정된 매핑정보는 수정이 불가하다.(인덱스를 삭제 후 다시 생성해야됨)
``` http request
PUT /인덱스명
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  },
  
  "mappings": {
    "properties": {
      "name": {
        "type": "keyword" //단순 문자열의 경우
      },
      "age": {
        "type": "integer"
      },
      "addr": {
        "type": "text" //형태소 분석이 필요한 경우
      }
    }
  }
}
```
- 인덱스 삭제
```
DELETE 인덱스명
```

## 문서 관리
- Single document API
    - index API : 한 건의 문서를 색인한다.
    ``` http request
    POST /인덱스명/_doc/1
    {
        "name":"박정웅",
        "age":"29",
        "addr":"서울시 서대문구"
    }
    ```
    - Get API : 한 건의 문서를 조회한다.
    ``` http request
    GET /인덱스명/_doc/1
    ```
    - Delete API : 한 건의 문서를 삭제한다.
    ``` http request
    DELETE /인덱스명/_doc/1
    ```
    - Update API : 한 건의 문서를 수정한다.
    ```
    #index API 같음
    ``` 
- Multi document API
    - Multi Get API : 다수의 문서를 조회한다.
    - Bulk API : 대량의 문서를 색인하다.
    - Delete By Query API : 다수의 문서를 삭제한다.
    - Update By Query API : 다수의 문서를 수정한다.
    - Reindex API : 인덱스의 문서를 다시 색인한다.

## 검색
- URL 지정
``` http request
GET /인덱스명/_doc/_search?q=박정웅
```
- QueryDSL
``` http request
POST /인덱스명/_doc/_search
{
  "query":{
    "term": {
        "name":"박정웅"
      }
  }
}
```
두가지 방식을 합하여 사용 가능

## 집계
- 버킷 집계 : 문서의 필들르 기준으로 버킷을 집계한다.
- 메트릭 집계 : 문서에서 추출된 값을 가지고 합, 최대, 최소, 평균을 계산한다.
- 매트릭스 집계 : 행렬의 값을 합하거나 곱한다.
- 파이프라인 집계 : 버킷에서 도출된 결과 문서를 다른 필드 값으로 재분류한다. 즉 집계된 값을 기준으로 다시 집계한다.

``` http request
POST /인덱스명/_search?size=0
{
  "aggs":{
    "names":{
      "terms": {
        "field":"name"
      }
    }
  }
}
#이름으로 그룹 카운드
```
