# API 표준

## 목적

이 문서는 UniBase 플랫폼의 REST API 디자인 표준을 정의합니다. 모든 엔드포인트에서 일관성을 보장하여 개발자의 학습 곡선을 줄이고 신뢰할 수 있는 자동화를 가능하게 합니다.

## 목차

- [API 디자인 원칙](#api-디자인-원칙)
- [URL 구조](#url-구조)
- [HTTP 메서드](#http-메서드)
- [요청 형식](#요청-형식)
- [응답 형식](#응답-형식)
- [오류 처리](#오류-처리)
- [버전 관리](#버전-관리)
- [인증 및 권한 부여](#인증-및-권한-부여)
- [속도 제한](#속도-제한)
- [문서화](#문서화)
- [테스트 표준](#테스트-표준)

## API 디자인 원칙

UniBase API는 다음과 같은 핵심 원칙을 준수합니다:

1. **RESTful 디자인**: 리소스는 적절한 HTTP 메서드를 사용하는 엔드포인트로 모델링됩니다
2. **일관된 구조**: 예측 가능한 URL 패턴 및 응답 형식
3. **포괄적인 문서화**: 모든 엔드포인트는 OpenAPI/Swagger를 사용하여 문서화됩니다
4. **버전 관리**: API는 이전 버전과의 호환성을 유지하면서 발전할 수 있도록 버전이 관리됩니다
5. **기본적으로 안전**: 모든 엔드포인트는 명시적으로 공개되지 않는 한 적절한 인증이 필요합니다

## URL 구조

### 기본 URL 형식

```
https://{environment}.api.unibase.io/v{version}/{module}/{resource}
```

예시:
- `https://dev.api.unibase.io/v1/users/profile`
- `https://prod.api.unibase.io/v1/farm/crops`

### 리소스 명명 규칙

- 컬렉션에는 복수형 명사 사용: `/users`, `/applications`, `/modules`
- 특정 리소스에는 단수형 명사 사용: `/users/{id}`, `/applications/{id}/config`
- 복합 리소스 이름에는 kebab-case 사용: `/user-profiles`, `/module-activations`
- 관련 리소스 중첩: `/applications/{id}/modules`
- 중첩은 최대 2단계로 제한

### 쿼리 매개변수

- 필터링, 정렬, 페이징 및 필드 선택에 사용
- 일관된 명명 규칙을 따름:
  - `filter[field]`: 필드 값으로 결과 필터링
  - `sort`: 정렬 방향 및 필드 (예: `sort=-created_at,name`)
  - `page[size]`: 페이지당 결과 수
  - `page[number]`: 페이징을 위한 페이지 번호
  - `fields`: 포함할 필드의 쉼표로 구분된 목록

## HTTP 메서드

| 메서드 | 목적 | 예시 |
|--------|---------|---------|
| GET | 리소스 검색 | GET /users |
| POST | 새 리소스 생성 | POST /users |
| PUT | 리소스 업데이트(전체 교체) | PUT /users/123 |
| PATCH | 리소스 부분 업데이트 | PATCH /users/123 |
| DELETE | 리소스 제거 | DELETE /users/123 |
| OPTIONS | 리소스에 대한 메타데이터 검색 | OPTIONS /users |

### 메서드 사용 지침

- **GET** 요청은 멱등성을 가져야 하며 데이터를 수정해서는 안 됩니다
- **POST** 요청은 생성된 리소스와 함께 201 Created와 Location 헤더를 반환해야 합니다
- **PUT** 요청은 멱등성을 가져야 하며 전체 리소스 표현이 필요합니다
- **PATCH** 요청은 JSON Patch 형식(RFC 6902) 또는 JSON Merge Patch(RFC 7396)를 사용해야 합니다
- **DELETE** 요청은 성공 시 204 No Content를 반환해야 합니다

## 요청 형식

### 헤더

| 헤더 | 목적 | 예시 |
|--------|---------|---------|
| Authorization | 인증 토큰 | Bearer eyJhbGciOiJIUzI1NiIsI... |
| Content-Type | 요청 본문의 미디어 타입 | application/json |
| Accept | 예상 응답 형식 | application/json |
| Accept-Language | 응답 언어 선호도 | ko-KR |
| X-API-Key | 애플리케이션 인증용 API 키 | apikey_v1_1234abcd |
| X-Request-ID | 추적을 위한 클라이언트 생성 요청 ID | 550e8400-e29b-41d4-a716-446655440000 |

### 요청 본문

- 모든 요청 본문은 유효한 JSON이어야 합니다
- 속성 이름에는 camelCase 사용
- 페이로드 크기를 줄이기 위해 필요한 필드만 포함
- 요청 전송 전 입력 검증

예시:

```json
{
  "username": "honggildong",
  "email": "hong.gildong@example.com",
  "firstName": "길동",
  "lastName": "홍",
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}
```

## 응답 형식

### 표준 응답 형식

모든 API 응답은 표준 형식을 따릅니다:

```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "timestamp": "2023-04-15T10:30:00.000Z",
    "requestId": "550e8400-e29b-41d4-a716-446655440000",
    "pagination": {
      "totalItems": 100,
      "totalPages": 10,
      "currentPage": 1,
      "pageSize": 10
    }
  },
  "error": null
}
```

오류 응답의 경우:

```json
{
  "success": false,
  "data": null,
  "meta": {
    "timestamp": "2023-04-15T10:30:00.000Z",
    "requestId": "550e8400-e29b-41d4-a716-446655440000"
  },
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "ID가 123인 사용자를 찾을 수 없습니다",
    "details": { ... },
    "status": 404
  }
}
```

### HTTP 상태 코드

| 코드 | 설명 | 사용 |
|------|-------------|-------|
| 200 | OK | 성공한 GET, PUT, PATCH 요청 |
| 201 | Created | 성공한 POST 요청 |
| 204 | No Content | 성공한 DELETE 요청 |
| 400 | Bad Request | 잘못된 요청 매개변수 또는 본문 |
| 401 | Unauthorized | 인증 누락 또는 잘못됨 |
| 403 | Forbidden | 인증은 유효하지만 권한 부족 |
| 404 | Not Found | 리소스를 찾을 수 없음 |
| 409 | Conflict | 요청이 현재 상태와 충돌함 |
| 422 | Unprocessable Entity | 유효한 요청이지만 의미적으로 잘못됨 |
| 429 | Too Many Requests | 속도 제한 초과 |
| 500 | Internal Server Error | 예상치 못한 서버 오류 |
| 503 | Service Unavailable | 서비스를 일시적으로 사용할 수 없음 |

## 오류 처리

### 오류 응답 구조

모든 오류 응답은 다음을 포함합니다:

1. 오류 범주를 반영하는 HTTP 상태 코드
2. 응답 본문의 표준 오류 형식
3. 프로그래밍 방식 처리를 위한 기계 판독 가능 오류 코드
4. 사람이 읽을 수 있는 오류 메시지
5. 디버깅을 위한 선택적 상세 정보

### 오류 코드

오류 코드는 다음 형식을 따릅니다: `{RESOURCE}_{ERROR_TYPE}`

예시:
- `USER_NOT_FOUND`
- `APPLICATION_ALREADY_EXISTS`
- `MODULE_ACTIVATION_FAILED`
- `VALIDATION_ERROR`
- `UNAUTHORIZED_ACCESS`

## 버전 관리

UniBase API는 URL 경로 버전 관리를 사용합니다:

```
/v1/users
/v2/users
```

버전 관리 지침:
- 새 엔드포인트는 최신 버전에 추가해야 합니다
- 호환성이 깨지는 변경은 새 버전이 필요합니다
- 이전 버전에 대한 이전 버전과의 호환성은 최소 하나 이상 유지됩니다
- 더 이상 사용되지 않는 엔드포인트에는 제거 날짜를 나타내는 Sunset 헤더가 포함됩니다

## 인증 및 권한 부여

### 인증 방법

API는 여러 인증 방법을 지원합니다:

1. **JWT 베어러 토큰**
   ```
   Authorization: Bearer eyJhbGciOiJIUzI1NiIsI...
   ```

2. **API 키** (애플리케이션 간)
   ```
   X-API-Key: apikey_v1_1234abcd
   ```

### 권한 부여 모델

- 시스템 전체 권한에 대한 역할 기반 액세스 제어(RBAC)
- 특정 애플리케이션 및 모듈에 대한 리소스 기반 권한
- 권한 확인은 컨트롤러 수준에서 발생합니다

## 속도 제한

다음을 기준으로 속도 제한이 적용됩니다:
- 클라이언트 IP 주소
- API 키 또는 사용자 ID
- 엔드포인트 범주(공개 vs. 비공개)

속도 제한 헤더:
- `X-RateLimit-Limit`: 윈도우당 요청 제한
- `X-RateLimit-Remaining`: 윈도우에 남은 요청
- `X-RateLimit-Reset`: 윈도우가 재설정되는 시간(Unix 타임스탬프)

속도 제한에 걸리면 API는 Retry-After 헤더와 함께 429 상태 코드를 반환합니다.

## 문서화

모든 API 엔드포인트는 다음을 사용하여 문서화됩니다:

1. **OpenAPI (Swagger)**: `/v{version}/swagger-ui.html`에서 사용 가능
2. **API 참조**: 개발자 포털에서 유지 관리되는 사람이 읽을 수 있는 문서
3. **변경 로그**: 버전 간 변경 사항에 대한 자세한 문서

문서에는 다음이 포함됩니다:
- 요청 매개변수
- 응답 스키마
- 요청 및 응답 예시
- 인증 요구 사항
- 속도 제한 정보

## 테스트 표준

API 테스트에는 다음이 포함되어야 합니다:

1. **단위 테스트**: 개별 컨트롤러 및 서비스 테스트
2. **통합 테스트**: 모의 데이터로 엔드포인트 테스트
3. **계약 테스트**: API가 지정된 계약을 준수하는지 확인
4. **성능 테스트**: 응답 시간 및 처리량 측정
5. **보안 테스트**: 취약점 확인

코드 커버리지 요구 사항: 컨트롤러 및 서비스 계층에 대해 최소 85% 