# Database Schema (v1.0)

## Purpose

This document specifies the core database schema for the UniBase platform. It defines the structure of common tables such as users, applications, and shared entities that are used across all modules.

## Table of Contents

- [Schema Overview](#schema-overview)
- [Core Tables](#core-tables)
  - [Users](#users)
  - [Applications](#applications)
  - [Modules](#modules)
  - [App_Module_Map](#app_module_map)
  - [User_App_Map](#user_app_map)
- [Authentication Tables](#authentication-tables)
- [Audit and Logging](#audit-and-logging)
- [Database Migrations](#database-migrations)
- [Foreign Key Relationships](#foreign-key-relationships)

## Schema Overview

The UniBase database schema is organized into the following categories:

1. **Core Tables**: Essential entities required by all modules (users, applications)
2. **Authentication Tables**: Identity, roles, permissions, and access control
3. **Module-Specific Tables**: Tables specific to each functional module
4. **Audit Tables**: Historical tracking of changes and actions
5. **Configuration Tables**: System and application settings

## Core Tables

### Users

The `users` table stores information about all users in the system.

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
```

### Applications

The `applications` table tracks all instances of applications created on the platform.

```sql
CREATE TABLE applications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    description TEXT,
    api_key VARCHAR(128) UNIQUE,
    status VARCHAR(20) NOT NULL DEFAULT 'active', -- 'active', 'inactive', 'development'
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    created_by UUID NOT NULL REFERENCES users(id),
    config JSONB NOT NULL DEFAULT '{}'::jsonb -- Application-specific configuration
);

CREATE INDEX idx_applications_name ON applications(name);
CREATE INDEX idx_applications_created_by ON applications(created_by);
```

### Modules

The `modules` table defines all available functional modules in the platform.

```sql
CREATE TABLE modules (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) NOT NULL UNIQUE,
    display_name VARCHAR(100) NOT NULL,
    description TEXT,
    version VARCHAR(20) NOT NULL,
    is_core BOOLEAN NOT NULL DEFAULT FALSE, -- Whether the module is a core module
    dependencies JSONB DEFAULT NULL, -- Module dependencies
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_modules_name ON modules(name);
```

### App_Module_Map

The `app_module_map` table maps which modules are activated for each application.

```sql
CREATE TABLE app_module_map (
    app_id UUID NOT NULL REFERENCES applications(id) ON DELETE CASCADE,
    module_id UUID NOT NULL REFERENCES modules(id) ON DELETE RESTRICT,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    config JSONB NOT NULL DEFAULT '{}'::jsonb, -- Module-specific configuration
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (app_id, module_id)
);

CREATE INDEX idx_app_module_map_app_id ON app_module_map(app_id);
CREATE INDEX idx_app_module_map_module_id ON app_module_map(module_id);
```

### User_App_Map

The `user_app_map` table assigns users to applications with specific roles.

```sql
CREATE TABLE user_app_map (
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    app_id UUID NOT NULL REFERENCES applications(id) ON DELETE CASCADE,
    role VARCHAR(50) NOT NULL, -- 'owner', 'admin', 'editor', 'viewer'
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, app_id)
);

CREATE INDEX idx_user_app_map_user_id ON user_app_map(user_id);
CREATE INDEX idx_user_app_map_app_id ON user_app_map(app_id);
```

## Authentication Tables

### Roles

The `roles` table defines system-wide roles for user permissions.

```sql
CREATE TABLE roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO roles (name, description) VALUES 
    ('ROLE_ADMIN', 'Platform administrators with full access'),
    ('ROLE_USER', 'Standard user with limited access'),
    ('ROLE_APP_OWNER', 'Application owner/creator');
```

### User_Roles

The `user_roles` table maps users to their system roles.

```sql
CREATE TABLE user_roles (
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role_id UUID NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, role_id)
);
```

### Auth_Tokens

The `auth_tokens` table manages user authentication sessions.

```sql
CREATE TABLE auth_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token_value VARCHAR(255) NOT NULL UNIQUE,
    token_type VARCHAR(20) NOT NULL, -- 'access', 'refresh'
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_used_at TIMESTAMP WITH TIME ZONE,
    device_info JSONB
);

CREATE INDEX idx_auth_tokens_user_id ON auth_tokens(user_id);
CREATE INDEX idx_auth_tokens_token_value ON auth_tokens(token_value);
CREATE INDEX idx_auth_tokens_expires_at ON auth_tokens(expires_at);
```

## Audit and Logging

### Audit_Log

The `audit_log` table records all significant actions in the system.

```sql
CREATE TABLE audit_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    action VARCHAR(50) NOT NULL, -- 'create', 'update', 'delete', 'login', etc.
    entity_type VARCHAR(50) NOT NULL, -- 'user', 'application', 'module', etc.
    entity_id UUID, -- ID of the affected entity
    details JSONB NOT NULL DEFAULT '{}'::jsonb, -- Additional details about the action
    ip_address VARCHAR(45), -- Supports IPv6
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_log_user_id ON audit_log(user_id);
CREATE INDEX idx_audit_log_entity_type_id ON audit_log(entity_type, entity_id);
CREATE INDEX idx_audit_log_timestamp ON audit_log(timestamp);
```

## Database Migrations

All schema changes are managed through Flyway migrations. Migration scripts are stored in the server repository under:

```
server/src/main/resources/db/migration/
```

Migration files follow the naming convention:

```
V{version}__{description}.sql
```

Example:

```
V1_0_0__initial_schema.sql
V1_0_1__add_avatar_to_users.sql
```

## Foreign Key Relationships

The following diagram illustrates the relationships between core tables:

```
users <─────────────┐ 
  ▲                 │
  │                 │
  │                 │
user_roles        user_app_map
  │                 │
  │                 │
  ▼                 ▼
roles          applications ───┐
                    ▲          │
                    │          │
                    │          │
                    │          │
                app_module_map ◄──── modules
```

## Notes on Schema Design

1. **UUID Primary Keys**: All tables use UUID primary keys for security and distributed generation.
2. **Timestamps**: All tables include `created_at` and relevant tables include `updated_at`.
3. **Soft Deletion**: Critical tables support soft deletion via `is_active` flags.
4. **JSONB for Configuration**: Configuration data is stored in JSONB columns for flexibility.
5. **Indexing Strategy**: Indexes are created for all foreign keys and frequently queried columns.

## Module-Specific Tables

Module-specific tables follow a similar schema pattern but are managed separately in their respective database migration files. All module tables should use the module name as a prefix:

Examples:
- `farm_crops`
- `farm_animals`
- `payment_transactions`
- `social_posts` 