# n8n 에이전트 단독 노드 비교 (Version 1.113.3)

## 📋 조사 결과

**n8n 1.113.3 버전에서는 3사의 에이전트 단독 노드 구조가 다음과 같습니다:**

---

## 1️⃣ OpenAI 단독 노드

### **OpenAI Node** (일반 작업용)
**경로**: `packages/nodes-base/nodes/OpenAi/`

**기본 정보**
- **노드명**: `OpenAi`
- **표시명**: OpenAI
- **버전**: 1, 1.1
- **상태**: Hidden (숨김 처리됨)
- **그룹**: Transform
- **인증**: `openAiApi`

**리소스 & 작업**

| 리소스 | 작업 | 설명 |
|--------|------|------|
| **Text** | Complete | 주어진 텍스트에 대한 완성 생성 |
| | Edit | 텍스트 편집 버전 생성 |
| | Moderate | 콘텐츠 정책 위반 여부 확인 |
| **Chat** | Complete | 채팅 완성 생성 (대화형) |
| **Image** | Create | DALL-E로 이미지 생성 |

**Text 작업 상세**
- **모델**: `gpt-3.5-turbo-instruct`, `text-davinci-edit-001`, `text-moderation-latest`
- **옵션**: Temperature, Max Tokens, Frequency/Presence Penalty

**Chat 작업 상세**
- **모델**: `gpt-3.5-turbo`, `gpt-4` 등
- **메시지 역할**: system, user, assistant
- **옵션**: Temperature, Max Tokens, Top P, Output Simplification

**Image 작업 상세**
- **모델**: `dall-e-2`, `dall-e-3`
- **해상도**:
  - DALL-E 2: 256x256, 512x512, 1024x1024
  - DALL-E 3: 1024x1024, 1792x1024, 1024x1792
- **옵션**: Quality (HD/Standard), Style (Natural/Vivid), 이미지 개수 (1-10)

---

### **OpenAI Assistant Node** (Assistant API용)
**경로**: `packages/@n8n/nodes-langchain/nodes/agents/OpenAiAssistant/`

**기본 정보**
- **노드명**: `OpenAiAssistant`
- **표시명**: OpenAI Assistant
- **버전**: 1, 1.1
- **상태**: Hidden
- **그룹**: Transform

**작업 모드**
1. **Create New Assistant**: 새 어시스턴트 생성
2. **Use Existing Assistant**: 기존 어시스턴트 사용

**네이티브 도구**
- **Code Interpreter**: 코드 실행
- **Knowledge Retrieval**: 지식 검색

**설정 옵션**
- Assistant ID
- Model Selection
- Base URL
- Max Retries
- Timeout
- Custom Tools Integration

---

## 2️⃣ Anthropic (Claude) 단독 노드

### ❌ **단독 노드 없음**

**현황**:
- n8n 1.113.3에는 Anthropic/Claude의 **에이전트 단독 노드가 존재하지 않습니다**
- Chat Model 노드(`LMChatAnthropic`)만 제공됨
- AI Agent 노드와 함께 사용해야 함

**사용 방법**:
```
AI Agent 노드 + Anthropic Chat Model 노드 (서브노드)
```

---

## 3️⃣ Google Gemini 단독 노드

### ❌ **단독 노드 없음**

**현황**:
- n8n 1.113.3에는 Google Gemini의 **에이전트 단독 노드가 존재하지 않습니다**
- Chat Model 노드(`LmChatGoogleGemini`)만 제공됨
- AI Agent 노드와 함께 사용해야 함

**사용 방법**:
```
AI Agent 노드 + Google Gemini Chat Model 노드 (서브노드)
```

---

## 📊 AI Agent 노드 (공통 에이전트)

**경로**: `packages/@n8n/nodes-langchain/nodes/agents/Agent/`

**기본 정보**
- **노드명**: `Agent`
- **표시명**: AI Agent
- **버전**: 1.0 ~ 2.2 (최신: 2.2)
- **그룹**: Transform
- **별칭**: LangChain, Chat, Conversational, Plan and Execute, ReAct, Tools

**에이전트 타입**

| 타입 | 설명 | 지원 모델 |
|------|------|----------|
| **Conversational Agent** | 대화형 에이전트 | 모든 Chat Model |
| **OpenAI Functions Agent** | OpenAI Function Calling | OpenAI Chat Model 필수 |
| **Plan and Execute Agent** | 계획 수립 후 실행 | 모든 Chat Model |
| **ReAct Agent** | 추론-행동 패턴 | 모든 Chat Model |
| **SQL Agent** | SQL 쿼리 생성/실행 | 모든 Chat Model |
| **Tools Agent** | 도구 사용 에이전트 | 모든 Chat Model |

**공통 파라미터**
- Prompt (프롬프트)
- Require Specific Output Format (특정 출력 형식 요구)

**공통 옵션**
- System Message (시스템 메시지)
- Max Iterations (최대 반복 횟수)
- Return Intermediate Steps (중간 단계 반환)
- Enable Streaming (스트리밍 활성화)
- Automatically Passthrough Binary Images (바이너리 이미지 자동 전달)

---

## 🔄 사용 패턴 비교

### OpenAI 사용 패턴
**패턴 1: 단독 노드 사용**
```
OpenAI Node (Text/Chat/Image 작업)
```

**패턴 2: Assistant API 사용**
```
OpenAI Assistant Node
```

**패턴 3: AI Agent 사용**
```
AI Agent + OpenAI Chat Model (서브노드)
```

---

### Anthropic (Claude) 사용 패턴
**유일한 방법**
```
AI Agent + Anthropic Chat Model (서브노드)
```

---

### Google Gemini 사용 패턴
**유일한 방법**
```
AI Agent + Google Gemini Chat Model (서브노드)
```

---

## 💡 주요 차이점

| 특징 | OpenAI | Anthropic | Gemini |
|------|--------|-----------|--------|
| 단독 작업 노드 | ✅ (Text, Chat, Image) | ❌ | ❌ |
| Assistant 전용 노드 | ✅ (OpenAI Assistant) | ❌ | ❌ |
| Chat Model 노드 | ✅ | ✅ | ✅ |
| AI Agent 통합 | ✅ | ✅ (필수) | ✅ (필수) |
| 네이티브 도구 | Code Interpreter, Retrieval | - | - |

---

## 🎯 결론

**OpenAI만 단독 에이전트 노드를 제공**합니다:
- 일반 작업용 `OpenAI` 노드
- Assistant API용 `OpenAI Assistant` 노드

**Anthropic과 Gemini는**:
- 에이전트 단독 노드가 없음
- 반드시 `AI Agent` 노드와 함께 Chat Model 서브노드로 사용해야 함

이는 OpenAI가 Assistant API 등 자체 에이전트 프레임워크를 제공하는 반면, Anthropic과 Google은 표준 Chat API만 제공하기 때문입니다.

---

## 📅 문서 정보
- **n8n 버전**: 1.113.3
- **작성일**: 2025-10-16
- **최종 업데이트**: 2025-10-16
