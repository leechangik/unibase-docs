# 📊 UniBase 광고 리포트 DB 설계 (`schema_v1.2_ads_report.md`)

## 문서 목적
이 문서는 광고 클릭, 리포트 생성 요청, 리포트 결과 저장에 대한 구조를 정의합니다. 향후 광고주가 다양한 기준(날짜, 앱, 유저 등)으로 보고서를 요청하고, 그 결과를 API 또는 UI를 통해 확인하는 기능을 확장할 수 있는 기반입니다.

---

## 📘 테이블 설계

### 1. `ad_clicks` (광고 클릭 기록)
| 컬럼명       | 타입         | 제약조건        | 설명                       |
|--------------|--------------|-----------------|----------------------------|
| click_id     | BIGINT       | PK, AUTO_INC    | 클릭 고유 ID               |
| impression_id| BIGINT       | FK              | 연결된 노출 로그 ID         |
| user_id      | BIGINT       | FK              | 클릭한 사용자 ID (옵션)    |
| click_time   | DATETIME     | DEFAULT CURRENT_TIMESTAMP | 클릭 시각    |
| metadata     | JSON         |                 | 위치, 디바이스 등 추가정보 |

### 2. `ad_reports` (광고 리포트 요청)
| 컬럼명       | 타입         | 제약조건        | 설명                           |
|--------------|--------------|-----------------|----------------------------------|
| report_id    | BIGINT       | PK, AUTO_INC    | 리포트 요청 고유 ID             |
| advertiser_id| BIGINT       | FK              | 광고주 ID                        |
| campaign_id  | BIGINT       | FK              | 대상 캠페인 ID (NULL 허용)       |
| report_type  | VARCHAR(50)  | NOT NULL        | DAILY, WEEKLY, CUSTOM 등         |
| date_range   | JSON         |                 | 시작일/종료일 포함 JSON (ISO 형식) |
| status       | VARCHAR(20)  | DEFAULT 'PENDING' | 상태 (PENDING, COMPLETED, FAILED) |
| requested_at | DATETIME     | DEFAULT CURRENT_TIMESTAMP | 요청 시각 |
| completed_at | DATETIME     |                 | 완료 시각 (완료 시 기록)         |
| metadata     | JSON         |                 | 기타 확장 설정                   |

### 3. `ad_report_results` (리포트 결과 요약)
| 컬럼명       | 타입         | 제약조건        | 설명                             |
|--------------|--------------|-----------------|----------------------------------|
| result_id    | BIGINT       | PK, AUTO_INC    | 결과 고유 ID                     |
| report_id    | BIGINT       | FK              | 연결된 리포트 ID                 |
| summary_json | JSON         |                 | 요약 리포트 (ex. CTR, 노출수 등) |
| exported_url | VARCHAR(255) |                 | 외부 저장소 경로 (PDF, CSV 등)   |
| generated_at | DATETIME     | DEFAULT CURRENT_TIMESTAMP | 생성 시각 |

---

## 🔐 확장 고려 사항
- 리포트 결과는 외부 파일 URL 또는 API 응답용 JSON 구조로 모두 사용 가능
- 향후 `ad_report_logs` 테이블로 리포트 생성 실패/재시도 히스토리 추가 가능
- `metadata` 필드는 리포트 필터 기준 등을 자유롭게 확장 가능

---

## 🔗 관계 요약
- `ad_clicks` → `ad_impressions` (1:N)
- `ad_reports` → `advertisers`, `ad_campaigns` (1:N)
- `ad_report_results` → `ad_reports` (1:1)

---

## 📂 저장 경로
```
/Users/changig/dev/unibase-docs/02_server/db/schema_v1.2_ads_report.md
``` 