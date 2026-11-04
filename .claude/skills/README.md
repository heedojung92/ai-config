# 가져온 skill

Claude Code가 이 저장소에서 쓰는 skill이다. 각 폴더는 아래 upstream commit을 그대로 복사했고, 표시한 것만 이 저장소 규칙에 맞게 고쳤다. 폴더의 `LICENSE`는 upstream 원본이다. 2026-11-04에 가져왔다.

| skill | 출처 | commit | 라이선스 | 수정 |
|---|---|---|---|---|
| `using-dbt-for-analytics-engineering` | dbt-labs/dbt-agent-skills `skills/dbt/skills/` | `b8facde20b7a92c40ce3e6b86b71c4b92aa44040` | Apache-2.0 | 있음 |
| `running-dbt-commands` | 같은 저장소 | 같음 | Apache-2.0 | 있음 |
| `working-with-dbt-mesh` | 같은 저장소 | 같음 | Apache-2.0 | 없음 |
| `migrating-airflow-2-to-3` | astronomer/agents `skills/` | `35b9610bb2710c37ca11326c4cbf7818b6725fec` | Apache-2.0 | 있음 |
| `kubernetes-skill` | LukasNiessen/kubernetes-skill (루트의 `SKILL.md`·`references/`) | `34f93c1e409156958cc7c410ab1aa88ab512f10f` | MIT | 없음 |
| `gradle-best-practices` | gradle/gradle-skills `skills/` | `b26b14955e3a3ada5d1e90a4e08e2f10b2b54b5d` | Apache-2.0 | 있음 |
| `gradle-wrapper-upgrade` | 같은 저장소 | 같음 | Apache-2.0 | 없음 |
| `kotlin-backend-jpa-entity-mapping` | Kotlin/kotlin-agent-skills `skills/` | `087f5689c1934135ed58718cdf524525e452b7e9` | Apache-2.0 | 없음 |

## 수정 내역

- `using-dbt-for-analytics-engineering`: 모든 dbt 명령을 사전 승인하는 `allowed-tools`를 지웠다. dbt MCP 서버를 설치·설정하도록 권하는 줄을 "이미 설정돼 있으면 쓰고, 요청이 없으면 설치하지 않는다"로 바꿨다.
- `running-dbt-commands`: 쓰기 전에 target을 확인하고 production target과 `--full-refresh`는 명시적 요청이 있을 때만 쓰는 항목을 추가했다.
- `migrating-airflow-2-to-3`: Edit마다 실행되는 PostToolUse hook을 지웠다. `ruff --fix --unsafe-fixes .`를 바로 돌리던 단계를 검사 먼저, 요청 범위의 `--fix`, 사용자 동의 후 `--unsafe-fixes` 순서로 바꿨다. 버전 권고에 현재 릴리스를 확인하라는 말을 붙였다.
- `gradle-best-practices`: 답을 받을 수 없을 때 확인 없이 수정을 적용하라는 절을 "요청하지 않은 변경은 적용하지 않는다"로 바꿨다. 무인 실행에서 구조 변경을 계획만 적고 진행하던 문구를 지웠다.

## 알려진 한계

- 모두 영어다. 한국어 요청에서 덜 불릴 수 있어 `CLAUDE.md`에 작업별 사용 표를 둔다.
- dbt·Airflow skill 본문은 이 저장소에 없는 다른 skill(`answering-natural-language-questions-with-dbt`, `testing-dags` 등)을 가리키는 곳이 있다. 없는 skill은 건너뛴다.
- `kubernetes-skill`은 개인 유지 저장소이고, kubeconform 예시 버전이 1.29~1.30이다. 대상 클러스터 버전으로 바꿔 쓴다. schema 예시가 외부 호스트(datreeio CRDs-catalog)를 가리킨다.
- `kotlin-backend-jpa-entity-mapping`의 `hashCode() = javaClass.hashCode()`는 Hibernate proxy 클래스와 실제 클래스에서 값이 다를 수 있다. proxy와 섞어 `Set`에 넣는 코드는 따로 확인한다.
- `gradle-wrapper-upgrade`는 `services.gradle.org`·`downloads.gradle.org`에 접속하고 배포본(100MB 이상)을 내려받는다. 네트워크가 막힌 환경에서는 명령 안내만 가능하다.
- skill 지시가 `AGENTS.md`·`guides/`와 다르면 이 저장소 규칙을 따른다.

## 추가·갱신 기준

- 추가는 공식 또는 유지보수자가 관리하고, 허용형 라이선스이며, 실제로 반복되는 작업에 쓰는 skill만 한다. 비슷한 skill이 많으면 서로 경쟁해 덜 불린다.
- 갱신할 때는 upstream 새 commit의 diff를 읽고 `allowed-tools`·hook·script·외부 호출·git·삭제 명령 변화를 확인한 뒤, 위 수정을 다시 적용하고 표의 commit을 바꾼다.
- 넣지 않은 것: Databricks·Snowflake skill(독점 라이선스), Confluent·Monte Carlo·Lenses(특정 유료 서비스나 넓은 도구 사전 승인 필요), Astronomer `authoring-dags`(실행 중인 Airflow API와 `af` CLI 필요, connection extra를 그대로 출력할 수 있음), Kotlin/kotlin-backend-agent-skills(버전 정보 없는 일반 조언 25개), HashiCorp agent-skills(MPL-2.0). Iceberg·Delta·Spark·Trino·medallion은 조건에 맞는 skill을 찾지 못했다.
- 도구를 실제로 쓰게 되면 검토할 후보: fluxcd/agent-skills `gitops-knowledge`(Flux), dbt-labs `adding-dbt-unit-test`, Astronomer `cosmos-dbt-core`(Airflow에서 dbt 실행), datacontract/datacontract-skills(ODCS 데이터 계약).
