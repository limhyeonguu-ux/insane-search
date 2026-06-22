# Changelog

## 0.6.0 — 2026-06-22

Engine overhaul — multi-AI reviewed (GPT-5.5 Pro + council) and effect-tested before shipping.

- **Diversity scheduler** (`fetch_chain.py`): the grid now materializes a plan and varies TLS family × URL transform first, so a small attempt budget touches every family/transform instead of burning out on one. Measured: family×transform class coverage 3/10 → 10/10 at the same cap. `max_attempts=None` is now exhaustive (honours R6); `tls_impersonate_avoid` targets are deprioritized, not deleted; jitter only on a failed attempt; new `grid_exhausted` / `stop_reason` diagnostics.
- **Validator v2** (`validators.py`): adds non-terminal `SUSPECT_OK`, JSON-aware validation (small API responses no longer mislabelled `CHALLENGE`), HARD vs SOFT markers (a `captcha` word can't override a matched selector), byte-accurate size, and 429/401/404/5xx status semantics. Measured: judgment errors 5/11 → 0/11 (incl. 2 false-successes removed).
- **Per-host SessionPool + cookie bridge** (`transport.py`, `executor.py`): cookies and connections persist across attempts/pages; a browser that clears a JS challenge hands its cookies + UA to curl_cffi (FlareSolverr pattern). Proven: an injected clearance cookie converts a 403/challenge into a 200. Adds `fetch_many()` and root warmup.
- **Playwright fallback hardening**: per-host profile isolation, `process.exit` → drained natural exit (no truncated HTML), single shared navigation deadline, JSON envelope (status / final URL / cookies / UA).
- **Patchright support (additive)**: if `patchright` is installed it is used as a drop-in (Runtime.enable-free) Playwright per its official best-practice (`channel='chrome'`, `no_viewport`, no stealth/headers); otherwise behaviour is unchanged. Measured on rebrowser-bot-detector: `runtimeEnableLeak` passes, `navigator.webdriver` hidden.
- **SSRF / redirect guard** (`safety.py`): blocks non-http(s) schemes and requests/redirects to private/loopback/link-local/metadata IPs (with DNS-rebinding check); every redirect hop is validated. `INSANE_ALLOW_PRIVATE=1` opts in for local use.
- **Requires curl_cffi ≥ 0.15.0**: `impersonate="chrome"` now resolves to Chrome 146 (was the stale Chrome 142), plus HTTP/3 fingerprints and an SSRF-safe redirect default. Setup and the runtime guard upgrade an existing older curl_cffi.
- Adds deterministic regression tests (`test_u1.py`, `test_u4.py`, `test_u7.py`).

## 0.5.2 — 2026-06-21

- The GitHub-star prompt is shown in the user's current language; on a fresh session with no language signal yet, it falls back to the language detected from your recent Claude sessions (else English).
- GitHub star is now **opt-in** — on first run the command asks once via AskUserQuestion (`네, ⭐ 눌러주기` / `아니요`) instead of auto-starring. The star logic moved into `setup.sh` and records the choice (`~/.gptaku-setup/<plugin>.star.json`) so it never re-asks. `setup.sh` no longer stars anything automatically.

This project follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.4.1] — 2026-05-04

### Changed
- SKILL.md R7 (WAF 조기 감지 시 API-first 병행 분기) — 분기 결정은 자동이지만 사용자가 결과 metadata에서 확인 가능. 어떤 우회 경로로 성공/실패했는지 명시

### Preserved (R1-R7 모두 보존)
- R1: WebFetch / 즉흥 curl 금지
- R2: 첫 200에서 탈출 금지 (4-계층 검증)
- **R3: No-Site-Name Rule** (bias_check.py CI 게이트) — fossil-방지 메타-패턴
- R4: 사이트 고유 정보는 CLI/user_hint로만
- R5: Phase 0 공식 API 우선
- R6: 격자 모두 돌린 뒤 "뚫을 수 없음" 결론
- R7: 병행 분기

→ insane-search는 4 진단 대상 중 fossil 의문이 가장 적게 검증된 케이스. R3 + bias_check.py는 다른 fossil-위험 플러그인에 차용 가능한 메타-패턴.

## [0.4.0] — 2026-04-22

### Added
- **`engine/` Python package** — single public entrypoint (`python3 -m engine URL` or `from engine import fetch`) that runs an exhaustive curl_cffi grid over WAF product profiles.
  - `fetch_chain.py` — grid scheduler with internal phases (probe → validate → detect → plan → execute → report), per-attempt jitter, and `FetchResult.trace[]` for diagnostics.
  - `validators.py` — 4-layer challenge classifier (`STRONG_OK / WEAK_OK / CHALLENGE / BLOCKED / UNKNOWN`) replacing naive HTTP 200 heuristics.
  - `waf_detector.py` — ranked `[(profile_id, confidence)]` detection with sticky `last_load_error()` for loader diagnostics.
  - `waf_profiles.yaml` — seven product profiles (`akamai_bot_manager`, `cloudflare_turnstile`, `f5_big_ip`, `aws_waf`, `datadome_probable`, `perimeterx_human`, `unknown_challenge`) with 25+ curl_cffi impersonate candidates and an empirically-derived `tls_impersonate_avoid` list.
  - `url_transforms.py` — generic URL mutations (`original`, `mobile_subdomain`, `am_prefix`, `drop_www`), no site-specific branches.
  - `executor.py` — capability-matched Playwright router, honours each profile's `fallback_when_challenge` ordering.
  - `templates/playwright_real_chrome.js` — Local Node + `channel:'chrome'` + stealth + persistent context, with home warmup and reload-retry against Akamai-grade WAFs.
  - `templates/playwright_mobile_chrome.js` — `devices[...]` emulation while keeping real-Chrome TLS.
  - `bias_check.py` — CI linter enforcing the No-Site-Name Rule via brand denylist + URL/domain regex, with `node_modules`/build-artefact exclusion.
  - `tests/test_smoke.py` — unit + online smoke coverage for validators, profile loader, URL transforms, and network round-trips.
- **SKILL.md harness rules R1–R7** — explicit constraints that keep Claude from improvising around the engine:
  - R1 CLI-first on any blocked URL
  - R2 no early break on HTTP 200
  - R3 No-Site-Name enforcement
  - R4 runtime-only hints
  - R5 Phase 0 official APIs take precedence
  - R6 exhaustive grid before declaring failure
  - **R7 — API-first parallel branch** when a WAF is detected early and the user intent is list/collect: engine keeps running in background while Claude reconnoiters via Playwright MCP `browser_network_requests` to discover internal JSON endpoints, then re-fetches via engine.
- **Full `references/` index (12/12 files)** grouped by role (engine extension, lightweight alternatives, platform APIs, in-tree code) with "when to read" + "what it covers" per entry.
- **`references/playwright.md`** rewritten as Approach 1 (MCP Chromium — Cloudflare-grade) vs Approach 2 (Local Node `channel:'chrome'` + stealth — Akamai-grade), selection driven automatically by profile `capabilities_needed` tags.

### Changed
- `plugin.json` version bumped to `0.4.0` (new public surface + new behavior justify minor bump).
- Graceful degradation paths for missing `PyYAML`, `curl_cffi`, `bs4`, or Node — failures surface as `UNKNOWN` verdicts + trace entries, never silently swallow.
- Per-attempt jitter between curl grid calls, env-tunable via `INSANE_JITTER_MS_MIN` / `INSANE_JITTER_MS_MAX`.

### Notes
- No site-specific logic is introduced anywhere in `engine/**` or `waf_profiles.yaml`; all site knowledge enters at call time (`success_selectors`, `user_hint`) or stays in comments / docs.
- Earlier history kept in git log.

## Earlier releases

Pre-0.4.0 history is documented in git commits only; no structured changelog was maintained before this release.
