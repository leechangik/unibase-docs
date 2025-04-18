# 사용자 활동 로그 테이블 설계 (v1.1)

## 목적

이 문서는 UniBase 플랫폼의 사용자 활동 추적과 분석을 위한 데이터베이스 테이블 구조를 정의합니다. 사용자 행동 데이터 수집, 실시간 모니터링, 성능 분석, 오류 추적을 위한 스키마를 제공합니다.

## 목차

- [앱 이벤트 로그 테이블](#앱-이벤트-로그-테이블)
- [사용자 세션 테이블](#사용자-세션-테이블)
- [오류 로그 테이블](#오류-로그-테이블)
- [성능 메트릭 테이블](#성능-메트릭-테이블)
- [푸시 알림 로그 테이블](#푸시-알림-로그-테이블)
- [인덱스 및 파티셔닝 전략](#인덱스-및-파티셔닝-전략)
- [로그 보존 정책](#로그-보존-정책)

## 앱 이벤트 로그 테이블

`app_event_logs` 테이블은 애플리케이션 내 사용자 활동과 이벤트를 기록합니다.

```sql
CREATE TABLE app_event_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    app_id UUID REFERENCES applications(id) ON DELETE SET NULL,
    module_id UUID REFERENCES modules(id) ON DELETE SET NULL,
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    session_id VARCHAR(100),
    event_type VARCHAR(50) NOT NULL, -- 'page_view', 'button_click', 'form_submit', 'purchase', 'feature_usage'
    event_name VARCHAR(100) NOT NULL,
    event_data JSONB NOT NULL DEFAULT '{}'::jsonb, -- 이벤트 관련 상세 데이터
    device_type VARCHAR(50), -- 'mobile', 'tablet', 'desktop'
    platform VARCHAR(50), -- 'android', 'ios', 'web'
    platform_version VARCHAR(50), -- OS 또는 브라우저 버전
    app_version VARCHAR(50), -- 앱 버전
    ip_address VARCHAR(45), -- IPv6 지원 크기
    country_code VARCHAR(2),
    city VARCHAR(100),
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_app_event_logs_app_id ON app_event_logs(app_id);
CREATE INDEX idx_app_event_logs_user_id ON app_event_logs(user_id);
CREATE INDEX idx_app_event_logs_event_type ON app_event_logs(event_type);
CREATE INDEX idx_app_event_logs_timestamp ON app_event_logs(timestamp);
CREATE INDEX idx_app_event_logs_session_id ON app_event_logs(session_id);
CREATE INDEX idx_app_event_logs_event_name ON app_event_logs(event_name);
CREATE INDEX idx_app_event_logs_platform ON app_event_logs(platform);
```

## 사용자 세션 테이블

`user_sessions` 테이블은 사용자 세션 정보를 저장합니다.

```sql
CREATE TABLE user_sessions (
    id VARCHAR(100) PRIMARY KEY, -- 세션 식별자
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    app_id UUID REFERENCES applications(id) ON DELETE SET NULL,
    device_id VARCHAR(100), -- 기기 식별자
    device_type VARCHAR(50), -- 'mobile', 'tablet', 'desktop'
    platform VARCHAR(50), -- 'android', 'ios', 'web'
    platform_version VARCHAR(50), -- OS 또는 브라우저 버전
    app_version VARCHAR(50), -- 앱 버전
    ip_address VARCHAR(45), -- IPv6 지원 크기
    country_code VARCHAR(2),
    city VARCHAR(100),
    start_time TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    end_time TIMESTAMP WITH TIME ZONE,
    duration_seconds INTEGER, -- 세션 지속 시간(초)
    page_views INTEGER DEFAULT 0, -- 페이지뷰 수
    events_count INTEGER DEFAULT 0, -- 이벤트 발생 수
    referrer_url TEXT, -- 유입 경로 URL
    user_agent TEXT, -- 브라우저/클라이언트 정보
    is_active BOOLEAN DEFAULT TRUE -- 세션 활성 상태
);

CREATE INDEX idx_user_sessions_user_id ON user_sessions(user_id);
CREATE INDEX idx_user_sessions_app_id ON user_sessions(app_id);
CREATE INDEX idx_user_sessions_start_time ON user_sessions(start_time);
CREATE INDEX idx_user_sessions_end_time ON user_sessions(end_time);
CREATE INDEX idx_user_sessions_is_active ON user_sessions(is_active);
CREATE INDEX idx_user_sessions_platform ON user_sessions(platform);
```

## 오류 로그 테이블

`error_logs` 테이블은 애플리케이션 오류 및 예외를 기록합니다.

```sql
CREATE TABLE error_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    app_id UUID REFERENCES applications(id) ON DELETE SET NULL,
    module_id UUID REFERENCES modules(id) ON DELETE SET NULL,
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    session_id VARCHAR(100),
    error_code VARCHAR(100),
    error_type VARCHAR(100) NOT NULL, -- 'api_error', 'client_error', 'network_error', 'permission_error'
    error_message TEXT NOT NULL,
    error_details JSONB, -- 오류 관련 상세 정보 (스택 트레이스 등)
    component VARCHAR(100), -- 오류 발생 컴포넌트
    severity VARCHAR(20) NOT NULL, -- 'low', 'medium', 'high', 'critical'
    device_type VARCHAR(50), -- 'mobile', 'tablet', 'desktop'
    platform VARCHAR(50), -- 'android', 'ios', 'web'
    platform_version VARCHAR(50), -- OS 또는 브라우저 버전
    app_version VARCHAR(50), -- 앱 버전
    ip_address VARCHAR(45), -- IPv6 지원 크기
    country_code VARCHAR(2),
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_resolved BOOLEAN DEFAULT FALSE, -- 해결 여부
    resolved_at TIMESTAMP WITH TIME ZONE, -- 해결 시간
    resolved_by UUID REFERENCES users(id) ON DELETE SET NULL, -- 해결한 관리자
    resolution_notes TEXT -- 해결 방법 메모
);

CREATE INDEX idx_error_logs_app_id ON error_logs(app_id);
CREATE INDEX idx_error_logs_user_id ON error_logs(user_id);
CREATE INDEX idx_error_logs_error_type ON error_logs(error_type);
CREATE INDEX idx_error_logs_severity ON error_logs(severity);
CREATE INDEX idx_error_logs_timestamp ON error_logs(timestamp);
CREATE INDEX idx_error_logs_is_resolved ON error_logs(is_resolved);
CREATE INDEX idx_error_logs_session_id ON error_logs(session_id);
```

## 성능 메트릭 테이블

`performance_metrics` 테이블은 앱 성능 측정 데이터를 저장합니다.

```sql
CREATE TABLE performance_metrics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    app_id UUID REFERENCES applications(id) ON DELETE SET NULL,
    module_id UUID REFERENCES modules(id) ON DELETE SET NULL,
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    session_id VARCHAR(100),
    metric_type VARCHAR(50) NOT NULL, -- 'page_load', 'api_response', 'render_time', 'memory_usage'
    metric_name VARCHAR(100) NOT NULL,
    value DECIMAL(10, 3) NOT NULL, -- 측정값
    unit VARCHAR(20) NOT NULL, -- 'ms', 'bytes', 's', 'percent'
    context JSONB, -- 측정 컨텍스트(페이지, API 등)
    device_type VARCHAR(50), -- 'mobile', 'tablet', 'desktop'
    platform VARCHAR(50), -- 'android', 'ios', 'web'
    platform_version VARCHAR(50), -- OS 또는 브라우저 버전
    app_version VARCHAR(50), -- 앱 버전
    network_type VARCHAR(20), -- 'wifi', 'cellular', 'ethernet'
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_performance_metrics_app_id ON performance_metrics(app_id);
CREATE INDEX idx_performance_metrics_metric_type ON performance_metrics(metric_type);
CREATE INDEX idx_performance_metrics_timestamp ON performance_metrics(timestamp);
CREATE INDEX idx_performance_metrics_session_id ON performance_metrics(session_id);
CREATE INDEX idx_performance_metrics_platform ON performance_metrics(platform);
```

## 푸시 알림 로그 테이블

`push_notification_logs` 테이블은 푸시 알림 발송 및 상호작용 데이터를 기록합니다.

```sql
CREATE TABLE push_notification_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    notification_id VARCHAR(100) NOT NULL, -- 알림 식별자
    app_id UUID REFERENCES applications(id) ON DELETE SET NULL,
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    device_id VARCHAR(100), -- 기기 식별자
    title VARCHAR(200) NOT NULL,
    body TEXT NOT NULL,
    data JSONB, -- 알림 데이터 페이로드
    status VARCHAR(20) NOT NULL, -- 'sent', 'delivered', 'failed', 'opened', 'ignored'
    sent_at TIMESTAMP WITH TIME ZONE, -- 발송 시간
    delivered_at TIMESTAMP WITH TIME ZONE, -- 전달 시간
    opened_at TIMESTAMP WITH TIME ZONE, -- 열람 시간
    error_message TEXT, -- 실패 시 오류 메시지
    channel VARCHAR(50), -- 'fcm', 'apns', 'web'
    platform VARCHAR(50), -- 'android', 'ios', 'web'
    campaign_id VARCHAR(100), -- 캠페인 식별자
    notification_type VARCHAR(50), -- 'marketing', 'transactional', 'reminder'
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_push_notification_logs_user_id ON push_notification_logs(user_id);
CREATE INDEX idx_push_notification_logs_app_id ON push_notification_logs(app_id);
CREATE INDEX idx_push_notification_logs_status ON push_notification_logs(status);
CREATE INDEX idx_push_notification_logs_sent_at ON push_notification_logs(sent_at);
CREATE INDEX idx_push_notification_logs_notification_type ON push_notification_logs(notification_type);
CREATE INDEX idx_push_notification_logs_campaign_id ON push_notification_logs(campaign_id);
```

## 인덱스 및 파티셔닝 전략

로그 테이블은 시간이 지남에 따라 대용량 데이터를 저장하게 됩니다. 효율적인 관리를 위한 전략:

### 파티셔닝

대용량 로그 테이블(`app_event_logs`, `error_logs`, `performance_metrics`)에 시간 기반 파티셔닝 적용:

```sql
-- 파티셔닝된 앱 이벤트 로그 테이블 예시
CREATE TABLE app_event_logs (
    id UUID NOT NULL,
    -- 다른 필드들
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
) PARTITION BY RANGE (timestamp);

-- 월별 파티션 생성 예시
CREATE TABLE app_event_logs_y2023m01 PARTITION OF app_event_logs
    FOR VALUES FROM ('2023-01-01') TO ('2023-02-01');
    
CREATE TABLE app_event_logs_y2023m02 PARTITION OF app_event_logs
    FOR VALUES FROM ('2023-02-01') TO ('2023-03-01');
```

### 인덱스 최적화

1. 모든 로그 테이블에 timestamp 인덱스 적용
2. 자주 사용되는 필터링 필드(user_id, app_id, session_id)에 인덱스 적용
3. 필요에 따라 복합 인덱스 적용
4. 각 파티션에 개별 인덱스 생성 고려

## 로그 보존 정책

데이터 증가 관리 및 규정 준수를 위한 보존 정책:

1. **상세 로그**: 3~6개월 보관 (app_event_logs, error_logs, performance_metrics)
2. **집계 데이터**: 무기한 보관 (일/주/월별 집계 테이블 별도 생성)
3. **자동화된 보관 처리**: 
   - 오래된 데이터는 저비용 스토리지로 자동 아카이빙
   - 규정에 따라 필요 시 영구 삭제 스크립트 적용
4. **개인정보 처리**:
   - 필요에 따라 익명화 처리
   - 개인 식별 정보는 해당 지역 규정(GDPR, CCPA 등)에 따라 처리

## 집계 및 분석 테이블

효율적인 분석을 위한 집계 테이블 권장:

```sql
-- 일별 앱 사용 통계 집계 테이블 예시
CREATE TABLE daily_app_usage_stats (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    app_id UUID REFERENCES applications(id) ON DELETE SET NULL,
    date DATE NOT NULL,
    active_users_count INTEGER NOT NULL DEFAULT 0,
    new_users_count INTEGER NOT NULL DEFAULT 0,
    sessions_count INTEGER NOT NULL DEFAULT 0,
    avg_session_duration DECIMAL(10, 2) NOT NULL DEFAULT 0,
    total_events_count INTEGER NOT NULL DEFAULT 0,
    error_count INTEGER NOT NULL DEFAULT 0,
    platform_stats JSONB, -- 플랫폼별 통계
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE UNIQUE INDEX idx_daily_app_usage_stats_app_date ON daily_app_usage_stats(app_id, date);
```

## 참고 사항

1. 로그 데이터의 빠른 수집을 위해 벌크 삽입 메커니즘 구현 권장
2. 데이터 볼륨이 매우 큰 경우 전용 로깅 데이터베이스(Elasticsearch, ClickHouse 등) 고려
3. 실시간 모니터링이 필요한 경우 Redis와 같은 인메모리 데이터베이스를 캐싱 레이어로 활용
4. 민감한 개인 식별 정보는 암호화 또는 익명화 처리를 권장 