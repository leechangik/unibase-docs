# 관리자 시스템 테이블 설계 (v1.1)

## 목적

이 문서는 UniBase 플랫폼의 관리자 시스템에 필요한 데이터베이스 테이블 구조를 정의합니다. 관리자 계정, 권한, 활동 로그 및 시스템 설정 관리를 위한 스키마를 제공합니다.

## 목차

- [관리자 계정 테이블](#관리자-계정-테이블)
- [관리자 역할 테이블](#관리자-역할-테이블)
- [관리자 권한 테이블](#관리자-권한-테이블)
- [역할-권한 매핑 테이블](#역할-권한-매핑-테이블)
- [관리자 활동 로그 테이블](#관리자-활동-로그-테이블)
- [시스템 설정 테이블](#시스템-설정-테이블)
- [공지사항 테이블](#공지사항-테이블)
- [관계 다이어그램](#관계-다이어그램)

## 관리자 계정 테이블

`admin_users` 테이블은 플랫폼 관리자 정보를 저장합니다.

```sql
CREATE TABLE admin_users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    phone_number VARCHAR(50),
    status VARCHAR(20) NOT NULL DEFAULT 'active', -- 'active', 'inactive', 'suspended'
    last_login_at TIMESTAMP WITH TIME ZONE,
    login_attempts INTEGER NOT NULL DEFAULT 0,
    locked_until TIMESTAMP WITH TIME ZONE,
    require_password_change BOOLEAN NOT NULL DEFAULT false,
    two_factor_enabled BOOLEAN NOT NULL DEFAULT false,
    two_factor_secret VARCHAR(100),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES admin_users(id) ON DELETE SET NULL,
    updated_by UUID REFERENCES admin_users(id) ON DELETE SET NULL,
    profile_image_url VARCHAR(255),
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_admin_users_email ON admin_users(email);
CREATE INDEX idx_admin_users_status ON admin_users(status);
```

## 관리자 역할 테이블

`admin_roles` 테이블은 관리자 역할 정의를 저장합니다.

```sql
CREATE TABLE admin_roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) NOT NULL UNIQUE,
    display_name VARCHAR(100) NOT NULL,
    description TEXT,
    is_system BOOLEAN NOT NULL DEFAULT false, -- 시스템 기본 역할 여부
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES admin_users(id) ON DELETE SET NULL,
    updated_by UUID REFERENCES admin_users(id) ON DELETE SET NULL,
    metadata JSONB DEFAULT '{}'::jsonb
);

-- 기본 역할 삽입
INSERT INTO admin_roles (name, display_name, description, is_system)
VALUES 
    ('super_admin', '최고 관리자', '모든 권한을 가진 슈퍼 관리자', true),
    ('admin', '관리자', '대부분의 관리 기능에 접근 가능한 일반 관리자', true),
    ('content_manager', '콘텐츠 관리자', '콘텐츠 관리 기능만 접근 가능', true),
    ('support', '고객 지원', '고객 지원 기능만 접근 가능', true),
    ('read_only', '읽기 전용', '읽기 전용 접근 권한', true);

CREATE INDEX idx_admin_roles_name ON admin_roles(name);
```

## 관리자 권한 테이블

`admin_permissions` 테이블은 시스템 내 개별 권한을 정의합니다.

```sql
CREATE TABLE admin_permissions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL UNIQUE,
    display_name VARCHAR(100) NOT NULL,
    description TEXT,
    category VARCHAR(50) NOT NULL, -- 'users', 'apps', 'content', 'system', 'reports'
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- 기본 권한 삽입 (일부 예시)
INSERT INTO admin_permissions (name, display_name, description, category)
VALUES 
    ('manage_users', '사용자 관리', '사용자 계정 생성, 수정, 삭제', 'users'),
    ('view_users', '사용자 보기', '사용자 목록 및 정보 조회', 'users'),
    ('manage_apps', '앱 관리', '앱 생성, 수정, 삭제', 'apps'),
    ('view_apps', '앱 보기', '앱 목록 및 정보 조회', 'apps'),
    ('manage_modules', '모듈 관리', '모듈 구성 및 설정', 'apps'),
    ('manage_content', '콘텐츠 관리', '콘텐츠 생성, 수정, 삭제', 'content'),
    ('view_analytics', '분석 보기', '분석 데이터 조회', 'reports'),
    ('manage_system', '시스템 설정', '시스템 설정 변경', 'system');

CREATE INDEX idx_admin_permissions_category ON admin_permissions(category);
```

## 관리자-역할 매핑 테이블

`admin_user_roles` 테이블은 관리자와 역할 간의 매핑을 저장합니다.

```sql
CREATE TABLE admin_user_roles (
    admin_user_id UUID NOT NULL REFERENCES admin_users(id) ON DELETE CASCADE,
    role_id UUID NOT NULL REFERENCES admin_roles(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES admin_users(id) ON DELETE SET NULL,
    PRIMARY KEY (admin_user_id, role_id)
);

CREATE INDEX idx_admin_user_roles_user_id ON admin_user_roles(admin_user_id);
CREATE INDEX idx_admin_user_roles_role_id ON admin_user_roles(role_id);
```

## 역할-권한 매핑 테이블

`admin_role_permissions` 테이블은 역할과 권한 간의 매핑을 저장합니다.

```sql
CREATE TABLE admin_role_permissions (
    role_id UUID NOT NULL REFERENCES admin_roles(id) ON DELETE CASCADE,
    permission_id UUID NOT NULL REFERENCES admin_permissions(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES admin_users(id) ON DELETE SET NULL,
    PRIMARY KEY (role_id, permission_id)
);

CREATE INDEX idx_admin_role_permissions_role_id ON admin_role_permissions(role_id);
CREATE INDEX idx_admin_role_permissions_permission_id ON admin_role_permissions(permission_id);
```

## 관리자 활동 로그 테이블

`admin_activity_logs` 테이블은 관리자의 모든 활동을 로깅합니다.

```sql
CREATE TABLE admin_activity_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    admin_user_id UUID REFERENCES admin_users(id) ON DELETE SET NULL,
    action VARCHAR(100) NOT NULL, -- 'login', 'logout', 'create', 'update', 'delete', 'view'
    entity_type VARCHAR(50) NOT NULL, -- 'user', 'app', 'module', 'system_setting'
    entity_id UUID, -- 대상 엔티티 ID
    description TEXT NOT NULL, -- 활동 상세 설명
    ip_address VARCHAR(45), -- IPv6 지원 크기
    user_agent TEXT, -- 브라우저/클라이언트 정보
    status VARCHAR(20) NOT NULL, -- 'success', 'failure'
    details JSONB, -- 추가 상세 정보 (변경 전/후 데이터 등)
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_admin_activity_logs_user_id ON admin_activity_logs(admin_user_id);
CREATE INDEX idx_admin_activity_logs_action ON admin_activity_logs(action);
CREATE INDEX idx_admin_activity_logs_entity ON admin_activity_logs(entity_type, entity_id);
CREATE INDEX idx_admin_activity_logs_created_at ON admin_activity_logs(created_at);
```

## 시스템 설정 테이블

`system_settings` 테이블은 플랫폼 시스템 설정을 저장합니다.

```sql
CREATE TABLE system_settings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    key VARCHAR(100) NOT NULL UNIQUE,
    value TEXT NOT NULL,
    data_type VARCHAR(20) NOT NULL, -- 'string', 'number', 'boolean', 'json'
    category VARCHAR(50) NOT NULL, -- 'security', 'email', 'notification', 'appearance'
    display_name VARCHAR(100) NOT NULL,
    description TEXT,
    is_public BOOLEAN NOT NULL DEFAULT false, -- 클라이언트에 노출 가능 여부
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_by UUID REFERENCES admin_users(id) ON DELETE SET NULL
);

-- 기본 시스템 설정 삽입
INSERT INTO system_settings (key, value, data_type, category, display_name, description, is_public)
VALUES 
    ('max_login_attempts', '5', 'number', 'security', '최대 로그인 시도 횟수', '계정 잠금 전 최대 로그인 시도 횟수', false),
    ('account_lockout_duration', '30', 'number', 'security', '계정 잠금 기간(분)', '로그인 실패 후 계정 잠금 기간(분)', false),
    ('password_expiry_days', '90', 'number', 'security', '비밀번호 만료 기간(일)', '비밀번호 변경 요구 전 기간(일)', false),
    ('platform_name', 'UniBase Platform', 'string', 'appearance', '플랫폼 이름', '관리자 대시보드에 표시되는 플랫폼 이름', true),
    ('support_email', 'support@unibase.example.com', 'string', 'email', '지원 이메일', '사용자 지원을 위한 이메일 주소', true);

CREATE INDEX idx_system_settings_category ON system_settings(category);
CREATE INDEX idx_system_settings_is_public ON system_settings(is_public);
```

## 공지사항 테이블

`announcements` 테이블은 관리자가 작성한 시스템 공지사항을 저장합니다.

```sql
CREATE TABLE announcements (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(200) NOT NULL,
    content TEXT NOT NULL,
    start_date TIMESTAMP WITH TIME ZONE NOT NULL,
    end_date TIMESTAMP WITH TIME ZONE,
    is_active BOOLEAN NOT NULL DEFAULT true,
    priority VARCHAR(20) NOT NULL DEFAULT 'normal', -- 'low', 'normal', 'high', 'urgent'
    target_audience VARCHAR(20) NOT NULL DEFAULT 'all', -- 'all', 'admin', 'user'
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES admin_users(id) ON DELETE SET NULL,
    updated_by UUID REFERENCES admin_users(id) ON DELETE SET NULL
);

CREATE INDEX idx_announcements_active_dates ON announcements(is_active, start_date, end_date);
CREATE INDEX idx_announcements_priority ON announcements(priority);
CREATE INDEX idx_announcements_target ON announcements(target_audience);
```

## 관계 다이어그램

관리자 시스템 테이블의 관계 다이어그램은 다음과 같습니다:

```
admin_users <─┬──── admin_user_roles ───┬─> admin_roles <─┬── admin_role_permissions ───> admin_permissions
              │                          │                 │
              │                          │                 │
              ▼                          │                 │
    admin_activity_logs                  │                 │
              ▲                          │                 │
              │                          │                 │
              │                          │                 │
system_settings ───┐                     │                 │
                   │                     │                 │
                   │                     │                 │
                   └─────────────────────┴─────────────────┘
                                  │
                                  │
                                  ▼
                            announcements
```

## 참고 사항

1. 관리자 권한 체계는 RBAC(Role-Based Access Control) 모델을 따릅니다.
2. 모든 관리자 활동은 `admin_activity_logs`에 기록하여 감사 추적을 지원합니다.
3. 시스템 설정은 카테고리별로 분류하여 관리합니다.
4. 보안 관련 설정(암호화 키, API 키 등)은 별도의 안전한 저장소를 사용하는 것을 권장합니다. 