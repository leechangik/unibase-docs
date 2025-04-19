# 🧠 Cursor Git 연동 및 커밋 전략 문서

## 📌 목적
Cursor가 소스 변경 작업을 완료한 후, GitHub 원격 저장소로 안전하게 커밋 및 푸시하도록 하기 위한 전략 문서입니다.  
문서와 실제 소스 경로를 기준으로 **경로 혼동 없이 정확한 Git 작업**을 수행하게 만드는 목적입니다.

---

## 📁 문서 위치
`/Users/changig/dev/unibase-docs/08_github/cursor_git_strategy_doc.md`

---

## 📁 관련 경로

| 항목 | 경로 |
|------|------|
| 공통 문서 기준 경로 | `/Users/changig/dev/unibase-docs/` |
| Flutter 소스 경로 | `/Users/changig/dev/unibase-platform-dev/flutter/` |
| Server 소스 경로 | `/Users/changig/dev/unibase-platform-dev/server/` |
| Git 루트 경로 | `/Users/changig/dev/unibase-platform-dev/` |
| GitHub 저장소 URL | `https://github.com/leechangik/unibase-platform-dev.git` |

---

## ✅ Git 작업 원칙 (Cursor 전용)

1. 항상 **지정된 루트 디렉토리**(`/Users/changig/dev/unibase-platform-dev`) 기준으로 `git` 명령어를 실행해야 함
2. 원격 저장소 URL은 **절대로 변경하지 말 것** → `https://github.com/leechangik/unibase-platform-dev.git`
3. 커밋 메시지는 반드시 명확한 전략 규칙을 따름:
   - 예: `feat: 4단계 user_apps 구조 추가 및 사용자-앱 관계 처리 완료`
4. 커밋 전에는 항상 다음 경로 기준으로 변경 사항 확인
   - server/
   - flutter/
   - 문서: `/Users/changig/dev/unibase-docs/`

---

## 🔐 인증 관련
- 최초 푸시 시 GitHub 인증 필요 (Token 사용)
- 이미 인증한 환경에서는 추가 인증 없음
- Token은 터미널에 붙여넣기하여 인증 처리 가능

---

## 🛠️ Git 푸시 전체 명령 흐름

```bash
cd /Users/changig/dev/unibase-platform-dev
git init

# origin 재설정
git remote remove origin 2>/dev/null || true
git remote add origin https://github.com/leechangik/unibase-platform-dev.git
git branch -M main

git add .
git commit -m "feat: 4단계 user_apps 구조 추가 및 사용자-앱 관계 처리 완료"
git push -u origin main
```

---

## ✅ 최종 출력 조건
- 커밋 메시지 출력
- 변경된 경로 요약
- 푸시 성공 여부 확인 메시지 (`✅ GitHub 푸시 완료`)

---

## 🚫 주의 사항
- Git 루트가 `unibase-platform`이 아닌 다른 디렉토리이면 절대 푸시 금지
- GitHub URL이 다르면 즉시 중단하고 경로 검토 요청할 것
- README 또는 문서만 변경될 경우, 명확한 메시지 필요 (ex. `docs: update git push strategy doc`)

---

이 전략 문서는 Cursor가 **Git 작업 시 실수를 방지하고, 프로젝트 소스와 문서의 동기화를 정확히 유지하기 위한 기준 문서**로 사용됩니다. 