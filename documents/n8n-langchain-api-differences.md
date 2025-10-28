# n8n LangChain 내부 OpenAI vs Claude vs Gemini API 차이점

## 📋 개요

n8n은 LangChain을 통해 세 가지 AI 제공자의 API를 통합합니다. 각 제공자는 서로 다른 LangChain 클래스와 API 엔드포인트를 사용합니다.

---

## 🔧 1. LangChain 클래스 비교

| 항목 | OpenAI | Anthropic | Google Gemini |
|------|--------|-----------|---------------|
| **LangChain 클래스** | `ChatOpenAI` | `ChatAnthropic` | `ChatGoogleGenerativeAI` |
| **패키지** | `@langchain/openai` | `@langchain/anthropic` | `@langchain/google-genai` |
| **노드 파일** | `LmChatOpenAi.node.ts` | `LmChatAnthropic.node.ts` | `LmChatGoogleGemini.node.ts` |

---

## 🌐 2. API 엔드포인트

### OpenAI
```typescript
baseURL: credentials.baseURL || 'https://api.openai.com/v1'
```
- **기본 엔드포인트**: `https://api.openai.com/v1`
- **커스텀 URL 지원**: ✅ (Azure OpenAI 등)
- **프록시 지원**: ✅ `getProxyAgent()`

### Anthropic (Claude)
```typescript
baseURL: credentials.url || 'https://api.anthropic.com'
```
- **기본 엔드포인트**: `https://api.anthropic.com`
- **커스텀 URL 지원**: ✅
- **프록시 지원**: ✅ `httpAgent` / `httpsAgent`

### Google Gemini
```typescript
baseUrl: credentials.host
```
- **기본 엔드포인트**: Google AI API (credentials에서 지정)
- **커스텀 URL 지원**: ✅
- **프록시 지원**: ❌

---

## 🔑 3. 인증 방식

### OpenAI
```typescript
const credentials = await this.getCredentials('openAiApi');

configuration = {
  apiKey: credentials.apiKey,
  baseURL: credentials.url,
  defaultHeaders: {
    [credentials.headerName]: credentials.headerValue  // 커스텀 헤더
  }
}
```
**특징**:
- API Key 기반
- 커스텀 HTTP 헤더 지원
- Azure OpenAI 등 다양한 엔드포인트 대응

### Anthropic
```typescript
const credentials = await this.getCredentials('anthropicApi');

model = new ChatAnthropic({
  anthropicApiKey: credentials.apiKey,
  anthropicApiUrl: baseURL
})
```
**특징**:
- API Key 기반 (간단)
- URL 오버라이드 가능

### Google Gemini
```typescript
const credentials = await this.getCredentials('googlePalmApi');

model = new ChatGoogleGenerativeAI({
  apiKey: credentials.apiKey,
  baseUrl: credentials.host
})
```
**특징**:
- API Key 기반
- Host URL 설정 필수

---

## ⚙️ 4. 모델 파라미터 비교

### OpenAI
```typescript
const model = new ChatOpenAI({
  openAIApiKey: credentials.apiKey,
  modelName: modelName,              // 'gpt-4o-mini'
  temperature: options.temperature,  // 0 ~ 2
  maxTokens: options.maxTokens,      // 최대 32,768
  topP: options.topP,
  frequencyPenalty: options.frequencyPenalty,  // -2 ~ 2
  presencePenalty: options.presencePenalty,    // -2 ~ 2
  timeout: options.timeout,
  maxRetries: options.maxRetries,
  modelKwargs: {
    response_format: { type: 'json_object' },  // JSON 모드
    reasoning_effort: 'high'                    // 추론 강도
  }
})
```

**고유 파라미터**:
- ✅ `frequencyPenalty` / `presencePenalty`
- ✅ `response_format` (JSON 모드)
- ✅ `reasoning_effort` (추론 강도)

---

### Anthropic (Claude)
```typescript
const model = new ChatAnthropic({
  anthropicApiKey: credentials.apiKey,
  model: modelName,                    // 'claude-3-5-sonnet-...'
  maxTokens: options.maxTokensToSample,  // 기본 4,096
  temperature: options.temperature,    // 0 ~ 1
  topK: options.topK,
  topP: options.topP,
  invocationKwargs: {
    thinking: {                        // 🌟 고유 기능
      type: 'enabled',
      budget_tokens: 1024
    }
  }
})
```

**고유 파라미터**:
- ✅ `thinking` (추론 모드)
- ✅ `thinkingBudget` (추론 토큰 예산)
- ✅ `invocationKwargs` (직접 API 파라미터 전달)

---

### Google Gemini
```typescript
const model = new ChatGoogleGenerativeAI({
  apiKey: credentials.apiKey,
  model: modelName,                  // 'gemini-2.5-flash'
  temperature: options.temperature,  // 0 ~ 1
  topK: options.topK,                // 기본 40
  topP: options.topP,                // 기본 0.9
  maxOutputTokens: options.maxOutputTokens,  // 기본 1,024
  safetySettings: [                  // 🌟 고유 기능
    {
      category: 'HARM_CATEGORY_DANGEROUS_CONTENT',
      threshold: 'BLOCK_MEDIUM_AND_ABOVE'
    }
  ]
})
```

**고유 파라미터**:
- ✅ `safetySettings` (콘텐츠 안전 필터)
- ✅ `maxOutputTokens` (출력 토큰 제한)

---

## 🔄 5. 콜백 및 에러 핸들링

### OpenAI
```typescript
callbacks: [
  new N8nLlmTracing(this)
],
onFailedAttempt: makeN8nLlmFailedAttemptHandler(this)
```
- 표준 LangChain 콜백
- 자동 재시도

### Anthropic
```typescript
callbacks: [
  new N8nLlmTracing(this, {
    tokensUsageParser  // 토큰 사용량 파싱
  })
],
onFailedAttempt: makeN8nLlmFailedAttemptHandler(this)
```
- 토큰 사용량 추적 강화
- 커스텀 파서

### Google Gemini
```typescript
callbacks: [
  new N8nLlmTracing(this, {
    errorDescriptionMapper  // 에러 메시지 매핑
  })
],
onFailedAttempt: makeN8nLlmFailedAttemptHandler(this)
```
- 에러 메시지 변환
- Google 특화 에러 핸들링

---

## 📊 6. API 요청 구조 비교

### OpenAI API 요청
```json
POST https://api.openai.com/v1/chat/completions
{
  "model": "gpt-4o-mini",
  "messages": [
    {"role": "system", "content": "..."},
    {"role": "user", "content": "..."}
  ],
  "temperature": 0.7,
  "max_tokens": 1000,
  "top_p": 1,
  "frequency_penalty": 0,
  "presence_penalty": 0,
  "response_format": {"type": "json_object"}
}
```

### Anthropic API 요청
```json
POST https://api.anthropic.com/v1/messages
{
  "model": "claude-3-5-sonnet-20241022",
  "max_tokens": 4096,
  "temperature": 0.7,
  "top_k": 40,
  "top_p": 0.9,
  "thinking": {
    "type": "enabled",
    "budget_tokens": 1024
  },
  "messages": [
    {"role": "user", "content": "..."}
  ]
}
```

### Google Gemini API 요청
```json
POST https://generativelanguage.googleapis.com/v1/models/gemini-2.5-flash:generateContent
{
  "contents": [
    {"role": "user", "parts": [{"text": "..."}]}
  ],
  "generationConfig": {
    "temperature": 0.7,
    "topK": 40,
    "topP": 0.9,
    "maxOutputTokens": 1024
  },
  "safetySettings": [...]
}
```

---

## 🔍 7. 주요 차이점 요약

| 특징 | OpenAI | Anthropic | Gemini |
|------|--------|-----------|--------|
| **메시지 구조** | `messages[]` | `messages[]` | `contents[]` |
| **파라미터 네임** | camelCase | snake_case | camelCase |
| **토큰 제한 키** | `max_tokens` | `max_tokens` | `maxOutputTokens` |
| **고유 기능** | `response_format`<br/>`reasoning_effort` | `thinking`<br/>`invocationKwargs` | `safetySettings` |
| **프록시 설정** | `fetchOptions.dispatcher` | `httpAgent` | 미지원 |
| **재시도 로직** | 내장 | 내장 | 내장 |
| **스트리밍** | ✅ | ✅ | ✅ |

---

## 💡 8. 실제 사용 차이

### 모델 선택 방식

**OpenAI (v1.2+)**:
```typescript
this.getNodeParameter('model.value', itemIndex)
// Resource Locator 사용 (동적 검색)
```

**Anthropic (v1.3+)**:
```typescript
this.getNodeParameter('model.value', itemIndex)
// Resource Locator + API 기반 모델 목록
```

**Gemini**:
```typescript
this.getNodeParameter('modelName', itemIndex)
// 직접 모델 이름 (단순 문자열)
```

---

### 옵션 설정 패턴

**OpenAI** - 최대 유연성:
```typescript
options = {
  baseURL, frequencyPenalty, maxTokens,
  presencePenalty, temperature, topP,
  responseFormat, reasoningEffort,
  timeout, maxRetries
}
```

**Anthropic** - 중간 수준:
```typescript
options = {
  maxTokensToSample, temperature,
  topK, topP, thinking, thinkingBudget
}
```

**Gemini** - 기본 설정 중심:
```typescript
options = {
  maxOutputTokens: 1024,  // 기본값 명시
  temperature: 0.7,
  topK: 40,
  topP: 0.9,
  safetySettings
}
```

---

## 🎯 핵심 차이점

### 1. **API 설계 철학**
- **OpenAI**: 범용성 (Azure, 커스텀 엔드포인트)
- **Anthropic**: 단순성 + 고급 기능 (Thinking)
- **Gemini**: 안전성 우선 (Safety Settings)

### 2. **파라미터 전달 방식**
- **OpenAI**: `modelKwargs`로 추가 파라미터
- **Anthropic**: `invocationKwargs`로 직접 API 제어
- **Gemini**: 구조화된 옵션 (중첩 객체)

### 3. **프록시 및 네트워크**
- **OpenAI**: 완전한 프록시 지원
- **Anthropic**: HTTP(S) 에이전트 커스터마이징
- **Gemini**: 프록시 미지원

### 4. **에러 처리**
- **OpenAI**: 표준 재시도
- **Anthropic**: 토큰 사용량 추적 강화
- **Gemini**: Google 특화 에러 매핑

---

## 📚 참고 코드 경로

```
packages/@n8n/nodes-langchain/nodes/llms/
├── LMChatOpenAi/
│   └── LmChatOpenAi.node.ts      (OpenAI 구현)
├── LMChatAnthropic/
│   └── LmChatAnthropic.node.ts   (Claude 구현)
└── LmChatGoogleGemini/
    └── LmChatGoogleGemini.node.ts (Gemini 구현)
```

---

## 📅 문서 정보
- **n8n 버전**: 1.113.3
- **작성일**: 2025-10-16
- **최종 업데이트**: 2025-10-16
