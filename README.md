# Brew Market Obsidian Notes

`brew-market` 프로젝트의 학습, 구현, 검증, 회고를 정리하는 Obsidian 노트 공간이다.

## 목표

단순히 기능을 만든 기록이 아니라, 다음 역량이 드러나도록 정리한다.

- 문제를 작은 단위로 나누고 검증 가능한 결과물로 만드는 능력
- Spring Boot, HTTP, 테스트, Git을 실제 개발 흐름으로 연결하는 능력
- 기능 구현 전에 시스템 경계와 검증 기준을 먼저 세우는 습관
- 학습 내용을 다음 구현 판단에 재사용할 수 있는 문서로 바꾸는 능력

## 폴더 구조

```text
obsidian
├─ chapters
│  ├─ chapter-01
│  │  └─ 01-spring-boot-and-http.md
│  └─ chapter-02
│     ├─ 00-overview.md
│     ├─ 01-docker-postgresql-postgis.md
│     └─ 02-jpa-hibernate-jdbc.md
├─ decisions
│  ├─ ADR-0001-backend-bootstrap-and-health-check.md
│  ├─ ADR-0002-postgis-and-persistent-local-database.md
│  └─ ADR-0003-spring-datasource-and-jpa-validation.md
├─ review
│  └─ development-record-checklist.md
└─ questions
   ├─ chapter-01-review-questions.md
   └─ chapter-02-review-questions.md
```

## 작성 원칙

- Chapter는 하나의 학습과 구현 목표로 묶고, 하위 파일은 한 번에 학습하고 구현 및 검증할 수 있는 주제로 나눈다.
- 파일명에는 Session 번호를 넣지 않고 `숫자-주제.md` 형식을 사용한다.
- 하나의 파일은 하나의 개념 묶음을 이해하고, 관련 코드를 구현하고, 하나의 검증 결과를 얻는 범위로 작성한다.
- 각 학습 파일은 `목표`, `개념`, `프로젝트 연결`, `구현`, `검증`, `해석`, `문제 해결`, `내 언어로 설명`, `다음 단위`가 드러나야 한다.
- 의사결정은 `선택`, `이유`, `대안`, `검증 방법`을 남긴다.
- 모든 기록은 나중에 README, 포트폴리오, 기술 설명으로 재사용 가능해야 한다.

## 복습 방법

- `chapters`에서 해당 챕터에서 무엇을 구현하고 어떻게 검증했는지 다시 본다.
- `questions`의 답을 가린 채 소리 내어 설명하고, 막히는 항목만 다시 학습한다.
- `decisions`에서는 선택만이 아니라 대안과 선택 비용까지 설명한다.
- `review/development-record-checklist.md`에서는 구현한 사실과 아직 보강해야 할 증거를 구분한다.
- 문서의 명령은 외우기보다 각 명령이 무엇을 보장하고 무엇을 보장하지 않는지 설명한다.
