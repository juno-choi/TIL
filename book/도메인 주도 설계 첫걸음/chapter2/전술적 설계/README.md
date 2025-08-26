# 🔴 전술적 설계

## 🟠 간단한 비즈니스 로직 구현

### 🟢 Transaction Script

하나의 요청을 처리하는 서비스 로직을 함수에 모아 구현하는 스타일이다.

```kotlin
@Service
class OrderService(
    private val orderRepository: OrderRepository,
    private val productRepository: ProductRepository
) {
    @Transactional
    fun placeOrder(cmd: PlaceOrderCommand): OrderId {
        val order = OrderEntity(customerId = cmd.customerId)
        var total = BigDecimal.ZERO

        cmd.items.forEach { item ->
            val product = productRepository.findById(item.productId) ?: error("No product")
            require(product.stock >= item.qty) { "재고 부족" }
            product.stock -= item.qty
            total += product.price * item.qty.toBigDecimal()
            order.items.add(OrderItemEntity(item.productId, item.qty, product.price))
        }

        order.total = total
        order.status = "PLACED"
        return orderRepository.save(order).id
    }
}
```

다음과 같은 소스로 가장 간단하면서 익숙한 로직이다.


| 장점                | 단점                                 |
|-------------------|------------------------------------|
| 단순한 로직으로 구현이 빠르고 이해가 쉬움 | 로직이 늘어날 수록 서비스가 비대해지고 스파게티 소스가 되어감 |
| 신규 인원 인보딩 리소스가 적게듦| 불변 조건 보장이 약해짐, 테스트 비용 증가           |

도메인 규칙이 단순하고 빠르게 제품을 출시해야 하는 MVP 모델에 자주 적용된다.

차후 복잡도 증가나 서비스가 커짐에 따라 DDD로 리팩토링할 수 있게 최소한(모듈의 경계, 테스트 코드) 갖춰두는게 좋다.

### 🟢 Active Record

도메인 객체 = 레코드 모델이자, 그 객체가 자신의 영속화(save, update, delete)를 책임지는 패턴


```kotlin
@Entity
class Order(
    @Id @GeneratedValue var id: Long? = null,
    var status: String = "NEW",
    @OneToMany(cascade = [CascadeType.ALL], mappedBy = "order")
    val items: MutableList<OrderItem> = mutableListOf()
) {
    fun addItem(product: Product, qty: Int) {
        require(qty > 0)
        items.add(OrderItem(this, product.id!!, qty, product.price))
    }

    fun place() {
        require(items.isNotEmpty()) { "아이템 없음" }
        status = "PLACED"
        // (AR 스타일) this.save() 같은 영속 호출을 객체가 직접 하려 함 → Spring/JPA에선 부자연스러움
    }
}

```

쉽게 말해 Entity 자체에 도메인 로직까지 모두 포함하는 설계 패턴

| 장점                     | 단점                                 |
|------------------------|------------------------------------|
| 간단한 앱에서는 개발 속도가 빠름     | 영속성 관심사와 도메인 규칙이 결합                |
| 객체.save() 와 같이 직관적인 코드 | 에그리게이트, 도메인 이벤트, 도메인 서비스로 확장하기 어려움 |

Active Record 패턴 적용시 이후에 DDD로 옮겨가는 부분에서 불리해지므로 트랜잭션 스크립트로 구현하는 것이 오히려 더 추천한다.

## 🟠 복잡한 비즈니스 로직 다루기

### 🟢 VO(Value Object) 도입

- 비즈니스 규칙을 단순 숫자/문자열로 처리하지 않고 의미 있는 타입으로 승격함
- VO로 도메인 규칙을 묶어서 다룸

```kotlin
class User(
   val email : String,
   val name : String,
   val nickname : String,
) {}
```
다음과 같이 단순한 문자열로 User의 정보를 가지고 갈 수 있지만

```kotlin
data class Email(
    val address : String,
) {
    // 이메일 패턴 체크 로직
}

class User(
   val email : Email,
   val name : String,
   val nickname : String,
)
```

Email로 승격하여 도메인 규칙, 로직을 정할 수 있다.

### 🟢 트랜잭션 경계(Transaction Boundary)

- 도메인 로직은 에그리게이트 단위에서 일관성을 유지해야 함
- 하나의 트랜잭션 경계에서 불변(invariant)을 보장하는 것이 핵심
- 서비스 레이어는 여러 에그리게이트 조합(=오케스트레이션)을 담당할 수 있지만, 에그리게이트 안의 로직은 에그리게이트 루트가 책임져야 함
    - 에그리게이트 루트의 책임을 모아둠으로서 연관된 에그리게이트끼리의 도메인 규칙/로직을 한눈에 파악하기 쉬워짐

`잘못된 예` 서비스가 규칙을 다 품고 있음 == 트랜잭션 스크립트
```kotlin
fun cancelOrder(orderId: Long) {
    val order = orderRepo.findById(orderId) ?: error("없음")
    if (order.status == "SHIPPED") error("이미 발송된 주문은 취소 불가")
    order.status = "CANCELLED"
    orderRepo.save(order)
}
```

`개선 예` 에그리게이트 루트에 도메인 규칙을 품음

```kotlin
class Order(...) {
    fun cancel() {
        require(status != OrderStatus.SHIPPED) { "이미 발송된 주문은 취소 불가" }
        status = OrderStatus.CANCELLED
    }
}

fun cancelOrder(orderId: Long) {
    val order = orderRepo.findById(orderId) ?: error("없음")
    order.cancel()
    orderRepo.save(order)
}
```

이렇게 되면 에그리게이트 루트에서 `발송 전 상품만 주문 취소 가능`이라는 규칙에 대한 불변성을 지킬 수 있다.

서비스는 얇아지고 도메인의 규칙은 에그리게이트 루트 안에서 일관되게 지켜질 수 있다.

### 🟢 도메인 서비스(Domain Service)

- 규칙이 여러 에그리게이트에 걸쳐 존재할 때 도입
- 엔티티/VO에 억지로 책임을 넣으려고 할때 이상하게 느껴지는 경우 별도의 도메인 서비스로 분리

```kotlin
class PaymentService {
    fun approve(order: Order, payment: Payment): Boolean {
        return payment.amount >= order.totalPrice()
    }
}
```

해당 로직은 Order에 넣기도 Payment에 넣기도 애매한 경우이다. 이럴 때, 도메인 서비스로 분리하여 처리할 수 있다.

단, 도메인 서비스는 남발하면 안되고 대부분의 도메인 규칙/로직은 도메인 내부에 두어야 한다.

## 🟠 시간 차원의 모델링

전통적인 CRUD 모델은 `현재 상태`만을 저장한다. 하지만 비즈니스에서는 과거에 어떤 일이 있었는가?가 중요한 경우가 많다.

그래서 `상태` 뿐만 아니라 `시간 흐름(=이벤트)`를 모델링 하는 방식이 필요하다

### 🟢 이벤트 소싱(Event sourcing)

- 현재 상태를 이벤트 흐름으로부터 도출하는 방식
- 엔티티 상태를 DB에 저장하지 않고, 도메인 이벤트(event stream)을 저장
- 에그리게이트를 복원할 때는 이벤트들을 순차 재생(replay) 해서 최종 상태로 만든다.
- 예를 들어 `Deposited(10000, time=09:00)` `Withdrew(5000, time=10:00)` 와 같은 이벤트 스트림을 저장했다면
    - 0원 → +10000 → -5000 → 최종 잔액 = 5000 결과를 도출해낼 수 있다.

### 🟢 이벤트 소싱 기반 도메인 모델
- 에그리게이트 루트는 이벤트를 생성하고 적용하는 역할을 한다
- 상태 변경은 이벤트에 의해만 발생
- 저장소는 save만 하고, DB에는 이벤트 로그를 남김

```kotlin
// 도메인 이벤트
sealed interface AccountEvent
data class Deposited(val amount: Int) : AccountEvent
data class Withdrew(val amount: Int) : AccountEvent

// 애그리게이트 루트
class Account(
    val id: String,
    private val events: MutableList<AccountEvent> = mutableListOf()
) {
    private var balance: Int = 0

    fun deposit(amount: Int) {
        apply(Deposited(amount))
    }

    fun withdraw(amount: Int) {
        require(balance >= amount) { "잔액 부족" }
        apply(Withdrew(amount))
    }

    private fun apply(event: AccountEvent) {
        when (event) {
            is Deposited -> balance += event.amount
            is Withdrew -> balance -= event.amount
        }
        events.add(event)
    }

    fun getUncommittedEvents(): List<AccountEvent> = events.toList()
    fun currentBalance() = balance
}
```

- Account는 이벤트를 apply함으로써 상태 변경
- DB에는 Deposited, Withdrew 이벤트가 기록됨
- 다시 불러올 때는 이벤트를 차례대로 apply 해서 balance를 복원

| 장점                                | 단점                                       |
|-----------------------------------|------------------------------------------|
| 완벽한 이력 보존                         | 단순 CRUD보다 구현이 어려움                        |
| 과거 상태/행위를 언제든 복원 가능               | 이벤트 스키마 변경 시 이벤트 마이그레이션 문제 발생            |
| 감사 추적(Audit), 규제 대응, AI 학습 데이터 유리 | 이벤트 로그가 계속 쌓이므로 스냅샷(Snapshot) 전략이 필요     |
| CQRS와 잘 어울림(조회 전용 DB 별도로 구축)      | 모든 도메인에는 적합하진 않음 - 규제/로그 중요성이 높은 도메인에 적절 |

### 🟢 DDD에서 바라보는 이벤트 소싱
- 이벤트 소싱은 단순한 기술이 아니라, 시간 차원의 도메인 모델링 패턴
- 모든 로그를 항상 계산할 순 없기 때문에 스냅샷 전략은 필수
- 핵심 도메인 중 시간 흐름/이력 중요성이 큰 곳에 잘 맞음
- 금융, 물류, 헬스케어, AI 등

## 🟠 아키텍처 패턴

코드베이스를 조직하는 적절한 방법 혹은 올바른 아키텍처 패턴을 선택하는 것은 단기적으로는 비즈니스 로직 구현을 지원하고, 장기적으로는 유지보수를 위해 매우 중요하다.

세가지 주요 아키텍처 패턴인 계층형, 포트와 어댑터, CQRS에 대해 알아보자.

### 🟢 계층형 아키텍처(Layered Architecture)

계층형 아키텍처는 가장 일반적이고 전통적인 아키텍처 패턴이다.

```text
프레젠테이션 계층(웹 UI, CLI, REST API) 
⬇️ 
비즈니스 로직 계층(엔티티, 규칙, 프로세스) 
⬇️ 
데이터 접근 계층(데이터베이스, 메세지 버스, 오브젝트 스토리지)
```

`프레젠테이션 계층`
- 사용자와 상호작용
- 웹 UI, CLI, REST API 포함되어 있는 계층
- 외부 환경으로부터 요청을 받고 결과를 반환하는 소통 수단

`비즈니스 로직 계층`
- 비즈니스 로직 구현
- 에릭 에반스는 이 계층이 소프트웨어의 중심이라 말함

`데이터 접근 계층`
- 데이터 저장
- 현재는 NoSQL 등장 이후로 여러 데이터베이스를 사용하는 시스템이 보편화됨.
  - NoSQL은 실시간 데이터 처리
  - 검색 인덱스는 동적 질의에 사용
  - S3 Object Storage에는 파일을 저장
- 프로그램의 기능 구현하는데 필요한 다양한 외부 정보 제공자와 연동하는 것도 포함된다.
  - 외부 데이터인 날씨 정보, 주식 데이터 등 외부 API 또는 관리형 서비스와 연동

### 🟢 계층 간 커뮤니케이션

계층은 탑다운(top-down) 커뮤니케이션 모델에 따라 연동된다. 각 계층은 바로 아래 계층에만 의존한다. 이렇게 하면 구현 관심사의 결합성을 낮추고 계층 간 공유할 지식이 줄어든다.

프레젠테이션 계층은 서비스 계층만 참조하고 데이터 접근 계층의 설계는 알 필요가 없다.

### 🟢 변종(variation)

계층형 아키텍처 패턴을 확장해서 서비스 계층을 추가하는 것이다.

`서비스 계층`
- 서비스 계층은 프로그램의 프레젠테이션 계층과 비즈니스 로직 계층 사이의 중간 역할을 한다.
- 프레젠테이션 계층과 비즈니스 로직 계층의 결합도를 낮춘다.
- 비즈니스 기능을 테스트하기 쉬워진다.
- 동일한 서비스 계층을 여러 프레젠테이션 계층에서 재사용할 수 있다.
- 모든 관련된 메서드를 한곳에 모아 모듈화를 할 수 있다.

```text
프레젠테이션 계층(웹 UI, CLI, REST API)
⬇️ 
서비스 계층(액션, 액션, 액션) 
⬇️ 
비즈니스 로직 계층(엔티티, 규칙, 프로세스) 
⬇️ 
데이터 접근 계층(데이터베이스, 메세지 버스, 오브젝트 스토리지)
```
다만, 서비스 계층이 항상 필요한 것은 아니다. 예를 들어, 비즈니스 로직이 트랜잭션 스크립트로 구현되어 있는 경우 서비스 계층을 나누어도 서비스에서 트랜잭션 스크립트를 호출하는 역할로서만 작동되기 때문이다.

### 🟢 용어

계층형 아키텍처에 사용되는 용어를 정리해보자.

`프레젠테이션 계층` = 사용자 인터페이스 계층

`서비스 계층` = 애플리케이션 계층

`비즈니스 로직 계층` = 도메인 계층 = 모델 계층

`데이터 접근 계층` = 인프라스트럭처 계층

### 🟢 계층형 아키텍처를 사용하는 경우

비즈니스 로직이 트랜잭션 스크립트 또는 레코드 패턴(entity 자체에 서비스 로직이 있는 경우)이 적용된 시스템은 계층형 아키텍처 패턴이 적합하다.

반면, 도메인 모델을 구현하는데 계층형 아키텍처 패턴을 적용하는 것은 어렵다. 도메인 모델에서는 비즈니스 엔티티(애그리게이트와 VO)가 하부의 인프라스트럭처에 대해 의존성이 없어야 하기 때문이다.

계층형 아키텍처의 톱다운 의존성은 이런 부분에 있어 어려움을 겪는다.

### 🟢 포트와 어댑터

포트와 어댑터 아키텍처는 계층형 아키텍처의 단점을 해결하고 좀 더 복잡한 비즈니스 로직을 구현하는데 적합하다.

```text
비즈니스 로직 계층(엔티티, 규칙, 프로세스)
⬆️
애플리케이션 계층(액션, 액션, 액션)
⬆️
인프라스트럭처 계층(데이터베이스, UI 프레임워크, 외부 API, 메세지 버스)
```

도메인이 인프라에 의존하면 안되는 부분을 DIP 원칙을 적용하여 인프라가 도메인에 의존하도록 만들었다.

### 🟢 인프라 구성요소의 연동

