# AWS 리소스 관리 테이블 설계 (v1.1)

## 목적

이 문서는 UniBase 플랫폼의 AWS 인프라 관리를 위한 데이터베이스 테이블 구조를 정의합니다. 클라우드 리소스 추적, 비용 모니터링, 구성 관리 및 자동화를 위한 스키마를 제공합니다.

## 목차

- [AWS 리소스 테이블](#aws-리소스-테이블)
- [인프라 환경 테이블](#인프라-환경-테이블)
- [리소스 비용 추적 테이블](#리소스-비용-추적-테이블)
- [DNS 관리 테이블](#dns-관리-테이블)
- [인프라 이벤트 로그 테이블](#인프라-이벤트-로그-테이블)
- [배포 관리 테이블](#배포-관리-테이블)
- [관계 다이어그램](#관계-다이어그램)

## AWS 리소스 테이블

`aws_resources` 테이블은 플랫폼에서 사용하는 모든 AWS 리소스 정보를 저장합니다.

```sql
CREATE TABLE aws_resources (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    resource_id VARCHAR(100) NOT NULL, -- AWS 리소스 ID (예: i-1234567890abcdef0)
    resource_arn VARCHAR(255), -- AWS 리소스 ARN
    resource_type VARCHAR(50) NOT NULL, -- 'ec2', 'rds', 's3', 'lambda', 'apigateway', 등
    name VARCHAR(100) NOT NULL, -- 리소스 이름 또는 식별자
    region VARCHAR(20) NOT NULL, -- AWS 리전 (예: ap-northeast-2)
    account_id VARCHAR(20) NOT NULL, -- AWS 계정 ID
    environment_id UUID REFERENCES aws_environments(id) ON DELETE SET NULL,
    status VARCHAR(50) NOT NULL, -- 'running', 'stopped', 'terminated', 등
    configuration JSONB NOT NULL DEFAULT '{}'::jsonb, -- 리소스 구성 정보
    tags JSONB, -- 리소스 태그
    creation_date TIMESTAMP WITH TIME ZONE NOT NULL,
    last_modified_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES users(id) ON DELETE SET NULL,
    is_managed BOOLEAN NOT NULL DEFAULT TRUE, -- 플랫폼에서 관리되는지 여부
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE UNIQUE INDEX idx_aws_resources_resource_id ON aws_resources(resource_id);
CREATE INDEX idx_aws_resources_type ON aws_resources(resource_type);
CREATE INDEX idx_aws_resources_environment_id ON aws_resources(environment_id);
CREATE INDEX idx_aws_resources_region ON aws_resources(region);
CREATE INDEX idx_aws_resources_status ON aws_resources(status);
```

## 인프라 환경 테이블

`aws_environments` 테이블은 논리적 환경(개발, 스테이징, 운영 등)을 정의합니다.

```sql
CREATE TABLE aws_environments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) NOT NULL UNIQUE, -- 'development', 'staging', 'production'
    description TEXT,
    account_id VARCHAR(20) NOT NULL, -- AWS 계정 ID
    default_region VARCHAR(20) NOT NULL, -- 기본 AWS 리전
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    deployment_pipeline_id UUID, -- CI/CD 파이프라인 연결
    vpc_id VARCHAR(50), -- 환경의 기본 VPC ID
    network_config JSONB, -- 네트워크 구성 정보
    security_group_ids VARCHAR[] DEFAULT ARRAY[]::VARCHAR[], -- 보안 그룹 ID 목록
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES users(id) ON DELETE SET NULL,
    updated_by UUID REFERENCES users(id) ON DELETE SET NULL,
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_aws_environments_name ON aws_environments(name);
CREATE INDEX idx_aws_environments_account_id ON aws_environments(account_id);
```

## 리소스 비용 추적 테이블

`aws_resource_costs` 테이블은 각 AWS 리소스의 비용 정보를 기록합니다.

```sql
CREATE TABLE aws_resource_costs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    resource_id UUID REFERENCES aws_resources(id) ON DELETE CASCADE,
    date DATE NOT NULL,
    cost_amount DECIMAL(12, 6) NOT NULL,
    currency VARCHAR(3) NOT NULL DEFAULT 'USD',
    cost_type VARCHAR(50) NOT NULL, -- 'compute', 'storage', 'network', 'data_transfer', 등
    usage_quantity DECIMAL(12, 6),
    usage_unit VARCHAR(20), -- 'GB', 'hours', 등
    blended_cost DECIMAL(12, 6),
    unblended_cost DECIMAL(12, 6),
    amortized_cost DECIMAL(12, 6),
    is_estimated BOOLEAN DEFAULT FALSE, -- 추정 비용 여부
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE UNIQUE INDEX idx_aws_resource_costs_resource_date ON aws_resource_costs(resource_id, date, cost_type);
CREATE INDEX idx_aws_resource_costs_date ON aws_resource_costs(date);
```

## DNS 관리 테이블

`dns_records` 테이블은 플랫폼에서 관리하는 DNS 레코드 정보를 저장합니다.

```sql
CREATE TABLE dns_records (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    domain_name VARCHAR(255) NOT NULL,
    record_name VARCHAR(255) NOT NULL,
    record_type VARCHAR(10) NOT NULL, -- 'A', 'CNAME', 'MX', 'TXT', 등
    record_value TEXT[] NOT NULL, -- 여러 값을 가질 수 있음
    ttl INTEGER NOT NULL DEFAULT 300,
    is_alias BOOLEAN NOT NULL DEFAULT FALSE, -- AWS 별칭 레코드 여부
    route53_zone_id VARCHAR(50), -- Route 53 호스팅 영역 ID
    route53_record_id VARCHAR(50), -- Route 53 레코드 ID
    resource_id UUID REFERENCES aws_resources(id) ON DELETE SET NULL, -- 연결된 AWS 리소스 (있는 경우)
    environment_id UUID REFERENCES aws_environments(id) ON DELETE SET NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES users(id) ON DELETE SET NULL,
    updated_by UUID REFERENCES users(id) ON DELETE SET NULL,
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE UNIQUE INDEX idx_dns_records_domain_name_record ON dns_records(domain_name, record_name, record_type);
CREATE INDEX idx_dns_records_resource_id ON dns_records(resource_id);
CREATE INDEX idx_dns_records_environment_id ON dns_records(environment_id);
```

## 인프라 이벤트 로그 테이블

`aws_infrastructure_events` 테이블은 인프라 변경 및 운영 이벤트를 기록합니다.

```sql
CREATE TABLE aws_infrastructure_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    resource_id UUID REFERENCES aws_resources(id) ON DELETE SET NULL,
    environment_id UUID REFERENCES aws_environments(id) ON DELETE SET NULL,
    event_type VARCHAR(50) NOT NULL, -- 'create', 'update', 'delete', 'start', 'stop', 'error'
    status VARCHAR(20) NOT NULL, -- 'success', 'failure', 'in_progress'
    description TEXT NOT NULL,
    details JSONB,
    event_source VARCHAR(50) NOT NULL, -- 'terraform', 'cloudformation', 'console', 'api', 'scheduled'
    triggered_by UUID REFERENCES users(id) ON DELETE SET NULL, -- 트리거한 사용자 (자동화인 경우 NULL)
    resource_type VARCHAR(50), -- AWS 리소스 유형
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_aws_infrastructure_events_resource_id ON aws_infrastructure_events(resource_id);
CREATE INDEX idx_aws_infrastructure_events_environment_id ON aws_infrastructure_events(environment_id);
CREATE INDEX idx_aws_infrastructure_events_event_type ON aws_infrastructure_events(event_type);
CREATE INDEX idx_aws_infrastructure_events_timestamp ON aws_infrastructure_events(timestamp);
```

## 배포 관리 테이블

`aws_deployments` 테이블은 애플리케이션 배포 정보를 저장합니다.

```sql
CREATE TABLE aws_deployments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    app_id UUID REFERENCES applications(id) ON DELETE SET NULL,
    environment_id UUID REFERENCES aws_environments(id) ON DELETE SET NULL,
    version VARCHAR(50) NOT NULL, -- 배포 버전
    deployment_type VARCHAR(50) NOT NULL, -- 'code_deploy', 'elastic_beanstalk', 'ecs', 'lambda'
    source_repo VARCHAR(255), -- 소스 저장소 URL
    commit_hash VARCHAR(50), -- 소스 커밋 해시
    branch VARCHAR(100), -- 소스 브랜치
    status VARCHAR(20) NOT NULL, -- 'pending', 'in_progress', 'completed', 'failed', 'rolled_back'
    start_time TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    end_time TIMESTAMP WITH TIME ZONE,
    triggered_by UUID REFERENCES users(id) ON DELETE SET NULL,
    config_override JSONB, -- 배포 구성 오버라이드
    deployment_resources JSONB, -- 배포된 리소스 정보
    rollback_version VARCHAR(50), -- 롤백 버전 (있는 경우)
    error_message TEXT, -- 실패 시 오류 메시지
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_aws_deployments_app_id ON aws_deployments(app_id);
CREATE INDEX idx_aws_deployments_environment_id ON aws_deployments(environment_id);
CREATE INDEX idx_aws_deployments_status ON aws_deployments(status);
CREATE INDEX idx_aws_deployments_start_time ON aws_deployments(start_time);
```

## AWS 인증 정보 테이블

`aws_credentials` 테이블은 AWS 계정 접근 정보를 안전하게 저장합니다.

```sql
CREATE TABLE aws_credentials (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL UNIQUE,
    account_id VARCHAR(20) NOT NULL,
    account_alias VARCHAR(100),
    credential_type VARCHAR(20) NOT NULL, -- 'iam_user', 'iam_role', 'assume_role'
    access_key_id VARCHAR(255), -- 암호화 권장
    secret_access_key VARCHAR(255), -- 암호화 필수
    assume_role_arn VARCHAR(255), -- AssumeRole인 경우 사용
    external_id VARCHAR(255), -- AssumeRole인 경우 사용
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    expiration_date TIMESTAMP WITH TIME ZONE, -- IAM 자격 증명 만료일
    last_used_date TIMESTAMP WITH TIME ZONE,
    created_by UUID REFERENCES users(id) ON DELETE SET NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_aws_credentials_account_id ON aws_credentials(account_id);
CREATE INDEX idx_aws_credentials_is_active ON aws_credentials(is_active);
```

## 백업 관리 테이블

`aws_backups` 테이블은 리소스 백업 정보를 저장합니다.

```sql
CREATE TABLE aws_backups (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    resource_id UUID REFERENCES aws_resources(id) ON DELETE SET NULL,
    backup_id VARCHAR(100) NOT NULL, -- AWS 백업 ID (예: snap-1234567890abcdef0)
    backup_type VARCHAR(50) NOT NULL, -- 'snapshot', 'ami', 'rds_snapshot', 's3_copy'
    name VARCHAR(100) NOT NULL,
    description TEXT,
    status VARCHAR(20) NOT NULL, -- 'created', 'available', 'error', 'deleted'
    creation_date TIMESTAMP WITH TIME ZONE NOT NULL,
    expiration_date TIMESTAMP WITH TIME ZONE, -- 자동 삭제 일자
    backup_size_gb DECIMAL(10, 2), -- 백업 크기 (GB)
    storage_location VARCHAR(255), -- 백업 저장 위치
    is_encrypted BOOLEAN NOT NULL DEFAULT TRUE, -- 암호화 여부
    kms_key_id VARCHAR(100), -- 암호화 키 ID
    is_automated BOOLEAN NOT NULL DEFAULT FALSE, -- 자동 백업 여부
    retention_policy VARCHAR(50), -- 'daily', 'weekly', 'monthly', 'yearly'
    created_by UUID REFERENCES users(id) ON DELETE SET NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    metadata JSONB DEFAULT '{}'::jsonb
);

CREATE UNIQUE INDEX idx_aws_backups_backup_id ON aws_backups(backup_id);
CREATE INDEX idx_aws_backups_resource_id ON aws_backups(resource_id);
CREATE INDEX idx_aws_backups_backup_type ON aws_backups(backup_type);
CREATE INDEX idx_aws_backups_status ON aws_backups(status);
CREATE INDEX idx_aws_backups_creation_date ON aws_backups(creation_date);
```

## 관계 다이어그램

AWS 리소스 관리 테이블의 관계 다이어그램은 다음과 같습니다:

```
aws_environments <─┬── aws_resources ──┬─> aws_backups
                   │                   │
                   │                   ├─> aws_resource_costs
                   │                   │
                   │                   └─> dns_records
                   │
                   ├── aws_deployments
                   │
                   └── aws_infrastructure_events
                          │
                          │
                          ▼
                    aws_credentials
```

## 확장 고려사항

### 1. 멀티 클라우드 지원

현재 AWS에 초점을 맞추었으나, 다음과 같이 멀티 클라우드를 지원하도록 확장 가능합니다:

```sql
-- 클라우드 제공자 테이블
CREATE TABLE cloud_providers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) NOT NULL UNIQUE, -- 'aws', 'gcp', 'azure'
    display_name VARCHAR(100) NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- 리소스 테이블 확장
ALTER TABLE aws_resources
ADD COLUMN cloud_provider_id UUID REFERENCES cloud_providers(id) ON DELETE SET NULL;
```

### 2. IaC(Infrastructure as Code) 통합

Terraform, CloudFormation 등의 IaC 도구와 통합을 위한 테이블:

```sql
CREATE TABLE iac_templates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    description TEXT,
    iac_type VARCHAR(50) NOT NULL, -- 'terraform', 'cloudformation', 'arm'
    version VARCHAR(50) NOT NULL,
    template_content TEXT NOT NULL, -- 템플릿 내용
    variables JSONB, -- 템플릿 변수
    environment_id UUID REFERENCES aws_environments(id) ON DELETE SET NULL,
    last_applied_date TIMESTAMP WITH TIME ZONE,
    created_by UUID REFERENCES users(id) ON DELETE SET NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

## 참고 사항

1. AWS 인증 정보와 같은 보안 민감 정보는 저장 시 반드시 암호화해야 합니다.
2. 대규모 인프라 환경의 경우, 리소스 비용 테이블에 파티셔닝 적용을 고려하세요.
3. 인프라 자동화 워크플로우 관리를 위한 추가 테이블이 필요할 수 있습니다.
4. 리소스 간 의존성 추적을 위한 별도 테이블 추가를 고려할 수 있습니다. 