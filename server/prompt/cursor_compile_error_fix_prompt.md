# ✅ Cursor용 5단계 컴파일 오류 자동 수정 프롬프트

## 📌 목적
5단계(subscription 도메인 및 사용자-앱 관계 포함) 구현 후 발생한 컴파일 오류를 자동으로 수정하기 위한 Cursor 프롬프트입니다.  
이 프롬프트는 순차적으로 Repository, Controller, DTO-Entity 매핑, 전체 구조 오류를 검토하고 수정하는 절차를 포함합니다.

---

## 📁 대상 작업 경로
`/Users/changig/workspace/unibase-platform/server`

---

## 📄 1단계: Repository 메서드 누락 보완

- UserRepository에 다음 메서드 추가:
```java
boolean existsByUsername(String username);
boolean existsByEmail(String email);
```

- AppRepository에 다음 메서드 추가:
```java
boolean existsByAppName(String appName);
```

- UserAppRepository에 다음 메서드 추가:
```java
boolean existsByUserIdAndAppId(Long userId, Long appId);
```

---

## 📄 2단계: Controller CRUD 메서드 추가

- UserController에 `getUserById(Long id)` 메서드 추가
- AppController에 `getAppById(Long id)` 메서드 추가
- UserAppController에 다음 메서드 추가:
  - `getUserApps()` → 전체 조회
  - `registerUserApp(UserAppDTO.Request dto)` → 신규 등록
  - `removeUserApp(Long id)` → 삭제
  - `updateStatus(Long id, String status)` → 구독 상태 변경

> 모든 응답은 `ApiResponse<T>` 형식으로 감싸서 반환해야 하며, 예외 발생 시 `GlobalExceptionHandler`로 처리되도록 구성

---

## 📄 3단계: DTO ↔ Entity 매핑 구조 보정

- UserDTO ↔ User 변환 메서드 통일:
  - `User from(UserDTO.Request dto)` 생성
  - `UserDTO.Response from(User entity)` 정적 팩토리 방식

- AppDTO, UserAppDTO도 위와 동일한 방식으로 매핑 통일
  - `Request → Entity`, `Entity → Response` 두 방향 함수 명확히 구분

---

## 📄 4단계: 기타 필드/메서드 보완

- AppDTO에서 `version` 필드가 존재하지 않는다면 제거 또는 주석 처리
- UserAppDTO에서 `status` 관련 필드 누락 시 보완
- 모든 DTO 클래스에서 내부 클래스와 외부 클래스 간 타입 불일치 확인 후 수정

---

## 🧪 빌드 확인

모든 수정이 끝난 후 다음 명령으로 전체 빌드 검증:
```bash
cd /Users/changig/workspace/unibase-platform/server
./gradlew build
```

> ✅ 빌드 성공 시 `✅ 5단계 오류 수정 및 재빌드 완료` 출력

---

## 🚨 유의사항
- 절대 패키지 경로(import) 변경하지 말 것
- 모든 클래스는 기존 구조에 따라 `com.unibase.app_xxx` 경로에 위치할 것
- Swagger 어노테이션 누락된 메서드는 자동 보완할 수 있음 