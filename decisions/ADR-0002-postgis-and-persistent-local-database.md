# ADR-0002 - PostGIS 이미지와 named volume 기반 로컬 DB

날짜: 2026.08.11  
상태: Accepted

## Context

`brew-market`은 지역 기반 거래 서비스를 목표로 한다. 앞으로 위치와 거리 데이터를 다룰 가능성이 있지만, Chapter 02에서는 애플리케이션 연동보다 먼저 팀원이 같은 데이터베이스 환경을 재현할 수 있는 로컬 실행 기준이 필요했다.

로컬에 PostgreSQL과 공간 확장을 직접 설치하면 개발자별 버전과 설정이 달라질 수 있다. 또한 컨테이너 내부에만 DB 데이터를 저장하면 컨테이너를 다시 만들 때 데이터가 사라질 수 있다.

## Decision

- Docker Compose로 `db` 서비스를 관리한다.
- `postgis/postgis:17-3.5` 이미지를 사용한다.
- PostgreSQL 데이터 디렉터리 `/var/lib/postgresql/data`를 named volume `postgres-data`에 연결한다.
- DB 초기화 값과 호스트 포트는 `.env`로 주입한다.
- `pg_isready` 기반 health check로 연결 준비 상태를 확인한다.

## Why

- PostgreSQL과 PostGIS 버전을 이미지 태그로 고정할 수 있다.
- 지역 기반 기능에 필요한 공간 데이터 기능을 이후 단계에서 사용할 수 있다.
- Docker Compose 명령으로 실행 절차를 통일할 수 있다.
- named volume이 DB 데이터의 수명을 컨테이너 수명과 분리한다.
- health check를 통해 단순 프로세스 실행과 DB 연결 준비 상태를 구분할 수 있다.

## Alternatives Considered

### 로컬에 PostgreSQL과 PostGIS 직접 설치

Docker 없이도 실행할 수 있지만 운영체제별 설치 과정과 버전 차이가 생긴다. 현재 단계에서는 환경 재현성을 우선해 선택하지 않았다.

### 일반 PostgreSQL 이미지 사용 후 PostGIS 직접 설치

확장 설치 과정을 직접 통제할 수 있지만 별도 이미지 빌드와 유지보수가 필요하다. 지금은 PostGIS가 포함된 이미지로 필요한 기반을 단순하게 확보하는 편이 적절하다.

### 컨테이너 파일 시스템에 데이터 저장

구성은 단순하지만 컨테이너 교체 시 데이터 유실 위험이 있다. 반복 개발 환경에서는 데이터 유지가 필요하므로 선택하지 않았다.

### 호스트 경로를 연결하는 bind mount

데이터 위치를 직접 볼 수 있지만 호스트 경로와 권한에 의존한다. 현재는 Docker가 관리하고 Compose 프로젝트 단위로 식별할 수 있는 named volume을 선택했다.

## Consequences

좋은 점:

- 개발 환경의 PostgreSQL과 PostGIS 버전이 명시된다.
- 컨테이너를 다시 만들어도 같은 볼륨을 연결해 데이터 디렉터리를 재사용할 수 있다.
- 실행, 상태 확인, 종료 절차가 Compose 명령으로 단순해진다.

주의할 점:

- `docker compose down -v`는 named volume과 DB 데이터를 제거한다.
- `POSTGRES_*` 값은 빈 데이터 디렉터리를 최초 초기화할 때 주로 사용된다. 이미 초기화된 볼륨에서는 `.env` 값만 바꿔도 기존 사용자와 비밀번호가 자동 변경되지 않는다.
- 이미지 태그를 고정했어도 정확한 패치 버전은 이미지 갱신 시 달라질 수 있다. 재현성이 더 중요해지면 digest 고정을 검토한다.
- PostGIS 기반 이미지를 선택한 것이 공간 검색 설계를 완료했다는 뜻은 아니다. 공간 타입, 좌표계, 인덱스, 검색 반경은 실제 요구사항과 함께 별도로 결정해야 한다.

## Verification

- `docker compose config`로 환경변수와 최종 Compose 구성을 확인했다.
- `docker compose ps -a`에서 `brew-market-db`가 `healthy` 상태임을 확인했다.
- `pg_isready`가 `accepting connections`를 반환했다.
- `SELECT version()`으로 PostgreSQL 17.5를 확인했다.
- `SELECT PostGIS_Version()`으로 PostGIS 3.5 활성화를 확인했다.
- 재실행 로그에서 기존 데이터 디렉터리를 발견하고 초기화를 건너뛴 것을 확인했다.

특정 데이터 행의 보존 여부는 아직 별도의 삽입·재조회 방식으로 검증하지 않았다.
