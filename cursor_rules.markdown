# Cursor Rules 정리

## 1. Cursor Rules란?
Cursor의 Rules(규칙) 시스템은 AI가 일관된 방식으로 동작하도록 지속적으로 지침을 제공하는 시스템입니다. 이 규칙들은 코드 생성, 편집, 워크플로우 자동화 등 다양한 상황에서 AI의 행동을 제어할 수 있습니다.

## 2. 규칙의 종류
- **Project Rules**: `.cursor/rules` 폴더에 저장, 프로젝트별로 관리, 버전 관리 가능
    - 특정 프로젝트에만 적용되는 규칙입니다.
    - 여러 프로젝트에서 각각 다른 규칙을 둘 수 있습니다.
    - User Rules와 함께 공존하며, 프로젝트 내에서만 우선적으로 적용됩니다.
    - **프로젝트 규칙을 사용하여 다음을 수행할 수 있습니다:**
        - 코드베이스에 대한 도메인별 지식 인코딩
        - 프로젝트별 워크플로 또는 템플릿 자동화
        - 스타일 또는 아키텍처 결정 표준화
- **User Rules**: Cursor 설정에서 관리, 모든 프로젝트에 전역적으로 적용
    - 사용자(나)에게 항상 적용되는 전역 규칙입니다.
    - 예를 들어, "항상 한국어로 답변해줘" 같은 규칙을 등록하면 모든 프로젝트에서 적용됩니다.
    - Project Rules와 함께 적용될 수 있습니다.
- **.cursorrules (Legacy)**: 프로젝트 루트에 `.cursorrules` 파일로 존재, 곧 폐기 예정
    - 예전 방식으로, 앞으로는 사용하지 않는 것이 좋습니다.
    - Project Rules와 User Rules를 사용하는 것이 권장됩니다.

## 3. 규칙의 작동 방식
- 규칙(Rules)은 대형 언어 모델이 각 요청마다 기억을 유지하지 못하는 한계를 보완하기 위해, 프롬프트 레벨에서 일관된 지침을 지속적으로 제공합니다.
- 규칙이 적용되면, 그 내용이 모델 컨텍스트의 시작 부분에 항상 포함되어 AI가 코드 생성, 편집, 워크플로우 등에서 일관된 행동을 하도록 만듭니다.
- 규칙은 Chat과 Cmd-K 모두에 적용됩니다.
    - Chat: AI와 대화창에서 질문하거나 코드 생성/수정 요청을 할 때 적용됩니다.
    - Cmd-K: 에디터에서 Cmd+K(또는 Ctrl+K)로 AI 기능을 직접 호출할 때 적용됩니다.

## 4. 규칙 파일 구조 (MDC 포맷)
- 메타데이터와 내용을 한 파일에 작성
- 주요 필드: description(설명), globs(적용 파일 패턴), alwaysApply(항상 적용 여부)

| 규칙 유형           | 설명                                                                 |
|---------------------|----------------------------------------------------------------------|
| Always              | 항상 모델 컨텍스트에 포함됨                                          |
| Auto Attached       | 글로브 패턴과 일치하는 파일이 참조될 때 포함됨                        |
| Agent Requested     | 규칙은 AI가 사용할 수 있으며, AI가 규칙을 포함할지 여부를 결정함. 설명 필요 |
| Manual              | 명시적으로 언급(@ruleName)된 경우에만 포함됨                         |

- 예시:

```mdc
---
description: RPC Service boilerplate
globs: 
alwaysApply: false
---

- 서비스 정의 시 내부 RPC 패턴 사용
- 서비스 이름은 snake_case로 작성

@service-template.ts
```

- 규칙이 트리거되면, 본문에서 @로 참조한 파일(예: @service-template.ts)의 내용이 AI의 추가 컨텍스트로 자동 포함됩니다. 이를 통해 예시 코드, 템플릿, 참고 구현 등을 규칙과 함께 AI가 참고할 수 있습니다.
- Cursor 에디터에서 `Cmd + Shift + P`(Windows는 `Ctrl + Shift + P`)를 누른 후 "새 커서 규칙"(New Cursor Rule) 명령어를 사용하면, 규칙 파일을 빠르게 생성할 수 있습니다.

## 5. 규칙 생성 및 적용 방법
- `Cmd + Shift + P` → "New Cursor Rule"
- 또는 Cursor Settings > Rules에서 직접 생성
- 파일 패턴, always 옵션, 수동 호출 등 다양한 방식으로 규칙을 적용할 수 있음
- `/Generate Cursor Rules` 명령을 사용하여 대화에서 직접 규칙을 생성할 수 있습니다. 상담원의 행동 방식에 대해 많은 결정이 내려진 대화를 나눈 경우, 이 명령으로 규칙을 생성해두면 나중에 재사용할 수 있어 매우 유용합니다.

## 6. 규칙 작성 Best Practice
- 500줄 이하로 간결하게
- 큰 개념은 여러 규칙으로 분리
- 구체적인 예시, 참조 파일 활용
- 모호한 지침은 피하고, 내부 문서처럼 명확하게 작성

## 7. 예시 규칙
### (1) 한국어 답변 강제 (User Rule)
```
항상 한국어로 답변해줘.
```

### (2) 프론트엔드 컴포넌트 스타일 가이드 (Project Rule)
```mdc
---
description: 프론트엔드 컴포넌트 스타일 가이드
globs: ["src/components/**"]
alwaysApply: true
---

- Tailwind만 사용해서 스타일링
- Framer Motion으로 애니메이션 구현
- 컴포넌트 이름은 PascalCase로 작성
```

### (3) API 검증 표준화 (Project Rule)
```mdc
---
description: API 엔드포인트 검증 표준
globs: ["src/api/**"]
alwaysApply: true
---

- 모든 검증에 zod 사용
- 반환 타입은 zod 스키마로 정의
- 타입은 스키마에서 export
```

### (4) Express 서비스 템플릿 (Project Rule)
```mdc
---
description: Express 서비스 생성 템플릿
globs: ["src/services/**"]
alwaysApply: false
---

- RESTful 원칙 준수
- 에러 핸들링 미들웨어 포함
- 적절한 로깅 설정

@express-service-template.ts
```

### (5) React 컴포넌트 구조 (Project Rule)
```mdc
---
description: React 컴포넌트 구조 가이드
globs: ["src/components/**"]
alwaysApply: false
---

- Props interface를 파일 상단에 작성
- 컴포넌트는 named export로 작성
- 스타일은 파일 하단에 위치

@component-template.tsx
```

### (6) 워크플로 자동화 (Project Rule)
```mdc
---
description: 앱 분석 워크플로 자동화
globs: []
alwaysApply: false
---

- 앱 분석 요청 시:
    1. `npm run dev`로 dev 서버 실행
    2. 콘솔 로그 수집
    3. 성능 개선점 제안
```

### (7) 코드 기반 문서화 (Project Rule)
```mdc
---
description: 코드 기반 문서화 자동화
globs: []
alwaysApply: false
---

- 코드 주석 추출
- README.md 분석
- 마크다운 문서 자동 생성
```

## 8. 참고
- [Cursor 공식 문서: Rules](https://docs.cursor.com/context/rules)
