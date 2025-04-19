## ✅ Flutter 공통 코드 작업을 위한 Cursor 프롬프트 기준 문서

### 1. 📂 작업 디렉토리 및 경로 기준
- 모든 Flutter 코드는 다음 루트 경로 내에서 작업합니다:
  ```
  /Users/changig/dev/unibase-platform-dev/flutter/
  ```

### 2. 🛑 절대 변경 금지 항목
- `lib/core/` 폴더 내 구조 및 기존 파일은 **절대 삭제/이동/수정하지 마십시오**.
- import 경로는 항상 `package:unibase_platform/`으로 시작해야 합니다.
- **절대 상대 경로 import(`../`)를 사용하지 마십시오.**

### 3. 📦 pubspec.yaml 프로젝트 설정
- `pubspec.yaml`의 name은 다음과 같이 설정되어 있습니다:
  ```yaml
  name: unibase_platform
  ```
- 따라서 import 시 다음과 같이 작성해야 합니다:
  ```dart
  import 'package:unibase_platform/core/services/api_client.dart';
  ```

### 4. 📁 공통 코드 작성 위치
모든 공통 코드는 아래 경로에만 작성합니다:
```
lib/core/
├── models/
├── services/
├── widgets/
├── layout/
└── config/
```
- 기존 코드는 수정하지 말고 **새 파일을 추가**하는 방식으로 확장하십시오.

### 5. 📁 앱별 모듈 작성 위치 (예시: idle_farm 앱)
```
lib/apps/idle_farm/
├── screens/
├── services/
├── widgets/
├── models/
└── config/
```
- 앱별 모듈은 `apps/{앱이름}/` 하위에서만 작업합니다.
- 공통 코드와 앱별 코드를 **절대 혼합하지 마십시오.**

### 6. ✅ 작업 기준 요약
- 절대경로 import 필수 (`package:unibase_platform/`)
- 공통 코드는 `lib/core/`에만 작성
- 앱별 코드는 `lib/apps/{앱이름}/` 하위에만 작성
- 폴더 구조는 절대 변경하지 말 것
- 기존 파일은 최대한 수정하지 않고, 새 파일을 추가하여 확장

### 7. 🌐 Git 원격 저장소
- 변경사항은 다음 원격 저장소로 푸시됩니다:
  ```
  https://github.com/leechangik/unibase-platform-dev
  ```

### 8. 🧠 필수 이해 사항
- 이 구조는 **플러터 앱 자동화 및 복제 기반 구조 설계**를 위해 엄격하게 고정된 구조입니다.
- 모든 작업자는 이 문서를 사전 정독한 후 작업을 시작해야 하며, **위 기준을 따르지 않으면 빌드 실패 및 구조 오류가 발생할 수 있습니다.** 