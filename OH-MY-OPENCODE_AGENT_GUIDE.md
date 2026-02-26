# Oh-My-OpenCode 에이전트 가이드

> 각 에이전트별 모델 구성, 인증 방법, 구독 필요 여부 및 추천 사항을 정리한 문서.
> 기준: oh-my-opencode / opencode 1.2.12 (2026년 2월)

---

## 목차

- [에이전트별 모델 구성](#에이전트별-모델-구성)
- [태스크 카테고리별 모델](#태스크-카테고리별-모델)
- [인증 방법 가이드](#인증-방법-가이드)
- [구독 필요 여부 및 추천](#구독-필요-여부-및-추천)
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

## 참고: 인증 파일 위치

| 파일 | 내용 |
|------|------|
| `~/.local/share/opencode/auth.json` | opencode 인증 토큰 (Anthropic · Google · OpenCode Zen) |
| `~/.config/opencode/oh-my-opencode.json` | 에이전트별 모델 설정 |
| `~/.config/opencode/opencode.json` | opencode 전체 설정 (provider · plugin) |
| `%APPDATA%\opencode\antigravity-accounts.json` | antigravity Google 계정 정보 |

---

*마지막 업데이트: 2026년 2월*
