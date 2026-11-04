@AGENTS.md

## Claude Code 전용

- 저장소 루트에서 시작한다. 하위 디렉터리에서 시작하면 첫 줄의 import가 외부 import로 처리돼 빠질 수 있고 루트 `.claude/settings.json`도 적용되지 않는다. 저장소 루트 `AGENTS.md`의 내용이 컨텍스트에 없으면 프로젝트 `AGENTS.md`가 로드돼 있어도 작업 전에 루트 파일을 읽는다.
- 루트 `.claude/settings.json`에는 비밀 파일 읽기 deny와 commit·PR attribution 끄기만 둔다. deny는 목록에 없는 파일, 디렉터리 단위 재귀 검색(`grep -r` 등), 스크립트 안의 파일 읽기를 막지 못하므로 `AGENTS.md`의 비밀값 규칙을 그대로 따른다. hook·응답 재생성·전역 설정 변경은 전제하지 않는다.
- `AGENTS.md`의 검증 루프를 Claude Code 기능으로 잇는다.
  - 여러 파일·공개 API·스키마를 바꾸는 작업은 바꿀 파일과 검사 명령을 담은 계획부터 세운다.
  - 크거나 위험한 변경을 마치면 `reviewer` 하위 에이전트에 요구·변경 파일·기준 ref를 주고 한 번 검토받는다. 작성 맥락을 모르는 리뷰어라 독립 리뷰가 된다.
  - 실제 실행 결과가 중요한 변경(서비스·앱·DAG 실행)은 사용자에게 `/verify`나 `/run`을 권한다.
  - 오래 걸리거나 지켜보지 않는 작업을 맡으면 사용자가 쓸 `/goal` 조건을 제안한다. 조건은 종료 상태 하나, 그것을 보여 줄 명령, 바꾸면 안 되는 것, 턴 상한으로 쓴다. 평가 모델은 명령을 실행하지 않고 대화에 나온 출력만 본다.
  - 세션을 넘기기 전에는 `/handoff`를, 사용자가 같은 것을 다시 바로잡은 뒤에는 `/retro`를 권한다.
- dbt·Airflow·Kotlin·K8s 파일을 읽거나 고치면 `.claude/rules/`의 해당 규칙이 자동으로 로드된다. 그 규칙도 `AGENTS.md`보다 앞서지 않는다.
- 아래 작업에는 `.claude/skills/`의 해당 skill을 먼저 쓴다. skill 지시가 `AGENTS.md`·가이드와 다르면(쓰기 대상, 승인, git, 비밀값) 이 저장소 규칙을 따른다. 출처와 수정 내역은 `.claude/skills/README.md`에 있다.

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
