# 🗂️ UniBase 전체 ERD 구조 개요 (`erd_overview.md`)

## 📌 문서 목적
이 문서는 UniBase 플랫폼의 데이터베이스 구조 전체를 시각적으로 요약한 ERD(엔터티 관계도)를 설명하며, 공통 테이블과 기능별 확장 테이블 간의 관계를 명확하게 보여줍니다. 이 ERD는 GPT, Cursor, 개발자 모두가 데이터 흐름과 의존성을 이해하는 기준 도구입니다.

---

## 📁 문서 위치
```
/Users/changig/dev/unibase-docs/02_server/db/erd_overview.md
```

---

## 🧱 포함된 테이블 분류

### ✅ 공통 기준 테이블 (`schema_v1.md` 기반)
- `users`: 사용자 계정 (Firebase 연동)
- `apps`: 앱 메타 정보
- `user_apps`: 사용자-앱 연결
- `subscriptions`: 구독 정보
- `multilingual_texts`: 다국어 문구 저장
- `common_codes`: 공통 코드 관리
- `app_design_settings`: 테마/레이아웃 JSON 저장

### 🧩 확장 기능 테이블 (`schema_v1.1_*`, `v1.2` 포함)
- `advertisers`: 광고주 등록
- `ad_campaigns`: 앱 기반 광고 캠페인
- `ad_impressions`: 노출 기록 저장
- `admin_users`: 관리자 계정 관리
- `app_event_logs`: 사용자 앱 이벤트 로그
- `aws_resources`: AWS 리소스 상태/식별자 관리
- `ad_clicks`: 광고 클릭 이벤트 기록 *(v1.2)*
- `ad_reports`: 광고주 리포트 요청 정보 *(v1.2)*
- `ad_report_results`: 리포트 요약 및 파일 링크 *(v1.2)*

---

## 🔗 주요 관계 요약

| 관계             | 설명 |
|------------------|------|
| `users` → `user_apps`        | 사용자-앱 연결 (N:M)
| `apps` → `user_apps`         | 앱과 사용자 연결
| `users` → `subscriptions`    | 사용자별 구독 상태 추적
| `apps` → `subscriptions`     | 앱별 구독 분류
| `apps` → `app_design_settings` | 앱별 테마/레이아웃 분리 적용
| `apps` → `ad_campaigns`      | 광고는 앱 단위로 진행됨
| `ad_campaigns` → `ad_impressions` | 노출 기록은 캠페인 단위로 발생
| `users` → `ad_impressions`   | 로그인 사용자 기준 노출 추적 가능
| `apps` → `app_event_logs`    | 클릭/전환 로그 저장
| `users` → `app_event_logs`   | 사용자 행동 데이터 분석
| `aws_resources` → `apps`     | 리소스별 앱 배포 연계 가능
| `ad_impressions` → `ad_clicks` | 광고 클릭 로그와 연결 *(v1.2)*
| `users` → `ad_clicks`        | 사용자 클릭 기록 *(v1.2)*
| `advertisers` → `ad_reports` | 광고주 리포트 요청 *(v1.2)*
| `ad_campaigns` → `ad_reports`| 캠페인 기준 리포트 요청 *(v1.2)*
| `ad_reports` → `ad_report_results` | 요청 → 결과 연결 *(v1.2)*

---

## 🖼️ ERD 시각 자료
> 아래 PNG 파일은 이 문서를 기반으로 한 시각적 관계도입니다:
> 📁 `/Users/changig/dev/unibase-docs/02_server/assets/unibase_erd_v1.2.png`

![UniBase ERD v1.2](../assets/unibase_erd_v1.2.png)

---

## 📂 문서 저장 위치
```
/Users/changig/dev/unibase-docs/02_server/db/erd_overview.md
```

이 문서는 모든 데이터 구조 및 관계의 기준 참조용 문서로, Cursor 또는 설계자들은 기능별 설계 시 반드시 이 ERD 구조와 정합성을 유지해야 합니다. 