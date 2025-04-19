# 🔧 Cursor DB 기반 서버 코드 수정 프롬프트

## 🎯 프롬프트 목적
공통 서버 소스 구조를 기반으로 Cursor가 실수 없이 정확한 위치에 코드를 수정할 수 있도록 지시합니다.  
수정 대상은 기존 common-core 모듈 내부의 공통 클래스이며, 구조적 일관성을 유지하고 의존성을 오염시키지 않도록 엄격하게 통제합니다.

## 📁 작업 경로 정의

### 문서 저장 경로
`/Users/changig/dev/unibase-docs/02_server/02_code/cursor_db_based_server_mod_prompt.md`

### 작업 기준 경로
`/Users/changig/workspace/unibase-platform/server`  
→ 해당 경로 내에서만 파일 생성 및 수정할 것

## 💾 연동 대상 DB 구조
- MySQL 기반, 테이블명 및 컬럼명은 snake_case, 실제 테이블은 이미 존재함
- 모든 테이블은 복수형(ex. users, apps, subscriptions)이며, 기본적으로 다음 공통 컬럼을 포함:
  - created_at, updated_at, created_by, updated_by, metadata

## 🏗 전체 서버 구조 특징
- Java + Spring Boot 기반
- common-core 모듈과 app 모듈(app_user, app_subscription 등)로 구성
- 공통 응답 포맷: ApiResponse<T> 사용
- JPA 기반 Entity + Repository + Service + Controller 구조
- DTO ↔ Entity 변환 구조화 필요 (MapStruct 또는 수동)

## ⚠️ 작업 시 반드시 지켜야 할 기준

1. 기존 소스의 패키지 구조, 클래스 이름, import 경로를 변경하지 말 것
2. 공통 모듈 수정 시 반드시 `/common/` 하위에 생성 또는 수정할 것
3. 새로운 도메인 추가 시:
   - Entity → DTO → Repository → Service → Controller 순서로 계층적으로 생성
   - DTO 클래스는 필드명 camelCase, 서버와 DB 간 변환 필요 시 @JsonProperty 사용
4. API 응답은 반드시 ApiResponse<T> 구조로 래핑해서 반환할 것
5. Controller의 @RequestMapping 경로는 `/api/{도메인명}` 패턴을 사용할 것

## 🔍 에러 방지 및 테스트 기준

- 변경 사항이 기존 코드와 충돌하지 않도록 사전 분석할 것
- Lombok 어노테이션(@Getter, @Builder 등) 누락 시 오류 발생 가능
- import 경로는 절대 상대 경로 금지, 항상 `com.unibase.xxx` 절대 경로 사용
- Entity와 DTO 간 필드 누락/오타 방지, JSON 직렬화 주석 확인

## 📝 API 예시 응답 포맷 (공통)

```json
{
  "status": "SUCCESS",
  "message": "조회 성공",
  "data": [
    {
      "userId": 1,
      "email": "test@example.com",
      "nickname": "홍길동"
    }
  ]
}
```

## ✅ 마지막 확인 사항

- 모든 생성 클래스는 어떤 목적과 어떤 테이블 기반인지 주석으로 명확히 명시할 것
- Cursor는 변경 후 수정/생성된 클래스 경로, 파일명, 설명을 콘솔에 출력할 것
- 변경 또는 생성된 API가 정상 작동할 수 있도록 구조 내 의존성 확인 후 완료 메시지를 출력할 것

## 🔄 변경 이력 관리

모든 변경 사항은 다음 형식으로 기록되어야 합니다:

```
[변경일시] YYYY-MM-DD HH:mm
[수정파일] com.unibase.xxx.YYY
[변경내용] 
- 변경 사항 1
- 변경 사항 2
[검증결과]
- 테스트 결과
- 구조 검증 결과
``` 