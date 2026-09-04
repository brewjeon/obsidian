# ADR-0001 - Backend Bootstrap and Health Check

날짜: 2026.08.09  
상태: Accepted

## Context

`brew-market`는 지역 기반 거래 서비스를 목표로 한다. Chapter 01에서는 도메인 기능보다 먼저 백엔드 프로젝트가 정상적으로 실행되고 HTTP 요청을 처리할 수 있는 기준선을 만드는 것이 필요했다.

초기 단계에서 검증해야 할 것은 다음과 같다.

- Java 21과 Spring Boot 프로젝트가 빌드 가능한가
- 애플리케이션 시작 클래스가 정상적으로 동작하는가
- HTTP 라우팅이 동작하는가
- JSON 응답을 반환할 수 있는가
- 자동화 테스트와 실제 HTTP 호출을 모두 수행할 수 있는가

## Decision

Spring Boot 백엔드에 `/health` API를 추가한다.

응답은 다음 JSON 형태로 둔다.

```json
{
  "status": "UP"
}
```

테스트는 Controller 레이어 중심으로 작성하고, 실제 실행 검증은 `bootRun`과 `curl`로 수행한다.

## Why

`/health`는 작지만 시스템의 시작 경로를 검증하기에 충분하다.

- Controller 매핑이 동작한다.
- JSON 응답 직렬화가 동작한다.
- 테스트 도구가 프로젝트에 연결된다.
- 내장 웹 서버가 실제 요청을 처리한다.
- 이후 CI, 배포, 모니터링의 출발점으로 확장할 수 있다.

## Alternatives Considered

첫 도메인 API를 바로 구현할 수도 있었다. 하지만 사용자, 게시글, 위치 같은 도메인은 요구사항과 데이터 모델 판단이 필요하다. Chapter 01에서는 도메인 판단보다 프로젝트 실행 가능성을 먼저 보증하는 것이 더 적절하다.

Spring Boot Actuator를 바로 도입할 수도 있었다. 하지만 현재는 학습과 최소 구현이 목적이므로 직접 `/health`를 구현해 HTTP 요청 처리 흐름을 이해하는 편이 낫다. 운영 단계가 가까워지면 Actuator 도입을 다시 검토한다.

## Consequences

좋은 점:

- 프로젝트가 실행 가능한 상태임을 빠르게 증명할 수 있다.
- 테스트와 수동 HTTP 검증의 차이를 학습할 수 있다.
- 이후 기능 개발 전에 기본 실행 경로가 깨졌는지 확인할 기준이 생긴다.

주의할 점:

- 현재 `/health`는 애플리케이션 프로세스가 응답하는지만 확인한다.
- DB, Redis, 외부 API 같은 의존성 상태는 아직 검증하지 않는다.
- 운영용 health check로 사용하려면 Actuator 또는 의존성별 readiness/liveness 구분이 필요하다.
