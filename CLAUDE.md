@AGENTS.md

## Claude Code 전용

- 저장소 루트에서 시작한다. 하위 디렉터리에서 시작하면 첫 줄의 import가 외부 import로 처리돼 빠질 수 있고 루트 `.claude/settings.json`도 적용되지 않는다. 저장소 루트 `AGENTS.md`의 내용이 컨텍스트에 없으면 프로젝트 `AGENTS.md`가 로드돼 있어도 작업 전에 루트 파일을 읽는다.
- 루트 `.claude/settings.json`에는 비밀 파일 읽기 deny, commit·PR attribution 끄기, Bash 작업 디렉터리 고정(상대 경로 deny가 `cd`를 따라 옮겨 가지 않게)만 둔다. deny는 목록에 없는 파일, 디렉터리 단위 재귀 검색(`grep -r` 등), 스크립트 안의 파일 읽기를 막지 못하므로 `AGENTS.md`의 비밀값 규칙을 그대로 따른다. hook·응답 재생성·전역 설정 변경은 전제하지 않는다.
- `AGENTS.md`의 검증 루프를 Claude Code 기능으로 잇는다.
  - 여러 모듈이나 공개 API·스키마를 바꾸는 작업은 바꿀 파일과 검사 명령을 먼저 적고 시작한다. 그보다 작은 작업은 `AGENTS.md`대로 바로 한다.
  - 크거나 위험한 변경에 두 번째 검토가 필요하면 `reviewer` 하위 에이전트에 요구·기준 ref·적용할 제약과 `git diff <ref>` 결과를 넘겨 한 번 맡긴다. 추적하지 않는 새 파일(`git status --short`의 `??`)은 diff에 나오지 않으므로 내용을 따로 넘긴다. 비밀이 담길 수 있는 파일은 내용 대신 경로만 알리고, 작성자의 추론은 넘기지 않는다.
  - 실제 실행 결과가 중요한 변경(서비스·앱)은 `/run`으로 실행해 보거나 사용자에게 `/verify`를 권한다. 외부 시스템에 쓰는 실행(DAG task, 배포 등)은 `AGENTS.md`의 권한 규칙을 따른다.
  - 오래 걸리거나 지켜보지 않는 작업을 맡으면 사용자가 쓸 `/goal` 조건을 제안한다. 조건은 종료 상태 하나, 그것을 보여 줄 명령, 바꾸면 안 되는 것, 멈출 조건(턴 상한, 권한·정보가 필요할 때)으로 쓴다. 평가 모델은 명령을 실행하지 않고 대화에 나온 출력만 본다. `/goal`은 권한 모드를 바꾸지 않으므로 지켜보지 않는 실행에는 auto mode가 필요하다.
  - 세션을 넘기기 전에는 `/handoff`를, 사용자가 같은 것을 다시 바로잡은 뒤에는 `/retro`를 권한다.
- dbt·Airflow·Kotlin·K8s 파일을 읽거나 고치면 경로가 맞는 `.claude/rules/`의 규칙이 그 뒤에 붙는다. 새 파일을 만들 때나 rules·skills·`reviewer`가 로드되지 않는 환경(claude.ai Project의 cloud thread 등)에서는 해당 `.claude/rules/*.md`와 `SKILL.md`를 먼저 직접 읽는다.
- 아래 작업에는 `.claude/skills/`의 해당 skill을 먼저 쓴다. 가져온 skill이나 규칙의 지시가 `AGENTS.md`·가이드와 다르면 `AGENTS.md`·가이드를 따르고, skill과 규칙이 다르면 규칙을 따른다. 출처와 수정 내역은 `.claude/skills/README.md`에 있다.

| 작업 | skill |
|---|---|
| dbt 모델·SQL·테스트 작성과 수정, 변경 영향 확인 | `using-dbt-for-analytics-engineering` |
| dbt 명령 실행(build·show·list·selector·defer) | `running-dbt-commands` |
| dbt 컬럼 이름·타입 변경과 삭제, model version·contract·access | `working-with-dbt-mesh` |
| Airflow 2→3 이전, Airflow 3 import·API 변경 확인 | `migrating-airflow-2-to-3` |
| K8s manifest·Helm·Kustomize·RBAC 작성과 리뷰 | `kubernetes-skill` |
| Gradle 빌드 점검·개선 | `gradle-best-practices` |
| Gradle wrapper 버전 올리기 | `gradle-wrapper-upgrade` |
| Kotlin JPA 엔티티, equals/hashCode, N+1 | `kotlin-backend-jpa-entity-mapping` |
