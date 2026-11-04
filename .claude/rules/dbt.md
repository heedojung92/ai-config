---
paths:
  - "**/dbt_project.yml"
  - "**/models/**/*.sql"
  - "**/models/**/*.yml"
  - "**/macros/**/*.sql"
  - "**/snapshots/**"
  - "**/seeds/**"
---
# dbt 작업 규칙

- 검사 루프: 바꾼 모델만 `dbt build --select <model>+`로 빌드·테스트하고 결과는 `dbt show --select <model> --limit 50`으로 본다. production manifest가 있으면 상위 모델을 다시 만들지 않고 `--defer --state <경로>`를 쓴다.
- target은 dev로 둔다. production target 실행, `--full-refresh`, snapshot 재생성은 명시적 요청이 있을 때만 한다.
- 테이블은 `ref()`·`source()`로만 가리키고 database·schema 이름을 하드코딩하지 않는다.
- incremental 모델은 `is_incremental()` 필터와 `unique_key`를 함께 정하고, 늦게 도착하는 데이터 범위(lookback)를 필터에 넣는다. `on_schema_change`의 기본값 `ignore`는 새 컬럼을 대상 테이블에 넣지 않는다.
- 모델 설명에 grain(한 행이 뜻하는 것)을 적는다. grain을 바꾸면 기본키 `unique`·`not_null` 테스트로 확인한다.
- 컬럼 이름·타입 변경과 삭제는 소비자를 깨는 변경이다. contract나 model version으로 다루고 `working-with-dbt-mesh` skill을 쓴다.
- 새 모델을 만들기 전에 기존 중간 모델에 컬럼을 더하는 쪽을 먼저 검토한다.
- 실패한 데이터 테스트는 데이터 문제인지 로직 문제인지 먼저 가린다. 테스트를 지우거나 기준을 낮춰 통과시키지 않는다.
