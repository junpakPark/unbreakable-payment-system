# Unbreakable Payment System

> 깨지면서 견고해지는 결제 시스템

스타트업이 성장하면서 마주치는 결제/정산 시스템의 진화 과정을 시뮬레이션한 프로젝트입니다.

## 이 프로젝트는

"일단 출시하자"로 시작해서, 장애를 겪고, 트래픽에 깨지고... 그럴때마다 한 단계씩 견고해지는 **결제 시스템의 성장기**입니다.

```
Phase 1: "일단 돌아가게 만들어서 출시"
    ↓
Phase 2: "출시했더니 여기가 느리네?"
    ↓
Phase 3: "구독 결제 추가해주세요"
    ↓
Phase 4: "동시에 결제하면 오류나요"
    ↓
Phase 5: "정산이 결제를 느리게 해요"
    ↓
Phase 6: "PG사 장애나면 우리도 죽어요"
```

모든 기술 도입에는 **측정 가능한 근거**가 있다고 믿습니다. "왜 Kafka를 도입했나요?"라는 질문에 "PostgreSQL 폴링이 초당 X건에서 한계에 도달했고, 부하테스트 결과 Kafka 전환 후 Y% 개선되었습니다"라고 답할 수 있는 능력을 기르는 것이 이 프로젝트를 시작하는 목적입니다.

## 핵심 원칙

| 원칙          | 설명                               |
|-------------|----------------------------------|
| **측정 후 도입** | 부하테스트로 병목이 증명되기 전까지 기술을 추가하지 않는다 |
| **단순하게 시작** | 저스펙 인스턴스, 단일 서버, RDBMS로 최대한 버틴다  |
| **TDD**     | 테스트 먼저 작성, 100% 커버리지 목표          |

## 기술 스택

| 영역          | 기술                              |
|-------------|---------------------------------|
| Language    | Kotlin + Java 21                |
| Framework   | Spring Boot 3.4                 |
| Database    | PostgreSQL                      |
| ORM         | Spring Data JPA                 |
| Frontend    | Thymeleaf + HTMX + Tailwind CSS |
| Test        | JUnit5 + AssertJ + RestAssured  |
| Monitoring  | Prometheus + Grafana + Loki     |
| Load Test   | k6                              |

## 아키텍처

### 레이어드 아키텍처

```
┌─────────────────────────────────────┐
│           Presentation              │
│    (Controller, Thymeleaf, API)     │
├─────────────────────────────────────┤
│           Application               │
│         (Service, UseCase)          │
├─────────────────────────────────────┤
│             Domain                  │
│   (Entity, VO, Repository 인터페이스) │
├─────────────────────────────────────┤
│          Infrastructure             │
│    (JPA Repository, PG Client)      │
└─────────────────────────────────────┘
```

## 도메인 구조

```
구매자
   │
   ▼
Order ────→ Payment ────→ Ledger
(주문)       (결제)        (원장)
  │            │
  └─ OrderItem │
               │
   ┌───────────┴───────────┐
   ▼                       ▼
Subscription            Wallet ────→ Settlement
(구독)                  (지갑)        (정산)
                          │
                          ▼
                        판매자
```

| 도메인              | 역할                             |
|------------------|--------------------------------|
| **Order**        | 주문 정보, 부분 환불을 위한 OrderItem 포함  |
| **Payment**      | PG 연동, 멱등성 키 기반 중복 방지, 타임아웃 처리 |
| **Ledger**       | 복식부기 원장, 모든 거래의 차변/대변 기록       |
| **Wallet**       | 판매자별 정산대기금, 음수 잔액 허용           |
| **Settlement**   | D+1 정산, VAT/원천징수 계산            |
| **Subscription** | 구독 상태 관리, 자동 재시도 (3회)          |

## 시작하기

### 요구사항

- Java 21+
- PostgreSQL 15+
- Docker (optional)

### 실행

```bash
# 테스트
./gradlew test

# 로컬 실행
./gradlew bootRun

# 빌드
./gradlew bootJar
```

### 환경 변수

```bash
DB_HOST=localhost
DB_PORT=5432
DB_NAME=payment
DB_USERNAME=payment
DB_PASSWORD=payment
```

## API 문서

실행 후 Swagger UI에서 확인할 수 있습니다.

```
http://localhost:8080/swagger-ui.html
```

## 모니터링

| 엔드포인트                  | 용도             |
|------------------------|----------------|
| `/actuator/health`     | 헬스체크           |
| `/actuator/prometheus` | Prometheus 메트릭 |

## 프로젝트 구조

```
src/main/kotlin/com/example/payment/
├── controller/          # Presentation Layer
├── service/             # Application Layer
├── domain/              # Domain Layer
│   ├── order/
│   ├── payment/
│   ├── ledger/
│   ├── wallet/
│   └── settlement/
└── infrastructure/      # Infrastructure Layer
    ├── persistence/
    └── pg/
```

## 라이선스

MIT License
