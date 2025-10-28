# n8n AI 노드 구조 비교 (OpenAI, Anthropic, Gemini)

## 📋 개요

n8n의 주요 AI 노드들은 LangChain 기반으로 구현되어 있으며, 공통된 인터페이스를 제공하면서도 각 AI 제공자의 고유 기능을 지원합니다.

---

## 🔍 상세 비교

### 1️⃣ OpenAI Chat Model

**기본 정보**
- **노드 ID**: `lmChatOpenAi`
- **표시명**: OpenAI Chat Model
- **버전**: 1, 1.1, 1.2
- **카테고리**: AI > Language Models > Chat Models

**인증**
- **타입**: `openAiApi`
- **필수 항목**: API Key
- **선택 항목**: Base URL (커스텀 엔드포인트 지원)

**핵심 파라미터**

| 파라미터 | 기본값 | 범위/옵션 | 설명 |
|---------|--------|----------|------|
| Model | `gpt-4o-mini` | 동적 모델 목록 | 사용할 GPT 모델 선택 |
| Temperature | - | 0 ~ 2 | 응답의 창의성 조절 |
| Maximum Tokens | - | 최대 32,768 | 최대 출력 토큰 수 |
| Frequency Penalty | - | -2 ~ 2 | 반복 단어 억제 |
| Presence Penalty | - | -2 ~ 2 | 새로운 주제 유도 |
| Response Format | text | text / JSON | 응답 형식 지정 |
| Reasoning Effort | - | low / medium / high | 추론 강도 설정 |
| Top P | - | 0 ~ 1 | Nucleus sampling |
| Timeout | - | ms | 요청 타임아웃 |
| Max Retries | - | 정수 | 재시도 횟수 |

**고유 기능**
- ✅ Custom Base URL 지원 (Azure OpenAI 등)
- ✅ JSON 응답 모드
- ✅ Reasoning Effort 조절
- ✅ Proxy agent 통합
- ✅ Tracing 기능

---

### 2️⃣ Anthropic Chat Model (Claude)

**기본 정보**
- **노드 ID**: `lmChatAnthropic`
- **표시명**: Anthropic Chat Model
- **버전**: 1, 1.1, 1.2, 1.3
- **카테고리**: AI > Language Models > Chat Models

**인증**
- **타입**: `anthropicApi`
- **필수 항목**: API Key
- **선택 항목**: Custom URL

**핵심 파라미터**

| 파라미터 | 기본값 | 범위/옵션 | 설명 |
|---------|--------|----------|------|
| Model | - | Claude 3.5 Sonnet<br/>Claude 3 Opus<br/>Claude 3 Haiku 등 | 사용할 Claude 모델 |
| Temperature | 0.7 | 0 ~ 1 | 응답의 창의성 조절 |
| Maximum Tokens | 4,096 | 정수 | 최대 출력 토큰 수 |
| Top K | - | 정수 | Top-K sampling |
| Top P | - | 0 ~ 1 | Nucleus sampling |
| **Thinking Mode** | off | on / off | 🌟 Claude 고유 추론 모드 |
| **Thinking Budget** | 1,024 | 최소 1,024 | 🌟 추론 할당 토큰 |

**고유 기능**
- ✅ **Thinking Mode** - Claude의 내부 추론 과정 활성화
- ✅ **Thinking Budget** - 추론에 할당할 토큰 수 설정
- ✅ 동적 모델 검색
- ✅ Token usage tracing
- ✅ Proxy 지원

---

### 3️⃣ Google Gemini Chat Model

**기본 정보**
- **노드 ID**: `lmChatGoogleGemini`
- **표시명**: Google Gemini Chat Model
- **버전**: 1
- **카테고리**: AI > Language Models > Chat Models

**인증**
- **타입**: `googlePalmApi`
- **필수 항목**: API Key
- **선택 항목**: Host URL

**핵심 파라미터**

| 파라미터 | 기본값 | 범위/옵션 | 설명 |
|---------|--------|----------|------|
| Model | `gemini-2.5-flash` | 동적 모델 목록 | 사용할 Gemini 모델 |
| Temperature | 0.7 | 0 ~ 1 | 응답의 창의성 조절 |
| Max Output Tokens | 1,024 | 정수 | 최대 출력 토큰 수 |
| Top K | 40 | 정수 | Top-K sampling |
| Top P | 0.9 | 0 ~ 1 | Nucleus sampling |
| **Safety Settings** | - | 설정 객체 | 🌟 콘텐츠 안전 설정 |

**고유 기능**
- ✅ **Safety Settings** - Google의 콘텐츠 안전 필터 설정
- ✅ Google API에서 실시간 모델 목록 로드
- ✅ 커스텀 에러 매핑
- ❌ Proxy 미지원

---

## 📊 종합 비교표

| 항목 | OpenAI | Anthropic | Gemini |
|------|--------|-----------|--------|
| **버전 수** | 3개 (1, 1.1, 1.2) | 4개 (1, 1.1, 1.2, 1.3) | 1개 |
| **기본 Max Tokens** | 32,768 | 4,096 | 1,024 |
| **Temperature 범위** | 0 ~ 2 | 0 ~ 1 | 0 ~ 1 |
| **고유 기능** | Reasoning Effort<br/>JSON 모드 | Thinking Mode<br/>Thinking Budget | Safety Settings |
| **Response Format** | ✅ text/JSON | ❌ | ❌ |
| **Proxy 지원** | ✅ | ✅ | ❌ |
| **Custom URL** | ✅ | ✅ | ✅ |
| **동적 모델 목록** | ✅ | ✅ | ✅ |
| **Tracing** | ✅ | ✅ | ✅ |

---

## 🏗️ 공통 아키텍처

### 노드 구조
```
LMChat[Provider].node.ts
├── 기본 정보 (name, displayName, version)
├── Credentials 설정
├── Properties (파라미터 정의)
│   ├── Model Selection
│   ├── Temperature
│   ├── Token Limits
│   ├── Sampling Parameters
│   └── Provider-specific Options
├── Methods
│   └── loadOptions (동적 옵션 로드)
└── Execute Logic
```

### 출력 타입
- **Connection Type**: `AiLanguageModel`
- **지원 연결**: AI Chain, AI Agent

### 파일 구조
```
packages/@n8n/nodes-langchain/nodes/llms/
├── LMChatOpenAi/
│   ├── LmChatOpenAi.node.ts
│   ├── openAi.svg
│   └── methods/
├── LMChatAnthropic/
│   ├── LmChatAnthropic.node.ts
│   ├── anthropic.svg
│   └── methods/
└── LmChatGoogleGemini/
    ├── LmChatGoogleGemini.node.ts
    ├── google.svg
    └── methods/
```

---

## 💡 선택 가이드

### OpenAI를 선택해야 할 때
- JSON 형식 응답이 필요한 경우
- Azure OpenAI 등 커스텀 엔드포인트 사용
- 높은 토큰 한도 필요 (최대 32K)
- Reasoning effort 조절이 필요한 복잡한 추론 작업

### Anthropic을 선택해야 할 때
- 긴 문맥 처리 (Claude 3 모델의 200K+ 컨텍스트)
- Thinking Mode로 추론 과정 확인 필요
- 안전하고 정확한 응답 우선

### Gemini를 선택해야 할 때
- Google 생태계 통합
- 콘텐츠 안전 필터링 중요
- 비용 효율적인 솔루션 (Flash 모델)

---

## 🔗 참고 자료

- [n8n Documentation](https://docs.n8n.io)
- [n8n GitHub Repository](https://github.com/n8n-io/n8n)
- OpenAI 노드: `packages/@n8n/nodes-langchain/nodes/llms/LMChatOpenAi/`
- Anthropic 노드: `packages/@n8n/nodes-langchain/nodes/llms/LMChatAnthropic/`
- Gemini 노드: `packages/@n8n/nodes-langchain/nodes/llms/LmChatGoogleGemini/`

---

## 📅 문서 정보
- **n8n 버전**: 1.113.3
- **작성일**: 2025-10-16
- **최종 업데이트**: 2025-10-16
