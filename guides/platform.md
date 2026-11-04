# 데이터 플랫폼·K8s·인프라

공통 원칙과 해당 작업에 관련된 절만 읽는다. 아래 목록은 영향 분석의 후보이며 매 작업마다 전부 실행하는 체크리스트가 아니다.

## 공통

- 대상 환경·버전·리소스·설정과 검증할 결과를 고정한다. 문서상 설정, 원하는 상태, 실제 실행 상태를 구분한다.
- 변경 전후의 의도된 차이, 유지할 계약, 영향받는 소비자, 실패 시 대응을 정한다.
- 로컬 정적 검증과 실제 환경 확인을 나눈다. target environment에서 실행하지 않았다면 그 사실을 밝힌다.
- 원격 명령·API·배포는 현재 도구·환경·사용자 권한 범위에서 수행한다. manifest·plan 파일의 존재가 실행 승인이나 적용 성공을 뜻하지 않는다.
- 조회는 읽기 전용 신원이 있으면 그 신원으로 시작한다(K8s `view` role, 읽기 전용 IAM, plan용 읽기 전용 자격 증명). HCP Terraform의 plan 권한은 보안상 write와 같다. 역할 이름을 믿지 말고 실제 권한을 확인한다(`kubectl auth can-i --list` 등). 도구의 read-only 옵션·확인 프롬프트는 사고를 줄이는 장치이고, 권한 경계는 자격 증명의 범위다.
- 실행 전에 context·namespace·account·region·workspace 등 해당 도구의 대상 선택을 확인한다. 이 확인을 반복 승인 의식으로 만들지 않는다.
- 비밀·연결정보·민감한 상태가 render·plan·log·state에 나올 수 있으므로 출력·공유 범위를 통제한다. 마스킹을 끄는 옵션(`kubectl diff --show-secrets`, OpenTofu `-show-sensitive` 등)은 쓰지 않는다.

## 데이터 플랫폼

| 변경 영역 | 확인할 계약·실패 형태 |
|---|---|
| 스키마·형식 | 타입·null·기본값·필드 의미·버전 호환·생산자와 소비자의 해석 |
| 배치·스케줄 | 의존성·처리 구간·시간대·겹침·부분 실패·재실행 |
| 스트리밍 | event time·처리 시각·지연 데이터·순서·offset/checkpoint·backpressure |
| 재처리·backfill | 중복·멱등성·기존 상태와 병합·대상 구간·다른 실행과의 간섭 |
| 저장·조회 | 파티션·파일 형식·predicate pushdown·인덱스·작은 파일·실행 비용 |
| 장애·관측 | retry·격리·부분 결과·알림·freshness·진단 가능한 상태 |

- 인공 또는 비식별 입력으로 정상·경계·중복·실패·재실행을 필요한 범위에서 확인한다.
- 재시도 성공과 데이터 중복·누락 방지는 다른 계약이다. exactly-once 같은 보장은 producer·processor·sink 전체 범위를 명시한다.
- 덮어쓰기 범위를 먼저 확인한다. Spark의 기본 static partition overwrite는 partition spec에 맞는 기존 파티션을 새 데이터에 없는 것까지 모두 지운 뒤 쓰고(spec이 없으면 테이블·경로 전체), dbt `--full-refresh`는 incremental 모델을 처음부터 다시 만든다.
- backfill은 대상 구간·재처리 정책·동시 실행 수를 정하고, 도구가 dry run을 지원하면(예: Airflow 3 `airflow backfill create --dry-run`) 생성될 실행을 먼저 본다. dry run 목록은 실제 재처리 결과의 증명이 아니다.
- 비용이 큰 조회는 가능하면 dry run의 처리량 추정과 조회별 상한(예: on-demand 과금의 BigQuery `maximumBytesBilled`)으로 먼저 막는다. 추정이 없거나 정확한 값이 아니면(BigQuery `totalBytesProcessedAccuracy`가 `PRECISE`가 아닌 경우) 모르는 값으로 다룬다.
- 작은 로컬 입력의 성공을 실제 데이터량·분산 실행·운영 성능의 증거로 쓰지 않는다.
- 스키마 변경은 소비자 호환성을, 상태 변경은 복구·재처리 가능성을 확인한다.
- 스케줄 등록·DAG 파싱·작업 성공·출력 데이터 정합을 구분한다.

## Kubernetes

| 변경 영역 | 확인할 계약·실패 형태 |
|---|---|
| 렌더링·API | 실제 배포 버전의 chart/kustomize 결과·API schema·CRD·immutable field |
| 연결 | label/selector·Service/port·Ingress·DNS·NetworkPolicy·외부 의존성 |
| 실행·가용성 | rollout·probe·종료 처리·replica·PDB·scheduling·resource request/limit |
| 상태·저장 | PVC·StorageClass·접근 모드·mount·재시작·데이터 보존 |
| 권한·설정 | service account·RBAC·security context·Secret/ConfigMap 참조 |
| 실제 상태 | controller reconcile·events·logs·실제 endpoint·소비자 동작 |

- YAML 문법 통과, schema 검증, API server 수락, rollout 완료, 실제 서비스 동작을 구분한다.
- Secret이 포함될 수 있는 렌더링 결과를 무심코 출력하거나 commit하지 않는다. `kubectl diff`의 Secret 마스킹은 v1 Secret의 `data`만 가리므로 Secret 관련 diff는 key 이름과 변경 여부로만 요약한다.
- CRD schema가 없으면 해당 검증이 빠졌다고 표시한다. 전체 검증 성공으로 포장하지 않는다.
- server dry-run과 `kubectl diff`는 실제 쓰기와 같은 권한이 필요하고 admission webhook에 전체 객체를 보낸다. 단순 로컬 lint와 같은 실행으로 취급하지 않는다. dry-run 실패가 항상 manifest 문제는 아니다(dry run을 지원하지 않는 webhook 등).
- `kubectl diff`의 종료 코드 1은 차이가 있다는 뜻이고, 1보다 크면 오류다.
- Argo CD·Flux가 관리하는 리소스는 Git이 원하는 상태다. 클러스터를 직접 바꾸면 drift가 되고 자동 동기화가 되돌릴 수 있다. 변경은 Git으로 하고 sync·health·실제 상태로 확인하며, 삭제 반영(prune) 설정을 확인한다. 자동 동기화 대상 branch로의 push·merge는 배포이므로 배포 요청이 있을 때만 한다.
- 복구는 이전 manifest 적용만으로 끝나는지 확인한다. 데이터·외부 자원 변경은 별도 고려한다.

## 인프라·IaC

| 변경 영역 | 확인할 계약·실패 형태 |
|---|---|
| 대상·상태 | 실제 account/project·region·workspace·state backend·drift |
| 계획 | 추가·수정·교체·삭제 범위·의존성·예상 밖 리소스·비용 영향 |
| 네트워크 | 경로·DNS·TLS·방화벽·라우팅·접근 주체 |
| 권한 | 실행 주체·IAM·최소 권한·비밀 취급 |
| 자원·상태 | 용량·장애 도메인·스토리지·백업·복구·유지보수 영향 |
| 적용·관측 | 실제 적용 결과·연결·서비스 계약·실패 후 상태 |

- formatter, 정적 validate, plan, apply, 실제 기능 관측을 구분한다.
- plan·init도 backend·provider·원격 API에 접근할 수 있다. 도구·옵션별 효과를 확인한다. 검토하지 않은 구성(검토 전 브랜치, 새로 들이거나 버전을 바꾼 module·provider)의 init·plan은 현재 자격 증명으로 그 코드를 실행하는 것과 같다. provider·`external` data source·backend 설정이 명령을 실행할 수 있으므로 먼저 검토하거나 자격 증명이 없는 환경에서 확인한다. `init -backend=false`도 provider·module은 내려받고, 이어지는 validate·plan은 그 provider를 실행한다.
- saved plan 파일·state·`show -json`·`output -json`·`output -raw`는 sensitive 값을 평문으로 담는다(OpenTofu의 state·plan 암호화는 파일만 보호한다). 사람이 읽는 출력은 sensitive로 표시된 값만 가린다. 이 산출물의 원문은 출력하지 않고, plan 출력도 그대로 일반 보고서·Git에 넣지 않는다.
- `apply <planfile>`과 `-auto-approve`는 확인 없이 적용하고, 삭제도 포함된다. saved plan 적용도 대화형 apply처럼 사용자의 명시적 승인이 있을 때만 한다. `destroy`·`-destroy`·`-target`·`-replace`·`import`·state 변경(rm·mv·push 등)·`force-unlock`은 사용자가 명시적으로 요청했을 때만 쓴다.
- `plan -detailed-exitcode`의 2는 변경이 있다는 뜻이고(output 변경·move·import 포함) 실패가 아니다.
- 모든 변경에 자동 rollback이 있다고 가정하지 않는다. 되돌릴 수 없는 변경이면 백업·복구·forward fix 조건을 밝힌다.
- 적용은 명시된 범위에서만 수행한다. 이 가이드는 원격 실행이나 변경 권한을 새로 부여하지 않는다.

## 검증 도구 선택

프로젝트가 이미 사용하는 도구를 먼저 쓴다. 언어·플랫폼과 관계없는 공통 검사 엔진을 새로 만들지 않는다. 프로젝트가 정책 검사(Checkov·Trivy·Conftest·Kyverno CLI 등)를 쓰면 검증에 포함하고 실행한 정책과 skip·soft-fail을 밝힌다. 실제 실행한 명령, 대상, 결과, 남은 경계를 프로젝트의 보고 또는 필요한 기술 기록에 남긴다.
