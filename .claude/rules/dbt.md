---
paths:
  - "**/dbt_project.yml"
  - "**/models/**/*.sql"
  - "**/models/**/*.yml"
  - "**/macros/**/*.sql"
  - "**/snapshots/**/*.sql"
  - "**/snapshots/**/*.yml"
  - "**/seeds/**/*.csv"
  - "**/seeds/**/*.yml"
---
# dbt 작업 규칙

- 검사 루프: 바꾼 모델을 `dbt build --select <model>`로 빌드·테스트한다. 하위 모델까지 볼 때는 `dbt list --select <model>+`로 범위를 확인한 뒤 `<model>+`로 넓힌다. production manifest가 있으면 상위 모델을 다시 만들지 않고 `--defer --state <경로>`를 쓴다.
- 결과는 행 원문보다 행 수·키 중복·null 수 같은 집계로 확인한다(예: `dbt show --inline "select count(*) from {{ ref('<model>') }}"`). 운영 데이터에서 온 행은 사용자가 허용했을 때만 출력하고, dbt skill의 `dbt show` 미리보기에도 이 규칙을 적용한다.
- target은 dev로 둔다. production target 실행, `--full-refresh`, snapshot 재생성은 명시적 요청이 있을 때만 한다.
- 테이블은 `ref()`·`source()`로만 가리키고 database·schema 이름을 하드코딩하지 않는다.
- incremental 모델은 `incremental_strategy`부터 확인한다. `merge`·`delete+insert`는 `is_incremental()` 필터와 `unique_key`를, `insert_overwrite`는 덮어쓸 파티션을 정하고 늦게 도착하는 데이터 범위(lookback)를 필터에 넣는다. `append`는 `unique_key`를 쓰지 않아 재실행하면 중복된다. `microbatch`(dbt 1.9 이상)는 dbt가 `event_time`으로 필터를 만들므로 필터 대신 `lookback` 설정을 쓴다. `on_schema_change`의 기본값 `ignore`는 새 컬럼을 대상 테이블에 넣지 않는다.
- 모델 설명에 grain(한 행이 뜻하는 것)을 적는다. grain을 바꾸면 기본키 `unique`·`not_null` 테스트로 확인한다.
- 컬럼 이름·타입 변경과 삭제는 소비자를 깰 수 있다. 하위 모델·exposure·외부 소비자를 먼저 찾고, 같은 변경에서 고칠 수 없는 소비자가 있으면 contract나 model version으로 다룬다. 판단에는 `working-with-dbt-mesh` skill을 쓴다.
- 새 모델을 만들기 전에 기존 중간 모델에 컬럼을 더하는 쪽을 먼저 검토한다.
- 실패한 데이터 테스트는 데이터 문제인지 로직 문제인지 먼저 가린다. 테스트를 지우거나 기준을 낮춰 통과시키지 않는다.
