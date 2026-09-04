# Chapter 02 - PostgreSQL/PostGIS와 JPA

## 챕터 목표

PostgreSQL/PostGIS 로컬 환경을 구성하고 Spring Boot와 연결할 수 있는 영속성 기반을 만든다.

이 챕터의 핵심은 "DB를 띄웠다"에서 끝나는 것이 아니라, 애플리케이션 코드가 어떤 의존성과 설정을 통해 PostgreSQL까지 도달하는지 설명할 수 있게 되는 것이다.

## 진행 상태

- [x] Docker Compose로 PostgreSQL/PostGIS 실행
- [x] `.env`와 `.env.example`로 로컬 DB 설정 분리
- [x] JPA, Hibernate, JDBC 역할 구분
- [x] Spring Boot DataSource 설정
- [x] PostgreSQL JDBC Driver 연결 의존성 추가
- [x] Hibernate `ddl-auto: validate` 기준 설정
- [ ] DB 연결과 Spring Context 로딩 검증
- [ ] BaseEntity와 JPA Auditing
- [ ] User Entity와 Repository
- [ ] 이메일 unique 제약조건
- [ ] Chapter 02 회고 및 복습

## 학습 파일

- [[01-docker-postgresql-postgis|Docker Compose로 PostgreSQL과 PostGIS 실행]]
- [[02-jpa-hibernate-jdbc|JPA, Hibernate, JDBC와 PostgreSQL 연결 설정]]

## 관련 의사결정

- [[../../decisions/ADR-0002-postgis-and-persistent-local-database|ADR-0002 - PostGIS 이미지와 named volume 기반 로컬 DB]]
- [[../../decisions/ADR-0003-spring-datasource-and-jpa-validation|ADR-0003 - Spring DataSource와 Hibernate validate 기준]]

## 현재 코드 기준 요약

`brew-market-back`는 Java 21, Spring Boot 4.1.0, Gradle 기반 백엔드다.

현재 백엔드에서 DB 연결을 위해 추가된 것은 다음과 같다.

- `spring-boot-starter-data-jpa`: Repository, EntityManager, Hibernate 기반 JPA 사용
- `org.postgresql:postgresql`: JDBC 레벨에서 PostgreSQL과 실제 통신
- `application.yml`: `.env` import, DataSource URL, 사용자명, 비밀번호, Hibernate DDL 전략 설정

현재 `application.yml`의 기준 설정은 다음 의도를 가진다.

- `.env`는 로컬 개발자의 실제 DB 값을 담고 Git에 올리지 않는다.
- `.env.example`은 필요한 변수 이름만 공유한다.
- `jdbc:postgresql://localhost:${POSTGRES_PORT:5432}/${POSTGRES_DB}`로 Docker Compose DB에 연결한다.
- `ddl-auto: validate`를 사용해 Hibernate가 스키마를 마음대로 생성하거나 변경하지 않게 한다.

## 남은 작업

이번 단계에서는 JPA와 PostgreSQL 연결을 위한 의존성과 설정을 추가했다. 실제 DB 연결 검증, Entity/Repository 구현, 스키마 관리 전략은 다음 작업으로 남겨둔다.

## 다음 단위

1. Docker DB가 켜진 상태에서 Spring Boot 테스트 또는 애플리케이션 실행으로 DataSource 연결을 검증한다.
2. 첫 Entity와 Repository를 추가한다.
3. Hibernate `validate`가 실패하거나 통과하는 조건을 직접 확인한다.
