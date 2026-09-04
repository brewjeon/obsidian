# Chapter 02.02 - JPA, Hibernate, JDBC와 PostgreSQL 연결 설정

날짜: 2026.09.03  
프로젝트: brew-market  
주제: Spring Boot에서 JPA 의존성과 PostgreSQL DataSource 설정 추가

## 이번 단위의 목표

Spring Boot 애플리케이션에서 DB 저장 요청이 어떤 흐름으로 PostgreSQL까지 전달되는지 이해하고, 그 흐름에 필요한 최소 의존성과 설정을 추가한다.

이번 작업의 결과는 `brew-market-back`의 최근 push인 `ca1b227 Add JPA PostgreSQL configuration`에 해당한다.

## 먼저 이해할 개념

### 전체 흐름

Spring Boot 코드에서 DB를 사용할 때 요청은 대략 다음 순서로 내려간다.

```text
애플리케이션 코드
  -> Spring Data JPA
  -> JPA
  -> Hibernate
  -> JDBC
  -> PostgreSQL JDBC Driver
  -> PostgreSQL/PostGIS
```

각 기술은 역할이 다르다.

- Spring Data JPA: Repository 인터페이스만으로 기본 CRUD 구현을 제공한다.
- JPA: Java 객체와 관계형 DB 테이블을 매핑하는 표준 명세다.
- Hibernate: JPA 명세를 실제로 실행하는 구현체다.
- JDBC: Java가 DB와 통신하는 낮은 수준의 표준 API다.
- PostgreSQL JDBC Driver: JDBC 호출을 PostgreSQL 프로토콜로 바꿔 실제 DB와 통신한다.

### Spring Data JPA

Spring Data JPA는 Repository 계층을 단순하게 만들어준다.

예를 들어 다음처럼 Repository 인터페이스를 만들면, 직접 구현 클래스를 작성하지 않아도 기본 저장, 조회, 삭제 메서드를 사용할 수 있다.

```java
public interface UserRepository extends JpaRepository<User, Long> {
}
```

이 프로젝트에서는 아직 `User` Entity와 Repository를 만들지 않았다. 이번 push는 그 전 단계인 JPA 사용 준비와 DB 연결 설정이다.

### JPA

JPA는 Java Persistence API의 약자다. Java 객체를 DB 테이블과 연결하고, 객체의 상태 변경을 DB에 반영하는 방법을 정의한다.

JPA 자체는 명세이므로 직접 DB와 통신하지 않는다. 실제 동작은 Hibernate 같은 구현체가 담당한다.

### Hibernate

Hibernate는 Spring Boot에서 기본적으로 많이 쓰는 JPA 구현체다. Entity 매핑 정보를 읽고 SQL을 만들거나 실행한다.

이번 설정에서는 Hibernate가 스키마를 자동으로 만들지 않고, 이미 존재하는 DB 스키마와 Entity 매핑이 맞는지만 확인하도록 `ddl-auto: validate`를 선택했다.

### JDBC와 PostgreSQL Driver

JDBC는 Java 애플리케이션이 DB에 연결하는 표준 인터페이스다. 하지만 JDBC만으로는 특정 DB와 통신할 수 없다.

PostgreSQL과 연결하려면 PostgreSQL JDBC Driver가 필요하다. 그래서 `build.gradle`에 `runtimeOnly 'org.postgresql:postgresql'`을 추가했다.

## 현재 프로젝트와의 연결

Chapter 02.01에서 Docker Compose로 `brew-market-db` 컨테이너를 띄웠다.

이번 작업은 그 DB에 Spring Boot 애플리케이션이 연결할 수 있도록 백엔드 설정을 추가한 것이다.

관련 파일:

- `brew-market-back/build.gradle`
- `brew-market-back/src/main/resources/application.yml`
- `brew-market-back/.env.example`
- `brew-market-back/docker-compose.yml`

## 구현한 것

### JPA 의존성 추가

`build.gradle`에 Spring Data JPA 스타터를 추가했다.

```gradle
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

이 의존성을 통해 Spring Data JPA, JPA API, Hibernate 관련 구성이 함께 들어온다.

### PostgreSQL JDBC Driver 추가

`build.gradle`에 PostgreSQL JDBC Driver를 런타임 의존성으로 추가했다.

```gradle
runtimeOnly 'org.postgresql:postgresql'
```

애플리케이션 코드는 JDBC 표준을 사용하지만, 실제 PostgreSQL 서버와 통신하려면 이 드라이버가 런타임에 필요하다.

### `.env` import 설정

`application.yml`에 다음 설정을 추가했다.

```yaml
spring:
  config:
    import: optional:file:.env[.properties]
```

이 설정은 프로젝트 루트의 `.env` 파일을 Spring 설정 값으로 읽게 한다.

`optional:`을 붙였기 때문에 `.env` 파일이 없다는 이유만으로 애플리케이션이 즉시 실패하지는 않는다. 다만 DataSource에 필요한 `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD` 값이 실제로 없으면 DB 연결 단계에서 실패할 수 있다.

### DataSource 설정

`application.yml`에 PostgreSQL 연결 정보를 추가했다.

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:${POSTGRES_PORT:5432}/${POSTGRES_DB}
    username: ${POSTGRES_USER}
    password: ${POSTGRES_PASSWORD}
```

Docker Compose에서 호스트의 `POSTGRES_PORT`를 컨테이너의 PostgreSQL 5432 포트에 연결했기 때문에 Spring Boot는 `localhost`로 접근한다.

`POSTGRES_PORT`는 값이 없으면 기본값 5432를 사용한다. DB 이름, 사용자명, 비밀번호는 `.env`에서 가져온다.

### Hibernate DDL 전략 설정

```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: validate
```

`validate`는 Hibernate가 Entity 매핑과 DB 스키마가 맞는지 검사하게 한다. 테이블을 자동 생성하거나 수정하지 않는다.

현재 프로젝트는 아직 Entity와 마이그레이션 도구가 없으므로, `validate`는 "스키마를 코드가 임의로 바꾸지 않는다"는 보수적인 기준을 먼저 세우는 역할이다.

## 검증한 것

최근 push에서 확인할 수 있는 검증은 커밋 기준으로 다음과 같다.

```text
ca1b227 Add JPA PostgreSQL configuration
 build.gradle                       |  2 ++
 src/main/resources/application.yml | 12 ++++++++++++
```

즉, JPA/PostgreSQL 연결에 필요한 의존성과 설정 파일이 추가된 것은 확인된다.

다만 이 단계에서 아직 확인하지 못한 것도 분명하다.

- Spring Boot 애플리케이션이 실제 DB에 연결 성공하는지
- `.env` 값이 현재 로컬 DB와 정확히 일치하는지
- Hibernate `validate`가 통과할 스키마가 존재하는지
- Entity와 Repository가 정상 동작하는지

이 검증은 다음 작업에서 별도로 해야 한다.

## 결과 해석

이번 push는 기능 구현이라기보다 영속성 계층을 사용하기 위한 기반 설정이다.

아직 게시글, 사용자, 위치 검색 같은 비즈니스 기능이 생긴 것은 아니다. 하지만 이제 백엔드가 PostgreSQL/PostGIS에 연결할 준비를 갖췄고, 다음 단계에서 Entity와 Repository를 추가할 수 있다.

## 내 언어로 설명

이번 작업은 Spring Boot가 PostgreSQL을 사용할 수 있게 길을 연결한 것이다.

JPA는 객체와 테이블을 매핑하는 규칙이고, Hibernate는 그 규칙을 실제로 실행하는 도구다. JDBC는 Java가 DB와 통신하는 표준 통로이며, PostgreSQL Driver는 그 통로를 PostgreSQL 서버가 이해하는 방식으로 연결해준다.

`application.yml`은 `.env`에서 DB 접속 값을 읽어 DataSource를 구성한다. `ddl-auto: validate`를 사용했기 때문에 Hibernate가 테이블을 자동으로 만들거나 바꾸지 않고, 코드의 Entity와 DB 스키마가 맞는지만 확인한다.

## 다음 단위

1. Docker Compose DB를 실행한 상태에서 Spring Boot 애플리케이션 또는 테스트를 실행한다.
2. DataSource 연결 실패가 나면 `.env`, 포트, DB 사용자, named volume 초기화 상태를 확인한다.
3. 첫 Entity와 Repository를 만들고 `ddl-auto: validate`가 어떤 조건에서 통과하는지 검증한다.

## 이번 챕터 학습 2줄

Spring Data JPA를 쓰려면 JPA/Hibernate만 이해하는 것이 아니라, 런타임에서 실제 DB와 통신하는 JDBC Driver와 DataSource 설정까지 연결해서 봐야 한다.  
`ddl-auto: validate`는 Hibernate가 스키마를 자동 생성하지 않게 하면서, 코드와 DB 스키마가 맞는지 확인하는 보수적인 시작점이다.

## 태그

#brew-market #spring-boot #jpa #hibernate #jdbc #postgresql #datasource
