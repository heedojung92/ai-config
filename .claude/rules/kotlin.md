---
paths:
  - "**/*.kt"
  - "**/*.kts"
---
# Kotlin 백엔드 규칙

- 검사 루프: 바꾼 클래스의 테스트만 `./gradlew test --tests '<패키지.클래스>'`로 먼저 돌리고, 끝내기 전에 `./gradlew check`나 프로젝트가 정한 명령을 돌린다. 시스템 `gradle`이 아니라 wrapper(`./gradlew`)를 쓴다.
- Kotlin·Spring Boot·JDK 버전은 빌드 파일에서 확인한다. `gradle/libs.versions.toml`이 있으면 의존성 버전은 그 파일에서만 바꾼다.
- `!!`로 null을 덮지 않는다. 외부 입력·DB·Java API 경계에서 nullable을 그대로 모델링한다.
- coroutine은 `GlobalScope`가 아니라 호출자 scope에서 시작한다. 막히는 호출(JDBC·파일·블로킹 클라이언트)은 `withContext(Dispatchers.IO)`로 감싸고, 요청 처리 경로에서 `runBlocking`을 쓰지 않는다.
- `runCatching`과 `catch (e: Exception)`은 `CancellationException`도 잡는다. suspend 코드에서는 잡는 예외를 좁히거나 취소 예외를 다시 던진다.
- Spring 빈이 proxy되려면 `kotlin("plugin.spring")`이, JPA 엔티티에는 `kotlin("plugin.jpa")`가 필요하다. 같은 클래스 안에서 호출한 `@Transactional`·`@Async` 메서드는 proxy를 거치지 않는다.
- 의존성은 생성자 `val`로 주입하고 필드 `lateinit` 주입을 쓰지 않는다.
- suspend 함수 테스트는 `runTest`로, MockK mock은 `coEvery`·`coVerify`로 다룬다. DB 동작은 mock 대신 프로젝트의 통합 테스트(Testcontainers 등)로 확인한다.
- JPA 엔티티는 `kotlin-backend-jpa-entity-mapping` skill, Gradle 빌드 정리는 `gradle-best-practices` skill을 쓴다.
