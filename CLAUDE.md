@AGENTS.md

## Claude Code 전용

- 저장소 루트에서 시작한다. 하위 디렉터리에서 시작하면 첫 줄의 import가 외부 import로 처리돼 빠질 수 있고 루트 `.claude/settings.json`도 적용되지 않는다. 저장소 루트 `AGENTS.md`의 내용이 컨텍스트에 없으면 프로젝트 `AGENTS.md`가 로드돼 있어도 작업 전에 루트 파일을 읽는다.
- 루트 `.claude/settings.json`에는 비밀 파일 읽기 deny와 commit·PR attribution 끄기만 둔다. deny는 목록에 없는 파일, 디렉터리 단위 재귀 검색(`grep -r` 등), 스크립트 안의 파일 읽기를 막지 못하므로 `AGENTS.md`의 비밀값 규칙을 그대로 따른다. hook·응답 재생성·전역 설정 변경은 전제하지 않는다.
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
