---
paths:
  - "**/dags/**/*.py"
---
# Airflow DAG 규칙

- 먼저 프로젝트의 Airflow 버전을 확인한다. 3.x는 `airflow.sdk`(`dag`, `task`, `DAG`, `Asset`)에서 import하고 `schedule`을 쓴다(`schedule_interval`·`execution_date`는 없다). 2.x 코드를 3.x로 옮기는 작업은 `migrating-airflow-2-to-3` skill을 쓴다.
- DAG 파일의 최상위 코드는 파싱할 때마다 실행된다. DB·API 호출, 무거운 import·계산은 task 안으로 옮긴다.
- task는 같은 data interval로 다시 실행해도 결과가 같아야 한다. 처리 구간은 현재 시각이 아니라 `data_interval_start`·`data_interval_end`(또는 `{{ ds }}` 같은 템플릿)에서 얻는다.
- 큰 데이터는 XCom으로 넘기지 않고 저장 위치(경로·테이블 이름)를 넘긴다.
- task 코드에서 metadata DB에 직접 접근하지 않는다(3.x에서는 실패한다). 필요한 값은 context나 REST API로 얻는다.
- 연결 정보·비밀값은 코드에 쓰지 않고 Connection·Variable·secrets backend로 받으며, connection 내용을 출력하지 않는다.
- 검사 루프: DAG 파일 import 오류를 먼저 확인하고(`python <파일>` 또는 DagBag import 테스트), `airflow dags test <dag_id> <날짜>`로 로컬에서 실행한다. 실제 환경의 trigger·backfill·clear는 명시적 요청이 있을 때만 한다.
