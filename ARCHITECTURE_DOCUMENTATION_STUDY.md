# 소프트웨어 아키텍처 문서화: 이유에서 검증까지

이 자료는 Paul Clements 외, *Documenting Software Architectures: Views and Beyond*(2016)의 핵심 개념을 실제 문서화 과정에서 자연스럽게 생기는 질문을 따라가면서 설명한다.

> **전체 흐름: 중요한 결정을 공유해야 한다 → 한 그림으로는 부족하다 → 이해관계자에 맞는 뷰를 고른다 → 구조와 동작을 명확하게 기록한다 → 뷰를 연결한다 → 독자가 사용할 수 있는지 검증한다.**

![아키텍처 문서화 전체 흐름 예시](diagrams/15-documentation-flow.png)

## 1. 왜 아키텍처를 문서화하는가

> **품질에 관한 결정은 시스템의 큰 구조에 영향을 준다.**

소프트웨어를 만들 때는 기능뿐 아니라 변경 용이성, 성능, 보안, 가용성 같은 품질도 결정해야 한다. 이런 품질은 특정 함수 하나만으로 달성할 수 없기 때문이다. 예를 들어 주문 처리와 알림을 동기 호출로 묶으면 구현은 단순해지지만, 알림 장애가 주문 실패로 전파될 수 있다.

> **아키텍처 결정은 여러 사람이 함께 참조할 수 있도록 문서화해야 한다.**

이처럼 이후 구현과 운영을 크게 제약하는 결정을 **아키텍처 결정**이라고 볼 수 있다. 이러한 결정은 개발·테스트·배포·운영 등 여러 사람의 업무에 영향을 주므로 머릿속이나 회의 기록에만 남겨서는 안 된다.

![품질 목표에서 아키텍처 문서까지의 흐름](diagrams/16-quality-to-documentation.png)

> **아키텍처 문서의 목적은 이해관계자가 업무에 필요한 결정을 내릴 수 있게 하는 것이다.**

그림을 많이 만드는 것 자체가 목적은 아니다. 이해관계자마다 문서를 통해 답하려는 질문은 다음과 같이 다르다.

| 이해관계자 | 결정에 필요한 질문 |
|---|---|
| 개발자 | 결제 기능을 바꾸면 어떤 코드가 영향을 받는가? |
| 테스터 | 어떤 인터페이스와 실패 경로를 검증해야 하는가? |
| 운영자 | 서버 하나가 중단되면 어떤 서비스가 영향을 받는가? |
| 보안 담당자 | 외부 입력은 어떤 신뢰 경계를 통과하는가? |
| 관리자 | 어느 팀이 어떤 부분을 책임지는가? |

## 2. 왜 하나의 그림으로는 부족한가

> **이해관계자의 질문이 다르면 필요한 구조도 다르다.**

코드 변경 질문에는 패키지와 의존성이 필요하지만, 장애 전파 질문에는 실행 중인 서비스와 통신 경로가 필요하다. 배포 위치 질문에는 서버와 네트워크가 필요하다.

```plantuml
@startuml
title 질문이 다르면 필요한 뷰도 다르다
left to right direction

actor 개발자
actor 운영자
actor 관리자

rectangle "코드 변경 영향?" as Q1
rectangle "장애 전파 경로?" as Q2
rectangle "팀별 책임?" as Q3

rectangle "모듈 뷰" as V1 #DDEBFF
rectangle "C&C 뷰" as V2 #E2F4DF
rectangle "할당 뷰" as V3 #FFF0D6

개발자 --> Q1
Q1 --> V1
운영자 --> Q2
Q2 --> V2
관리자 --> Q3
Q3 --> V3
@enduml
```

> **서로 다른 관심사를 한 그림에 모두 넣지 말고 뷰(view)로 분리해야 한다.**

시스템을 하나의 거대한 그림에 모두 넣으면 서로 다른 의미의 상자와 선이 섞인다. 그림이 복잡해질 뿐 아니라, 같은 화살표가 코드 의존성인지 런타임 호출인지 알 수 없게 된다.

```plantuml
@startuml
title 아키텍처를 바라보는 세 가지 핵심 뷰타입
top to bottom direction
skinparam componentStyle rectangle

rectangle "소프트웨어 아키텍처" as Architecture
rectangle "모듈 뷰타입\n구현 단위와 정적 관계" as Module #DDEBFF
rectangle "컴포넌트-커넥터 뷰타입\n런타임 요소와 상호작용" as CC #E2F4DF
rectangle "할당 뷰타입\n소프트웨어와 환경의 대응" as Allocation #FFF0D6

Architecture --> Module : 코드는 어떻게 조직되는가?
Architecture --> CC : 실행 중 어떻게 협력하는가?
Architecture --> Allocation : 어디에 배치되고 누가 맡는가?

legend bottom
  화살표는 "해당 관점으로 기술한다"는 분류 관계다.
  시스템 내부의 호출이나 데이터 흐름을 뜻하지 않는다.
endlegend
@enduml
```

### 뷰, 뷰타입, 스타일

> **뷰는 실제 표현이고, 뷰타입과 스타일은 그 표현을 구성하는 규칙이다.**

- **뷰**: 특정 이해관계자의 관심사를 위해 실제 시스템 일부를 표현한 결과물
- **뷰타입**: 표현할 요소와 관계의 종류를 정한 범주
- **스타일**: 한 뷰타입 안에서 사용할 요소·관계·제약을 더 구체화한 규칙

### PDF에 제시된 뷰타입별 스타일

| 뷰타입 | 스타일 또는 스타일 계열 | 핵심 요소와 관계 | 적용 예시 |
|---|---|---|---|
| 모듈(Module) | 분해(Decomposition) | 모듈과 `is-part-of` 관계 | `Order` 모듈을 application, domain, adapter 하위 모듈로 분해한다. |
| 모듈(Module) | 사용(Uses) | 모듈과 `uses` 관계 | `Order Application`이 `Payment Port`의 올바른 구현에 의존한다. |
| 모듈(Module) | 일반화(Generalization) | 모듈과 `is-a` 관계 | 여러 결제 어댑터가 공통 `Payment Adapter` 타입을 특수화한다. |
| 모듈(Module) | 계층(Layered) | 계층과 `allowed-to-use` 관계 | Presentation 계층은 Application 계층을 거쳐 Domain 기능을 사용한다. |
| 컴포넌트-커넥터(C&C) | 데이터스트림(Datastream) | 필터, 파이프, 비동기 데이터 스트림 | 주문 데이터가 검증·변환·저장 필터를 순서대로 통과한다. 대표 세부 스타일은 pipe-filter다. |
| 컴포넌트-커넥터(C&C) | 호출-반환(Call-Return) | 서비스 제공자·요청자와 request-reply 관계 | Web App이 Order Service를 동기 호출한다. 대표 세부 스타일은 client-server와 peer-to-peer다. |
| 컴포넌트-커넥터(C&C) | 공유 데이터(Shared-Data) | 저장소, 데이터 접근자, 읽기·쓰기 관계 | 여러 서비스가 공용 저장소를 통해 데이터를 공유한다. 대표 세부 스타일은 repository와 blackboard다. |
| 컴포넌트-커넥터(C&C) | 발행-구독(Publish-Subscribe) | 발행자·구독자와 이벤트 버스 | Order Service가 `OrderPlaced`를 발행하면 여러 워커가 구독한다. 세부 스타일로 implicit invocation과 event-only가 있다. |
| 컴포넌트-커넥터(C&C) | 통신 프로세스(Communicating Processes) | 프로세스·스레드와 메시지·동기화 관계 | 주문 프로세스와 재고 프로세스가 메시지를 교환하며 동시에 실행된다. |
| 할당(Allocation) | 배포(Deployment) | 소프트웨어 요소와 실행 플랫폼의 `allocated-to` 관계 | Order Service 프로세스를 컨테이너 플랫폼 노드에 배포한다. |
| 할당(Allocation) | 구현(Implementation) | 모듈과 파일·디렉터리·저장소의 `allocated-to` 관계 | `order.application` 모듈을 `src/order/application` 디렉터리에 둔다. |
| 할당(Allocation) | 작업 배정(Work Assignment) | 소프트웨어 요소와 개발 조직의 `allocated-to` 관계 | 주문 모듈은 Order Team이, 결제 어댑터는 Payment Team이 담당한다. |

*출처: Clements et al.,* Documenting Software Architectures: Views and Beyond, *Chapter 2, pp. 59–98; Chapter 4, pp. 117–125; Chapter 6, pp. 156–168.*

## 3. 질문에 따라 세 가지 뷰타입을 선택한다

> **세 뷰타입은 서로 다른 질문에 답하므로 필요한 경우 함께 사용할 수 있다.**

세 뷰타입은 하나만 선택해야 하는 경쟁 관계가 아니다.

### 3.1 모듈 뷰타입: 코드는 어떻게 나뉘는가

> **모듈 뷰는 구현 단위와 정적 관계를 보여준다.**

![PDF Figure 9의 모듈 uses 관계와 분해 예시](diagrams/14-module-uses-from-book.png)

*출처: Clements et al.,* Documenting Software Architectures: Views and Beyond, *p. 70, Figure 9.*

- 요소: 패키지, 클래스, 계층, 서브시스템
- 관계: 포함(`is-part-of`), 사용(`uses`), 일반화(`is-a`)
- 답하는 질문: 책임 분리는 적절한가? 변경 영향은 어디까지인가? 어떤 코드를 재사용하는가?
- 답하기 어려운 질문: 실행 순서는 무엇인가? 어느 서버에서 실행되는가?

`Order Application ..> Payment Port : uses`는 주문 코드가 올바르게 구현되려면 결제 포트의 정의가 필요하다는 뜻이다. 실제 실행 시 네트워크로 결제사를 호출한다는 의미는 아니다.

### 3.2 컴포넌트-커넥터 뷰타입: 실행 중 어떻게 협력하는가

> **컴포넌트-커넥터(C&C) 뷰는 런타임 요소와 상호작용을 보여준다.**

```plantuml
@startuml
title 온라인 서점 - 런타임 컴포넌트-커넥터 뷰
left to right direction
skinparam componentStyle rectangle

actor Customer
component "Web App" as web
component "Catalog Service" as catalog
component "Order Service" as order
component "Payment Gateway" as payment
queue "Event Broker" as broker
component "Inventory Worker" as inventory
component "Notification Worker" as notification
database "Order DB" as orderDb

Customer --> web : HTTPS
web --> catalog : HTTPS/JSON\n동기 질의
web --> order : HTTPS/JSON\n동기 명령
order --> payment : HTTPS/JSON\n동기 승인
order --> orderDb : SQL transaction
order -[#blue,dashed]-> broker : OrderPlaced\n비동기 발행
broker -[#blue,dashed]-> inventory : OrderPlaced\n비동기 구독
broker -[#blue,dashed]-> notification : OrderPlaced\n비동기 구독

legend bottom
  실선 화살표: 요청자가 시작하는 동기 상호작용
  파란 점선 화살표: 비동기 이벤트 이동
  SQL 선은 Order Service만 Order DB를 직접 접근함을 뜻한다.
endlegend
@enduml
```

- 요소: 서비스, 프로세스, 클라이언트, 데이터 저장소
- 관계: 호출, 메시지, 이벤트, 스트림, 공유 데이터 접근
- 답하는 질문: 요청과 데이터는 어디로 흐르는가? 병목과 장애는 어디로 전파되는가?
- 답하기 어려운 질문: 소스 파일은 어디에 있는가? 어느 팀이 구현하는가?

주문 서비스가 결제 게이트웨이를 호출하고 이벤트 브로커에 주문 이벤트를 발행하는 구조는 C&C 뷰로 표현한다.

### 3.3 할당 뷰타입: 소프트웨어는 환경과 어떻게 대응되는가

> **할당 뷰는 소프트웨어 요소를 소프트웨어가 아닌 환경에 대응시킨다.**

```plantuml
@startuml
title 온라인 서점 - 배포 할당 뷰
skinparam componentStyle rectangle

cloud "Internet" as internet

node "Public subnet" as public {
  node "Web node\nreplicas: 2..6" as webNode {
    artifact "web-app" as webArtifact
  }
}

node "Application subnet" as app {
  node "Container platform" as platform {
    artifact "catalog-service\nreplicas: 2..10" as catalogArtifact
    artifact "order-service\nreplicas: 2..10" as orderArtifact
    artifact "notification-worker\nreplicas: 1..5" as notificationArtifact
  }
  queue "managed event broker" as broker
}

node "Data subnet" as data {
  database "order-db\nprimary + standby" as db
}

internet --> webNode : TLS 443
webNode --> platform : TLS 443
orderArtifact --> broker : TLS 9093
notificationArtifact --> broker : TLS 9093
orderArtifact --> db : TLS 5432

legend bottom
  중첩은 아티팩트가 해당 실행 환경에 배포됨을 뜻한다.
  화살표는 허용된 네트워크 통신과 포트를 뜻한다.
  subnet 경계는 네트워크 신뢰 경계다.
endlegend
@enduml
```

- 배포 스타일: 서비스와 서버·컨테이너의 대응
- 구현 스타일: 모듈과 저장소·디렉터리의 대응
- 작업 배정 스타일: 모듈과 개발팀의 대응
- 답하는 질문: 어디에서 실행되는가? 코드는 어디에 있는가? 누가 책임지는가?

### 같은 시스템을 세 관점으로 보기

> **서로 다른 뷰의 요소는 1:1로 대응할 필요가 없다.**

하나의 모듈로 여러 서비스 인스턴스를 만들 수 있고, 여러 모듈을 하나의 서비스로 패키징할 수도 있다.

```plantuml
@startuml
title 같은 온라인 서점을 세 관점으로 보기
left to right direction

package "모듈 뷰" #DDEBFF {
  [Order Module] as M1
  [Payment Adapter] as M2
  M1 ..> M2 : uses
}

package "C&C 뷰" #E2F4DF {
  [Order Service] as C1
  [Payment Gateway] as C2
  C1 --> C2 : HTTPS 호출
}

package "배포 뷰" #FFF0D6 {
  node "Container Node" as N1 {
    artifact "order-service" as A1
  }
}

M1 ..> C1 : 구현됨
C1 ..> A1 : 패키징됨
@enduml
```

## 4. 필요한 뷰는 이해관계자의 질문에서 결정된다

> **필요한 뷰는 이해관계자의 질문을 기준으로 선택해야 한다.**

“모든 뷰를 만들면 안전하다”는 생각은 문서 비용과 중복을 늘린다. 반대로 팀이 익숙한 다이어그램만 만들면 중요한 질문이 빠진다. 따라서 뷰 선택은 다음 인과관계를 따른다.

```text
이해관계자의 업무
    ↓
업무 중 내려야 하는 결정
    ↓
결정에 필요한 질문
    ↓
질문을 가장 잘 드러내는 뷰
```

```plantuml
@startuml
title 이해관계자 중심 뷰 선택
start
:이해관계자 식별;
:업무 질문 수집;
:질문에 맞는 뷰 후보 선택;
:중요도와 작성 비용 비교;
:최소 뷰 집합 작성;
:대표 독자가 질문에 답함;
if (답할 수 있는가?) then (예)
  :뷰 집합 확정;
else (아니오)
  :누락된 뷰 또는 설명 보완;
endif
stop
@enduml
```

다음 표처럼 질문과 뷰의 연결을 먼저 작성하면 불필요한 그림을 줄일 수 있다.

| 이해관계자 질문 | 필요한 정보 | 우선 뷰 |
|---|---|---|
| 결제 변경 시 어떤 코드가 바뀌는가? | 정적 의존성 | 모듈 uses 뷰 |
| 알림 장애가 주문에 전파되는가? | 런타임 호출과 이벤트 | C&C 뷰 |
| DB 장애 시 대기 노드가 있는가? | 노드와 복제 관계 | 배포 뷰 |
| 주문 기능은 어느 팀이 담당하는가? | 책임 대응 | 작업 배정 뷰 |

## 5. 선택한 뷰를 오해 없이 표현한다

> **뷰의 상자와 화살표는 의미를 명확히 정의해야 한다.**
> **표기법은 뷰의 목적에 필요한 차이만 명확히 드러내야 한다.**

뜻이 불분명하면 같은 표현에도 서로 다른 해석이 생긴다. `A → B`는 호출, 데이터 흐름, 코드 의존성, 배포, 제어 이전 등 여러 뜻일 수 있다.

그래서 모든 다이어그램은 다음을 명시해야 한다.

1. 상자가 나타내는 요소 타입
2. 선과 화살표가 나타내는 관계
3. 방향이 나타내는 의미
4. 색과 선 모양의 의미
5. 그림에서 의도적으로 생략한 정보

```plantuml
@startuml
title 모호한 화살표를 피하는 법
left to right direction
skinparam componentStyle rectangle
skinparam packageStyle rectangle

package "나쁜 예: 의미를 추측해야 함" #FFE4E1 {
  component "A" as BadA
  component "B" as BadB

  BadA --> BadB

  note bottom of BadA
    A와 B는 무엇인가?
    모듈, 프로세스, 서버 중 어느 것인가?
  end note

  note bottom of BadB
    화살표는 무엇인가?
    호출, 데이터 흐름, 의존성 중 어느 것인가?
  end note
}

package "좋은 예: 요소·관계·방향을 명시함" #E2F4DF {
  component "Order Service\n<<runtime service>>" as Order
  component "Payment Gateway\n<<external service>>" as Payment
  queue "Event Broker\n<<message broker>>" as Broker

  Order -[#black]-> Payment : HTTPS POST /payments\n동기 결제 승인 요청
  Payment -[#green,dashed]-> Order : HTTP response\n승인 결과 데이터
  Order -[#blue,dotted]-> Broker : OrderPlaced\n비동기 이벤트 발행
}

BadB -[hidden]-> Order

legend bottom
  |= 표기 |= 의미 |
  | 검은 실선 화살표 | 호출자가 시작하는 동기 요청 |
  | 초록 점선 화살표 | 요청에 대한 응답 데이터 |
  | 파란 점선 화살표 | 비동기 이벤트 이동 |

  선의 라벨은 프로토콜, 메시지 또는 관계 이름을 나타낸다.
  방향은 각 관계의 시작점에서 대상 또는 데이터 이동 방향을 가리킨다.
endlegend
@enduml
```

모든 정보를 한 그림에 넣을 필요는 없다. 동기 요청과 비동기 이벤트를 구분하는 것이 목적이라면 그 차이만 명확히 표현하고, 상세 오류 코드는 인터페이스 문서로 분리한다.

## 6. 그림을 설명 자료로 완성한다

> **다이어그램은 설명 자료를 더해야 완전한 뷰가 된다.**

다이어그램은 뷰의 입구이지 전체 문서가 아니다. 그림만으로는 요소의 책임, 제약, 인터페이스, 설계 이유를 충분히 전달할 수 없다. 따라서 하나의 뷰는 다음 순서로 완성한다.

![완전한 뷰를 구성하는 설명 자료](diagrams/18-complete-view.png)

### 6.1 목적과 범위

- 대상 독자와 답하려는 질문
- 시스템 및 문서 기준 버전
- 포함 범위와 제외 범위

### 6.2 기본 표현

- 뷰타입과 적용 스타일
- 표기법과 범례

### 6.3 요소 목록
그림의 모든 핵심 요소는 요소 목록에서 설명되어야 한다. 목록에는 있지만 그림에서 생략한 요소가 있다면 그 이유를 기록한다.

[Example]
| 요소 | 타입 | 책임 | 주요 속성 | 인터페이스 |
|---|---|---|---|---|
| Order Service | 런타임 서비스 | 주문 검증과 생성 | stateless | Order API, OrderPlaced |
| Event Broker | 메시지 커넥터 | 이벤트 전달 | at-least-once | publish, subscribe |

### 6.4 설계 배경

- 품질 목표와 제약
- 중요한 결정과 선택 이유
- 검토했지만 선택하지 않은 대안
- 선택으로 생긴 부정적 결과와 대응책
- 알려진 위험과 미결정 사항

> **결정 근거에는 원인과 결과를 모두 기록해야 한다.**

결과만 기록하면 조건이 변했을 때 결정을 유지해야 하는지 판단하기 어렵다.

![결정의 맥락과 긍정적·부정적 결과](diagrams/19-decision-rationale.png)

```text
맥락: 주문 응답은 500ms 이내여야 하고 알림 장애가 주문을 막으면 안 된다.
결정: 주문 후속 처리를 비동기 이벤트로 분리한다.
긍정적 결과: 응답 지연과 장애 전파가 줄어든다.
부정적 결과: 중복 이벤트와 최종 일관성을 처리해야 한다.
대응: 멱등성 키, outbox, dead-letter queue를 사용한다.
```

## 7. 인터페이스는 제공 기능과 환경 가정을 함께 기록한다

> **인터페이스는 제공 기능과 필요한 환경을 함께 규정해야 한다.**

구조가 요소의 존재와 관계를 보여준다면, 인터페이스는 요소가 실제로 협력하기 위한 계약을 보여준다. 제공하는 API만 기록하고 필요한 환경을 생략하면 통합 실패가 발생한다.

![제공 인터페이스와 요구 환경의 계약](diagrams/20-interface-contract.png)

예를 들어 주문 API가 결제 승인 전에 호출되어야 하는 초기화 절차, 요구하는 인증 토큰, 의존하는 이벤트 스키마 버전을 숨기면 호출자는 계약을 만족할 수 없다. 따라서 인터페이스에는 다음을 포함한다.

- 제공(provided) 기능과 요구(required) 자원
- 입력·출력의 타입, 범위, 단위
- 선행 조건, 후행 조건, 호출 순서
- 오류, 타임아웃, 재시도, 멱등성
- 성능과 보안 제약
- 버전과 호환성 정책

> **인터페이스 문서에는 사용자가 의존해도 되는 정보만 공개한다.**

![공개 계약과 비공개 구현의 구분](diagrams/21-public-vs-private-interface.png)

내부 구현까지 공개하면 작은 구현 변경도 계약 변경으로 번진다.

## 8. 구조로 부족한 부분은 동작으로 보완한다

> **순서나 상태가 결과에 영향을 주면 동작 문서로 보완해야 한다.**

C&C 뷰는 어떤 요소가 연결될 수 있는지 보여주지만, 실제 메시지 순서와 조건까지 모두 보여주지는 않는다.

[동작으로 보완 가능한 다이어그램]
- 시퀀스 다이어그램: 참여자 사이의 메시지 순서와 분기
- 상태 다이어그램: 한 요소의 생명주기와 허용 전이
- 활동 다이어그램: 업무 흐름과 병렬 작업

```plantuml
@startuml
title 주문 처리의 정상 및 실패 흐름
actor 고객
participant "Order Service" as Order
participant "Payment Gateway" as Payment
database "Order DB" as DB

고객 -> Order : 주문 요청
Order -> Payment : 결제 승인
alt 승인 성공
  Payment --> Order : 승인 ID
  Order -> DB : 주문 저장
  Order --> 고객 : 주문 번호
else 승인 실패
  Payment --> Order : 거절 사유
  Order --> 고객 : 결제 실패 안내
end
@enduml
```

> **동작 문서는 중요한 대표 시나리오를 우선해야 한다.**

모든 기능을 시퀀스로 그릴 필요는 없다. 품질 목표, 실패 처리, 동시성 또는 외부 연동에 영향을 주는 시나리오를 먼저 기록한다.

```plantuml
@startuml
title 주문 생성 - 성공 및 결제 실패 동작
actor Customer
participant "Web App" as Web
participant "Order Service" as Order
participant "Payment Gateway" as Payment
database "Order DB" as DB
queue "Event Broker" as Broker

Customer -> Web : 주문 제출
Web -> Order : POST /orders\nIdempotency-Key
Order -> Order : 상품·금액 검증
Order -> Payment : 결제 승인 요청

alt 승인 성공
  Payment --> Order : 승인 ID
  Order -> DB : 주문 + outbox 저장
  DB --> Order : commit
  Order --> Web : 201 Created
  Web --> Customer : 주문 번호 표시
  Order -> Broker : OrderPlaced 발행
else 승인 실패
  Payment --> Order : 거절 사유
  Order --> Web : 422 Payment Declined
  Web --> Customer : 결제수단 변경 안내
end

note over Order,Broker
  이벤트 전달은 최소 한 번(at-least-once)을 가정한다.
  소비자는 주문 ID를 기준으로 멱등 처리한다.
end note
@enduml
```

## 9. 여러 뷰를 연결해 하나의 시스템으로 만든다

> **같은 시스템을 설명하는 여러 뷰를 하나의 전체로 이해하려면 뷰 간 요소의 대응 관계를 명시해야 한다.**

PDF는 뷰 간 매핑을 문서화할 때 한 뷰의 각 요소에 대응하는 다른 뷰의 요소를 기록하고, 그 대응이 부분적인지 전체적인지도 밝히라고 권고한다. 대응은 항상 1:1이 아니다. 하나의 모듈로 여러 런타임 인스턴스를 만들 수 있고, 여러 모듈을 하나의 런타임 요소로 패키징할 수도 있다.

다음 예에서는 `Order Module`과 `Payment Adapter`가 두 개의 `Order Service` 인스턴스에 구현되고, 각 인스턴스가 같은 컨테이너 플랫폼에 배포된다. 이 매핑을 통해 코드 요소에서 실행 요소와 배포 환경까지 이어서 찾을 수 있다.

```plantuml
@startuml
left to right direction

package "모듈 뷰" #DDEBFF {
  [Order Module] as OrderModule
  [Payment Adapter] as PaymentAdapter
}

package "C&C 뷰" #E2F4DF {
  [Order Service #1] as Order1
  [Order Service #2] as Order2
}

package "배포 뷰" #FFF0D6 {
  node "Container Platform" as Platform
}

OrderModule ..> Order1 : implements
OrderModule ..> Order2 : implements
PaymentAdapter ..> Order1 : included in
PaymentAdapter ..> Order2 : included in
Order1 ..> Platform : deployed on
Order2 ..> Platform : deployed on
@enduml
```

*출처: Clements et al.,* Documenting Software Architectures: Views and Beyond, *pp. 253–254, “Mapping Between Views”.*

## 10. 문서는 독자가 사용할 수 있을 때 완성된다

> **아키텍처 문서는 실제 독자가 자신의 질문에 답할 수 있을 때 완성된다.**

문법적으로 정확한 PlantUML과 빈칸 없는 표만으로는 충분하지 않다. 문서화는 한 번 쓰고 끝나는 작업이 아니라 작성, 사용, 검토, 갱신의 순환이다.

```plantuml
@startuml
title 문서화와 검증 순환
left to right direction

rectangle "질문 수집" as Q
rectangle "뷰 선택" as S
rectangle "문서 작성" as W
rectangle "독자 검토" as R
rectangle "문서 갱신" as U

Q --> S
S --> W
W --> R
R --> U : 모호함·누락 발견
U --> R : 다시 검증
R --> Q : 새로운 질문
@enduml
```

### 검토 체크리스트

- [ ] 뷰마다 대상 독자와 답할 질문이 명시되어 있는가?
- [ ] 요소, 관계, 방향, 범례의 뜻을 추측할 필요가 없는가?
- [ ] 그림의 핵심 요소가 요소 목록에 설명되어 있는가?
- [ ] 제공 인터페이스뿐 아니라 요구 조건도 기록되어 있는가?
- [ ] 중요한 정상·실패 동작이 구조와 연결되어 있는가?
- [ ] 설계 결정에 원인, 대안, 긍정·부정 결과가 있는가?
- [ ] 모듈, 런타임, 배포 요소를 서로 추적할 수 있는가?
- [ ] 미결정 사항이 `TBD`, 담당자, 기한과 함께 표시되어 있는가?
- [ ] 문서 버전이 구현 또는 릴리스 기준과 일치하는가?
- [ ] 대표 독자가 문서만 사용해 실제 질문에 답해 보았는가?

## 11. 정리

> **아키텍처 문서화는 결정의 원인에서 독자의 검증과 지속적인 갱신까지 이어지는 과정이다.**

![원인에서 검증과 갱신까지의 문서화 순환](diagrams/23-documentation-cycle.png)

1. 품질 목표와 제약 때문에 시스템 전체에 영향을 주는 결정이 생긴다.
2. 이 결정을 여러 이해관계자가 사용하므로 문서화가 필요하다.
3. 이해관계자의 질문이 서로 다르므로 하나의 그림으로는 부족하다.
4. 질문에 맞춰 모듈, C&C, 할당 뷰를 선택한다.
5. 선택한 뷰의 요소와 관계를 범례로 명확히 정의한다.
6. 그림을 요소 목록, 인터페이스, 설계 근거로 보완한다.
7. 정적 구조로 설명되지 않는 순서와 조건은 동작 모델로 기록한다.
8. 뷰 간 매핑으로 코드 변경부터 배포 영향까지 추적한다.
9. 실제 독자가 질문에 답해 봄으로써 목적 적합성을 검증한다.
10. 구현과 결정이 바뀌면 기준 시점에 문서를 갱신하고 다시 검토한다.
