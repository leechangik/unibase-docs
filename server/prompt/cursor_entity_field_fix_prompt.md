# ✅ Cursor - 5단계 Entity 필드 오류 수정 프롬프트

## 📌 목적
5단계(subscription 및 사용자-앱 연결 구조)에서 발생한 Entity 클래스의 필드 누락/불일치를 수정하기 위한 프롬프트입니다.  
DTO와의 호환성, JPA 필드 정의, 연관된 클래스와의 일관성을 유지하도록 설계되어야 합니다.

---

## 📁 대상 작업 경로
`/Users/changig/workspace/unibase-platform/server`

---

## 📁 수정 대상 및 내용

### [1] UserApp 엔티티
**파일 위치:** `com.unibase.app_userapp.entity.UserApp`

#### ✅ 수정 내용
- 필드 추가:
```java
@Column(name = "subscription_status")
private String status;
```
- 관련 DTO (`UserAppDTO`)에 해당 필드 추가: `String status`
- 변환 로직에도 반영:
  - `UserAppDTO.Response.from(UserApp entity)` 에서 status 포함
  - `UserApp.from(UserAppDTO.Request dto)` 에서 dto.getStatus() → entity.setStatus() 추가

---

### [2] App 엔티티
**파일 위치:** `com.unibase.app_app.entity.App`

#### ✅ 수정 내용
- 불필요한 필드 제거:
```java
// 제거 대상
private String version;
```
- 관련 DTO (`AppDTO`)에서도 version 필드 제거
- 매핑 로직에서 version 포함된 부분이 있으면 주석 처리 또는 제거

---

## 🛠️ 구현 조건
- 기존 필드 주석 처리 X → 완전히 제거 또는 추가
- DTO ↔ Entity 매핑 메서드에 정확히 반영
- @Column 어노테이션 정확히 명시
- import 경로 상대 경로 금지 → 절대 경로 사용

---

## 🧪 빌드 검증
```bash
cd /Users/changig/workspace/unibase-platform/server
./gradlew build
```
> ✅ 빌드 성공 시 `✅ Entity 필드 오류 수정 완료` 출력

---

이 프롬프트는 `.cursor-tasks/prompt.txt`로 전달 시 Cursor가 실수 없이 **Entity → DTO → 매핑 전체 일관성**을 유지하면서 오류를 해결하게 설계되어 있습니다. 