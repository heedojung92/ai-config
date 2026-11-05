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
| `.claude/skills/` | Claude Code skill: 외부 skill(dbt·Airflow·K8s·Gradle·Kotlin JPA)과 이 저장소의 `/handoff`·`/retro`. 출처·commit·수정 내역은 [skills/README](.claude/skills/README.md) |
| `.claude/rules/` | dbt·Airflow·Kotlin·K8s 파일을 읽거나 고칠 때 Claude Code가 자동으로 불러오는 도메인 규칙 |
| `.claude/agents/reviewer.md` | 파일을 고칠 수 없는 독립 리뷰용 하위 에이전트 |
| `.claude/settings.json` | Claude Code의 비밀 파일 읽기 deny(같은 경로의 Edit·Write도 막으므로 그 파일은 사용자가 고친다), attribution 끄기, Bash 작업 디렉터리 고정. 루트에서 시작한 로컬 세션과 claude.ai Project 밖의 단일 저장소 cloud 세션에만 적용되는 보조 장치이고 접근 통제를 대신하지 않는다 |

claude.ai Project의 cloud thread는 루트 `CLAUDE.md`만 읽고 경로별 rules·skills·agents·settings는 쓰지 않는다(이 저장소의 rules는 모두 경로별이다).

## 프로젝트

실제 작업이 생기면 `projects/<project-name>/`을 만들고 언어·도구의 기존 디렉터리 관용을 사용한다. 프로젝트마다 다음 파일을 둔다.

| 파일 | 내용 |
|---|---|
| `README.md` | 사람용 개요: 목적과 경계, 구조. 명령은 `AGENTS.md`를 가리킨다 |
| `AGENTS.md` | 에이전트용 운영 정보: 환경, 검증 명령, 실행·변경 절차, 저장소만 읽어서는 알기 어려운 제약 |
| `CLAUDE.md` | `@AGENTS.md` 한 줄. 루트 `CLAUDE.md`가 있으면 Claude Code는 기본 설정에서 하위 `AGENTS.md`를 이 import로만 읽는다 |
| `NOTES.md` | 세션을 넘기는 작업에만 둔다. 항목은 루트 `AGENTS.md`를 따른다 |

프로젝트 `AGENTS.md`에는 실제로 실행할 수 있는 명령과 확인한 사실만 적는다. 명령을 모르면 placeholder를 검증 명령이라고 적지 않는다.

- 환경: 언어·런타임·도구 버전, 비밀 없는 재현 조건
- 검증: 빠른 대상 검사와 전체 검사, lint·타입·빌드, 변경을 관측하는 방법(로컬 실행·endpoint·로그·render·plan), 원격 시스템에 접근하는 명령과 필요한 권한, CI가 강제하는 검사
- 실행·변경: 필요할 때만 대상 환경 선택, 권한 범위, 적용·복구 절차
- 제약: 느린 테스트나 환경 특이점처럼 매번 다시 알아내야 하는 사실. 루트 규칙은 반복하지 않고, 예외는 범위를 적는다. 비밀·권한·git·원격 변경 규칙은 완화하지 않는다

현재 프로젝트가 없으면 공통 검증 명령도 없다. 새 스택에서 비밀이 담길 수 있는 파일 이름이 생기면 `AGENTS.md`의 목록, `.gitignore`, `.claude/settings.json` deny를 함께 보강한다.

별도 저장소의 코드를 다룰 때는 그 저장소를 직접 대상으로 작업하고 복제된 소스 사본을 이곳에 쌓지 않는다. 그 저장소의 규칙·검증 명령·권한을 따른다. 이 저장소의 경로별 rules와 `/` 기준 deny는 이 저장소 경로에만 걸리므로, 별도 저장소에서 같은 규칙이 필요하면 그 저장소에 둔다.

## 검증과 기록

검증은 프로젝트가 사용하는 빌드·린트·테스트·렌더링·정적 분석·CI로 수행한다. 공통 검사 프레임워크나 자동 hook은 두지 않는다.

작은 작업은 변경과 검증 결과를 보고한다. 세션을 넘기는 작업만 해당 프로젝트의 `NOTES.md`에 기록한다.

## 루프와 규칙

에이전트가 스스로 확인하며 끝내려면 결과가 통과·실패로 나오는 검사가 있어야 한다. 기본 루프는 `AGENTS.md`의 검증 루프이고, 확인 강도는 작업에 맞게 고른다. `/goal`·Stop hook·`/code-review`·`/loop`·`/verify`는 Claude Code 기능이고, 이 저장소는 hook을 설치하지 않는다.

| 강도 | 방법 | 쓰는 때 |
|---|---|---|
| 지시 | 요청에 완료 조건과 검사 명령을 함께 쓴다 | 대부분의 작업 |
| `/goal` | 매 턴이 끝날 때 별도 모델이 대화만 보고 조건을 판정하고, 충족되거나 불가능하다고 판정될 때까지 작업을 이어 간다 | 오래 걸리거나 지켜보지 않는 작업 |
| Stop hook | 검사가 실패하면 끝내지 못하게 막는다(종료 코드 2). 도구 호출 없이 8번 연속 막히면 Claude Code가 턴을 끝내고, 도구 호출이 끼면 횟수가 다시 시작된다 | 예외 없이 지켜야 하는 검사. 기본으로 두지 않고 빠른 검사가 있는 프로젝트에서 고른다 |
| 독립 리뷰 | `/code-review` 또는 `reviewer` 하위 에이전트. 맡기기 전에 diff에 비밀 파일이 없는지 본다 | 크거나 위험한 변경을 끝낸 뒤 |

Stop hook을 쓰려면 별도 저장소는 그 저장소의 `.claude/settings.json`에, 이 저장소의 프로젝트는 개인 `.claude/settings.local.json`에 둔다. 루트의 local hook은 모든 프로젝트 작업에서 실행되므로 바뀐 경로가 그 프로젝트일 때만 검사하게 쓴다. `projects/<name>/.claude/`의 설정은 `/cd`로 그 디렉터리로 옮긴 세션에만 적용되고, 옮기면 루트 `.claude/settings.json` 대신 그 디렉터리 설정이 쓰일 수 있다.

`/goal` 조건은 종료 상태 하나, 그것을 보여 줄 명령, 바꾸면 안 되는 것, 멈출 조건(턴 상한, 권한·정보가 필요할 때)으로 쓴다. 평가 모델은 명령을 직접 실행하지 않고 대화에 나온 출력만 보므로, 바꾸면 안 되는 것도 Claude가 실행해 보여 줄 수 있게 쓴다. `/goal`은 권한 모드를 바꾸지 않으므로 지켜보지 않고 돌리려면 auto mode에서 쓴다.

```text
/goal ./gradlew :order:test --tests 'com.example.order.*'가 exit 0으로 끝나고, git diff --stat <시작 commit>에 기존 테스트 파일과 빌드 파일이 없고, git push는 하지 않았다. 30턴이 지나거나 권한·정보가 필요하면 멈추고 남은 실패와 필요한 것을 보고한다
```

지켜보지 않고 맡기기 전에는 완료 조건, 스스로 할 검사, 건드려도 되는 범위(브랜치·worktree·권한), 멈출 조건(턴·비용 상한), 결과가 갈 곳과 승인할 사람을 정한다. CI·배포처럼 시간이 지나야 바뀌는 상태는 `/loop <확인할 일>`로 확인한다. 간격을 주지 않으면 Claude가 일이 끝났을 때 멈추고, `/loop 15m <확인할 일>`처럼 간격을 주면 취소를 요청할 때까지 최대 7일 돈다. 같은 것을 두 번 고쳐도 틀리면 필요할 때 `/retro`로 바로잡은 것을 먼저 남기고, `/clear` 후 알게 된 것을 담아 다시 요청한다.

`/verify`는 실행 방법을 저장소 루트(monorepo면 바꾼 패키지)의 `.claude/skills/verify/SKILL.md`에 기록할 수 있다. 루트에 기록된 `verify` skill은 다음 세션부터 모든 프로젝트의 commit 전에 실행하라는 지시를 붙이고, 하위 디렉터리의 기록은 루트에서 시작한 세션에 그 지시를 붙이지 않는다. `/run-skill-generator`는 env 값을 포함한 실행 방법을 `.claude/skills/run-<name>/`에 기록하고 바로 commit하므로 push 전에 검토하고, 비밀값이 들어갔으면 commit을 고친다. 기록된 파일은 비밀값·연결 정보·`allowed-tools`·hook·`!` 명령이 없는지 검토한다.

| 명령 | 하는 일 |
|---|---|
| `/handoff` | 세션을 넘기기 전에 프로젝트 `NOTES.md`를 갱신한다. 오래 걸리거나 지켜보지 않을 작업이면 다음 행동에 `/goal` 조건 후보를 넣는다 |
| `/retro` | 이 세션에서 바로잡은 것을 모아 규칙 개선안을 만든다. 고른 안만 반영한다 |

규칙은 지켜야 하는 강도와 범위에 따라 놓는다.

| 위치 | 담는 것 | 로드 |
|---|---|---|
| `AGENTS.md` | 모든 작업의 행동 규칙과 검증 루프 | 항상 |
| `projects/<name>/AGENTS.md` | 그 프로젝트의 명령·환경·제약 | 그 프로젝트를 다룰 때 |
| `.claude/rules/*.md` | 도메인 파일의 함정 | 경로가 맞는 파일을 읽거나 고칠 때(Explore·Plan 하위 에이전트 포함) |
| `.claude/skills/` | 여러 단계 절차 | 필요할 때나 사용자가 부를 때 |
| `.claude/settings.json`, hook | 도구가 막아야 하는 것(비밀 파일 읽기 deny 등) | 루트에서 시작한 세션에서 Claude Code가 적용한다. deny는 우회될 수 있고 hook은 기본으로 두지 않는다 |

## 지침 관리

- 규칙은 같은 실수가 반복되거나 사용자가 요청할 때 추가하고, 계기를 커밋 메시지에 적는다. 도구로 잡을 수 있는 실수는 프로젝트의 linter·테스트·CI로 막는다.
- 모델·도구가 바뀌면 각 줄이 아직 필요한지 다시 읽고, 지워도 실수가 늘지 않는 줄은 지운다. 비밀·권한·git·원격 변경 규칙과 AI 생성 표시 금지는 이 기준으로 지우지 않는다.
- Claude Code에서는 시작할 때 로드된 지침을 `/context`로(필요할 때 로드되는 하위 `CLAUDE.md`는 목록에 없다), 오래된 경로·충돌하는 지시를 `/doctor prompt-audit`(2.1.283 이상, 모델 호출 사용)로 점검할 수 있다. 결과는 판단이므로 그대로 반영하지 않는다.
