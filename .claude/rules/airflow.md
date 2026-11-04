---
paths:
  - "**/dags/**/*.py"
---
# Airflow DAG 규칙

- 먼저 프로젝트의 Airflow 버전을 확인한다. 3.x는 `airflow.sdk`(`dag`, `task`, `DAG`, `Asset`)에서 import하고 `schedule`을 쓴다(`schedule_interval`·`execution_date`는 없다). 2.x 코드를 3.x로 옮기는 작업은 `migrating-airflow-2-to-3` skill을 쓴다.
- DAG 파일의 최상위 코드는 파싱할 때마다 실행된다. DB·API 호출, 무거운 import·계산은 task 안으로 옮긴다.
- task는 같은 실행을 다시 돌려도 결과가 같아야 하고, 처리 구간은 현재 시각이 아니라 실행 context에서 얻는다. 3.x는 cron 문자열 `schedule`에 기본으로 `CronTriggerTimetable`을 써서 `data_interval_start`와 `data_interval_end`가 같고 `{{ ds }}`는 구간 시작일이 아니라 실행일이다. 2.x처럼 구간이 필요하면 `CronDataIntervalTimetable`이나 `[scheduler] create_cron_data_intervals` 설정을 먼저 확인한다. 수동·asset 실행에서는 `logical_date`가 None일 수 있다.
- 큰 데이터는 XCom으로 넘기지 않고 저장 위치(경로·테이블 이름)를 넘긴다.
- task 코드에서 metadata DB에 직접 접근하지 않는다(3.x에서는 실패한다). 필요한 값은 context나 REST API로 얻는다.
- 연결 정보·비밀값은 코드에 쓰지 않고 Connection·Variable·secrets backend로 받으며, connection 내용을 출력하지 않는다.
- 검사 루프: DAG import 오류는 DagBag import 테스트로 먼저 확인한다(`python <파일>`은 파일 끝의 `dag.test()`까지 실행할 수 있다). `airflow dags test`와 `dag.test()`는 task를 실제로 실행해 connection이 가리키는 시스템과 metadata DB에 쓰므로, 로컬·테스트용 connection과 DB임을 확인했을 때만 돌린다. 실제 환경의 trigger·backfill·clear는 명시적 요청이 있을 때만 한다.
