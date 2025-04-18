# 📜 서버 공통 프롬프트: 서버 소스 수정 시 반드시 지켜야 할 규칙

> 이 문서는 Cursor에 서버 기능 개발 또는 수정 요청 시 **항상 선행적으로 제공해야 하는 공통 기준 프롬프트**입니다.

---

## ✅ 작업 유형 구분
- 이 프롬프트는 **공통 모듈 또는 신규 앱 모듈 개발/수정 작업**에 적용됩니다.
- 기존 앱 경로(`apps/idle_farm`, `apps/test_ping` 등)의 컨트롤러, 서비스, 엔티티는 절대 수정 금지입니다.

---

## 📂 디렉토리 및 경로 규칙

1. **모든 Java import는 절대경로만 사용해야 합니다.**
   - ✅ `import com.example.unibase.common.response.ApiResponse;`
   - ❌ 상대경로, IDE 자동 import 경로 등은 절대 금지

2. **신규 기능 추가는 공통 모듈 또는 `apps/app_{기능명}` 하위 디렉토리로만 생성합니다.**
   - `apps/ad_report`, `apps/log_event`, `apps/admin_dashboard` 등

3. **모든 소스 파일은 정확한 지정 경로에 생성되어야 합니다.**
   - 예: `src/main/java/com/example/unibase/apps/ad_report/controller/AdReportController.java`

---

## 🧩 기능 수정/생성 제한

- `apps/` 하위 기존 앱 컨트롤러, 서비스, 리포지토리는 **읽기만 가능**하며 **절대 수정하지 마세요**.
- 공통 예외 처리 구조(GlobalExceptionHandler, CustomException, ErrorCode)는 변경 없이 그대로 활용해야 합니다.
- `@Transactional`, `@Service`, `@RestController` 등 어노테이션은 실무 기준에 맞춰 정확히 작성해야 합니다.

---

## 🔐 안정성 및 검증 기준

- 기존 기능과 충돌, 중복 경로, 엔티티 명 일치 등은 철저히 피해야 합니다.
- 중복 클래스명, 잘못된 import, 경로 누락 등은 전체 서버 빌드 실패 원인이 되므로 반드시 방지해야 합니다.
- 실제 서비스처럼 구조적으로 설계하되, **코드가 자동 생성되어도 전체 프로젝트에서 오류 없이 통합되도록 설계해야 합니다.**

---

## 📎 연동 방식
- 이 문서는 항상 Cursor 작업 시 **첫 번째 프롬프트**로 전달됩니다.
- 이후 기능별 프롬프트는 이 기준 위에서만 작성되어야 하며, 이 기준을 무시한 요청은 재작업 사유가 됩니다.

---

## 📁 저장 경로
```
/Users/changig/dev/unibase-docs/03_prompts/server/prompt.base_server_guideline.md
``` 