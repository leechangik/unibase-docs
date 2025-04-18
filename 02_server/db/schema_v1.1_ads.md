# 광고 추적 및 캠페인 테이블 설계 (v1.1)

## 목적

이 문서는 UniBase 플랫폼의 광고 관리, 추적 및 캠페인 운영에 필요한 데이터베이스 테이블 구조를 정의합니다. 광고 노출, 클릭, 전환 추적과 수익 관리를 위한 스키마를 제공합니다.

## 목차

- [광고주 테이블](#광고주-테이블)
- [광고 캠페인 테이블](#광고-캠페인-테이블)
- [광고 소재 테이블](#광고-소재-테이블)
- [광고 노출 로그 테이블](#광고-노출-로그-테이블)
- [광고 클릭 로그 테이블](#광고-클릭-로그-테이블)
- [광고 수익 테이블](#광고-수익-테이블)
- [인덱스 전략](#인덱스-전략)

## 광고주 테이블

`advertisers` 테이블은 UniBase 플랫폼에 광고를 게재하는 광고주 정보를 저장합니다.

```sql
CREATE TABLE advertisers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    company_name VARCHAR(100) NOT NULL,
    contact_email VARCHAR(255) NOT NULL,
    contact_phone VARCHAR(50),
    status VARCHAR(20) NOT NULL DEFAULT 'active', -- 'active', 'inactive', 'suspended'
    billing_address TEXT,
    payment_method JSONB DEFAULT NULL,
    balance DECIMAL(12, 2) DEFAULT 0.00,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES users(id) ON DELETE SET NULL,
    updated_by UUID REFERENCES users(id) ON DELETE SET NULL,
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_advertisers_name ON advertisers(name);
CREATE INDEX idx_advertisers_status ON advertisers(status);
```

## 광고 캠페인 테이블

`ad_campaigns` 테이블은 광고주가 생성한 광고 캠페인 정보를 저장합니다.

```sql
CREATE TABLE ad_campaigns (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    advertiser_id UUID NOT NULL REFERENCES advertisers(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    status VARCHAR(20) NOT NULL DEFAULT 'draft', -- 'draft', 'active', 'paused', 'completed', 'canceled'
    start_date TIMESTAMP WITH TIME ZONE NOT NULL,
    end_date TIMESTAMP WITH TIME ZONE,
    daily_budget DECIMAL(10, 2),
    total_budget DECIMAL(12, 2) NOT NULL,
    remaining_budget DECIMAL(12, 2) NOT NULL,
    target_apps JSONB, -- 특정 앱 타겟팅 설정
    target_demographics JSONB, -- 인구통계학적 타겟팅 설정
    target_platforms VARCHAR[] DEFAULT ARRAY['android', 'ios', 'web'], -- 타겟 플랫폼
    pricing_model VARCHAR(20) NOT NULL DEFAULT 'cpc', -- 'cpc', 'cpm', 'cpa'
    bid_amount DECIMAL(10, 4) NOT NULL, -- CPC/CPM/CPA 입찰가
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES users(id) ON DELETE SET NULL,
    updated_by UUID REFERENCES users(id) ON DELETE SET NULL,
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_ad_campaigns_advertiser_id ON ad_campaigns(advertiser_id);
CREATE INDEX idx_ad_campaigns_status ON ad_campaigns(status);
CREATE INDEX idx_ad_campaigns_dates ON ad_campaigns(start_date, end_date);
```

## 광고 소재 테이블

`ad_creatives` 테이블은 광고 캠페인에 사용되는 소재(이미지, 텍스트 등)를 저장합니다.

```sql
CREATE TABLE ad_creatives (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    campaign_id UUID NOT NULL REFERENCES ad_campaigns(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(20) NOT NULL, -- 'banner', 'interstitial', 'native', 'video'
    format VARCHAR(20) NOT NULL, -- 'image', 'video', 'html', 'text'
    content_url VARCHAR(255), -- 이미지/비디오 URL
    thumbnail_url VARCHAR(255), -- 썸네일 URL
    headline VARCHAR(100), -- 광고 제목
    description TEXT, -- 광고 설명
    call_to_action VARCHAR(50), -- 행동 유도 버튼 텍스트
    landing_url VARCHAR(255) NOT NULL, -- 랜딩 페이지 URL
    width INTEGER, -- 광고 너비(픽셀)
    height INTEGER, -- 광고 높이(픽셀)
    status VARCHAR(20) NOT NULL DEFAULT 'active', -- 'active', 'inactive'
    approval_status VARCHAR(20) NOT NULL DEFAULT 'pending', -- 'pending', 'approved', 'rejected'
    rejection_reason TEXT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES users(id) ON DELETE SET NULL,
    updated_by UUID REFERENCES users(id) ON DELETE SET NULL,
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_ad_creatives_campaign_id ON ad_creatives(campaign_id);
CREATE INDEX idx_ad_creatives_type ON ad_creatives(type);
CREATE INDEX idx_ad_creatives_status ON ad_creatives(status);
CREATE INDEX idx_ad_creatives_approval ON ad_creatives(approval_status);
```

## 광고 노출 로그 테이블

`ad_impressions` 테이블은 광고 노출 이벤트를 로깅합니다.

```sql
CREATE TABLE ad_impressions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    creative_id UUID NOT NULL REFERENCES ad_creatives(id),
    campaign_id UUID NOT NULL REFERENCES ad_campaigns(id),
    app_id UUID REFERENCES applications(id),
    user_id UUID REFERENCES users(id),
    session_id VARCHAR(100),
    device_id VARCHAR(100),
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    platform VARCHAR(20) NOT NULL, -- 'android', 'ios', 'web'
    device_type VARCHAR(20) NOT NULL, -- 'mobile', 'tablet', 'desktop'
    ip_address VARCHAR(45),
    country_code VARCHAR(2),
    city VARCHAR(100),
    impression_cost DECIMAL(10, 6) NOT NULL, -- 노출당 비용
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_ad_impressions_creative_campaign ON ad_impressions(creative_id, campaign_id);
CREATE INDEX idx_ad_impressions_app_id ON ad_impressions(app_id);
CREATE INDEX idx_ad_impressions_user_id ON ad_impressions(user_id);
CREATE INDEX idx_ad_impressions_timestamp ON ad_impressions(timestamp);
CREATE INDEX idx_ad_impressions_platform ON ad_impressions(platform);
CREATE INDEX idx_ad_impressions_country ON ad_impressions(country_code);
```

## 광고 클릭 로그 테이블

`ad_clicks` 테이블은 광고 클릭 이벤트를 로깅합니다.

```sql
CREATE TABLE ad_clicks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    impression_id UUID REFERENCES ad_impressions(id),
    creative_id UUID NOT NULL REFERENCES ad_creatives(id),
    campaign_id UUID NOT NULL REFERENCES ad_campaigns(id),
    app_id UUID REFERENCES applications(id),
    user_id UUID REFERENCES users(id),
    session_id VARCHAR(100),
    device_id VARCHAR(100),
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    platform VARCHAR(20) NOT NULL, -- 'android', 'ios', 'web'
    device_type VARCHAR(20) NOT NULL, -- 'mobile', 'tablet', 'desktop'
    ip_address VARCHAR(45),
    country_code VARCHAR(2),
    city VARCHAR(100),
    click_cost DECIMAL(10, 6) NOT NULL, -- 클릭당 비용
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_ad_clicks_creative_campaign ON ad_clicks(creative_id, campaign_id);
CREATE INDEX idx_ad_clicks_impression_id ON ad_clicks(impression_id);
CREATE INDEX idx_ad_clicks_app_id ON ad_clicks(app_id);
CREATE INDEX idx_ad_clicks_user_id ON ad_clicks(user_id);
CREATE INDEX idx_ad_clicks_timestamp ON ad_clicks(timestamp);
CREATE INDEX idx_ad_clicks_country ON ad_clicks(country_code);
```

## 광고 수익 테이블

`ad_revenue` 테이블은 앱별 광고 수익을 기록합니다.

```sql
CREATE TABLE ad_revenue (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    app_id UUID NOT NULL REFERENCES applications(id),
    date DATE NOT NULL,
    impressions INTEGER NOT NULL DEFAULT 0,
    clicks INTEGER NOT NULL DEFAULT 0,
    revenue_gross DECIMAL(12, 4) NOT NULL DEFAULT 0,
    revenue_net DECIMAL(12, 4) NOT NULL DEFAULT 0, -- 플랫폼 수수료 제외 순수익
    currency VARCHAR(3) NOT NULL DEFAULT 'USD',
    platform_fee DECIMAL(5, 2) NOT NULL DEFAULT 30.00, -- 플랫폼 수수료 (%)
    source VARCHAR(50) NOT NULL, -- 'admob', 'facebook', 'applovin', 'unity', 'internal'
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE UNIQUE INDEX idx_ad_revenue_app_date_source ON ad_revenue(app_id, date, source);
CREATE INDEX idx_ad_revenue_date ON ad_revenue(date);
CREATE INDEX idx_ad_revenue_app_id ON ad_revenue(app_id);
```

## 인덱스 전략

광고 데이터 특성을 고려한 인덱스 전략:

1. **시간 기반 쿼리 최적화**: 광고 노출 및 클릭은 시간 기반 조회가 많으므로 timestamp에 인덱스 적용
2. **조인 최적화**: 캠페인, 광고소재 간 조인이 빈번하므로 외래 키에 인덱스 설정
3. **분석 쿼리 지원**: 국가, 플랫폼별 분석을 위한 인덱스 설정
4. **보고서 최적화**: ad_revenue 테이블에서 앱별, 날짜별 보고서를 위한 복합 인덱스 설정

## 참고 사항

1. 대용량 로그 테이블(`ad_impressions`, `ad_clicks`)의 경우, 일정 기간 후 시계열 파티셔닝이나 롤업 전략 적용 권장
2. 광고 노출 및 클릭 데이터는 성능을 위해 TimescaleDB와 같은 시계열 데이터베이스 활용 고려
3. 광고 통계 분석을 위한 뷰나 구체화된 뷰 별도 설계 필요
4. 개인정보 보호 관련 필드는 암호화 또는 익명화 처리 권장 