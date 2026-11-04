# Engineering Workspace

개발, 코드 구조 검토, 코드 리뷰, 데이터 플랫폼, Kubernetes, 인프라 작업을 위한 저장소다.

## 시작

1. 작업 목표와 대상 프로젝트를 정한다.
2. 공통 지침은 [AGENTS.md](AGENTS.md)를 따른다. 작업별 가이드는 그 안의 표에서 고른다.
3. Claude Code는 저장소 루트에서 시작한다. 하위 디렉터리에서 시작하면 `CLAUDE.md`의 `@AGENTS.md` import가 빠질 수 있고 `.claude/settings.json`이 적용되지 않는다.

| 파일 | 역할 |
|---|---|
| `AGENTS.md` | 공통 지침의 정본과 작업별 가이드 선택. 대부분의 코딩 에이전트가 자동으로 읽는다 |
| `CLAUDE.md` | `@AGENTS.md` import와 Claude Code 전용 사항 |
| `guides/` | [개발](guides/development.md)·[리뷰](guides/review.md)·[플랫폼](guides/platform.md) 판단 기준. 해당 작업에서만 읽는다 |
| `.claude/skills/` | Claude Code용 외부 skill(dbt·Airflow·K8s·Gradle·Kotlin JPA). 출처·commit·수정 내역은 [skills/README](.claude/skills/README.md) |
| `.claude/settings.json` | Claude Code의 비밀 파일 읽기 deny와 attribution 끄기. 루트에서 시작한 로컬 세션과 claude.ai Project 밖의 단일 저장소 cloud 세션에만 적용되는 보조 장치이고 접근 통제를 대신하지 않는다 |

## 프로젝트

실제 작업이 생기면 `projects/<project-name>/`을 만들고 언어·도구의 기존 디렉터리 관용을 사용한다. 프로젝트마다 다음 파일을 둔다.

| 파일 | 내용 |
|---|---|
| `README.md` | 사람용 개요: 목적과 경계, 구조. 명령은 `AGENTS.md`를 가리킨다 |
| `AGENTS.md` | 에이전트용 운영 정보: 환경, 검증 명령, 실행·변경 절차, 저장소만 읽어서는 알기 어려운 제약 |
| `CLAUDE.md` | `@AGENTS.md` 한 줄. 루트 `CLAUDE.md`가 있으면 Claude Code는 하위 `AGENTS.md`를 이 import로만 읽는다 |
| `NOTES.md` | 세션을 넘기는 작업에만 둔다. 항목은 루트 `AGENTS.md`를 따른다 |

프로젝트 `AGENTS.md`에는 실제로 실행할 수 있는 명령과 확인한 사실만 적는다. 명령을 모르면 placeholder를 검증 명령이라고 적지 않는다.

- 환경: 언어·런타임·도구 버전, 비밀 없는 재현 조건
- 검증: 빠른 대상 검사와 전체 검사, lint·타입·빌드, 변경을 관측하는 방법(로컬 실행·endpoint·로그·render·plan), 원격 시스템에 접근하는 명령과 필요한 권한, CI가 강제하는 검사
- 실행·변경: 필요할 때만 대상 환경 선택, 권한 범위, 적용·복구 절차
- 제약: 느린 테스트나 환경 특이점처럼 매번 다시 알아내야 하는 사실. 루트 규칙은 반복하지 않고, 예외는 범위를 적는다

현재 프로젝트가 없으면 공통 검증 명령도 없다.

별도 저장소의 코드를 다룰 때는 그 저장소를 직접 대상으로 작업하고 복제된 소스 사본을 이곳에 쌓지 않는다. 그 저장소의 규칙·검증 명령·권한을 따른다.

## 검증과 기록

검증은 프로젝트가 사용하는 빌드·린트·테스트·렌더링·정적 분석·CI로 수행한다. 공통 검사 프레임워크나 자동 hook은 두지 않는다.

작은 작업은 변경과 검증 결과를 보고한다. 세션을 넘기는 작업만 해당 프로젝트의 `NOTES.md`에 기록한다.

## 지침 관리

- 규칙은 같은 실수가 반복될 때 추가하고, 계기가 된 실패를 커밋 메시지에 적는다. 도구로 잡을 수 있는 실수는 프로젝트의 linter·테스트·CI로 막는다.
- 모델·도구가 바뀌면 각 줄이 아직 필요한지 다시 읽고, 지워도 실수가 늘지 않는 줄은 지운다. 비밀·권한·git·원격 변경 규칙과 AI 생성 표시 금지는 이 기준으로 지우지 않는다.
- 실제 스택이 생기면 `.gitignore`와 `.claude/settings.json`의 deny 목록을 함께 보강한다(예: state·plan 파일, kubeconfig).
- Claude Code에서는 시작할 때 로드된 지침을 `/context`로(필요할 때 로드되는 하위 `CLAUDE.md`는 목록에 없다), 오래된 경로·충돌하는 지시를 `/doctor prompt-audit`(2.1.283 이상, 모델 호출 사용)로 점검할 수 있다. 결과는 판단이므로 그대로 반영하지 않는다.
