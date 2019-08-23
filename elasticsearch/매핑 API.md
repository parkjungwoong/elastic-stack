# 매핑 API

## 목차
- [매핑 API](#매핑-API)
- [고려사항](#고려사항)
- [매핑 파라미터](#매핑-파라미터)
- [매핑 탬플릿](#매핑-탬플릿)

## 매핑 API
색인될 문서의 데이터 모델링, RDB의 스키마와 비슷한 개념\
색인시 데이터가 저장될 형태를 결정하는 설정이다.\
document 안에 Field가 있고 Field는 데이터 타입과 메타데이터로 구성되는데 이 두개(데이터 타입, 메타데이터)가 색인과정에서 어떻게 역색인 될것인가를 결정

## 고려사항
- 문자열을 분석이 필요한가
- _source에 어떤 필드를 갖을 것인가
- 시간 데이터 필드는 어떤것으로 할 것인가
- 미지정 필드에 대한 요청 처리는 어떻게 할 것인가

각 항목에 대한 내용을 매핑 파라미터값을 조정하여 해결한다.

## 매핑 파라미터
전체 내용은 [공식문서](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html) 참고

- analyzer(문자열을 분석이 필요한가)
    - **analyzer** 옵션은 해당 필드의 데이터를 형태소 분석, text 데이터 타입의 필드는 analyzer 매핑 파라미터를 기본적으로 사용해야됨, 별도의 분석기를 지정하지 않으면 Standard Analyzer로 형태소 분석 수행
    - elasticsearch 6.4버전 이후 부터 [nori 한글 형태소](https://www.elastic.co/guide/en/elasticsearch/plugins/7.2/analysis-nori.html) 분석기가 공식 지원되나 기본으로 설치되어 있는 상태가 아니기 때문에 플러그인 형태로 설치가 필요하다.

- normalizer(문자열을 분석이 필요한가)
    - ID, 이름과 같이 정확한 값을 조건을 기반으로 검색할 필드에 지정
    - ex)Cafe, cafe => 같은 단어로 인식함
    - text 필드에서는 사용 권장 안함, 관련 내용은 [여기](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-term-query.html) 참고
    - keyword 타입에서 주로 사용
    
- coerce(미지정 필드에 대한 요청 처리는 어떻게 할 것인가)
    - 색인 시 자동 형변환에 대한 처리 유무
    
- copy_to
    - 필드의 값을 지정한 다른 필드에 복사한다.
    - Ex) 주소 도로명, 주소상세 필드 두개의 keyword 타입의 필드가 있다면 주소라는 text 타입의 필드로 앞의 두개의 필드 값을 복사한다.\
    주소 도로명:대왕판교로, 주소상세:유스페이스 => 주소:대왕판교로 유스페이스

- fielddata
    - text타입의 필드에서 집계나 정렬을 수행하는 경우에 지정
    - heep공간에 생성하는 메모리 캐시로 최소한으로만 사용해야됨(기본은 비활성화)
    
- doc_values
    - elasticearch에서 사용하는 기본 캐시(heep이 아닌 OS의 파일시스템 캐시를 사용)
    - text 필드 제외한 모든 타입에서 기본적으로 사용
    - 집계, 정렬이 필요없다면 비활성화 하여 디스크 공간 절약할 필요가 있음
    
- dynamic (미지정 필드에 대한 요청 처리는 어떻게 할 것인가)
    - 매핑에 필드를 추가할 때 동적으로 생성을 허용하는지에 대한 옵션(3가지 있음)
    - true : 새로 추가되는 필드를 매핑에 추가한다.
    - false : 매핑은 하지 않지만 _source에는 표기된다. (해당 필드는 색인되지 않아 검색 안됨)
    - strict : 예외 발생, 문서 자체가 색인되지 않음
    
- enabled
    - 색인대상에서 제외하고 싶은 경우 false로 설정한다.
    - _source에는 포함되지만 검색은 되지 않는다.

## 매핑 탬플릿
생성되는 인덱스가 인덱스명-YYYY-MM-DD와 같이 생성될 경우 매핑 템플릿을 사용하여 인덱스 매핑을 지정해줄 수 있다.

```
PUT _template/템블릿명
{
  #인덱스 패턴을 지정해줌
  "index_patterns": ["te*", "bar*"], 
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    ...
  }
}
```
- 지정된 필드외 다른 필드가 들어올 경우 무시하기 위한 옵션
    - dynamic : true(허용), false(무시), static(에러, 인덱스 생성 안됨) 
```
"mappings": {
    dynamic: false,
    ...
}
```