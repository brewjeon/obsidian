# ADR-0003 - Spring DataSource와 Hibernate validate 기준

날짜: 2026.09.03  
상태: Accepted

## Context

`brew-market-back`는 Chapter 02.01에서 Docker Compose 기반 PostgreSQL/PostGIS 로컬 DB를 준비했다. 다음 단계에서는 Spring Boot 애플리케이션이 이 DB에 연결할 수 있어야 한다.

DB 연결 값은 개발자 로컬 환경마다 달라질 수 있고, 비밀번호는 Git에 올라가면 안 된다. 또한 아직 Entity와 마이그레이션 전략이 정리되지 않은 상태에서 Hibernate가 스키마를 자동 생성하거나 변경하면 실제 DB 상태를 코드가 암묵적으로 바꿀 위험이 있다.

## Decision

- `spring-boot-starter-data-jpa`를 추가한다.
- PostgreSQL JDBC Driver를 `runtimeOnly` 의존성으로 추가한다.
- Spring Boot 설정에서 `.env`를 `optional:file:.env[.properties]`로 import한다.
- DataSource URL, 사용자명, 비밀번호를 `POSTGRES_*` 환경변수로 구성한다.
- Hibernate DDL 전략은 `ddl-auto: validate`로 시작한다.

## Why

- Spring Data JPA를 통해 Repository 기반 영속성 계층을 만들 수 있다.
- PostgreSQL JDBC Driver가 있어야 JDBC 호출이 실제 PostgreSQL 서버와 통신할 수 있다.
- `.env`를 사용하면 로컬 DB 접속 정보와 Git에 올리는 예시 설정을 분리할 수 있다.
- `optional:` import는 `.env` 파일이 없는 환경에서도 설정 파일 로딩 자체는 가능하게 한다.
- `validate`는 Hibernate가 스키마를 자동 변경하지 않게 하므로 초기 단계에서 데이터베이스 구조를 더 명시적으로 관리할 수 있다.

## Alternatives Considered

### `ddl-auto: create` 또는 `update`

초기 개발 속도는 빠를 수 있지만, Hibernate가 DB 스키마를 자동 생성하거나 변경한다. 현재는 스키마 변경 기준과 마이그레이션 도구가 정해지지 않았으므로 선택하지 않았다.

### 환경변수를 OS 또는 IDE 실행 설정에만 둠

가능하지만 새 개발자가 필요한 변수 목록을 파악하기 어렵다. `.env.example`로 필요한 값의 이름을 공유하는 편이 재현성이 좋다.

### PostgreSQL Driver를 implementation으로 추가

컴파일 코드가 직접 PostgreSQL Driver API에 의존하지 않으므로 `runtimeOnly`가 더 적절하다. 애플리케이션은 JDBC 표준과 Spring DataSource 설정을 통해 드라이버를 사용한다.

## Consequences

좋은 점:

- DB 연결 설정이 코드와 분리된다.
- JPA 기반 Repository 구현을 시작할 수 있다.
- Hibernate가 스키마를 조용히 바꾸는 상황을 피할 수 있다.

주의할 점:

- `.env`가 없거나 값이 맞지 않으면 DB 연결은 실패할 수 있다.
- `validate`는 필요한 테이블이 없으면 애플리케이션 시작 시 실패할 수 있다.
- 아직 마이그레이션 도구가 없으므로 스키마 생성과 변경 방법을 다음 단계에서 결정해야 한다.

## Verification

- `ca1b227 Add JPA PostgreSQL configuration` 커밋에서 `build.gradle`에 JPA 스타터와 PostgreSQL Driver가 추가된 것을 확인했다.
- 같은 커밋에서 `application.yml`에 `.env` import, DataSource, Hibernate `ddl-auto: validate` 설정이 추가된 것을 확인했다.

아직 실제 DB 연결과 Spring Context 로딩은 검증하지 않았다.
