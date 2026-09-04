# Chapter 02 Review Questions

## 이 문서의 사용법

답을 바로 읽지 말고 먼저 소리 내어 설명한다. 설명이 막히면 해당 답을 읽은 뒤, 문서를 닫고 다시 설명한다.

명령어를 외우는 것보다 "무엇을 검증하려고 이 명령을 실행하는가"를 말할 수 있어야 한다.

## 1. Docker 이미지와 컨테이너는 무엇이 다른가

이미지는 애플리케이션 실행에 필요한 파일과 기본 환경을 담은 불변에 가까운 원본이다. 컨테이너는 이미지를 기반으로 생성되어 실제 프로세스가 실행되는 인스턴스다.

현재 프로젝트에서는 `postgis/postgis:17-3.5`가 이미지이고 `brew-market-db`가 실행 중인 컨테이너다. 컨테이너를 제거해도 이미지와 named volume은 별도로 남을 수 있다.

설명 핵심:

> 이미지는 실행 환경의 원본이고, 컨테이너는 그 이미지에 쓰기 가능한 계층과 실행 상태를 더한 인스턴스입니다.

## 2. 왜 일반 PostgreSQL 이미지가 아니라 PostGIS 이미지를 사용했는가

`brew-market`은 지역 기반 거래 서비스를 목표로 하므로 이후 위치, 거리, 범위 검색을 다룰 가능성이 높다. PostGIS는 PostgreSQL에 공간 타입, 공간 함수, 공간 인덱스 기능을 추가한다.

현재는 공간 검색 기능을 구현한 것이 아니라, PostgreSQL 17과 PostGIS 3.5가 실행되는 로컬 기반만 준비했다. 미래 기능을 구현한 것처럼 설명해서는 안 된다.

설명 핵심:

> PostGIS는 별도 DB가 아니라 PostgreSQL의 공간 데이터 확장입니다. 지역 기반 검색 가능성을 고려해 PostGIS 포함 이미지를 선택했고, Chapter 02에서는 버전과 활성화 여부까지만 검증했습니다.

## 3. `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`는 언제 사용되는가

PostGIS 이미지의 entrypoint가 비어 있는 PostgreSQL 데이터 디렉터리를 최초 초기화할 때 DB와 사용자를 만드는 데 사용한다.

named volume에 이미 데이터베이스가 초기화되어 있다면 값을 바꾸고 컨테이너만 다시 실행해도 기존 DB 사용자나 비밀번호가 자동으로 바뀌지 않는다. 로그의 `Skipping initialization`이 이 차이를 보여준다.

## 4. `${POSTGRES_PORT:-5432}`는 무슨 뜻인가

Compose가 `POSTGRES_PORT` 값을 읽고, 값이 없거나 비어 있으면 기본값 `5432`를 사용한다는 뜻이다.

왼쪽은 호스트 포트이고 오른쪽은 컨테이너 포트다.

```text
${POSTGRES_PORT:-5432}:5432
호스트 포트              컨테이너 포트
```

호스트에서 이미 5432 포트를 사용 중이면 `.env`의 `POSTGRES_PORT`를 다른 값으로 바꿔 충돌을 피할 수 있다.

## 5. health check에서 `$${POSTGRES_USER}`처럼 `$`를 두 번 쓰는 이유는 무엇인가

`$$`는 Compose 단계에서 `$` 하나로 전달된다. 그 결과 `POSTGRES_USER`와 `POSTGRES_DB`는 Compose가 호스트에서 미리 치환하는 대신 컨테이너 내부 셸이 컨테이너 환경변수로 해석한다.

이 차이는 `docker compose config`로 최종 설정을 확인할 수 있다.

## 6. 컨테이너가 `Up`인 것과 `healthy`인 것은 무엇이 다른가

`Up`은 컨테이너의 주 프로세스가 실행 중이라는 뜻이다. `healthy`는 정의한 health check가 성공했다는 뜻이다.

현재 health check는 `pg_isready`를 사용하므로 PostgreSQL이 지정한 사용자와 DB 기준으로 연결 요청을 받을 준비가 되었는지를 확인한다. 실제 비즈니스 쿼리의 정확성이나 데이터 무결성까지 보장하지는 않는다.

## 7. named volume은 왜 컨테이너를 다시 만들어도 데이터를 유지하는가

현재 PostgreSQL 데이터 경로 `/var/lib/postgresql/data`는 컨테이너 파일 시스템이 아니라 Docker가 별도로 관리하는 `brew-market-back_postgres-data`에 저장된다.

새 컨테이너가 같은 볼륨을 마운트하면 기존 데이터 디렉터리를 다시 읽을 수 있다. `docker compose down`은 기본적으로 이 볼륨을 지우지 않지만 `docker compose down -v`는 지운다.

주의할 점:

로그에서 기존 데이터 디렉터리를 재사용한 것은 확인했지만, 특정 행의 데이터가 전후로 동일한지까지 검증하려면 테스트 데이터를 삽입하고 컨테이너 재생성 후 다시 조회해야 한다.

## 8. bind mount와 named volume은 무엇이 다른가

bind mount는 호스트의 특정 경로를 컨테이너에 직접 연결한다. 파일 위치가 눈에 보이고 직접 편집하기 쉽지만 호스트 경로와 권한에 더 의존한다.

named volume은 Docker가 저장 위치와 수명 주기를 관리한다. 현재처럼 DB 데이터를 로컬 경로 구조에서 분리하고 Docker 명령으로 관리할 때 적합하다.

## 9. `.env`와 `.env.example`은 왜 둘 다 필요한가

`.env`는 현재 개발자의 실제 로컬 설정이며 비밀값을 포함할 수 있으므로 Git에서 제외한다. `.env.example`은 필요한 변수의 이름과 예시를 공유하는 문서이므로 Git에 포함한다.

`.env.example`에 실제 비밀번호를 넣으면 `.env`를 제외한 의미가 약해진다. 현재 작업 트리의 `.env.example`은 커밋 전에 반드시 안전한 예시 값으로 되돌려야 한다.

## 10. `docker compose config`는 무엇을 검증하는가

Compose 파일 문법, 환경변수 치환, 기본값 적용, 실제 생성될 서비스·네트워크·볼륨 구성을 확인한다.

컨테이너가 정상 실행되거나 DB 쿼리가 성공한다는 사실까지 검증하지는 않는다. 실행 상태는 `docker compose ps`, DB 준비 상태는 `pg_isready`, 기능 활성화는 SQL 조회처럼 별도의 검증이 필요하다.

## 11. 이번 검증 명령은 각각 무엇을 증명했는가

| 검증 | 확인한 것 | 확인하지 못한 것 |
|---|---|---|
| `docker compose config` | 설정 해석과 환경변수 치환 | 컨테이너 실행 성공 |
| `docker compose ps -a` | 컨테이너 실행 및 health 상태 | 실제 쿼리 결과 |
| `pg_isready` | PostgreSQL 연결 수락 준비 | 스키마와 데이터 정확성 |
| `SELECT version()` | 실행 중인 PostgreSQL 버전 | PostGIS 활성화 |
| `SELECT PostGIS_Version()` | PostGIS 활성화와 버전 | 공간 검색 설계의 정확성 |
| 볼륨 inspect와 재실행 로그 | 동일 데이터 디렉터리 재사용 | 특정 데이터 행의 보존 |

## 12. 비밀번호를 `.env.example`에 커밋했다면 어떻게 대응해야 하는가

파일에서 값을 지우는 것만으로 끝나지 않는다. 이미 원격 저장소에 올라갔다면 노출된 비밀번호를 폐기하고 새 값으로 교체해야 한다. Git 기록에서 제거할 필요가 있는지도 저장소 공개 범위와 민감도에 따라 판단한다.

로컬 개발용 비밀번호라도 다른 환경에서 재사용했다면 실제 보안 사고로 이어질 수 있으므로 비밀번호 재사용을 피해야 한다.

## 직접 해볼 복습

1. `docker compose config` 출력에서 포트, 볼륨, health check를 찾아 설명한다.
2. `docker compose ps`에서 `Up`과 `healthy`를 구분해 설명한다.
3. 테스트 테이블과 행 하나를 만든다.
4. `docker compose down` 후 다시 `up -d` 한다.
5. 같은 행이 조회되는지 확인해 데이터 보존을 직접 증명한다.
6. `docker compose down -v`가 위험한 이유를 실행 전에 설명한다. 실제 삭제 실습은 필요한 데이터가 없는 환경에서만 한다.

## 스스로 다시 설명해보기

- PostGIS는 PostgreSQL과 어떤 관계인가?
- 이미지 태그 `17-3.5`의 두 숫자는 각각 무엇인가?
- 컨테이너를 삭제해도 데이터가 남는 조건은 무엇인가?
- `.env`가 없을 때 Compose 설정은 어떻게 되는가?
- 호스트의 5432 포트가 이미 사용 중이면 어떻게 해결할 수 있는가?
- health check가 성공해도 서비스가 완전히 정상이라고 단정할 수 없는 이유는 무엇인가?
- 환경변수의 초기화 값과 이미 생성된 DB의 실제 설정이 달라질 수 있는 이유는 무엇인가?

## 13. Spring Data JPA, JPA, Hibernate, JDBC는 각각 무엇을 담당하는가

Spring Data JPA는 Repository 구현을 자동으로 제공해 개발자가 CRUD 코드를 덜 작성하게 해준다.

JPA는 Java 객체와 관계형 DB 테이블을 매핑하는 표준 명세다. Hibernate는 그 명세를 실제로 실행하는 구현체다.

JDBC는 Java에서 DB에 접근하는 표준 API이고, PostgreSQL JDBC Driver는 JDBC 호출을 PostgreSQL 서버가 이해하는 방식으로 처리한다.

설명 핵심:

> Spring Data JPA는 편의 계층, JPA는 명세, Hibernate는 구현체, JDBC는 Java의 DB 통신 표준, PostgreSQL Driver는 실제 PostgreSQL 연결 도구입니다.

## 14. `spring-boot-starter-data-jpa`만 추가하면 PostgreSQL에 연결할 수 있는가

아니다. JPA 스타터는 JPA와 Hibernate 기반을 제공하지만, PostgreSQL 서버와 실제로 통신하려면 PostgreSQL JDBC Driver도 필요하다.

현재 프로젝트에서는 다음 두 의존성이 함께 필요하다.

```gradle
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
runtimeOnly 'org.postgresql:postgresql'
```

## 15. PostgreSQL Driver를 `runtimeOnly`로 둔 이유는 무엇인가

애플리케이션 코드가 PostgreSQL Driver의 클래스를 직접 import해서 컴파일하지 않기 때문이다.

코드는 Spring DataSource와 JDBC 표준을 통해 DB에 접근하고, 런타임에 PostgreSQL Driver가 실제 연결을 담당한다. 그래서 컴파일 의존성보다 런타임 의존성이 적절하다.

## 16. `spring.config.import=optional:file:.env[.properties]`는 무엇을 의미하는가

Spring Boot가 로컬 `.env` 파일을 properties 형식으로 읽어 설정 값에 포함한다는 뜻이다.

`optional:`이 붙어 있으므로 `.env` 파일이 없다는 이유만으로 설정 로딩이 바로 실패하지는 않는다. 하지만 DataSource가 요구하는 `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD` 값이 없으면 실제 DB 연결 단계에서 실패할 수 있다.

## 17. `ddl-auto: validate`는 무엇을 하고 무엇을 하지 않는가

`validate`는 Entity 매핑과 DB 스키마가 맞는지 검사한다.

테이블을 만들거나 컬럼을 추가하거나 기존 스키마를 수정하지 않는다. 그래서 스키마를 자동 변경하지 않는 보수적인 설정이다.

현재 프로젝트에서는 아직 마이그레이션 도구가 없으므로, Hibernate가 DB 구조를 암묵적으로 바꾸지 않게 하려는 의도로 선택했다.

## 18. 현재 JPA 설정 이후 아직 검증하지 못한 것은 무엇인가

의존성과 설정 파일이 추가된 것은 커밋으로 확인했다. 하지만 다음은 아직 별도 검증이 필요하다.

- Spring Boot가 실제 Docker PostgreSQL에 연결되는지
- `.env` 값이 현재 DB 사용자와 비밀번호에 맞는지
- Entity가 생겼을 때 `ddl-auto: validate`가 통과하는지
- Repository로 저장과 조회가 되는지

따라서 이번 push는 "DB 기반 연결 설정"이지 "영속성 기능 완성"은 아니다.
