---
paths:
  - "**/k8s/**"
  - "**/kubernetes/**"
  - "**/manifests/**"
  - "**/charts/**"
  - "**/helm/**"
  - "**/kustomize/**"
  - "**/overlays/**"
  - "**/kustomization.yaml"
  - "**/Chart.yaml"
  - "**/values*.yaml"
---
# Kubernetes manifest 규칙

- 검사 루프: `helm lint`와 `helm template` 또는 `kustomize build` 결과를 대상 클러스터 버전의 schema로 검사한다(예: `kubeconform -strict -kubernetes-version <버전>`). CRD schema가 없어 건너뛴 리소스는 보고한다. 클러스터에 닿는 `kubectl diff`·server dry-run은 그다음 단계이고, 적용은 명시적 요청이 있을 때만 한다.
- 대상 클러스터 버전과 context·namespace를 먼저 확인하고, 그 버전이 제공하는 apiVersion만 쓴다.
- Deployment·StatefulSet의 `spec.selector`는 만든 뒤 바꿀 수 없다. selector 라벨을 바꾸면 재생성이 필요하다고 보고한다.
- 이미지는 `latest`가 아닌 고정 tag나 digest로 지정한다.
- 컨테이너마다 CPU·메모리 requests를 둔다. 메모리 limit을 넘으면 OOMKilled되고 CPU limit은 throttling을 일으키므로, limit은 그 영향을 보고 팀 정책에 맞춰 정한다.
- readinessProbe는 트래픽을 받을 수 있는지, livenessProbe는 프로세스가 고장 났는지만 본다. liveness가 DB 같은 외부 의존성에 걸리면 연쇄 재시작이 난다.
- 기본 securityContext는 `runAsNonRoot: true`, `allowPrivilegeEscalation: false`, `capabilities.drop: ["ALL"]`, `seccompProfile.type: RuntimeDefault`이다. 풀어야 하면 이유를 적는다.
- Secret 값은 manifest·values 파일에 커밋하지 않는다. base64는 암호화가 아니다. 프로젝트의 비밀 관리 방식(External Secrets·Sealed Secrets 등)을 따른다.
- manifest 작성·리뷰의 세부 절차는 `kubernetes-skill`을 쓴다.
