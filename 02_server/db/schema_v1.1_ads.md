# 📘 UniBase 광고 추적 테이블 설계 (`schema_v1.1_ads.md`)

## 문서 목적
이 문서는 광고주 등록, 캠페인 관리, 광고 노출 기록을 관리하는 **광고 시스템용 테이블 설계서**입니다. 광고주는 앱에 광고를 요청하고, 플랫폼은 노출 정보를 추적하여 광고주에게 리포트를 제공합니다.

---

## 📊 테이블 설계

### 1. `advertisers` (광고주 정보)
| 컬럼명         | 타입         | 제약조건        | 설명             |
|----------------|--------------|-----------------|------------------|
| advertiser_id  | BIGINT       | PK, AUTO_INC    | 광고주 고유 ID   |
| company_name   | VARCHAR(255) | NOT NULL        | 회사명           |
| contact_email  | VARCHAR(255) | NOT NULL        | 담당 이메일      |
| created_at     | DATETIME     | DEFAULT CURRENT_TIMESTAMP | 생성일 |
| updated_at     | DATETIME     | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | 수정일 |

### 2. `ad_campaigns` (광고 캠페인)
| 컬럼명         | 타입         | 제약조건        | 설명                   |
|----------------|--------------|-----------------|------------------------|
| campaign_id    | BIGINT       | PK, AUTO_INC    | 캠페인 고유 ID         |
| advertiser_id  | BIGINT       | FK              | 광고주 ID (연결)        |
| app_id         | BIGINT       | FK              | 광고 대상 앱 ID         |
| campaign_name  | VARCHAR(255) | NOT NULL        | 캠페인명                |
| start_date     | DATETIME     |                 | 캠페인 시작일           |
| end_date       | DATETIME     |                 | 캠페인 종료일           |
| budget         | DECIMAL(18,2)|                 | 예산                   |
| created_at     | DATETIME     | DEFAULT CURRENT_TIMESTAMP | 생성일 |
| updated_at     | DATETIME     | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | 수정일 |

### 3. `ad_impressions` (광고 노출 기록)
| 컬럼명         | 타입         | 제약조건        | 설명                         |
|----------------|--------------|-----------------|------------------------------|
| impression_id  | BIGINT       | PK, AUTO_INC    | 광고 노출 고유 ID            |
| campaign_id    | BIGINT       | FK              | 연결된 캠페인 ID             |
| app_id         | BIGINT       | FK              | 앱 ID                        |
| user_id        | BIGINT       |                 | 사용자 ID (로그인 유저 한정) |
| impression_time| DATETIME     | DEFAULT CURRENT_TIMESTAMP | 노출 시각         |
| metadata       | JSON         |                 | 노출 환경 및 위치 등 부가 정보 |

---

## 🔗 테이블 관계
- `advertisers` → `ad_campaigns`: 1:N 관계
- `ad_campaigns` → `ad_impressions`: 1:N 관계
- `ad_campaigns.app_id` → `apps.app_id`: 외래키 참조

---

## 🧩 활용 예시
- 광고주는 하나 이상의 캠페인을 등록하고, 각 캠페인은 특정 앱에서 노출됨
- 앱 내에서 광고가 노출될 때마다 `ad_impressions`에 기록되어 광고주가 추적 가능함
- 추후 리포팅 및 정산 기능 개발을 위한 기초 데이터로 활용됨

---

## 📂 위치
> 이 문서는 `/Users/changig/dev/unibase-docs/02_server/db/schema_v1.1_ads.md` 경로에 존재하며, Cursor 및 GPT는 광고 관련 자동화 작업 시 반드시 참조해야 합니다. 