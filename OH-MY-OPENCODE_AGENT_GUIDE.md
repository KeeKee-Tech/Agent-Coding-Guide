# Oh-My-OpenCode 최적 사용 가이드

> OpenCode + Oh-My-OpenCode를 가장 잘 쓰는 방법에 대한 나침반.

---

## 1. 인증 — 이렇게 해라

| 모델 그룹 | 인증 방법 | 비고 |
|-----------|----------|------|
| **Claude** | **API 키** 사용 | OAuth는 Anthropic이 어뷰징으로 규정 → 계정 정지 위험 |
| **Gemini** | **antigravity OAuth** (Google 계정) | 구독 없이 사용 가능 |
| **GPT · GLM · Kimi** | **OpenCode Zen API 키** | 단일 키로 모든 모델 접근 |

> **Claude만 주의**: OAuth 대신 반드시 Anthropic API 키로 설정할 것.  
> OpenAI는 ChatGPT 구독 OAuth를 공식 허용하므로 GPT는 어느 방법이든 괜찮다.

---

## 2. 구독 — 이것만 해라

**Google AI Pro ($20/월) 하나로 충분하다.**

- Gemini 할당량이 5시간마다 갱신 → 사실상 제한 없이 사용 가능
- 무료 상태는 주간 한도라 금방 막힘

Claude는 구독 대신 **API 키 pay-as-you-go**로 사용하는 게 맞다.  
GPT는 **OpenCode Zen 쿼터**가 소진될 때 ChatGPT Plus 구독을 고려하면 된다.

---

## 3. 모델 설정 — 최종 목표

| 에이전트 / 카테고리 | 최적 모델 |
|--------------------|----------|
| sisyphus (오케스트레이터) | `anthropic/claude-opus-4-6` |
| oracle (추론 컨설턴트) | `opencode/gpt-5.2` |
| explore (코드 탐색) | `xai/grok-code-fast-1` |
| librarian (레퍼런스 탐색) | `opencode/glm-4.7` |
| prometheus / metis (계획·분석) | `anthropic/claude-opus-4-6` |
| momus (리뷰) | `opencode/gpt-5.2` |
| hephaestus (코딩 전문) | `opencode/gpt-5.3-codex` |
| visual-engineering / artistry | `google/gemini-3-pro-preview` |
| writing | `google/gemini-3-flash-preview` |
| deep | `opencode/gpt-5.3-codex` |
| ultrabrain | `opencode/gpt-5.3-codex` (xhigh) |
| quick | `anthropic/claude-haiku-4-5` |

---

## 4. 업그레이드 순서

### Phase 1 — 지금 당장 (무료)
현재 상태 유지. 단, **Claude OAuth를 API 키로 교체**하는 것이 최우선.

```bash
opencode auth login  # → Anthropic → API Key
```

### Phase 2 — +$20/월
**Google AI Pro 구독** → Gemini 계열 할당량 확보.  
이것만 해도 Gemini 에이전트 전체가 실용적으로 변한다.

### Phase 3 — +$40~60/월 (최종)
Claude 에이전트를 **claude-opus-4-6**으로 업그레이드.  
`sisyphus`, `prometheus`, `metis`, `unspecified-high` 카테고리가 대상.

---

## 5. OpenCode 업그레이드

```bash
# 최신 버전으로 업그레이드 (내장 명령어)
opencode upgrade

# 또는 npm으로 직접
npm install -g opencode-ai@latest
```

특정 버전으로 고정하려면:

```bash
opencode upgrade v1.2.10
```

---

## 6. Oh-My-OpenCode 업그레이드 & 재설정

### 업그레이드

```bash
bunx oh-my-opencode install
```

설치와 업그레이드가 동일 명령어. 실행하면 최신 버전으로 갱신되고, 기존 설정(`oh-my-opencode.json`)은 **자동 백업** 후 재생성된다.

### 환경설정 재설정

구독 구성이 바뀌었거나 설정을 처음부터 다시 잡고 싶을 때:

```bash
# 1. 기존 설정 파일 삭제 (백업은 .bak 파일로 남아있음)
rm ~/.config/opencode/oh-my-opencode.json

# 2. 재설치 — TUI 마법사가 뜨면서 구독 여부를 다시 물어봄
bunx oh-my-opencode install
```

TUI 없이 플래그로 바로 지정하려면:

```bash
# Claude API키 + OpenCode Zen 사용 기준
bunx oh-my-opencode install --no-tui --claude=yes --openai=no --gemini=no --opencode-zen=yes
```

| 플래그 | 의미 |
|--------|------|
| `--claude=yes` | Claude 구독/API키 보유 |
| `--claude=max20` | Claude Max $20 플랜 |
| `--openai=yes` | ChatGPT Plus/Pro 보유 |
| `--gemini=yes` | Google AI Pro 보유 |
| `--opencode-zen=yes` | OpenCode Zen API 키 보유 |
| `--no-tui` | 대화형 마법사 없이 바로 실행 |

### 설정 파일 위치

| 파일 | 설명 |
|------|------|
| `~/.config/opencode/oh-my-opencode.json` | 에이전트별 모델 설정 |
| `~/.config/opencode/opencode.json` | 플러그인·프로바이더 설정 |
| `~/.config/opencode/oh-my-opencode.json.bak.*` | 자동 백업 (날짜별) |

---

## 참고: 최종 설정 JSON

```json
{
  "agents": {
    "sisyphus":                { "model": "anthropic/claude-opus-4-6",     "variant": "high" },
    "oracle":                  { "model": "opencode/gpt-5.2",              "variant": "high" },
    "librarian":               { "model": "opencode/glm-4.7" },
    "explore":                 { "model": "xai/grok-code-fast-1" },
    "prometheus":              { "model": "anthropic/claude-opus-4-6",     "variant": "max" },
    "metis":                   { "model": "anthropic/claude-opus-4-6",     "variant": "max" },
    "momus":                   { "model": "opencode/gpt-5.2",              "variant": "medium" },
    "hephaestus":              { "model": "opencode/gpt-5.3-codex",        "variant": "medium" },
    "atlas":                   { "model": "anthropic/claude-sonnet-4-5" },
    "multimodal-looker":       { "model": "google/gemini-3-flash-preview" },
    "frontend-ui-ux-engineer": { "model": "google/antigravity-gemini-3-pro-high" },
    "document-writer":         { "model": "google/antigravity-gemini-3-flash" }
  },
  "categories": {
    "quick":              { "model": "anthropic/claude-haiku-4-5" },
    "unspecified-low":    { "model": "anthropic/claude-sonnet-4-5" },
    "unspecified-high":   { "model": "anthropic/claude-opus-4-6",   "variant": "max" },
    "visual-engineering": { "model": "google/gemini-3-pro-preview", "variant": "high" },
    "artistry":           { "model": "google/gemini-3-pro-preview", "variant": "high" },
    "writing":            { "model": "google/gemini-3-flash-preview" },
    "deep":               { "model": "opencode/gpt-5.3-codex",      "variant": "medium" },
    "ultrabrain":         { "model": "opencode/gpt-5.3-codex",      "variant": "xhigh" }
  }
}
```

---

*마지막 업데이트: 2026년 2월*
