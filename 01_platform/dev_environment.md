# Development Environment Specification

## Purpose

This document specifies the standardized development environment for UniBase platform development. It ensures consistency across development teams and environments, minimizing "it works on my machine" issues and streamlining onboarding.

## Table of Contents

- [Development Stack](#development-stack)
- [Version Control](#version-control)
- [Local Development Setup](#local-development-setup)
- [Docker Environment](#docker-environment)
- [IDE Configuration](#ide-configuration)
- [CI/CD Pipeline Requirements](#cicd-pipeline-requirements)
- [Environment Variables](#environment-variables)

## Development Stack

The following table outlines the core technologies and their versions used in the UniBase platform:

| Technology | Version | Purpose |
|------------|---------|---------|
| Flutter | 3.16.9 | Cross-platform client development |
| Dart | 3.2.3 | Programming language for Flutter |
| Java | 17 | Backend services |
| Spring Boot | 3.2.0 | Backend framework |
| PostgreSQL | 15.x | Primary database |
| Redis | 7.0.x | Caching layer |
| Node.js | 20.x LTS | Development tooling and scripts |
| Docker | 24.x | Containerization |
| Kubernetes | 1.28.x | Container orchestration |
| AWS CLI | 2.15.x | Cloud infrastructure management |

## Version Control

### Git Configuration

All repositories should use the following Git configuration:

```bash
git config --global core.autocrlf input
git config --global init.defaultBranch main
```

### Branch Strategy

We follow a trunk-based development model with the following branches:

- `main`: Production code, always deployable
- `develop`: Integration branch for features
- `feature/*`: Feature branches
- `hotfix/*`: Emergency fixes for production

### Commit Message Format

Commit messages should follow the [Conventional Commits](https://www.conventionalcommits.org/) format:

```
type(scope): subject

body
```

Example:

```
feat(auth): implement JWT token refresh

- Add refresh token endpoint
- Update token validation middleware
- Add unit tests for refresh flow
```

## Local Development Setup

### Prerequisites

- macOS 14.x, Windows 11, or Ubuntu 22.04 LTS
- Minimum 16GB RAM, 256GB SSD, 4 CPU cores
- Docker Desktop with Kubernetes enabled
- Flutter SDK installed via official channels
- JDK 17
- Node.js 20.x LTS
- Visual Studio Code or IntelliJ IDEA

### Setup Script

Each repository contains a `scripts/setup.sh` (or `.bat` for Windows) that automates environment setup.

```bash
# Example setup script usage
cd /path/to/repo
./scripts/setup.sh
```

## Docker Environment

All projects include Docker Compose configurations for local development:

```yaml
# Example docker-compose.yml
version: '3.8'

services:
  api:
    build:
      context: ./server
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=dev
      - DB_URL=jdbc:postgresql://db:5432/unibase
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=unibase
      - POSTGRES_PASSWORD=unibase
      - POSTGRES_DB=unibase
    volumes:
      - db-data:/var/lib/postgresql/data

  redis:
    image: redis:7.0
    ports:
      - "6379:6379"

volumes:
  db-data:
```

## IDE Configuration

### Visual Studio Code

Recommended extensions:

- Dart
- Flutter
- Java Extension Pack
- Spring Boot Extension Pack
- PostgreSQL
- Docker
- Kubernetes
- GitLens
- EditorConfig

Configuration files are provided in each repository's `.vscode` directory.

### IntelliJ IDEA

Recommended plugins:

- Dart
- Flutter
- Spring Boot Assistant
- Database Navigator
- Docker
- Kubernetes
- GitToolBox

Project configuration files are provided in each repository's `.idea` directory.

## CI/CD Pipeline Requirements

CI/CD pipelines should be configured to:

1. Build and test in an environment identical to the development environment
2. Use the same Docker images as local development
3. Run automated tests before deployment
4. Deploy to staging and production environments

GitHub Actions workflow example:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
        
    - name: Build with Maven
      run: mvn clean package
      
    - name: Run tests
      run: mvn test
      
    - name: Build Docker image
      run: docker build -t unibase-api .
      
    # Additional deployment steps...
```

## Environment Variables

The following environment variables must be configured in all development environments:

| Variable | Purpose | Example |
|----------|---------|---------|
| `UNIBASE_ENV` | Environment designation | `dev`, `stage`, `prod` |
| `DB_URL` | Database connection string | `jdbc:postgresql://localhost:5432/unibase` |
| `DB_USERNAME` | Database username | `unibase_user` |
| `DB_PASSWORD` | Database password | `********` |
| `REDIS_HOST` | Redis host | `localhost` |
| `REDIS_PORT` | Redis port | `6379` |
| `AWS_PROFILE` | AWS CLI profile | `unibase-dev` |
| `API_KEY` | API key for external services | `sk_test_******` |

Environment variables should be managed using:

- Local development: `.env` files (not committed to Git)
- CI/CD: Secrets management in GitHub Actions/Jenkins
- Production: AWS Parameter Store or Kubernetes Secrets

## Troubleshooting

Common development environment issues and solutions:

1. **Flutter/Dart version mismatches**:
   - Use `flutter doctor` to diagnose
   - Run `flutter channel stable && flutter upgrade` to update

2. **Database connection issues**:
   - Verify PostgreSQL is running: `docker ps | grep postgres`
   - Check connection settings in application properties

3. **Docker resource limitations**:
   - Increase Docker Desktop resource allocation
   - Minimum recommended: 8GB RAM, 2 CPUs, 60GB disk

## Version Control

This environment specification is version controlled. Changes to the development environment should be proposed, reviewed, and communicated to all development teams.

Current version: 1.0.0 (2024-04-01) 