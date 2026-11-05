# 가져온 skill

Claude Code가 이 저장소에서 쓰는 skill이다. 아래 표의 폴더는 upstream commit을 그대로 복사했고, 표시한 것만 이 저장소 규칙에 맞게 고쳤다. 폴더의 `LICENSE`는 upstream 저장소 루트의 원본이다(upstream skill 폴더에는 없다). 2026-11-04에 가져왔다.

`handoff`·`retro`는 이 저장소에서 만든 skill이다. `disable-model-invocation: true`라 사용자가 `/handoff`·`/retro`로 부를 때만 실행된다.

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

- `using-dbt-for-analytics-engineering`: `dbt`·`jq` 명령과 Read·Write·Edit·Glob·Grep을 사전 승인하는 `allowed-tools`를 지웠다. dbt MCP 서버를 설치·설정하도록 권하는 줄을 "이미 설정돼 있으면 쓰고, 요청이 없으면 설치하지 않는다"로 바꿨다. `dbt show`로 데이터를 보라는 절에 운영 데이터의 행은 집계로만 보고 기록 파일에 넣지 않는다는 문단을 더했다.
- `running-dbt-commands`: 쓰기 전에 target을 확인하고 production target과 `--full-refresh`는 명시적 요청이 있을 때만 쓰며 `profiles.yml`을 출력하지 않는 항목(5)과, 운영 데이터 행을 출력하지 않는 항목(6)을 추가했다. `--full-refresh` 예시 주석에 같은 제한을 적었다.
- `migrating-airflow-2-to-3`: Edit마다 실행되는 PostToolUse hook을 지웠다. `ruff --fix --unsafe-fixes .`를 바로 돌리던 단계(요약과 명령 블록)를 요청한 경로(`<paths>`)에 대해 검사 먼저, `--fix`, 사용자 동의 후 `--unsafe-fixes` 순서로 바꿨다. 버전 권고의 `3.0.11`이 Astro Runtime 3.0-11임을 밝히고(Apache Airflow 3.0.x는 3.0.6까지) 현재 릴리스를 확인하라는 말을 붙였다. cron data interval 설정 이름을 `AIRFLOW__SCHEDULER__CREATE_CRON_DATA_INTERVALS`로 바로잡고(`SKILL.md`, `reference/migration-patterns.md`. upstream 표기는 `..._INTERVAL`·`AIRFLOW__CORE__...`로 틀려 있었다), 이 설정이 배포 전체에 적용되므로 DAG 하나는 timetable을 명시하라는 말을 더했다.
- `gradle-best-practices`: 답을 받을 수 없을 때 확인 없이 수정을 적용하라는 절을 "요청하지 않은 변경은 적용하지 않는다"로 바꿨다. 무인 실행에서 구조 변경을 계획만 적고 진행하던 문구를 지웠다.

## 알려진 한계

- 가져온 skill은 모두 영어다. 한국어 요청에서 덜 불릴 수 있어 `CLAUDE.md`에 작업별 사용 표를 둔다.
- dbt·Airflow skill 본문은 이 저장소에 없는 다른 skill(`adding-dbt-unit-test`, `answering-natural-language-questions-with-dbt`, `building-dbt-semantic-layer`, `testing-dags`, `debugging-dags`, `deploying-airflow`)을 가리킨다. 없는 skill은 건너뛴다.
- `kubernetes-skill`은 개인 유지 저장소이고, kubeconform 예시 버전이 1.29~1.30이다. 대상 클러스터 버전을 `x.y.z` 형식으로 바꿔 쓴다. schema 예시가 외부 호스트(datreeio CRDs-catalog의 `main` branch)를 가리킨다.
- 고치지 않고 둔 줄 중 이 저장소 규칙과 다른 것: `kubernetes-skill`의 클러스터에 쓰는 확인 예시(`kubectl get secrets -A -o json | kubectl replace -f -`, `kubectl run`, `rollout undo`, PVC `patch`, `flux reconcile`, `helm test`), 버전·checksum 없이 `releases/latest`를 내려받고 Actions를 tag로 고정한 CI 예시, 평문 비밀번호가 든 kustomize `secretGenerator` 예시, `working-with-dbt-mesh`의 unsuffixed relation을 지우는 프로젝트 전체 post-hook(`drop_relation_if_exists`)과 production deployment 실행 안내, `running-dbt-commands`의 `--select` 없는 `dbt run` 예시, `discovering-data.md`의 `SELECT *` 표본, Airflow `removed-methods.md`의 `airflow_db` connection 예시, Gradle fix 문서의 "commit it". `AGENTS.md`의 원격 변경·비밀·git 규칙이 우선한다.
- `kotlin-backend-jpa-entity-mapping`의 `hashCode() = javaClass.hashCode()`는 Hibernate proxy 클래스와 실제 클래스에서 값이 다를 수 있다. proxy와 섞어 `Set`에 넣는 코드는 따로 확인한다.
- `gradle-wrapper-upgrade`는 `services.gradle.org`·`downloads.gradle.org`에 접속하고 배포본(100MB 이상)을 내려받는다. 네트워크가 막힌 환경에서는 명령 안내만 가능하다.
- skill 지시가 `AGENTS.md`·`guides/`와 다르면 `AGENTS.md`·`guides/`를, `.claude/rules/`와 다르면 규칙을 따른다.

## 추가·갱신 기준

- 추가는 공식 또는 유지보수자가 관리하고, 허용형 라이선스이며, 실제로 반복되는 작업에 쓰는 skill만 한다. 비슷한 skill이 많으면 서로 경쟁해 덜 불린다.
- 갱신할 때는 upstream 새 commit의 diff를 읽고 frontmatter(`allowed-tools`·`hooks`·`context`·`agent`·`model`·`disable-model-invocation`·`user-invocable`), `!` 셸 명령, script·외부 호출·git·삭제 명령 변화를 확인한다. 위 수정을 다시 적용하고 수정한 파일마다 안내 줄을 둔다. 표의 commit, 안내 줄의 commit, 가져온 날짜를 함께 바꾸고, `LICENSE`는 upstream 저장소 루트에서 다시 복사하며 NOTICE 파일이 생겼는지 본다. 그 skill을 가리키는 `CLAUDE.md` 표와 `.claude/rules/`의 줄도 다시 확인한다.
- `/verify`·`/run-skill-generator`가 기록한 skill도 같은 기준으로 검토하고, 남기면 이 문서에 적는다.
- 넣지 않은 것: Databricks·Snowflake skill(독점 라이선스), Confluent·Monte Carlo·Lenses(특정 유료 서비스나 넓은 도구 사전 승인 필요), Astronomer `authoring-dags`(실행 중인 Airflow API와 `af` CLI 필요, connection extra를 그대로 출력할 수 있음), Kotlin/kotlin-backend-agent-skills(버전 정보 없는 일반 조언 25개), HashiCorp agent-skills(MPL-2.0). Iceberg·Delta·Spark·Trino·medallion은 조건에 맞는 skill을 찾지 못했다.
- 도구를 실제로 쓰게 되면 검토할 후보: fluxcd/agent-skills `gitops-knowledge`(Flux), dbt-labs `adding-dbt-unit-test`, Astronomer `cosmos-dbt-core`(Airflow에서 dbt 실행), datacontract/datacontract-skills(ODCS 데이터 계약).
