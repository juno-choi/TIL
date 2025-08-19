# 🔴 daily로 얻는 지식 정리

### 🟢 mysql에서 복합 unique key 설정

예를 들어 `nickname`, `name` column을 unique 복합 key로 지정했을 때, name값이 null이라면 name을 중복으로 인식하지 않는다.

```text
UNIQUE(nickname, name) 제약조건에서 name이 NULL이면,
NULL ≠ NULL 이므로 중복으로 간주되지 않는다.
```

이 문제를 회피하는 방법으론 우선 null이 포함되는 키를 unique key로 잡지 않는 것과 기본값을 null이 아닌 공백으로 처리하는 것이 필요하다.

또한 이러한 설정은 DB마다 다르기에 사용 중인 DB의 특성에 맞게 설정하는 것이 필요하다.


### 🟢 partial index

postgresql은 partial index(부분 인덱스)를 지원한다. index를 설정할때 where 조건으로 특정 데이터들에게만 index를 설정할 수 있게 해준다.

mysql에서는 지원하지 않는 설정이다.

### 🟢 cascade

JPA에서 사용되는 cascade 종류 중 `PERSIST` `MERGE`에 대해서만 정리해둔다.

`PERSIST` : 새로운 엔티티를 저장할 때, 연관된 엔티티도 같이 저장됨

`MERGE` : 준영속(detached) 상태의 엔티티를 다시 merge할 때, 연관된 엔티티도 같이 병합됨
