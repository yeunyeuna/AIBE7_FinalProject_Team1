# AGENTS.md

## Project

전자제품 관리 및 중고거래 서비스.
초기 MVP는 노트북을 대상으로 한다.

## Tech Stack

- Java
- Spring Boot 4
- Gradle
- Spring Web MVC
- Spring Data JPA
- PostgreSQL
- Flyway
- Redis
- Spring Security + JWT
- Docker Compose
- JUnit 5
- Mockito
- Spotless

## Architecture

기본 구조:

Controller → Service → Repository → Entity

- Controller에 비즈니스 로직을 작성하지 않는다.
- Controller에서 Repository를 직접 호출하지 않는다.
- API Request / Response는 DTO를 사용한다.
- Entity를 API 응답으로 직접 반환하지 않는다.

## JPA

- 기본적으로 Lazy Loading을 사용한다.
- 불필요한 EAGER Loading을 사용하지 않는다.
- N+1 발생 가능성을 확인한다.
- 필요한 경우 Fetch Join, EntityGraph 등을 사용한다.
- Transaction 범위를 명확하게 관리한다.

## Database

- DB 스키마 변경은 반드시 Flyway Migration으로 관리한다.
- 기존 Migration 파일을 임의로 수정하지 않는다.
- 새로운 변경은 새로운 Migration 파일로 추가한다.
- `ddl-auto=update`에 의존하지 않는다.

## Security

- 인증이 필요한 API의 인증 / 인가 여부를 확인한다.
- 다른 사용자의 데이터에 접근할 수 없는지 확인한다.
- Password, Token, Secret, API Key를 코드나 로그에 노출하지 않는다.
- Refresh Token은 Redis에서 관리한다.

## Testing

새로운 기능이나 주요 비즈니스 로직에는 테스트를 작성한다.

작업 완료 후 가능한 경우 다음을 실행한다.

```bash
./gradlew spotlessCheck
./gradlew test
./gradlew build
```

실행하지 않은 테스트를 실행했다고 말하지 않는다.

Development Rules
작업 전에 관련 기존 코드를 먼저 확인한다.
기존 프로젝트 구조와 스타일을 우선한다.
요청하지 않은 대규모 리팩터링을 하지 않는다.
관련 없는 파일을 수정하지 않는다.
새로운 의존성 추가 전에 기존 의존성으로 해결 가능한지 확인한다.
과도한 추상화나 불필요한 디자인 패턴을 추가하지 않는다.
Git
main 브랜치에 직접 Push하지 않는다.
DB 변경 시 Flyway Migration을 함께 작성한다.
사용자가 요청하지 않은 Merge, Force Push, Branch 삭제를 수행하지 않는다.

## Coding Convention

팀 합의본이다. 위의 공통 규칙과 함께 적용한다.

### 1. 기본 원칙

- 기존 코드의 구조와 스타일을 우선한다. 요청하지 않은 대규모 리팩터링은 하지 않는다.
- 관련 없는 파일은 수정하지 않는다.
- 새 의존성을 추가하기 전에 기존 의존성으로 해결되는지 확인한다.
- 과도한 추상화와 불필요한 디자인 패턴을 넣지 않는다.
- 코드는 Spotless(google-java-format)가 정리한다. 포맷은 논쟁하지 않고 `./gradlew spotlessApply`로 맞춘다.

### 2. 포맷

| 항목 | 기준 |
| --- | --- |
| 포매터 | google-java-format (Spotless) |
| 들여쓰기 | 공백 2칸 |
| 한 줄 길이 | 100자 권장 |
| import | 사용하지 않는 import 제거, 와일드카드(`*`) 금지 |
| 파일 끝 | 줄바꿈으로 끝냄, 줄 끝 공백 제거 |
| 인코딩 | UTF-8 |

```bash
./gradlew spotlessApply   # 자동 정리
./gradlew spotlessCheck   # PR 전 검사
```

### 3. 패키지 구조

```
org.example.springtestci
├── {domain}                  auth, product, document, history, listing, chat, trade, ownership, ai
│   ├── controller
│   ├── service
│   ├── repository
│   ├── entity
│   └── dto
└── common                    error, response, security, storage, config, audit, time
```

- 기본 패키지 구조는 클린 아키텍처를 기준으로 삼는다.
- 도메인 패키지 안에서만 계층을 나눈다. 계층 → 도메인 순으로 나누지 않는다.
- 도메인 사이는 다른 도메인의 Repository를 직접 쓰지 않고, 인터페이스(예: `ProductOwnerReader`)나 서비스로 연결한다.
- 여러 도메인이 함께 쓰는 코드만 `common`에 둔다.

### 4. 네이밍

| 대상 | 규칙 | 예 |
| --- | --- | --- |
| 클래스 | UpperCamelCase, 역할 접미사 | `DocumentService`, `DocumentController` |
| 메서드·변수 | lowerCamelCase, 동사로 시작 | `createFileUrl`, `findOwnerId` |
| 상수 | UPPER_SNAKE_CASE | `MAX_DOCUMENTS_PER_PRODUCT` |
| 패키지 | 소문자 한 단어 | `document`, `repository` |
| 요청 DTO | `{행위}Request` | `ExtractionCreateRequest` |
| 응답 DTO | `{대상}Response` | `DocumentResponse` |
| 테스트 | `{대상}Test` | `DocumentServiceTest` |
| 불리언 | `is·has·can`으로 시작 | `isProtected`, `canTransitionTo` |
| DB 테이블·컬럼 | snake_case, 테이블은 복수형 | `document_extractions`, `uploader_id` |
| 시각 컬럼 | `_at` 접미사 | `protected_at` |
| 날짜 컬럼 | `_on` 접미사 | `first_purchase_on` |
| 외래키 컬럼 | `{대상}_id` | `product_id` |

- 약어를 임의로 만들지 않는다(`doc`, `mgr` 같은 줄임말 금지). 사용해야 한다면 주석으로 구별할 수 있도록 반드시 원래 용어를 첨부한다.

### 5. 계층별 규칙

#### 5-1. Controller

- 요청을 받아 검증·변환하고 서비스에 넘기는 일만 한다. 비즈니스 로직을 두지 않는다.
- Repository를 직접 호출하지 않는다.
- 요청·응답은 DTO를 쓰고, Entity를 그대로 반환하지 않는다.
- 응답은 `ApiResponse<T>`로 감싼다.
- 경로는 `/api/v1/...` 아래에 둔다.
- 요청 본문은 `@Valid`로 검증한다.
- 사용자 ID는 요청 값이 아니라 인증 토큰(JWT subject)에서 꺼낸다.

```java
@PostMapping("/{extractionId}/confirm")
ApiResponse<ExtractionResponse> confirm(
    @AuthenticationPrincipal Jwt jwt,
    @PathVariable Long documentId,
    @PathVariable Long extractionId,
    @Valid @RequestBody ExtractionConfirmRequest request) {
  return ApiResponse.success(
      ExtractionResponse.from(
          reviewService.confirm(documentId, extractionId, userId(jwt), request.reviewedData())));
}
```

#### 5-2. Service

- 트랜잭션 범위를 명확히 한다. 쓰기는 `@Transactional`, 조회는 `@Transactional(readOnly = true)`.
- 외부 API 호출은 DB 트랜잭션 안에서 하지 않는다.
- 다른 사용자의 데이터에 접근할 수 없는지 매번 확인한다.
- 권한이 없는 요청이 데이터의 존재를 드러내지 않도록 `404`로 응답한다.
- 동시 요청이 문제 되는 곳은 행 잠금(`PESSIMISTIC_WRITE`)으로 순서를 보장한다.

#### 5-3. Repository

- 메서드 이름 쿼리를 기본으로 하고, 복잡한 조회만 `@Query`를 쓴다.
- N+1 가능성을 확인하고, 필요하면 Fetch Join이나 EntityGraph를 쓴다.

#### 5-4. Entity

- 연관관계는 기본적으로 Lazy를 쓰고 불필요한 EAGER를 쓰지 않는다.
- 기본 생성자는 `protected`로 두고, 생성은 정적 팩토리 메서드(`register`, `request`)로 한다.
- `setter`를 열지 않고, 상태 변경은 의미 있는 메서드(`confirm`, `reject`, `protect`)로 한다.
- 상태 전이는 엔티티 안에서 검증해 잘못된 전이를 막는다.
- 생성·수정 시각은 `BaseTimeEntity`를 상속한다.

#### 5-5. DTO

- 요청·응답 DTO는 `record`로 만든다.
- 검증 어노테이션(`@NotNull`, `@Size`, `@PositiveOrZero`)은 DTO에 둔다.
- 응답 DTO에는 내부 값(저장소 키, 내부 ID, 업로더 등)을 넣지 않는다.
- Entity → DTO 변환은 DTO의 정적 메서드 `from(...)`으로 한다.

### 6. 예외와 오류 코드

- 비즈니스 예외는 `CommonException(ErrorCode)`로 던진다.
- 오류 코드는 `ErrorCode` enum에 모으고, 형식은 `{도메인}_{3자리 번호}`다.

| 접두어 | 도메인 |
| --- | --- |
| `COMMON` | 공통 |
| `AUTH` | 인증 |
| `PRODUCT` | 제품 |
| `DOC` | 문서 |

- 새 도메인은 접두어를 하나 정해 쓴다(`HIST`, `LIST`, `CHAT`, `TRADE` 등).
- 예상하지 못한 예외는 내부 정보를 응답에 노출하지 않는다.
- 예외 발생 시 반드시 로그를 기록하고 사용자 보고를 남긴다. 기능적으로 문제가 없더라도 예외를 무시하지 않는다.

### 7. 데이터베이스와 마이그레이션

- 스키마 변경은 반드시 Flyway 마이그레이션으로 한다.
- 이미 머지된 마이그레이션 파일은 수정하지 않고, 새 파일을 추가한다.
- `ddl-auto=update`에 의존하지 않는다(`validate` 사용).
- 파일 이름은 `V{yyyyMMdd}_{두자리 순번}__{설명}.sql`로 한다. 예: `V20261013_01__create_document_tables.sql`
    - 현재는 `V1__`, `V2__`, `V5__`가 섞여 있어 번호 충돌이 실제로 한 번 발생했다.
    - 이미 생성된 마이그레이션이나 데이터베이스가 있으면 현재 규격에 맞춰 가능할 경우 수정한다.
- 시각은 `TIMESTAMP WITH TIME ZONE`(UTC 기준)을 쓴다.
- 값 범위와 상태 조합은 코드뿐 아니라 `CHECK` 제약으로도 막는다.
- 다른 도메인의 테이블을 참조하는 FK는, 선행 테이블이 머지된 뒤 별도 마이그레이션으로 추가할 수 있다.

### 8. 보안과 로깅

- 비밀번호, 토큰, 비밀 키, API 키를 코드와 로그에 남기지 않는다.
- 개인정보(이메일·전화번호·카드번호·파일 내용)를 로그에 남기지 않는다.
- 파일 이름, 이미지, AI 응답 본문은 로그에 남기지 않고, 실패는 분류 코드만 기록한다.
- 인증이 필요한 API는 인증·인가를 확인한다.
- 환경별 값은 환경 변수로 관리하고, `.env`는 커밋하지 않는다.
- Refresh Token은 Redis에서 관리한다.

### 9. 주석과 문서화

- 주석은 기능 사항 우선으로 이해하기 쉽게 설명을 적는다. 이름과 코드로 알 수 있는 것은 쓰지 않는다.
- 클래스와 공개 메서드의 설명은 한글 Javadoc으로 쓴다.
- 임시 구현이나 알려진 한계는 이유와 함께 주석으로 남긴다(`TODO`는 담당·이슈 번호를 함께).

```java
/** 현재 소유자가 마스킹을 확인한 사본을 올린다. 업로드 파일명은 확장자 판별에만 쓰고 저장하지 않는다. */
@Transactional
public Document upload(...) { ... }
```

### 10. 테스트

- 새 기능과 주요 비즈니스 로직에는 테스트를 작성한다.
- 단위 테스트는 JUnit 5 + Mockito(`@ExtendWith(MockitoExtension.class)`)를 쓴다.
- 테스트 메서드 이름은 동작을 설명하는 한글 문장을 허용한다.
    - 예: `본인이_올린_보호되지_않은_사본은_삭제할_수_있다`
- 하나의 테스트는 하나의 동작만 검증한다(AAA: 준비 → 실행 → 검증).
- 정상 경로뿐 아니라 권한 없음, 중복 요청, 잘못된 상태를 검증한다.
- 발생 가능성이 낮은 상태나, 과도한 중복 검사는 배제한다. 엣지 케이스 테스트는 수행하지 않고, 발생 가능성 보고로만 남긴다.
- 실행하지 않은 테스트를 실행했다고 보고하지 않는다.

### 11. Git

#### 11-1. 브랜치

- `main`에 직접 푸시하지 않는다.
- 포크에서 브랜치를 만들어 작업하고 원본 `main`으로 PR을 보낸다.
- 브랜치 이름 형식은 `{종류}/{작업 내용}`이다.

| 종류 | 용도 | 예 |
| --- | --- | --- |
| `feature/` | 기능 추가 | `feature/doc-upload-api` |
| `fix/` | 버그 수정 | `fix/chat-migration-version` |
| `docs/` | 문서 | `docs/readme` |
| `chore/` | 설정·잡무 | `chore/gitignore` |

#### 11-2. 커밋 메시지

형식: `{종류}: {한글 설명}`

| 종류 | 용도 |
| --- | --- |
| `feat` | 기능 추가 |
| `fix` | 버그 수정 |
| `docs` | 문서 |
| `style` | 포맷·화면 스타일(동작 변경 없음) |
| `refactor` | 동작 변경 없는 구조 개선 |
| `test` | 테스트 추가·수정 |
| `chore` | 빌드·설정 |

- 제목은 변경 내용을 한 줄로 쓰고, 이유가 필요하면 본문에 적는다.
- 한 커밋은 하나의 기능만을 담도록 우선하되, 여러 기능이 포함될 경우 각 기능 명세를 분리해서 작성한다.

#### 11-3. Pull Request

- 변경 범위가 겹치지 않게 PR을 쪼개기를 권장한다. 앞 PR 위에 쌓일 때는 base를 명시한다.
- 공통 설정(`SecurityConfig`, `application.yaml`, 공용 저장소)을 바꾸면 추가 리뷰를 받는다.
- 설명에는 변경 내용, 테스트 결과, 확인이 필요한 사항을 쓴다.
- 요청하지 않은 Merge, Force Push, 브랜치 삭제를 하지 않는다.

### 12. PR 전 체크리스트

- [ ] `./gradlew spotlessCheck` 통과
- [ ] `./gradlew test` 통과
- [ ] `./gradlew build` 통과
- [ ] 새 기능에 테스트가 있다
- [ ] DB 변경은 새 Flyway 마이그레이션으로 추가했다 (기존 파일 미수정)
- [ ] 인증·인가와 타인 데이터 접근을 확인했다
- [ ] 비밀번호·토큰·키·개인정보가 코드와 로그에 없다
- [ ] Entity를 응답으로 직접 반환하지 않는다
- [ ] 관련 없는 파일을 수정하지 않았다
- [ ] 공통 파일을 바꿨다면 PR 설명에 적었다

Final Response

작업 완료 후 간단히 정리한다.

변경한 내용
변경한 주요 파일
테스트 결과
추가 확인이 필요한 사항