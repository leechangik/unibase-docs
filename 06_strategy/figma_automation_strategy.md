# 🧠 UniBase 앱 복제 기반 Figma 자동화 전략

## 📌 목적
이 문서는 글로벌 앱 복제 시, GPT + Cursor + Figma API를 연계하여 **디자인 자동화 프로세스를 구현하는 전체 전략**을 설명합니다. 수동으로 Figma를 열지 않고도, 앱 UI 스크린샷만으로 Figma 디자인이 자동 생성되도록 구성합니다.

---

## 🔁 전체 흐름

### 1️⃣ 복제 대상 앱 스크린샷 제공
- **사용자**는 복제하고자 하는 앱의 주요 UI 화면을 스크린샷으로 저장함 (JPG, PNG)
- 이 스크린샷은 동시에 **GPT**와 **Cursor**에게 제공됨
  - 📤 GPT: 스크린샷을 분석하여 UI 구조 및 구성요소 설명서 생성
  - 📤 Cursor: GPT가 만든 설명서 + 이미지 기반으로 더 정교한 스크립트 자동 생성 가능

### 2️⃣ GPT의 역할 (디자인 구조 분석 및 프롬프트 생성)
- UI 레이아웃(그리드, 섹션), 컬러, 폰트, 버튼 등 시각적 구성요소 분석
- 분석 결과를 기반으로 **디자인 가이드 문서 + Figma API JSON 구조 설명서** 생성
- Cursor가 사용할 수 있도록 **자동화 프롬프트 (`prompt.figma_ui_{앱명}.txt`)** 생성

### 3️⃣ Cursor의 역할 (자동화 코드 생성)
- GPT가 작성한 설명서 기반으로 **Node.js 또는 Python** 기반 스크립트 생성 (예: `figma_push.js`)
- 이 스크립트는 Figma API를 호출하여 디자인 요소(Frame, Text, Rectangle 등)를 Figma 파일에 자동 생성함

### 4️⃣ 사용자 수행 단계: Figma Access Token 수동 제공
- Figma API를 사용하기 위해서는 **사용자의 Personal Access Token**이 필요함
- 이 토큰은 GPT나 Cursor가 자동 발급할 수 없음
- 사용자가 Figma 계정에서 직접 토큰을 생성한 후, Cursor 스크립트에 붙여넣어야 함
  - 예: `const TOKEN = "figd_ABC123..."` 형태로 스크립트 상단에 입력

### 5️⃣ 실행 및 반복
- 사용자는 Cursor가 생성한 스크립트를 터미널에서 실행
- Figma에 자동으로 디자인 요소 생성 완료
- 이후 이 흐름은 복제할 앱마다 반복 수행 가능

---

## 🛠️ 디렉토리 구성 예시
```
.cursor-tasks/
├── prompt.base_figma_automation.txt             ← 공통 전략 프롬프트 (항상 함께 전달)
├── prompt.figma_ui_layout_puzzle_01.txt         ← 앱별 프롬프트 (GPT가 생성)
├── figma_push.js                                ← Cursor가 생성한 실행 스크립트
├── img/
│   └── puzzle_clone_01.png                      ← 복제할 앱 스크린샷
```

---

## ⚠️ 주의사항 요약
- Cursor는 Figma를 직접 열지 못함 → API로만 접근 가능
- Access Token은 사용자 계정에서 수동 발급 후 붙여넣기 필요
- 복잡한 디자인은 GPT가 구조화해야 정확도 향상
- GPT 설명 + 스크린샷 + 구조화된 프롬프트 세트가 함께 있어야 정확한 자동화 가능

---

## 📍 저장 경로 제안
```
/Users/changig/dev/unibase-docs/06_strategy/figma_automation_strategy.md
```

이 문서는 모든 Figma 디자인 자동화 작업의 기준이며, 각 앱 복제 시 반드시 참조되어야 합니다. 