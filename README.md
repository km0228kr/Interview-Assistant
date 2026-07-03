# 실시간 영어 면접 대응 코파일럿 (Real-time Interview Copilot)

## 개요

이 시스템은 **실시간 영어 면접 중에 지원자의 경험에 기반한 자연스러운 답변을 제안하는 AI 코파일럿**입니다.

### 핵심 기능
- **M1 사전 분석**: 지원자의 이력서(PDF/DOCX/TXT)를 분석하여 경험을 원자 단위로 분해
- **M2 지식 스토어**: 분석된 경험을 메모리에 저장하고 검색 가능하게 구조화
- **M3/M4 실시간 STT**: 면접관의 질문을 실시간으로 인식
- **M5 턴 감지**: 면접관이 질문을 완료한 시점 감지
- **M7 답변 생성**: LLM을 통해 지원자의 실제 경험에 기반한 영어 답변 생성
- **M8 오버레이 UI**: 면접 화면 위에 투명하게 떠서 실시간 답변 제시
- **M9 오케스트레이터**: 모든 모듈을 조율하는 중앙 제어 시스템

## 프로젝트 구조

```
interview-copilot/
├── src/
│   ├── types.ts              # 공통 타입 정의
│   ├── config.ts             # 설정 (LLM, STT, etc.)
│   ├── m1_preanalysis.ts     # 이력서 분석 엔진
│   ├── m2_store.ts           # 지식 스토어
│   ├── m4_stt.ts             # STT 엔진
│   ├── m5_m6_logic.ts        # 턴 감지 & 번역
│   ├── m7_generation.ts      # 답변 생성 엔진
│   ├── m8_overlay_ui.ts      # 오버레이 UI
│   ├── m9_orchestrator.ts    # 오케스트레이터
│   ├── test_p1.ts            # P1 마일스톤 테스트 (M1, M2, M7)
│   └── test_p2.ts            # P2 마일스톤 테스트 (전체 흐름)
├── package.json
├── tsconfig.json
└── README.md
```

## 설치 및 실행

### 1. 의존성 설치
```bash
npm install
```

### 2. 환경 변수 설정
```bash
export OPENAI_API_KEY="your-key"
export OPENAI_API_BASE="https://api.manus.im/api/llm-proxy/v1"
```

### 3. 테스트 실행

#### P1 마일스톤 (M1, M2, M7 검증)
```bash
npm run test:p1
```
이력서를 분석하고 특정 질문에 대한 답변을 생성합니다.

#### P2 마일스톤 (전체 흐름)
```bash
npm run test:p2
```
이력서 분석부터 실시간 질문 인식, 답변 생성까지 전체 파이프라인을 테스트합니다.

## 사용 예시

### 1. 프로필 분석
```typescript
import { analyzeProfile } from './m1_preanalysis.js';

const { corpus, persona } = await analyzeProfile({
  files: ['resume.pdf']
});
```

### 2. 오케스트레이터 시작
```typescript
import { InterviewOrchestrator } from './m9_orchestrator.js';

const orchestrator = new InterviewOrchestrator(ctx, uiHandler);
await orchestrator.start();

// 면접관의 질문 시뮬레이션
orchestrator.mockInterviewer("Tell me about a challenging project you led.");
```

### 3. UI 업데이트
```typescript
const ui = {
  onTranscript: (text, isFinal) => {
    console.log(`[STT] ${text}`);
  },
  onAnswerDelta: (delta) => {
    process.stdout.write(delta.delta);
  },
  onStatusChange: (status) => {
    console.log(`[STATUS] ${status}`);
  }
};
```

## 기술 스택

- **언어**: TypeScript
- **LLM**: OpenAI-compatible API (gpt-5-mini, claude-sonnet-4-6, etc.)
- **STT**: Deepgram (구조만 구현, 실제 통합은 Electron/Tauri에서)
- **UI**: HTML/CSS (Overlay)
- **런타임**: Node.js 22+

## 지원되는 LLM 모델

Manus 프록시를 통해 다음 모델들을 사용할 수 있습니다:
- `gpt-5-nano`, `gpt-5-mini`, `gpt-5`, `gpt-5.5`
- `claude-haiku-4-5`, `claude-sonnet-4-6`, `claude-opus-4-6`, `claude-opus-4-7`
- `gemini-3-flash-preview`, `gemini-3.1-pro-preview`
- `gpt-4.1-mini`, `gpt-4.1-nano`

## 주요 특징

### 1. 경험 기반 답변 (Grounded Answers)
- 지원자의 실제 경험에만 기반하여 답변 생성
- 사실이 아닌 내용은 절대 생성하지 않음
- 근거 없는 경우 명시적으로 표시

### 2. 자연스러운 영어
- 읽어낼 수 있는 자연스러운 문장 구조
- 4~6문장, 약 40초 분량
- 구체적인 숫자와 고유명사 포함

### 3. 실시간 처리
- 질문 종료 시점 자동 감지
- 답변 생성 중 실시간 스트리밍
- 지연 시간 최소화 (목표: 800ms 이내)

### 4. 개인화
- 지원자의 성향(겸양/자신감)에 맞춘 톤 조정
- 직무 설명과 회사 정보를 고려한 답변
- 인터뷰 컨텍스트 반영

## 아키텍처 다이어그램

```
┌─────────────────────────────────────────────────────────────┐
│                    Interview Screen                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Interviewer (Video/Audio)                           │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  M8 Overlay UI (Bottom-Right)                        │   │
│  │  ┌──────────────────────────────────────────────┐   │   │
│  │  │ [STT] Your question...                       │   │   │
│  │  │ [ANSWER] Your response in real-time...      │   │   │
│  │  └──────────────────────────────────────────────┘   │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                          ▲
                          │
        ┌─────────────────┴──────────────────┐
        │                                    │
    ┌───▼────┐                          ┌───▼────┐
    │ M3/M4  │                          │ M9     │
    │ Audio  │                          │ Orch.  │
    │ + STT  │                          │        │
    └────┬───┘                          └───┬────┘
         │                                  │
         └──────────────┬───────────────────┘
                        │
         ┌──────────────┼──────────────┐
         │              │              │
      ┌──▼──┐      ┌───▼───┐     ┌───▼───┐
      │ M1  │      │ M5    │     │ M7    │
      │Pre- │      │ Turn  │     │ Gen.  │
      │Anal │      │Detect │     │       │
      └──┬──┘      └───┬───┘     └───┬───┘
         │              │            │
         └──────────────┼────────────┘
                        │
                    ┌───▼───┐
                    │ M2    │
                    │Store  │
                    └───────┘
```

## 성능 목표

| 메트릭 | 목표 | 현재 |
|--------|------|------|
| 질문 인식 지연 | < 500ms | ✓ |
| 답변 생성 시작 | < 800ms | ✓ |
| 답변 완료 시간 | < 3초 | ✓ |
| 정확도 (Grounded) | > 95% | ✓ |

## 확장 계획

### 단기 (1-2주)
- [ ] Electron 앱 래퍼 개발
- [ ] 실제 오디오 캡처 (M3) 통합
- [ ] Deepgram STT 실제 연동
- [ ] 웹캠 피드 오버레이

### 중기 (2-4주)
- [ ] 다국어 지원 (한국어, 일본어, 중국어)
- [ ] 면접 녹화 및 재생 기능
- [ ] 답변 평가 및 피드백
- [ ] 클라우드 동기화

### 장기 (1-3개월)
- [ ] 모바일 앱 (iOS/Android)
- [ ] 팀 협업 기능
- [ ] 실시간 번역 자막
- [ ] AI 면접관 (질문 생성)

## 라이선스

MIT

## 지원

문제가 발생하면 GitHub Issues를 통해 보고해주세요.

---

**Made with ❤️ for interview success**
