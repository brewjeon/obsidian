# Chapter 01.01 - Spring Boot와 HTTP 요청

날짜: 2026.08.09 일요일  
프로젝트: brew-market  
주제: 프로젝트 기반 Spring Boot 시작, HTTP 요청과 검증 흐름 만들기

## 이번 챕터의 목표

`brew-market`의 백엔드와 프론트엔드 저장소를 최소 형태로 준비하고, 백엔드 서버가 HTTP 요청을 받을 수 있는 상태인지 `/health` API로 검증한다.

이번 챕터의 핵심은 기능을 많이 만드는 것이 아니라, 앞으로 기능을 쌓을 수 있는 실행 가능한 기준선을 만드는 것이다.

## 구현한 것

- `brew-market-back` 저장소 확인 및 Spring Boot 백엔드 프로젝트 구성
- Java 21, Spring Boot 4.1.0, Gradle 기반 설정
- Spring Web, Validation 의존성 추가
- Spring Boot 시작 클래스 `BrewMarketBackApplication` 구성
- `/health` API 구현
- `/health` Controller 테스트 작성
- `brew-market-back`, `brew-market-front` README 작성
- commit 및 push 완료

## 검증한 것

```powershell
./gradlew.bat test
./gradlew.bat bootRun
curl.exe -i http://localhost:8080/health
```

검증 결과:

- 테스트가 통과하면 Controller가 기대한 JSON 응답을 만드는지 확인할 수 있다.
- `bootRun`이 성공하면 Spring Boot 애플리케이션 컨텍스트와 내장 웹 서버가 실행 가능한지 확인할 수 있다.
- `curl` 호출이 HTTP 200을 반환하면 실제 네트워크 경로로 요청과 응답이 오가는지 확인할 수 있다.

## 결과물

- 실행 가능한 `brew-market-back`
- 최소 README가 있는 `brew-market-front`
- `/health` HTTP 200 응답
- 자동화 테스트
- Chapter 01 개발 기록

## 이번 챕터의 설계 판단

`/health` API는 비즈니스 기능은 아니지만 프로젝트의 첫 기준점으로 적합하다. 이유는 서버 실행, 라우팅, JSON 직렬화, 테스트, 실제 HTTP 호출을 모두 가장 작은 단위로 확인할 수 있기 때문이다.

테스트와 실제 HTTP 호출을 모두 사용한 것도 좋은 선택이다. 테스트는 빠르게 반복 가능한 코드 수준의 보증이고, `curl`은 애플리케이션이 실제 서버로 떠서 외부 요청을 처리하는지 확인하는 운영 관점의 검증이다.

## 내 언어로 설명

Spring Boot 프로젝트를 만든 뒤 바로 도메인 기능을 구현하지 않고 `/health`부터 만든 이유는, 개발 환경과 실행 경로가 정상인지 먼저 확인하기 위해서입니다. 이 API는 작은 기능이지만 컨트롤러 매핑, JSON 응답, 테스트, 실제 HTTP 호출을 모두 검증할 수 있어서 이후 기능 개발의 기준선이 됩니다.

테스트와 `curl`은 검증하는 층이 다릅니다. MockMvc 테스트는 컨트롤러가 기대한 HTTP 응답을 만드는지 빠르게 확인하고, `curl`은 애플리케이션이 실제 포트에서 요청을 받을 수 있는지 확인합니다.

## 개선할 점

- README의 실행 명령을 테스트 실행과 서버 실행으로 분리해서 더 명확하게 만든다.
- Health 응답 형식을 단순 문자열 Map에서 명시적인 DTO로 바꿀지 검토한다.
- 다음 단계부터는 API 응답 규칙, 에러 응답 규칙, 패키지 구조를 작게라도 정리한다.

## 다음 액션

- Chapter 02에서 첫 도메인 후보를 정한다.
- 게시글, 사용자, 위치 중 어떤 경계를 먼저 잡을지 결정한다.
- API 설계 전에 요청/응답 예시를 먼저 작성한다.
- 테스트 이름을 비즈니스 의도 중심으로 쓰는 연습을 시작한다.

## 태그

#brew-market #spring-boot #http #testing #chapter-01
