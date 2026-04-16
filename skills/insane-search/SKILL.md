---
name: insane-search
description: >
  WebFetch가 차단되거나 실패할 때, 또는 X/Twitter, Reddit, YouTube, GitHub,
  Mastodon, Medium, Substack, Stack Overflow, Threads, 네이버 등
  플랫폼에 접근할 때 사용하는 우회 접근 전략. 1,858개 미디어 사이트(yt-dlp),
  범용 웹(Jina Reader), 공개 API(HN, Bluesky, arXiv 등)를 활용한다.
  트위터/X 못 열어, 레딧 안 읽혀, 유튜브 자막 뽑아줘, 깃헙 검색, 사이트 차단됨,
  스레드 안 열려, 마스토돈, 미디엄, 서브스택, 스택오버플로우, 네이버 블로그,
  디시인사이드, 에펨코리아, 요즘IT, 긱뉴스, 클리앙, 쿠팡, 링크드인, 당근마켓,
  twitter access, reddit blocked, youtube subtitles, github search, arxiv papers,
  threads, mastodon, medium, substack, stackoverflow, naver blog, dcinside, fmkorea,
  coupang, linkedin, yozm, wishket.
  Make sure to use this skill whenever WebFetch returns 402/403/blocked,
  when accessing social media or developer platforms,
  or when extracting media content (video, audio, subtitles).
  Do NOT trigger for simple web searches that WebSearch can handle directly.
---

# Insane Search

> URL 접근이 차단될 때, 플랫폼별 최적 방법을 자동으로 안내한다.

## 사이트별 접근 인덱스

아래 테이블에서 접근하려는 사이트를 찾고, 해당 방법의 reference를 참조한다.

### 소셜 미디어

| 사이트 | 도메인 | 방법 | 상세 |
|--------|--------|------|------|
| X/Twitter | x.com, twitter.com | Syndication API, oEmbed | [twitter.md](references/twitter.md) |
| Reddit | reddit.com | JSON API (.json + Mobile UA) | [json-api.md](references/json-api.md) |
| Threads | threads.com | Jina Reader | [jina.md](references/jina.md) |
| Bluesky | bsky.app | AT Protocol 공개 API | [public-api.md](references/public-api.md) |
| Mastodon | mastodon.social 등 | 공개 API (인스턴스별) | [public-api.md](references/public-api.md) |

### 미디어/영상

| 사이트 | 도메인 | 방법 | 상세 |
|--------|--------|------|------|
| YouTube | youtube.com, youtu.be | yt-dlp (자막/메타/검색/댓글) | [media.md](references/media.md) |
| Vimeo | vimeo.com | yt-dlp | [media.md](references/media.md) |
| Twitch | twitch.tv | yt-dlp (VOD/클립) | [media.md](references/media.md) |
| TikTok | tiktok.com | yt-dlp | [media.md](references/media.md) |
| SoundCloud | soundcloud.com | yt-dlp (검색 가능: scsearch) | [media.md](references/media.md) |
| 1,858개 미디어 사이트 | 다양 | yt-dlp --dump-json | [media.md](references/media.md) |

### 개발/기술

| 사이트 | 도메인 | 방법 | 상세 |
|--------|--------|------|------|
| GitHub | github.com | gh CLI / REST API | [public-api.md](references/public-api.md) |
| V2EX | v2ex.com | JSON API | [json-api.md](references/json-api.md) |
| Stack Overflow | stackoverflow.com | SE API v2.3 (WebFetch 도메인 차단) | [public-api.md](references/public-api.md) |
| Hacker News | news.ycombinator.com | Firebase JSON API | [json-api.md](references/json-api.md) |
| Lobste.rs | lobste.rs | JSON API | [json-api.md](references/json-api.md) |
| dev.to | dev.to | 공개 API | [json-api.md](references/json-api.md) |
| npm | npmjs.com | Registry API | [json-api.md](references/json-api.md) |
| PyPI | pypi.org | JSON API | [json-api.md](references/json-api.md) |

### 학술/지식

| 사이트 | 도메인 | 방법 | 상세 |
|--------|--------|------|------|
| arXiv | arxiv.org | Atom API | [public-api.md](references/public-api.md) |
| CrossRef | doi.org | REST API | [public-api.md](references/public-api.md) |
| Wikipedia | wikipedia.org | REST API | [json-api.md](references/json-api.md) |
| OpenLibrary | openlibrary.org | JSON API | [public-api.md](references/public-api.md) |
| Wayback Machine | web.archive.org | CDX API | [public-api.md](references/public-api.md) |

### 한국 플랫폼

| 사이트 | 도메인 | 방법 | 상세 |
|--------|--------|------|------|
| 네이버 블로그 | blog.naver.com | 모바일 URL + iPhone UA | [naver.md](references/naver.md) |
| 네이버 뉴스 | news.naver.com | Jina Reader | [naver.md](references/naver.md) |
| 네이버 증권 | finance.naver.com | Jina Reader | [naver.md](references/naver.md) |
| 네이버 TV | tv.naver.com | yt-dlp | [naver.md](references/naver.md) |
| 네이버 금융 시세 | api.finance.naver.com | 비공식 JSON API (무인증) | [naver.md](references/naver.md) |
| 쿠팡 | coupang.com | curl_cffi safari + JSON-LD | [tls-impersonate.md](references/tls-impersonate.md) |
| 요즘IT | yozm.wishket.com | curl Chrome UA (Jina 차단됨) | [fallback.md](references/fallback.md) |
| 당근마켓 | daangn.com | Jina Reader (불안정) | [jina.md](references/jina.md) |
| 클리앙 | clien.net | Jina Reader | [jina.md](references/jina.md) |
| 루리웹 | ruliweb.com | Jina Reader | [jina.md](references/jina.md) |
| 뽐뿌 | ppomppu.co.kr | Jina Reader + RSS | [jina.md](references/jina.md) |
| 긱뉴스 | news.hada.io | Jina Reader | [jina.md](references/jina.md) |
| 벨로그 | velog.io | RSS (`v2.velog.io/rss/@{user}`) | [json-api.md](references/json-api.md) |
| 브런치 | brunch.co.kr | Jina Reader + RSS | [jina.md](references/jina.md) |
| 한국경제 | hankyung.com | Jina Reader + RSS | [jina.md](references/jina.md) |
| 44bits | 44bits.io | Jina Reader | [jina.md](references/jina.md) |
| 커리어리 | careerly.co.kr | Jina Reader | [jina.md](references/jina.md) |
| 디시인사이드 | dcinside.com | curl 모바일 UA (Jina 빈 본문) | [fallback.md](references/fallback.md) |
| 에펨코리아 | fmkorea.com | curl_cffi (Jina 430 차단) | [tls-impersonate.md](references/tls-impersonate.md) |
| 티스토리 | *.tistory.com | WebFetch / RSS | [json-api.md](references/json-api.md) |
| SBS/JTBC/Kakao | sbs.co.kr, jtbc.co.kr | yt-dlp | [media.md](references/media.md) |
| Chzzk/Soop | chzzk.naver.com, sooplive.co.kr | yt-dlp | [media.md](references/media.md) |

### 뉴스/미디어

| 사이트 | 도메인 | 방법 | 상세 |
|--------|--------|------|------|
| Medium | medium.com | Jina Reader | [jina.md](references/jina.md) |
| Substack | *.substack.com | Jina Reader + RSS | [jina.md](references/jina.md) |
| 다음 뉴스 | news.daum.net | Jina Reader | [jina.md](references/jina.md) |
| Google News | news.google.com | RSS 피드 (무인증) | [rss.md](references/rss.md) |
| 조선/중앙/동아/경향/SBS/MBC | 각 언론사 | RSS 직접 구독 (무인증) | [rss.md](references/rss.md) |
| LinkedIn | linkedin.com | JSON-LD Person/Org + Jobs Guest API | [metadata.md](references/metadata.md) |

### RSS 피드

| 사이트 | 도메인 | 방법 | 상세 |
|--------|--------|------|------|
| RSS/Atom 범용 | 다양 | feedparser (자동설치) | [rss.md](references/rss.md) |
| SearXNG | searx.space | 무인증 메타검색 JSON | [rss.md](references/rss.md) |

## 접근 순서 — 적응형 스케줄러

```
Phase 0: 특수 엔드포인트 — 인덱스에 있으면 먼저 시도 (정확성 최고)
  ↓ 실패 또는 인덱스에 없음
Phase 1: 경량 프로브 (병렬) — WebFetch + Jina + curl UA/URL 변형
  ↓ 403/WAF/챌린지/빈 SPA 감지
Phase 2: TLS 임퍼소네이션 — curl_cffi (자동설치, safari→chrome→firefox)
  ↓ TLS 우회도 실패 또는 JS 챌린지
Phase 3: Playwright MCP — 실제 브라우저 (최후 수단)
  ↓ login/paywall 감지
종료: "인증 필요" 알림

사이드카: 캐시/아카이브 (Phase 1과 동시, 원본 성공 시 참고만)
```

**원칙**: 어떤 방법도 미리 제외하지 않는다. 의존성이 없으면 설치하고 시도한다.

상세: [fallback.md](references/fallback.md)

## 빠른 참조 — 범용 명령어

```bash
# 범용 웹 (Jina Reader)
curl -s "https://r.jina.ai/{URL}"

# 미디어 메타데이터 (yt-dlp — 1,858 사이트)
yt-dlp --dump-json "URL"

# Reddit
curl -sL -H "User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X) AppleWebKit/605.1.15" "https://www.reddit.com/r/{sub}/hot.json?limit=10"

# X/Twitter 타임라인
curl -sL "https://syndication.twitter.com/srv/timeline-profile/screen-name/{handle}"

# Hacker News
curl -sL "https://hacker-news.firebaseio.com/v0/topstories.json?limitToFirst=10&orderBy=%22%24key%22"

# YouTube 자막
yt-dlp --write-sub --write-auto-sub --sub-lang "en,ko" --skip-download -o "/tmp/%(id)s" "URL"
```

## 응답 검증

curl로 받은 응답의 성공/실패 판정 기준은 [fallback.md](references/fallback.md)의 "응답 검증" 참조.
모든 HTML 응답에서 메타데이터(OGP/JSON-LD)도 같이 추출 — [metadata.md](references/metadata.md) 참조.
