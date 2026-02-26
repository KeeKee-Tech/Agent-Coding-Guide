# Oh-My-OpenCode 에이전트 가이드

> 각 에이전트별 모델 구성, 인증 방법, 구독 필요 여부 및 추천 사항을 정리한 문서.
> 기준: oh-my-opencode / opencode 1.2.12 (2026년 2월)

---

## 목차

- [에이전트별 모델 구성](#에이전트별-모델-구성)
- [태스크 카테고리별 모델](#태스크-카테고리별-모델)
- [인증 방법 가이드](#인증-방법-가이드)
- [구독 필요 여부 및 추천](#구독-필요-여부-및-추천)
- [공식 추천 모델 조합 및 업그레이드 로드맵](#공식-추천-모델-조합-및-업그레이드-로드맵)
- [참고: 인증 파일 위치](#참고-인증-파일-위치)

---

## 에이전트별 모델 구성

`~/.config/opencode/oh-my-opencode.json` 기준.

| 에이전트 | 모델 | Variant | 역할 |
|---------|------|---------|------|
| **sisyphus** | `anthropic/claude-sonnet-4-6` | `high` | 메인 오케스트레이터 |
| **oracle** | `opencode/gpt-5.2` | `high` | 고난이도 추론 · 디버깅 컨설턴트 |
| **librarian** | `opencode/glm-4.7-free` | — | 외부 문서 · 레퍼런스 탐색 |
| **explore** | `anthropic/claude-haiku-4-5` | — | 코드베이스 내부 탐색 |
| **prometheus** | `anthropic/claude-sonnet-4-6` | `max` | 작업 계획 수립 |
| **metis** | `anthropic/claude-sonnet-4-6` | `max` | 사전 분석 · 모호성 파악 |
| **momus** | `opencode/gpt-5.2` | `medium` | 작업 계획 검토 · 리뷰 |
| **atlas** | `opencode/kimi-k2.5-free` | — | 보조 에이전트 |
| **hephaestus** | `opencode/gpt-5.3-codex` | `medium` | 고난이도 코딩 전문 |
| **multimodal-looker** | `google/gemini-3-flash-preview` | — | 이미지 · PDF 분석 |
| **frontend-ui-ux-engineer** | `google/antigravity-gemini-3-pro-high` | — | 프론트엔드 UI/UX 구현 |
| **document-writer** | `google/antigravity-gemini-3-flash` | — | 문서 작성 |

---

## 태스크 카테고리별 모델

`task(category="...")` 위임 시 사용되는 모델.

| 카테고리 | 모델 | Variant | 용도 |
|---------|------|---------|------|
| `quick` | `anthropic/claude-haiku-4-5` | — | 단순 수정 · 빠른 작업 |
| `unspecified-low` | `anthropic/claude-sonnet-4-6` | — | 일반 저난이도 작업 |
| `unspecified-high` | `anthropic/claude-sonnet-4-6` | — | 일반 고난이도 작업 |
| `visual-engineering` | `google/gemini-3-pro-preview` | `high` | 프론트엔드 · UI/UX · 디자인 |
| `artistry` | `google/gemini-3-pro-preview` | `high` | 창의적 접근이 필요한 작업 |
| `writing` | `google/gemini-3-flash-preview` | — | 문서 · 기술 문서 작성 |
| `deep` | `opencode/gpt-5.3-codex` | `medium` | 심층 분석 · 복잡한 문제 해결 |
| `ultrabrain` | `opencode/gpt-5.3-codex` | `xhigh` | 극고난이도 로직 · 수학 |

---

## 인증 방법 가이드

현재 설정된 인증 방식 (`opencode auth list` 기준):

```
● Anthropic    (oauth)
● Google       (oauth)  ← antigravity 플러그인 방식
● OpenCode Zen (api)
```

---

### 1. Anthropic — Claude 모델

**해당 에이전트**: sisyphus, explore, prometheus, metis  
**해당 카테고리**: quick, unspecified-low, unspecified-high

| 항목 | 내용 |
|------|------|
| 현재 인증 방식 | OAuth (Claude 구독 계정 연동) |
| API 키 방식 | 가능 (`opencode auth login` → Anthropic → API Key 입력) |
| **⚠️ 주의** | Anthropic이 서드파티 도구에서 구독 OAuth 사용을 **어뷰징으로 규정, 계정 정지 경고** 공지. 기술적으로는 현재 작동하나 위험 있음 |
| 권장 방향 | 장기적으로 **Anthropic API 키** 방식으로 전환 권장 |

> 반면 **OpenAI는 opencode에서 ChatGPT 구독 OAuth 사용을 공식 허용**했다.  
> Claude는 위험, ChatGPT는 허용이라는 점이 명확한 차이점이다.

---

### 2. Google — Gemini 모델 (antigravity)

**해당 에이전트**: multimodal-looker, frontend-ui-ux-engineer, document-writer  
**해당 카테고리**: visual-engineering, artistry, writing

| 항목 | 내용 |
|------|------|
| 현재 인증 방식 | antigravity OAuth (Google 계정 연동) |
| **antigravity 정체** | Google Cloud Shell Editor / Gemini Code Assist 서비스를 활용하는 방식. Google IDE 플러그인으로 위장해 Gemini API에 접근 |
| 필요 조건 | **Google 계정만 있으면 됨** (Gemini 구독 불필요) |
| Dual Quota 시스템 | `gemini-antigravity` 풀 + `gemini-cli` 풀 2개 독립 운용 → 사실상 **2배 할당량** |

#### Gemini Plus 구독 vs antigravity

흔히 혼동되는 부분:

| 서비스 | 용도 | antigravity와 관계 |
|--------|------|--------------------|
| **Gemini Plus/Advanced** (`gemini.google.com`) | 웹 채팅 인터페이스 | ❌ **직접 연관 없음** |
| **Gemini Code Assist** (개발자 도구) | IDE 내 코딩 보조 | ✅ **antigravity가 이걸 활용** |
| **Google AI Pro/Ultra 구독** | 개발자 API 할당량 포함 | ✅ **antigravity 할당량에 직접 영향** |

---

### 3. OpenCode Zen — GPT · GLM · Kimi 모델

**해당 에이전트**: oracle, momus, hephaestus, librarian, atlas

| 항목 | 내용 |
|------|------|
| 인증 방식 | OpenCode Zen API 키 (`sk-...` 형태) |
| 제공 모델 | `gpt-5.2`, `gpt-5.3-codex`, `glm-4.7-free`, `kimi-k2.5-free` 등 |
| 특징 | OpenAI · Zhipu · Moonshot 등 여러 회사 모델을 **단일 API 키**로 접근하는 프록시 서비스 |
| ChatGPT 구독 필요? | ❌ **불필요**. OpenCode Zen 키 하나로 GPT 모델 사용 가능 |
| 주의사항 | OpenCode Zen 크레딧/쿼터 소진 시 사용 불가 |

#### ChatGPT Plus OAuth 전환도 가능

ChatGPT Plus/Pro 구독이 있다면 OpenCode Zen 대신 ChatGPT OAuth 방식으로 전환 가능.  
(OpenAI가 공식 허용한 방식)

```bash
opencode auth login  # → OpenAI → ChatGPT Plus/Pro 선택
```

---

## 구독 필요 여부 및 추천

### 현재 설정 기준 요약

| 모델 그룹 | 구독 없이 사용 가능? | 비고 |
|---------|:---------------:|------|
| Claude (Anthropic) | ✅ OAuth 작동 중 | 계정 정지 위험 있음 |
| Gemini (Google) | ✅ antigravity 무료 사용 | 무료 할당량 제한 있음 |
| GPT 계열 | ✅ OpenCode Zen으로 사용 | Zen 쿼터에 의존 |

---

### 구독별 상세 추천

#### Google AI Pro/Ultra — antigravity 할당량 직접 연동

| 구독 | 월 비용 | antigravity RPM | antigravity RPD | 갱신 주기 | 추천 |
|------|:------:|:---------------:|:---------------:|:-------:|:----:|
| **무료** | $0 | 제한적 | 주간 한도 | 주 1회 | 가벼운 사용 |
| **Google AI Pro** | ~$20 | 120 | 1,500 | **5시간마다** | ✅ 일반 개발자 추천 |
| **Google AI Ultra** | ~$250 | 120 | 2,000 | **5시간마다** | 매우 높은 사용량 |

> **핵심 포인트**  
> - Google AI Pro 구독 시 antigravity 할당량이 **직접 증가**하고, **5시간마다 갱신**된다.  
> - 무료 티어는 주간 할당량이라 많이 쓰면 막힌다.  
> - visual-engineering, artistry, writing 카테고리 및 Gemini 에이전트를 자주 쓴다면 Pro 구독은 실질적 가치가 있다.

#### Anthropic API 키 — Claude 계정 정지 위험 해소

| 항목 | 내용 |
|------|------|
| 비용 | 사용량 기반 과금 (구독 없음) |
| 장점 | 계정 정지 위험 없음, 안정적 운용 |
| 단점 | 사용량에 따라 비용 발생 |
| 권장 대상 | Claude 모델을 주력으로 많이 쓰는 경우 |

#### ChatGPT Plus/Pro — GPT OAuth 전환 시

| 구독 | 월 비용 | 장점 | 추천 |
|------|:------:|------|:----:|
| **ChatGPT Plus** | ~$20 | OpenCode Zen 쿼터 독립, GPT 직접 접근 | OpenCode Zen 부족 시 고려 |
| **ChatGPT Pro** | ~$200 | 높은 사용량, o1 Pro 등 | 매우 높은 사용량 |

> OpenCode Zen 쿼터가 충분하다면 ChatGPT 구독은 필수가 아니다.

---

### 최종 추천 우선순위

```
1순위. Google AI Pro 구독 ($20/월) ← 가장 즉각적인 효과
   → antigravity 할당량 5시간 갱신으로 실용적 무제한
   → visual-engineering, artistry, writing 카테고리 + Gemini 에이전트 전체 혜택

2순위. Anthropic API 키 발급 (pay-as-you-go)
   → 현재 OAuth 방식의 계정 정지 위험 해소
   → Claude 모델(sisyphus, explore, prometheus, metis) 안정적 운용

3순위. ChatGPT Plus 구독 ($20/월) — 선택사항
   → OpenCode Zen 쿼터 문제 발생 시 대안
   → oracle, momus, hephaestus 등 GPT 에이전트 독립적 운용
```

---

## 공식 추천 모델 조합 및 업그레이드 로드맵

oh-my-opencode가 공식적으로 설계한 최적 모델 조합을 향해 단계적으로 업그레이드하는 로드맵.  
현재 설정에서 시작해 Phase 3 최종 목표까지 순서대로 진행하면 된다.

---

### 현재 vs 공식 추천 비교표

| 에이전트/카테고리 | 현재 설정 | 공식 최종 추천 | 변경 이유 |
|----------------|----------|--------------|---------|
| **sisyphus** | `claude-sonnet-4-6` (high) | `claude-opus-4-6` (high) | 오케스트레이터는 최고 성능 필요 |
| **oracle** | `gpt-5.2` (high) | `gpt-5.2` (high) | ✅ 이미 최적 |
| **librarian** | `glm-4.7-free` | `glm-4.7` (유료) | free 모델은 성능/속도 제한 |
| **explore** | `claude-haiku-4-5` | `xai/grok-code-fast-1` | 코드 탐색 특화 모델로 교체 |
| **prometheus** | `claude-sonnet-4-6` (max) | `claude-opus-4-6` (max) | 계획 수립은 최고 모델 필요 |
| **metis** | `claude-sonnet-4-6` (max) | `claude-opus-4-6` (max) | 분석 깊이 향상 |
| **momus** | `gpt-5.2` (medium) | `gpt-5.2` (medium) | ✅ 이미 최적 |
| **hephaestus** | `gpt-5.3-codex` (medium) | `gpt-5.3-codex` (medium) | ✅ 이미 최적 |
| **atlas** | `kimi-k2.5-free` | `claude-sonnet-4-5` | 보조 에이전트 품질 향상 |
| **unspecified-high** | `claude-sonnet-4-6` | `claude-opus-4-6` (max) | 고난이도 작업 품질 향상 |
| **deep** | `gpt-5.3-codex` (medium) | `gpt-5.3-codex` (medium) | ✅ 이미 최적 |
| **ultrabrain** | `gpt-5.3-codex` (xhigh) | `gpt-5.3-codex` (xhigh) | ✅ 이미 최적 |

---

### Phase 1 — 현재 상태 (무료 중심, $0 추가 비용)

> **목표**: 현재 설정을 그대로 유지하며 안정적으로 사용하는 단계.  
> 무료 모델(glm-4.7-free, kimi-k2.5-free)과 OpenCode Zen에 의존.

```json
{
  "agents": {
    "sisyphus":               { "model": "anthropic/claude-sonnet-4-6",     "variant": "high" },
    "oracle":                 { "model": "opencode/gpt-5.2",                "variant": "high" },
    "librarian":              { "model": "opencode/glm-4.7-free" },
    "explore":                { "model": "anthropic/claude-haiku-4-5" },
    "prometheus":             { "model": "anthropic/claude-sonnet-4-6",     "variant": "max" },
    "metis":                  { "model": "anthropic/claude-sonnet-4-6",     "variant": "max" },
    "momus":                  { "model": "opencode/gpt-5.2",                "variant": "medium" },
    "hephaestus":             { "model": "opencode/gpt-5.3-codex",          "variant": "medium" },
    "atlas":                  { "model": "opencode/kimi-k2.5-free" },
    "multimodal-looker":      { "model": "google/gemini-3-flash-preview" },
    "frontend-ui-ux-engineer":{ "model": "google/antigravity-gemini-3-pro-high" },
    "document-writer":        { "model": "google/antigravity-gemini-3-flash" }
  },
  "categories": {
    "quick":              { "model": "anthropic/claude-haiku-4-5" },
    "unspecified-low":    { "model": "anthropic/claude-sonnet-4-6" },
    "unspecified-high":   { "model": "anthropic/claude-sonnet-4-6" },
    "visual-engineering": { "model": "google/gemini-3-pro-preview",   "variant": "high" },
    "artistry":           { "model": "google/gemini-3-pro-preview",   "variant": "high" },
    "writing":            { "model": "google/gemini-3-flash-preview" },
    "deep":               { "model": "opencode/gpt-5.3-codex",        "variant": "medium" },
    "ultrabrain":         { "model": "opencode/gpt-5.3-codex",        "variant": "xhigh" }
  }
}
```

**Phase 1 주의사항**:
- Claude OAuth는 계정 정지 위험이 있으므로 모니터링 필요
- 무료 모델(`glm-4.7-free`, `kimi-k2.5-free`)은 속도와 품질이 제한적
- OpenCode Zen 쿼터 소진 시 GPT/GLM/Kimi 계열 에이전트 중단

---

### Phase 2 — 안정화 단계 (월 ~$20 추가)

> **목표**: Claude 계정 정지 위험 해소 + Gemini 할당량 확보.  
> 인증을 안전하게 바꾸고, Gemini 에이전트를 실용적으로 사용하는 단계.

**변경 사항**:
1. **Anthropic API 키 발급** → OAuth에서 API 키로 전환 (계정 정지 위험 해소)
2. **Google AI Pro 구독** ($20/월) → antigravity 할당량 5시간 갱신 확보

```bash
# Anthropic API 키 전환
opencode auth login  # → Anthropic → API Key 선택 → sk-ant-... 입력

# Google AI Pro 구독 후 이미 antigravity OAuth가 설정되어 있으면 자동 반영
```

**모델 설정은 Phase 1과 동일**, 인증만 변경.

**Phase 2 달성 효과**:
- Claude 모델: 안전하고 안정적인 사용 ✅
- Gemini 에이전트: 5시간마다 할당량 갱신으로 사실상 제한 없이 사용 ✅
- GPT/GLM: OpenCode Zen 그대로 (Zen 쿼터에 의존)

---

### Phase 3 — 최적 설정 (월 ~$40~60 추가, 최종 목표)

> **목표**: oh-my-opencode 공식 추천 모델 조합 완성.  
> 핵심 에이전트를 최고 성능 모델로 교체하고, 무료 모델에서 탈피.

**추가 변경 사항**:
1. `sisyphus`, `prometheus`, `metis` → **claude-opus-4-6** 업그레이드
2. `explore` → **xai/grok-code-fast-1** 교체 (코드 탐색 특화)
3. `librarian` → **glm-4.7** 유료 버전 교체
4. `atlas` → **claude-sonnet-4-5** 교체
5. `unspecified-high` 카테고리 → **claude-opus-4-6 (max)**

```json
{
  "agents": {
    "sisyphus":               { "model": "anthropic/claude-opus-4-6",      "variant": "high" },
    "oracle":                 { "model": "opencode/gpt-5.2",               "variant": "high" },
    "librarian":              { "model": "opencode/glm-4.7" },
    "explore":                { "model": "xai/grok-code-fast-1" },
    "prometheus":             { "model": "anthropic/claude-opus-4-6",      "variant": "max" },
    "metis":                  { "model": "anthropic/claude-opus-4-6",      "variant": "max" },
    "momus":                  { "model": "opencode/gpt-5.2",               "variant": "medium" },
    "hephaestus":             { "model": "opencode/gpt-5.3-codex",         "variant": "medium" },
    "atlas":                  { "model": "anthropic/claude-sonnet-4-5" },
    "multimodal-looker":      { "model": "google/gemini-3-flash-preview" },
    "frontend-ui-ux-engineer":{ "model": "google/antigravity-gemini-3-pro-high" },
    "document-writer":        { "model": "google/antigravity-gemini-3-flash" }
  },
  "categories": {
    "quick":              { "model": "anthropic/claude-haiku-4-5" },
    "unspecified-low":    { "model": "anthropic/claude-sonnet-4-5" },
    "unspecified-high":   { "model": "anthropic/claude-opus-4-6",    "variant": "max" },
    "visual-engineering": { "model": "google/gemini-3-pro-preview",  "variant": "high" },
    "artistry":           { "model": "google/gemini-3-pro-preview",  "variant": "high" },
    "writing":            { "model": "google/gemini-3-flash-preview" },
    "deep":               { "model": "opencode/gpt-5.3-codex",       "variant": "medium" },
    "ultrabrain":         { "model": "opencode/gpt-5.3-codex",       "variant": "xhigh" }
  }
}
```

**Phase 3 달성 효과**:
- 메인 오케스트레이터(sisyphus)가 Opus급으로 업그레이드 → 작업 품질 대폭 향상 ✅
- 계획/분석 에이전트(prometheus, metis)도 Opus급 → 더 정밀한 사전 계획 ✅
- 코드 탐색(explore)이 grok-code-fast-1로 교체 → 속도 + 코드 특화 성능 ✅
- librarian 유료 전환 → 레퍼런스 검색 품질 향상 ✅

---

### 단계별 로드맵 요약

```
Phase 1 (현재) ─────────────────────────────────────────────► 무료 중심
   claude-sonnet-4-6 (sisyphus) + glm-4.7-free + kimi-k2.5-free
   추가 비용: $0/월

      ↓ Anthropic API 키 발급 + Google AI Pro 구독

Phase 2 (안정화) ────────────────────────────────────────────► 인증 안전화
   동일 모델, Claude 계정 정지 위험 해소 + Gemini 5시간 갱신
   추가 비용: ~$20/월 (Google AI Pro)

      ↓ claude-opus-4-6 업그레이드 + grok-code-fast-1 교체

Phase 3 (최적, 최종 목표) ───────────────────────────────────► 최고 성능
   claude-opus-4-6 (sisyphus/prometheus/metis) + grok-code-fast-1 (explore)
   추가 비용: ~$40~60/월 (Anthropic API 사용량 포함)
```

---

### Phase 업그레이드 체크리스트

#### Phase 1 → Phase 2

- [ ] Anthropic API 키 발급 (`console.anthropic.com`)
- [ ] `opencode auth login` → Anthropic → API Key 방식으로 재설정
- [ ] Google AI Pro 구독 (Google One → AI Pro)
- [ ] antigravity 할당량 갱신 주기 확인 (5시간마다)

#### Phase 2 → Phase 3

- [ ] Anthropic API 잔액 충전 (claude-opus-4-6은 sonnet 대비 5배 비용)
- [ ] oh-my-opencode.json에서 sisyphus 모델을 `anthropic/claude-opus-4-6`으로 변경
- [ ] prometheus, metis 모델도 `anthropic/claude-opus-4-6`으로 변경
- [ ] explore를 `xai/grok-code-fast-1`으로 변경 (XAI 인증 필요 시 별도 설정)
- [ ] unspecified-high 카테고리를 `anthropic/claude-opus-4-6` (max)으로 변경
- [ ] OpenCode Zen에서 `glm-4.7` (유료) 사용 가능 여부 확인 후 librarian 교체

---

## 참고: 인증 파일 위치

| 파일 | 내용 |
|------|------|
| `~/.local/share/opencode/auth.json` | opencode 인증 토큰 (Anthropic · Google · OpenCode Zen) |
| `~/.config/opencode/oh-my-opencode.json` | 에이전트별 모델 설정 |
| `~/.config/opencode/opencode.json` | opencode 전체 설정 (provider · plugin) |
| `%APPDATA%\opencode\antigravity-accounts.json` | antigravity Google 계정 정보 |

---

*마지막 업데이트: 2026년 2월*
