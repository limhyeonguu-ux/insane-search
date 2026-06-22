[English](README.md) | 한국어

# insane-search

> **포기는 배추 셀 때나 하는 말이다.**

`403`. WAF. CAPTCHA. 텅 빈 SPA. 로그인 벽. 남들이 다 두 손 들고 돌아설 때, insane-search는 그제야 소매를 걷습니다. 정문이 막혔으면 쪽문을, 쪽문도 막혔으면 진짜 브라우저를 끌고 와 사이트가 숨겨둔 API까지 캐냅니다. "여긴 막혔어요" 소리 듣던 사이트도 결국엔 길이 납니다.

API 키도, 가입도, 설정도 없습니다. 깔아만 두면 Claude Code가 더는 "이건 못 읽겠는데요" 하고 발을 빼지 않습니다.

[빠른 시작](#빠른-시작) • [동작 방식](#동작-방식) • [인덱스](#인덱스에-있는-것) • [레퍼런스](#레퍼런스) • [요구사항](#요구사항)

---

## 빠른 시작

### 1. 마켓플레이스 등록

```
/plugin marketplace add https://github.com/fivetaku/gptaku_plugins.git
```

### 2. 플러그인 설치

```
/plugin install insane-search
```

### 3. Claude Code 재시작

이게 전부입니다. 설정도, API 키도, 환경변수도 없습니다.

### 4. 그냥 평소처럼 시키세요

차단된 사이트는 알아서 뚫고 들어갑니다.

```
"r/LocalLLaMA에서 요즘 뜨는 거 뭐야?"
"openclaw가 X에 최근에 뭐라고 올렸어?"
"이 유튜브 영상 요약해줘"
"쿠팡에서 10만 원 밑으로 키보드 좀 찾아줘"
"이 네이버 블로그 글 읽어줘"
```

---

## 왜 insane-search?

- **'차단'이라는 말을 모릅니다** — "이 사이트는 접근 불가" 같은 지레짐작이 없습니다. 모든 사이트가 체인을 끝까지 탑니다. 쿠팡? 뚫립니다. LinkedIn? JSON-LD째로 발라냅니다. 요즘IT? Chrome인 척 한 번에 끝.
- **무기는 알아서 챙깁니다** — TLS 지문 위장용 `curl_cffi`가 없다? 깔아서 씁니다. `feedparser`가 없다? 깔아서 씁니다. `yt-dlp`가 없다? 역시 깔아서 씁니다. 당신이 눈치챌 틈도 없이.
- **비장의 한 수가 아니라 다섯 수** — WebFetch → Jina → curl UA/URL 변형 → TLS 위장(safari/chrome/firefox) → 진짜 브라우저. 앞 단계가 벽에 부딪혀야만 다음 수로 넘어갑니다.
- **숨은 API까지 캐냅니다** — Phase 3은 페이지만 그려보고 마는 게 아닙니다. 브라우저의 네트워크를 엿보다가 사이트가 속으로 쓰는 진짜 JSON API를 낚아채서, 그대로 다시 쓰게 돌려줍니다.
- **세팅 부담 제로** — API 키도, OAuth도, 개발자 포털도 없습니다. 전부 공개 엔드포인트와 알아서 깔리는 라이브러리로 굴러갑니다.

---

## 동작 방식

Claude Code가 URL에 접근해야 할 때, insane-search는 4단계 적응형 스케줄러를 돌립니다. 각 Phase는 앞 단계가 실패하거나 차단 신호를 잡았을 때만 깨어납니다.

```
Phase 0: 특수 엔드포인트 인덱스
  ↓ 인덱스에 없거나 실패
Phase 1: 경량 프로브 (병렬)
  • WebFetch + Jina Reader
  • curl Chrome / 모바일 / Googlebot UA
  • URL 변형: m.{domain}, .json, /rss, /feed
  • 사이드카: AMP 캐시, archive.today, Wayback (low-trust)
  ↓ 403/429/WAF 헤더/챌린지 본문 감지
Phase 2: TLS 위장
  • curl_cffi safari → chrome → firefox
  • 없으면 알아서 설치: pip install curl_cffi
  ↓ TLS 위장 실패 또는 JS 챌린지 감지
Phase 3: 진짜 브라우저
  • Playwright MCP (browser_navigate → snapshot → evaluate)
  • 숨은 API까지 덤으로 발견 (network_requests)
  ↓ login/paywall 감지
종료: "인증 필요" — 여기서부턴 더 가봐야 헛수고
```

**핵심 원칙**: 어떤 방법도 미리 빼지 않는다. 의존성이 없다고 건너뛰지 말고, 깔아서 시도한다. "이 사이트는 어렵겠지" 하고 지레 접지 마라 — 사이트도 변하고, 어제 안 되던 게 오늘은 먹힐 수 있다.

HTML 응답마다 OGP 태그와 JSON-LD 구조화 데이터도 같이 훑습니다. 본문을 통째로 못 건져도 제목, 요약, 가격, 프로필은 손에 쥐어집니다.

---

## 인덱스에 있는 것

범용 체인이 알아서 못 찾아내는 특수 엔드포인트만 인덱스에 박아둡니다. 나머지 — 네이버 블로그, 쿠팡, LinkedIn, Medium, 한국 뉴스, Substack, 웬만한 커뮤니티 — 는 인덱스에 없어도 적응형 스케줄러가 알아서 처리합니다.

### 플랫폼 전용 API

| 플랫폼 | 방법 | 레퍼런스 |
|--------|------|----------|
| X/Twitter | `syndication.twitter.com/srv/timeline-profile/...` + oEmbed | `twitter.md` |
| Reddit | URL + `.json` + Mobile UA | `json-api.md` |
| Bluesky | AT Protocol (`public.api.bsky.app/xrpc/...`) | `public-api.md` |
| Mastodon | 인스턴스별 공개 API | `public-api.md` |
| Hacker News | Firebase API | `json-api.md` |
| Stack Overflow | SE API v2.3 | `public-api.md` |
| Lobste.rs / V2EX / dev.to | 공개 JSON API | `json-api.md` |

### 미디어 (CLI 도구 필수)

| 플랫폼 | 방법 | 레퍼런스 |
|--------|------|----------|
| YouTube / Vimeo / Twitch / TikTok / SoundCloud + 1,853개 | `yt-dlp --dump-json` | `media.md` |

### 학술 & 레지스트리

| 플랫폼 | 방법 | 레퍼런스 |
|--------|------|----------|
| arXiv | Atom API | `public-api.md` |
| CrossRef | REST API | `public-api.md` |
| Wikipedia | REST API | `json-api.md` |
| OpenLibrary | JSON API | `public-api.md` |
| GitHub | `gh` CLI / REST API | `public-api.md` |
| npm / PyPI | Registry API | `json-api.md` |
| Wayback Machine | CDX API | `public-api.md` |

### 한국 전용

| 플랫폼 | 방법 | 레퍼런스 |
|--------|------|----------|
| 네이버 금융 시세 | `api.finance.naver.com/siseJson.naver` (비공식, 무인증) | `naver.md` |

**그 외 전부 Phase 1~3이 알아서 처리** — 쿠팡 (curl_cffi safari), LinkedIn (JSON-LD 추출), Medium (Jina), 웬만한 한국 커뮤니티 (Jina 또는 curl), `/rss`·`/feed`가 열려 있는 모든 사이트.

---

## 레퍼런스

스킬은 기법별 레퍼런스 파일로 나뉩니다.

| 파일 | 내용 |
|------|------|
| `fallback.md` | Phase 0→3 적응형 스케줄러, 에스컬레이션 신호, 응답 검증 |
| `jina.md` | Jina Reader (`r.jina.ai`, 무인증) |
| `json-api.md` | 공개 JSON API (Reddit, HN, dev.to, Wikipedia, npm, PyPI 등) |
| `public-api.md` | Bluesky, Mastodon, Stack Exchange, arXiv, CrossRef, OpenLibrary, GitHub, Wayback |
| `media.md` | 1,858개 미디어 사이트용 yt-dlp |
| `twitter.md` | Twitter Syndication API + oEmbed |
| `naver.md` | 네이버 블로그 모바일 URL, 네이버 금융 JSON API |
| `rss.md` | 한국 언론 RSS 9개, Google News RSS, feedparser, SearXNG |
| `tls-impersonate.md` | curl_cffi 다중 타겟 (safari/chrome/firefox) + 자동 설치 |
| `playwright.md` | Playwright MCP 풀 도구 (snapshot, evaluate, network_requests) |
| `cache-archive.md` | Google AMP 캐시, archive.today, Wayback Machine |
| `metadata.md` | OGP, JSON-LD, Schema.org, Next.js RSC 페이로드 추출 |

---

## 요구사항

**필수:** Claude Code 하나면 됩니다.

**자동 설치** (첫 사용 때 스킬이 알아서 깝니다):

```bash
pip install curl_cffi    # WAF 막힌 사이트용 TLS 위장
pip install feedparser   # RSS/Atom 파싱
pip install yt-dlp       # 1,858개 미디어 사이트
```

**선택 — 깔면 커버리지가 넓어집니다:**

```bash
brew install gh                      # GitHub (REST API보다 빠름)
claude mcp add playwright -- npx @playwright/mcp@latest   # JS 렌더링 사이트
```

의존성이 없다고 방법을 건너뛰지 않습니다 — 깔아서, 시도합니다.

---

## insane-search는 이런 게 아닙니다

- **스크래퍼가 아닙니다** — 방법을 고르는 레이어일 뿐입니다. 공개 API와 표준 기법을 상황에 맞게 엮을 뿐.
- **API 키로 굴러가지 않습니다** — 전부 무인증 공개 엔드포인트, 아니면 URL 변형.
- **손으로 적어둔 정답지가 아닙니다** — 인덱스는 최소한(~15 그룹)이고, 나머지는 적응형 스케줄러가 직접 찾아냅니다.
- **편견을 학습하지 않습니다** — "접근 거부" 명단 같은 건 없습니다. 뚫릴 구석이 있는 사이트라면 체인이 기어이 길을 냅니다.

---

## 사용 예시

명령어 없습니다. 평소처럼 말만 하면 됩니다. URL이 막혔거나 특수 처리가 필요한 플랫폼이면 알아서 발동합니다.

```
"지금 해커뉴스 프론트페이지에 뭐 떴어?"
→ Firebase API로 탑 스토리 + 점수 + 댓글

"이번 주 arXiv AI 논문 좀 뽑아줘"
→ arXiv Atom API, 날짜 필터

"쿠팡에서 100만 원 이하 노트북 추려줘"
→ Phase 2: curl_cffi safari → JSON-LD ItemList

"이 Medium 글 요약해줘"
→ Phase 1: Jina Reader → 깔끔한 마크다운

"Claude Code 레딧 반응 좀 봐줘"
→ Reddit JSON API + 모바일 UA → 글 + 댓글
```

---

## 라이선스

MIT

---

<div align="center">

**웹에 있는 거라면, insane-search는 결국 들어갑니다.**

</div>
