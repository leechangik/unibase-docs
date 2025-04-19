# 🔧 Cursor 공통 서버 코드 수정 프롬프트

## 🎯 프롬프트 목적
공통 서버 소스 구조를 기반으로 Cursor가 실수 없이 정확한 위치에 코드를 수정할 수 있도록 지시합니다.  
수정 대상은 기존 common-core 모듈 내부의 공통 클래스이며, 구조적 일관성을 유지하고 의존성을 오염시키지 않도록 엄격하게 통제합니다.

## 📁 작업 경로 정의

### 문서 저장 경로
`/Users/changig/dev/unibase-docs/server/prompt/cursor_common_code_prompt.md`

### 작업 기준 경로
`/Users/changig/workspace/unibase-platform/server`  
→ 해당 경로 내에서만 작업 가능하며, 절대 외부 경로에 파일을 생성하지 말 것

## 🗂 공통 코드 위치 구조 예시
- `com.unibase.common.response.ApiResponse`
- `com.unibase.common.exception.CustomException`
- `com.unibase.common.exception.ErrorCode`
- `com.unibase.common.entity.BaseEntity`

## ⚠️ 공통 수정 시 반드시 지켜야 할 조건

1. 기존 클래스의 패키지 경로(import 포함)를 절대 변경하지 말 것
2. 모든 새로운 공통 클래스는 common-core 모듈 내부에 추가할 것
3. App 모듈과 의존성을 역참조하지 말 것
4. import 경로는 절대 상대 경로 사용 금지 → 반드시 `com.unibase.xxx` 형태로 작성할 것
5. 공통 클래스를 수정 시 하위 모듈 전체 영향을 고려하고, 구조 보존을 우선할 것
6. 오류 발생 가능성이 있는 부분은 반드시 주석으로 사전 설명 후 처리할 것

## 📝 예시: ApiResponse 클래스 수정 시

### 수정 위치
`src/main/java/com/unibase/common/response/ApiResponse.java`

### 변경 방식
- 기존 구조 보존 → 성공/실패 응답 포맷에 확장 필드 추가 (status_code 등)
- 모든 DTO와의 통일성을 유지해야 하며, Swagger 문서 기준 예시 포맷을 변경하지 말 것

## 🔍 추가 규칙

1. 오류 발생 가능성이 있는 구조 변경은 사전에 주석으로 `// 변경 이유:` 를 남길 것
2. Lombok 사용 시 `@Getter`, `@Builder`, `@AllArgsConstructor` 등을 반드시 명시
3. 단위 테스트가 깨지지 않도록, public API 시그니처는 변경하지 말 것

## ✅ 마지막 확인 사항

- Cursor는 수정 후 반드시 경로, 파일명, 클래스명, 변경 내용을 콘솔에 출력할 것
- 변경 사항이 기존 구조와 충돌하지 않는지 확인 후 완료 메시지를 출력할 것

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