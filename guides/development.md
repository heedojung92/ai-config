# 개발 가이드

구현·디버깅·리팩터링·설계 직전에 필요한 절을 읽는다. 공통 규칙은 [AGENTS.md](../AGENTS.md), 리뷰 절차는 [review.md](review.md), 데이터·플랫폼 기준은 [platform.md](platform.md)에 있다.

## 구조와 설계

- 먼저 현재 구조를 읽는다: 진입점, 모듈·패키지 경계, 의존성 방향, 설정이 주입되는 경로, 테스트 위치.
- 모듈은 기능·계약·변경 이유로 나눈다. 함께 바뀌는 코드는 함께 두고, 다른 이유로 바뀌는 코드는 분리한다.
- 의존성은 안정된 계약 쪽을 향하게 한다. 순환 의존, 하위 모듈이 상위 구현을 참조하는 경우를 확인한다.
- 구현이 하나뿐이고 교체 계획이 없는 interface·impl·adapter·plugin 계층을 만들지 않는다. 시험 대역이 필요하면 그 경계에서만 추상화한다.
- 공개 API·스키마·설정 키·기본값은 계약이다. 바꾸기 전에 소비자와 호환성(구버전 클라이언트, 롤링 배포 중 버전 혼재, 이미 저장된 데이터)을 확인한다.
- 설계안은 현실적인 대안의 계약·영향·운영·유지보수 비용을 비교해 고르고, 고른 이유를 남긴다.

## 구현

- 목표에 필요한 변경만 한다. 근처 코드 정리, 포맷 일괄 변경, 의존성 교체를 섞지 않는다.
- 프로젝트의 기존 관용(언어 버전, 스타일, 에러 처리, 로깅, 설정 방식)을 따른다.
- 오류를 숨기지 않는다. 기본값으로 대체하거나 예외를 삼키기 전에 그 결과를 누가 소비하는지 확인한다.
- 실패 경로(timeout, 부분 실패, 재시도, 중복 호출, 취소)를 정상 경로와 함께 설계한다.
- 새 의존성은 기존 의존성·표준 라이브러리로 안 될 때만 추가하고 버전·라이선스·전이 의존을 확인한다.

## 진단

- 증상을 재현 가능한 입력·환경·명령으로 고정한다. 재현되지 않으면 시도한 조건과 확인 못 한 부분을 남긴다.
- 관측을 먼저 모은다: 오류 원문, 스택, 로그 시각, 입력, 버전, 설정, 최근 변경.
- 원인 후보를 둘 이상 세우고, 각 후보가 맞을 때만 나타나는 관측을 정해 가른다.
- 증상이 보이는 곳에서 멈추지 않고 값이 생성·변경되는 곳까지 거슬러 올라간다.
- 수정 후 같은 입력으로 증상이 사라졌는지 확인한다. 사용자가 "여전히 같다"고 하면 진단의 전제부터 다시 본다.

## 검증

- 수락 조건은 요구·계약에서 정한다. 구현 코드를 복사해 기대값을 만들지 않는다.
- 결함 수정은 가능하면 실패하는 시험을 먼저 만들고, 수정 전 실패와 수정 후 통과를 확인한다.
- 프로젝트에 있는 빌드·테스트·정적 분석 도구를 먼저 쓴다. 검증만을 위해 새 프레임워크·도구를 들여오지 않는다.
- 시험 결과에서 실행 건수, skip, 대상 필터를 확인한다. 0건 실행은 통과가 아니다.
- mock 통과는 그 mock이 흉내 낸 계약까지만 증명한다. 실제 DB·HTTP·클러스터와의 계약은 따로 확인하거나 미확인으로 남긴다.
- 확인한 범위(변경본, 입력, 환경)를 결론에 적는다. 작은 로컬 입력의 결과로 운영 규모의 성능·동시성을 보증하지 않는다.

## Kotlin·JVM

### 버전과 toolchain

- Kotlin, JDK, Gradle wrapper, Gradle 플러그인, 프레임워크의 버전을 프로젝트 파일에서 확인한다(`gradle/wrapper/gradle-wrapper.properties`, 버전 카탈로그, `toolchain`·`jvmTarget` 설정).
- 호환성은 각 도구의 공식 호환 표와 실제 빌드 결과로 확인한다. 과거에 쓰던 버전이나 임의의 최신 버전을 기본값으로 고정하지 않는다.
- 빌드는 프로젝트의 `./gradlew`로 한다. `./gradlew -version`, `./gradlew javaToolchains`로 실제 쓰이는 Gradle·JVM을 확인한다.

### 모듈과 가시성

- 모듈 간 의존 방향을 `settings.gradle(.kts)`와 각 모듈의 `dependencies`에서 확인한다. `api`와 `implementation`은 의존을 소비자에게 노출하는지가 다르다.
- 모듈 밖에 노출할 필요가 없는 선언은 `internal`·`private`로 둔다. Java 소비자에게 `internal`은 bytecode상 public으로 보인다.

### null, 예외, 직렬화, 상태, 동등성

- `!!`, `?: 기본값`, `lateinit`으로 데이터 오류를 숨기지 않는다. null이 가능한 이유와 처리 책임을 계약에 연결한다.
- Java에서 온 값은 platform type이다. 경계에서 nullability를 확정하고(검사, 어노테이션) 그 근거를 남긴다.
- Kotlin에는 checked exception이 없다. Java 호출자가 있으면 `@Throws`가 필요한지 본다.
- `runCatching`이나 넓은 `catch (e: Exception)`은 coroutine의 `CancellationException`까지 삼킨다.
- 직렬화에서 필드 누락, 명시적 null, 기본값이 각각 어떻게 해석되는지 채택한 라이브러리(kotlinx.serialization, Jackson Kotlin module 등) 기준으로 확인한다. 기본값이 누락 데이터를 정상처럼 보이게 할 수 있다.
- `val`은 참조만 고정한다. `List` 타입으로 노출한 `MutableList`, 공유 객체의 가변 필드, 스레드 간 공유 상태를 확인한다.
- `data class`의 equals·hashCode는 주 생성자 프로퍼티만 쓰고, 배열 프로퍼티는 참조로 비교한다. 엔티티·캐시 키·Set 원소로 쓸 때 이것이 계약과 맞는지 본다.

### coroutine

- scope의 소유자와 취소 주체를 확인한다. `GlobalScope`나 소유자 없는 scope에서 시작한 작업은 요청·컴포넌트 수명과 분리된다.
- structured concurrency: `coroutineScope`에서는 자식 하나의 실패가 형제와 부모를 취소하고, `supervisorScope`에서는 형제를 취소하지 않는다. 부분 실패의 의도와 맞는지 본다.
- 취소는 협조적이다. `withTimeout`은 suspend 지점에서만 작동하고 블로킹 호출을 멈추지 않는다. JDBC는 query timeout·`Statement.cancel` 같은 드라이버 수단이 따로 필요하다.
- `suspend`는 JDBC·파일 I/O를 비동기로 바꾸지 않는다. 블로킹 호출은 전용 dispatcher(`Dispatchers.IO` 또는 크기를 제한한 풀)에서 실행하고, 그 병렬도가 연결 풀 크기와 맞는지 본다. `Dispatchers.Default`에서 블로킹하면 CPU 작업이 굶는다.
- `CancellationException`은 다시 던진다. 정리 코드가 suspend해야 하면 `withContext(NonCancellable)`을 쓴다.

### 트랜잭션과 context

- 트랜잭션·MDC·보안 context가 ThreadLocal에 묶여 있으면, coroutine이 다른 스레드에서 재개될 때 따라가지 않을 수 있다.
- 트랜잭션과 coroutine의 연결은 채택한 라이브러리(Spring 트랜잭션 관리자의 종류, Exposed, jOOQ 등)의 문서와 실제 시험으로 확인한다.
- 트랜잭션 경계 안에서 외부 호출이나 긴 작업을 하지 않는지 본다.

### JDBC·HTTP·파일 streaming

- 연결·statement·result set·stream을 누가 닫는지 확인한다(`use {}`). `Sequence`·`Flow`·`Iterator`로 결과를 지연 반환하면 연결 scope 밖에서 쓰이지 않는지 본다.
- JDBC fetch size가 실제로 적용되는 조건은 드라이버마다 다르다(예: PostgreSQL은 autocommit 해제와 forward-only result set, MySQL Connector/J는 `useCursorFetch` 또는 streaming 모드). 채택한 드라이버 문서로 확인하고 메모리 사용을 측정한다.
- HTTP streaming 응답은 헤더를 보낸 뒤 실패하면 상태 코드를 바꿀 수 없다. 소비자가 부분 응답을 어떻게 판별하는지, 클라이언트가 끊었을 때 서버 쪽 작업이 취소되는지 확인한다.
- 소비자가 느릴 때(backpressure) 버퍼가 끝없이 쌓이지 않는지, 전체를 메모리에 모으는 지점이 없는지 본다.

### 의존성과 산출물

- 선언한 버전, 실제 resolve 결과, 패키징된 산출물을 구분한다.
  - resolve 결과: `./gradlew :<module>:dependencies --configuration runtimeClasspath`
  - 특정 의존성이 그 버전으로 정해진 이유: `./gradlew :<module>:dependencyInsight --dependency <name> --configuration runtimeClasspath`
  - 플러그인·빌드스크립트 classpath: `./gradlew buildEnvironment`
  - 산출물: 생성된 jar·이미지 안의 실제 파일(`jar tf`, `unzip -l`)
- BOM·platform·충돌 해결 때문에 선언과 다른 버전이 들어올 수 있다.
- 기존 Gradle task(`./gradlew tasks --all`), 테스트, 정적 분석(detekt·ktlint 등이 이미 있을 때)을 먼저 쓴다.
