# Chapter 02.01 - Docker Compose로 PostgreSQL과 PostGIS 실행

날짜: 2026.08.11 화요일  
프로젝트: brew-market  
주제: Docker Compose 기반 로컬 PostgreSQL·PostGIS 환경 구성과 데이터 유지 검증

## 이번 챕터의 목표

`brew-market-back`에서 사용할 로컬 데이터베이스를 Docker Compose로 실행하고, PostgreSQL과 PostGIS가 정상 동작하는지 확인한다.

컨테이너를 다시 생성하더라도 데이터가 유지되도록 PostgreSQL의 데이터 저장 경로를 named volume으로 분리하고, 로컬 비밀값은 Git에 포함하지 않는 환경변수 관리 기준을 만든다.

## 구현한 것

- `brew-market-back/docker-compose.yml` 추가
- PostgreSQL 17과 PostGIS 3.5가 포함된 `postgis/postgis:17-3.5` 이미지 사용
- 컨테이너 이름을 `brew-market-db`로 지정
- DB 이름, 사용자, 비밀번호, 호스트 포트를 환경변수로 분리
- PostgreSQL 데이터 경로를 named volume `postgres-data`에 연결
- `pg_isready` 기반 health check 추가
- 로컬 설정 파일 `.env` 추가
- 공유용 환경변수 템플릿 `.env.example` 추가
- `.env`는 제외하고 `.env.example`은 추적하도록 `.gitignore` 수정
- README에 DB 실행, 상태 확인, 종료 명령 추가
- `.gitignore`에 남아 있던 Git 충돌 표시 제거

## `docker-compose.yml` 설정

현재 프로젝트에서 작성한 설정은 다음과 같다.

```yaml
services:
  db:
    image: postgis/postgis:17-3.5
    container_name: brew-market-db
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    ports:
      - "${POSTGRES_PORT:-5432}:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres-data:
```

### `services`와 `db`

`services`는 Docker Compose가 관리할 실행 단위를 선언한다. 현재는 데이터베이스 하나만 필요하므로 `db` 서비스만 정의했다.

`db`는 Compose 내부의 서비스 이름이다. `docker compose logs db`, `docker compose exec db ...`처럼 특정 서비스를 지정할 때 사용한다.

### `image`

```yaml
image: postgis/postgis:17-3.5
```

컨테이너를 생성할 원본 이미지다. 현재 프로젝트는 PostgreSQL 17과 PostGIS 3.5가 함께 구성된 이미지를 사용한다.

### `container_name`

```yaml
container_name: brew-market-db
```

실제로 생성되는 컨테이너 이름을 고정한다. Docker 컨테이너 목록이나 로그에서 프로젝트의 DB를 `brew-market-db`라는 이름으로 식별할 수 있다.

### `environment`

```yaml
environment:
  POSTGRES_DB: ${POSTGRES_DB}
  POSTGRES_USER: ${POSTGRES_USER}
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

PostgreSQL을 처음 초기화할 때 사용할 데이터베이스 이름, 사용자, 비밀번호를 `.env`에서 읽어 컨테이너에 전달한다. 비밀번호를 Compose 파일에 직접 작성하지 않기 위한 설정이기도 하다.

### `ports`

```yaml
ports:
  - "${POSTGRES_PORT:-5432}:5432"
```

호스트 포트를 컨테이너 내부 PostgreSQL 포트 `5432`에 연결한다. `POSTGRES_PORT`가 없으면 기본값 `5432`를 사용한다. 현재 실행 결과에서는 호스트 `5432`와 컨테이너 `5432`가 연결되었다.

### 서비스의 `volumes`

```yaml
volumes:
  - postgres-data:/var/lib/postgresql/data
```

PostgreSQL이 데이터를 저장하는 `/var/lib/postgresql/data`를 named volume `postgres-data`에 연결한다. DB 데이터를 컨테이너 자체의 수명과 분리하는 설정이다.

### `healthcheck`

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
  interval: 10s
  timeout: 5s
  retries: 5
```

`pg_isready`로 PostgreSQL이 실제 연결을 받을 준비가 되었는지 확인한다. 10초마다 검사하고, 한 번의 검사 제한 시간은 5초이며, 5번 연속 실패하면 `unhealthy`로 판단한다.

`$${POSTGRES_USER}`처럼 `$`를 두 번 쓴 것은 Compose가 먼저 값을 치환하지 않고 컨테이너 내부 셸이 환경변수를 읽게 하기 위해서다.

### 최상위 `volumes`

```yaml
volumes:
  postgres-data:
```

Compose가 관리할 named volume을 선언한다. 실제로 생성된 이름은 Compose 프로젝트명이 붙은 `brew-market-back_postgres-data`였다.

## 이미지와 컨테이너의 차이

현재 프로젝트의 이미지는 `postgis/postgis:17-3.5`다. PostgreSQL과 PostGIS를 실행하는 데 필요한 파일과 기본 환경을 담은 원본이다.

컨테이너 `brew-market-db`는 그 이미지로 생성되어 실제 실행되는 인스턴스다. 하나의 이미지로 여러 컨테이너를 만들 수 있고, 컨테이너를 제거해도 이미지 자체는 별도로 남을 수 있다.

```text
postgis/postgis:17-3.5 이미지
                ↓ 생성 및 실행
brew-market-db 컨테이너
```

## PostgreSQL과 PostGIS의 관계

PostgreSQL은 관계형 데이터베이스이고, PostGIS는 PostgreSQL에 위치, 좌표, 거리, 범위 같은 공간 데이터 처리 기능을 추가하는 확장 기능이다.

PostGIS는 별도의 데이터베이스가 아니라 PostgreSQL 위에서 동작한다. 현재 사용한 `postgis/postgis:17-3.5` 이미지도 PostgreSQL 17을 기반으로 PostGIS 3.5를 사용할 수 있게 구성되어 있다.

실제 조회에서는 PostgreSQL 17.5와 PostGIS 3.5가 확인되었다.

## named volume으로 데이터가 유지되는 이유

컨테이너 내부에만 DB 데이터를 저장하면 컨테이너를 제거할 때 데이터도 함께 사라질 수 있다. 현재 설정은 PostgreSQL 데이터 디렉터리를 Docker가 별도로 관리하는 `brew-market-back_postgres-data` 볼륨에 저장한다.

따라서 컨테이너를 내리거나 다시 생성해도 같은 볼륨을 연결하면 기존 데이터 디렉터리를 다시 사용할 수 있다. 실제 재실행 로그에서도 다음 메시지가 확인되었다.

```text
PostgreSQL Database directory appears to contain a database; Skipping initialization
```

일반적인 `docker compose down`은 named volume을 유지하지만, `docker compose down -v`는 볼륨까지 제거하므로 저장된 DB 데이터가 삭제될 수 있다.

## `.env`와 `.env.example`의 차이

`.env`는 현재 개발 환경에서 Docker Compose가 실제로 읽는 파일이다. 로컬 DB 비밀번호처럼 저장소에 공개하면 안 되는 값을 포함하므로 `.gitignore`에 등록했다.

`.env.example`은 프로젝트 실행에 필요한 환경변수의 이름과 형식을 공유하는 템플릿이다. 다른 개발자는 이 파일을 복사해 자신의 `.env`를 만들 수 있다. Git으로 공유되는 파일이므로 실제 비밀번호 대신 `change-me` 같은 예시 값만 두어야 한다.

현재 `.gitignore`의 관련 설정은 다음과 같다.

```gitignore
.env
!.env.example
```

## 검증한 것

### Compose 설정 해석

```powershell
docker compose config
```

검증 결과:

- Compose 파일이 문법 오류 없이 해석되었다.
- `.env` 값이 컨테이너 환경변수에 적용되었다.
- 호스트 `5432` 포트와 컨테이너 `5432` 포트가 연결되었다.
- `postgres-data`가 `/var/lib/postgresql/data`에 연결되었다.

### 컨테이너와 health check 상태

```powershell
docker compose ps -a
```

검증 결과:

```text
NAME             IMAGE                    SERVICE   STATUS
brew-market-db   postgis/postgis:17-3.5   db        Up (healthy)
```

컨테이너가 실행 중이었고 health check도 통과했다. 포트는 `0.0.0.0:5432->5432/tcp`로 연결되었다.

### PostgreSQL 연결 준비 상태

```powershell
docker compose exec -T db pg_isready -U brew_market -d brew_market
```

검증 결과:

```text
/var/run/postgresql:5432 - accepting connections
```

PostgreSQL이 정상적으로 연결을 받을 수 있는 상태임을 확인했다.

### PostgreSQL과 PostGIS 버전

```sql
SELECT version();
SELECT PostGIS_Version();
```

검증 결과:

```text
PostgreSQL 17.5
PostGIS 3.5 USE_GEOS=1 USE_PROJ=1 USE_STATS=1
```

PostgreSQL이 정상 실행되고 PostGIS 확장도 현재 데이터베이스에서 동작함을 확인했다.

### named volume

`brew-market-back_postgres-data` 볼륨이 생성되어 `/var/lib/postgresql/data`에 연결된 것을 확인했다. 컨테이너 재실행 로그에서 기존 데이터 디렉터리를 발견하고 초기화를 건너뛴 것도 확인했다.

## 결과물

- 실행 가능한 `brew-market-db` 컨테이너
- health check가 통과한 PostgreSQL 17.5
- 사용할 수 있는 PostGIS 3.5 확장
- 컨테이너와 수명이 분리된 PostgreSQL named volume
- 실제 값과 공유용 예시를 분리할 수 있는 환경변수 파일 구조
- 로컬 DB 실행 방법이 추가된 README

## 발생한 문제와 해결

### `.gitignore`에 Git 충돌 표시가 남아 있던 문제

`.gitignore`에 `<<<<<<<`, `=======`, `>>>>>>>` 충돌 표시가 남아 있었다. 이전 병합 과정에서 충돌 표시가 완전히 정리되지 않은 것이 원인이었다.

충돌 표시를 제거하고 필요한 ignore 규칙을 하나로 합친 뒤 `.env` 제외 규칙과 `.env.example` 예외 규칙을 추가했다.

### `.env.example`의 비밀값 관리 문제

현재 작업 트리의 `.env.example`에 실제 비밀번호처럼 보이는 값이 들어가 있다. `.env`와 `.env.example`의 역할을 혼동하면 공유용 파일을 통해 비밀값이 커밋될 수 있다.

커밋 전 `.env.example`의 비밀번호를 `change-me` 같은 예시 값으로 되돌리고, 실제 값은 Git에서 제외된 `.env`에만 두어야 한다. 이미 외부에 공유한 비밀번호라면 값도 교체해야 한다.

## 개선할 점

- 커밋 전에 `.env.example`에 실제 비밀값이 없는지 확인한다.
- named volume 유지 여부를 더 명확히 검증하려면 테스트 데이터를 만든 뒤 컨테이너를 재생성하고 같은 데이터를 다시 조회한다.
- README에 `.env.example`을 `.env`로 복사하는 초기 설정 절차를 추가한다.

## 다음 액션

- `.env.example`의 비밀번호를 안전한 예시 값으로 정리한다.
- DB 실행과 종료 절차를 README만 보고 재현할 수 있는지 확인한다.
- 데이터가 필요한 다음 기능을 시작하기 전에 로컬 DB 환경의 재현성을 한 번 더 확인한다.

## 이번 챕터 학습 2줄

Docker 이미지는 실행 환경의 원본이고, 컨테이너는 그 이미지로 생성되어 실제 동작하는 인스턴스다.  
PostgreSQL 데이터를 named volume에 분리하면 컨테이너를 다시 생성해도 기존 데이터를 계속 사용할 수 있다.

## 태그

#brew-market #docker #docker-compose #postgresql #postgis #database #chapter-02
