---
paths:
  - "**/dbt_project.yml"
  - "**/models/**/*.sql"
  - "**/models/**/*.yml"
  - "**/models/**/*.yaml"
  - "**/macros/**/*.sql"
  - "**/snapshots/**/*.sql"
  - "**/snapshots/**/*.yml"
  - "**/seeds/**/*.csv"
  - "**/seeds/**/*.yml"
  - "**/tests/**/*.sql"
  - "**/analyses/**/*.sql"
---
# dbt 작업 규칙

- 검증 루프: 바꾼 모델을 `dbt build --select <model>`로 빌드·테스트한다. incremental 모델은 한 번 더 빌드해 증분 경로(`is_incremental()`)도 확인한다. 하위 모델까지 볼 때는 `dbt list --select <model>+`로 범위를 확인한 뒤 `<model>+`로 넓힌다. production manifest가 있으면 상위 모델을 다시 만들지 않고 `--defer --state <경로>`를 쓴다.
- 결과는 행 수·키 중복·null 수 같은 집계로 확인한다(예: `dbt show --inline "select count(*) from {{ ref('<model>') }}"`). 운영 데이터에서 온 행 원문은 출력하지 않는다(`AGENTS.md`). dbt skill의 `dbt show` 미리보기도 집계로 대신하고, skill이 만드는 `data_discovery.md` 같은 기록에도 행 원문을 넣지 않는다.
- target은 dev로 두고, 기본 target이 불분명하면 `--target`을 명시한다. production target으로 쓰는 실행, `--full-refresh`, snapshot 재생성은 명시적 요청이 있을 때만 한다.
- 테이블은 `ref()`·`source()`로만 가리키고 database·schema 이름을 하드코딩하지 않는다.
- incremental 모델은 `incremental_strategy`와 adapter부터 확인한다. `merge`·`delete+insert`는 `is_incremental()` 필터와 `unique_key`를 정한다. `insert_overwrite`가 덮어쓰는 범위는 adapter와 설정에 따라 테이블 전체일 수 있다. Snowflake는 언제나 테이블 전체를 바꾸고, Spark·Databricks는 `partition_by`와 dynamic overwrite 설정(Databricks SQL warehouse의 `use_replace_on_for_insert_overwrite`, Spark ODBC cluster의 `partitionOverwriteMode`)이 있을 때만 파티션 단위로 바꾼다. compile된 SQL(`target/run/`)로 범위를 확인하고, 파티션 단위일 때만 덮어쓸 파티션과 늦게 도착하는 데이터 범위(lookback)를 필터에 넣는다. 테이블 전체를 바꾸는 모델에 증분 필터를 넣으면 필터 밖의 행이 사라진다. `append`는 `unique_key`를 쓰지 않아 같은 구간을 다시 처리하면 중복된다. `microbatch`(dbt 1.9 이상)는 `event_time`·`begin`·`batch_size`가 필요하고 `event_time`을 정한 상위 모델만 dbt가 걸러 읽으므로, 필터 대신 `lookback` 설정을 쓴다. `on_schema_change`의 기본값 `ignore`는 새 컬럼을 대상 테이블에 넣지 않는다.
- 모델 설명에 grain(한 행이 뜻하는 것)을 적는다. grain을 바꾸면 기본키 `unique`·`not_null` 테스트로 확인한다.
- 컬럼 이름·타입 변경과 삭제는 소비자를 깰 수 있다. 하위 모델·exposure·외부 소비자를 먼저 찾고, 같은 변경에서 고칠 수 없는 소비자가 있으면 model version으로 이전 기간을 둔다. contract는 깨지는 변경을 빌드에서 잡을 뿐 이전 기간을 대신하지 않는다. 판단에는 `working-with-dbt-mesh` skill을 쓴다.
- 새 모델을 만들기 전에 기존 중간 모델에 컬럼을 더하는 쪽을 먼저 검토한다.
- 실패한 데이터 테스트는 데이터 문제인지 로직 문제인지 먼저 가린다. 테스트를 지우거나 기준을 낮춰 통과시키지 않는다.
