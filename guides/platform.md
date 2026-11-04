# 플랫폼 가이드

데이터 파이프라인, 분석 모델, K8s·GitOps, IaC, 관측 작업 직전에 관련 절만 읽는다. 각 절은 작업별 선택 기준이다. 한 작업에서 전부 검사하거나 한 프로젝트에 모든 기술을 들이지 않는다. 공통 규칙은 [AGENTS.md](../AGENTS.md), 코드 수준 기준은 [development.md](development.md), 리뷰 절차는 [review.md](review.md)에 있다.

## 공통

- 대상 선택을 먼저 확인한다: 환경, context, namespace, account, region, workspace, state backend, DB·카탈로그.
- 확인 수준을 구분해 보고한다: 문법 통과 → schema 통과 → 원격 dry-run·plan → 반영(sync·rollout·apply·실행) 완료 → 실제 기능·데이터 확인.
- 원격에 접속하거나 상태·메타데이터를 바꾸는 명령은 로컬 검사로 취급하지 않는다. 현재 환경과 사용자 권한 범위 안에서만 실행한다.
- 렌더 결과, plan, state, 로그, 쿼리 결과에 비밀·개인정보가 있을 수 있다. 출력·커밋 전에 확인한다.
- 이름으로 테이블·컬럼·인덱스·자원·지표를 추측하지 않는다. 카탈로그·API·실제 출력에서 확인한다.

## 데이터 플랫폼

### 흐름과 계약

- 입력 → 처리 → 저장 → 소비의 흐름을 추적하고 단계마다 스키마·타입·시간·상태 계약을 확인한다.
- 시간 값의 의미(event time, 적재 시각, 처리 시각), 시간대, 경계 포함 여부(`>=`와 `<`)를 확인한다.

### 배치

- 처리 구간(한 번에 처리하는 시간 범위), 시간대, upstream 의존성을 확인한다. Airflow를 쓴다면 run의 data interval과 실제 쿼리 조건이 일치하는지 본다.
- 중복 실행, 동시 실행, 부분 실패 때의 결과를 확인한다. 같은 구간을 다시 실행해도 같은 결과가 나오는지(멱등성)는 쓰기 방식(파티션 덮어쓰기, merge, append)에 달려 있다.
- backfill·재시도가 다른 구간과 downstream에 주는 영향을 확인한다.

### 스트리밍

- event time과 처리 시각, 순서가 보장되는 범위, 지연 데이터의 허용·처리 정책을 확인한다.
- offset·checkpoint를 언제 커밋하는지, sink 쓰기와 어떤 순서인지 확인한다. 이 순서가 중복(at-least-once)이나 유실을 정한다.
- backpressure와 consumer lag, 재시작·재배치 때의 동작을 확인한다.

### Hive·Impala 등 조회 엔진

- 카탈로그의 실제 스키마·파티션·저장 형식·위치를 확인한다(예: `DESCRIBE FORMATTED`, `SHOW PARTITIONS`).
- 통계 유무와 실행계획을 확인한다(예: Impala의 `SHOW TABLE STATS`·`SHOW COLUMN STATS`, `EXPLAIN`). 파티션 pruning이 실제로 되는지 계획에서 본다.
- `COMPUTE STATS`, `REFRESH`, `INVALIDATE METADATA`, `MSCK REPAIR TABLE`은 메타데이터를 바꾸고 부하를 줄 수 있다. 읽기 검사로 취급하지 않는다.
- 파일이 써진 것과 엔진이 그 파일을 보는 것은 별개다. 메타데이터가 갱신되는 경로를 확인한다.

### 완료 판단

- 다음을 각각 확인한다: 작업 성공, 데이터 중복·누락 없음, 데이터 최신성, 소비자가 실제로 쓸 수 있는 상태.
- 작은 로컬 입력으로 운영 규모의 성능을 보증하지 않는다. 성능 주장은 실제 규모의 실행계획·측정에 근거한다.

## 분석 엔지니어링

분석 SQL과 모델은 버전·계약·시험이 있는 소프트웨어로 다룬다.

### 모델 계약

- 먼저 정한다: 한 행의 의미(grain), 고유키·복합키, 시간 기준, null·중복 정책.
- 원천·정제·제공 모델의 책임을 나누고 lineage와 소비자를 확인한다. 계층 이름을 맞추려고 불필요한 모델을 만들지 않는다.
- 컬럼 이름·타입·의미·grain을 바꾸면 소비자 호환성을 확인한다.

### 조인과 집계

- 조인 전에 양쪽 키의 고유성을 확인하고 cardinality(1:1, 1:N, N:M)를 정한다. fan-out은 조인 전후의 행 수와 키 distinct 수로 확인한다.
- inner join으로 빠지는 행, left join 뒤 WHERE 조건 때문에 inner join처럼 작동하는 경우를 본다.
- 집계 수준이 grain과 맞는지, 필터가 분자·분모 중 어디에 적용되는지, 지표의 분모·분자·단위를 확인한다.

### 증분 모델

- watermark 기준 컬럼과 그 의미, 늦게 도착한 데이터·정정·삭제가 반영되는 방식을 확인한다.
- 처리 구간이 겹칠 때 중복이 생기지 않는지, 재실행과 full refresh의 결과가 같은지 확인한다.

### 품질과 비교

- 데이터 품질(고유성, not null, 허용값, 참조 무결성)과 freshness를 소비자 요구에 연결해 시험한다.
- 행 수만 비교하지 않는다. 키 기준으로 누락·중복·값 차이를 확인한다.
  - 중복: `GROUP BY <key> HAVING COUNT(*) > 1`
  - 누락·값 차이: 두 결과를 키로 `FULL OUTER JOIN`해서 한쪽에만 있는 키와 값이 다른 키를 센다.
  - 필요하면 구간(날짜·파티션)별로 나눠 본다.
- 예상값은 요구·계약·독립 계산에서 얻는다. 검사 대상 모델의 SQL을 복사해 기대값을 만들지 않는다.

### 도구

- dbt 등은 프로젝트에서 실제로 채택한 경우에만 그 도구의 model·test·snapshot 등을 쓴다. 채택 여부와 버전은 프로젝트 README에서 확인한다.
- `dbt build`·`run`·`test`·`seed`·`snapshot` 같은 명령은 DB 읽기·쓰기를 포함할 수 있다. 다른 명령도 adapter에 따라 DB 연결을 쓸 수 있으므로 로컬 lint처럼 취급하지 않고 대상 target을 확인한다.

## K8s·GitOps

### 배포 경로

- 코드 commit → build → image tag/digest → 배포 선언 → controller 반영 → 실제 서비스 동작을 연결해 추적한다.
- tag는 다른 이미지를 가리키도록 바뀔 수 있다. 실제로 돌고 있는 이미지는 Pod status의 `imageID`(digest)로 확인한다.
- 대상 context와 namespace를 명시한다(`kubectl config current-context`, `-n <namespace>`).

### 렌더와 API

- chart·kustomize의 렌더 결과를 본다(`helm template`, `kubectl kustomize` 또는 `kustomize build`). 배포되는 것은 values가 아니라 렌더 결과다.
- 대상 클러스터의 API 버전과 CRD 존재를 확인한다.
- immutable field(예: Deployment의 `spec.selector`, Job의 `spec.template`, StatefulSet의 `volumeClaimTemplates`)를 바꾸면 갱신이 거부되거나 재생성이 필요하다.
- label/selector와 Service·port(`port`, `targetPort`, 포트 이름), Ingress·DNS·NetworkPolicy가 실제로 연결되는지 본다. Service의 endpoint가 비어 있지 않은지 확인한다.

### 동작과 수명

- probe: readiness·liveness·startup의 역할을 구분한다. 시작이 느린 앱에 liveness가 엄격하면 재시작 루프가 생긴다.
- rollout과 종료: `maxSurge`·`maxUnavailable`, SIGTERM 처리, `preStop`, `terminationGracePeriodSeconds`, endpoint 제거와 프로세스 종료의 순서.
- PDB와 drain·rollout의 상호작용, scheduling 조건(affinity, taint·toleration, topology spread).
- resource request는 scheduling을, limit은 CPU throttling과 OOMKilled를 결정한다. JVM은 heap 설정과 컨테이너 메모리 limit의 관계를 확인한다.
- PVC·StorageClass: reclaim policy, volume binding mode, 자원 삭제 때 데이터가 남는지. 데이터를 지울 수 있는 변경은 보존 방법을 먼저 확인한다.

### 권한과 관리 주체

- RBAC·service account의 실제 권한을 확인한다(`kubectl auth can-i <verb> <resource> --as=system:serviceaccount:<ns>:<sa>`). security context(runAsUser, readOnlyRootFilesystem, capabilities)도 본다.
- Secret 참조는 이름과 key가 실제로 있는지 확인한다. env로 주입한 값은 Secret이 바뀌어도 Pod 재시작 전까지 반영되지 않는다.
- 자원의 실제 관리 주체(GitOps controller, Helm release, 수동 kubectl, 파일 provisioning)를 `metadata.managedFields`와 관리 label·annotation으로 확인한다. 한 자원을 둘 이상이 관리하지 않게 한다. self-heal이 켜진 GitOps 자원을 수동으로 고치면 되돌려진다.

### 확인 수준

- YAML 문법 통과, schema 통과, server dry-run 통과(`kubectl apply --dry-run=server`, `kubectl diff`), sync·rollout 완료, 실제 기능 성공을 구분한다.
- server dry-run과 `kubectl diff`는 API 서버에 접속하고 admission webhook을 거칠 수 있다.
- GitOps 도구의 동기화 상태와 건강 상태는 별개다(예: Argo CD의 Synced와 Healthy).

## 인프라·IaC

- 대상 선택: account·subscription·project, region, workspace, state backend를 확인한다.
- plan에서 추가·변경·교체(삭제 후 생성)·삭제를 구분한다. 교체·삭제되는 자원의 데이터, 의존 자원, 다운타임을 확인한다.
- drift: 코드와 실제 자원의 차이가 누구의 변경인지 확인하고, 덮어쓸지 코드에 반영할지 정한다.
- 의존성과 적용 순서, 비용 변화, 복구 방법(백업, 스냅샷, 재생성 절차)을 확인한다. 모든 변경에 자동 rollback이 있지는 않다.
- Terraform 기준으로 `init`은 provider·module을 내려받고 backend를 설정하며, `plan`은 원격 API를 읽고 state lock을 잡을 수 있다. 다른 IaC 도구도 채택한 도구 기준으로 원격 접근 여부를 확인한다. 로컬 검사로 취급하지 않는다.
- plan 출력과 state에는 비밀이 있을 수 있다.
- 네트워크·DNS·TLS·스토리지 변경은 코드 쪽 연결(접속 주소, 인증서 신뢰, 이름 해석, mount 경로와 권한)까지 따라가 확인한다.

## 관측·진단

### 신호 구분

- collector 성공, 프로세스 생존, endpoint 성공(예: HTTP 200), 실제 처리 결과를 구분한다. 앞의 것이 뒤의 것을 보장하지 않는다.
- 결측·stale·UNKNOWN을 정상이나 장애로 자동 해석하지 않는다. 마지막 성공 시각을 확인한다.

### 지표

- 단위(초·밀리초, 바이트), 지표 종류(counter·gauge·histogram), 계산 시간창을 확인한다. rate를 계산하는 창은 scrape 간격보다 충분히 길어야 한다.
- label cardinality가 저장과 쿼리 비용을 키우지 않는지 본다. 사용자 ID·요청 ID 같은 값을 label로 쓰지 않는다.
- scrape 지연과 수집 누락을 고려한다.

### 대시보드

- datasource, 쿼리, 변환(transformation), 표시 의미(단위, reduce 방식, null 처리, 패널별 시간 범위, 변수 값)를 함께 검증한다.
- mock·샘플 데이터와 실제 telemetry를 구분한다. 화면에 값이 나왔다고 실제 데이터라고 결론 내리지 않는다.

### 계측의 영향과 조정

- 계측이 본래의 응답·상태·지연·메모리·권한을 바꾸는지 확인한다(예: 응답 감싸기, 동기 전송, 메트릭 endpoint 노출).
- worker 수·메모리·timeout을 조정하기 전에 측정값과 원인 가설을 확보한다. 조정 뒤 같은 지표로 효과를 확인한다.
