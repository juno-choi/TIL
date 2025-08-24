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

### 🟢 캐시 스탬피드(Thundering Herd)

- 설명
    - Cache Aside(캐시 미스 발생 시 적재) 전략 사용시 발생할 수 있는 문제
    - 캐시가 비었을 때 동시 요청이 온다면 여러 쓰레드에서 DB read 후 Redis에 write 행위 발생
    - 성능적으로 문제 발생
- 해결
    - Lock을 통한 해결
        - 확실하게 여러 쓰레드가 동작하는 경우는 없어짐
        - 락을 대기하기에 성능적 이슈는 여전
    - 외부 재계산(External Recomputation) 방식
        - 배치나 스케줄러 같은 외부 시스템을 사용하여 TTL 전에 재계산하여 Redis에 write
        - 요청이 없는 경우에도 계속해서 계산을 하기에 메모리 낭비가 있음
    - 확률적 조기 재계산(Probablistic Early Recomputation) 방식
        - TTL 전에 각 쓰레드 별로 랜덤한 확률로서 재계산을 진행함
        - 확률이기 때문에 횟수가 줄어들 수 있는거지 완벽한 해결책은 아님

        ```kotlin
        fun shouldRefreshCache(createdAt: Long, ttl: Long): Boolean {
            val now = System.currentTimeMillis()
            val delta = now - createdAt
            val probability = exp(-1.0 * delta / ttl)
            return Random.nextDouble() < probability
        }
        ```