# AGENTS.md — nuclear-mini-player (for AI assistants; humans see README.md)

One maintainer plus friends helping out — keep every change small,
reviewable, and justified. No drive-by refactors.


Honesty is non-negotiable: say what you actually did versus what you
were asked to do. Never claim a test ran, a build passed, or hardware
was verified unless it truly happened in this session. Report failures
and shortcuts plainly, mark uncertainty as uncertainty, and never hide
a skipped step behind vague wording.

People over AI, always: this project is human-built and human-reviewed.
AI assists, humans decide. Write code, messages, and docs a person can
follow with no assistant involved — clear names, plain language, no
cleverness. The goal is users who understand their system, not users
dependent on one.
Nuclear music-player plugin. Single-file DOM-injected miniplayer. No
build, no deps, no tests. v1.8.x architecture (state object, TEXT i18n,
detached popup via window.open + BroadcastChannel, queue+playlists).

## Layout

- `index.js` (entire plugin), `package.json` (manifest + nuclear metadata)
- `.github/workflows/release.yml` — only CI
- Keep files at repo root. Registry expects flat layout.

## Lifecycle

- `module.exports = { onLoad, onEnable, onDisable }` — Nuclear host API.
- `onLoad`: only `api.Settings.register([...])` (9 settings).
- `onEnable`: settings, styles, player/queue/toast build, subscriptions,
  media session, theme watcher, rAF loop, initial state.
- `onDisable`: run every `state.cleanup` fn, abort lyrics fetch, cancel
  rAF, close detached window, remove all DOM nodes, clear caches.

## Host-API notes

- Lyrics come only from host providers via `api.Lyrics` (guarded —
  older hosts lack it). Panel scrapers were removed; do not re-add.
- Detached popup needs user gesture (button click) or it gets blocked.
- `api.Http.fetch` rejects AbortSignal (Tauri) — strip it in fetchJson.
- Never assume `api.Queue.goToPrevious/goToNext`, `api.Playlists`, or
  `api.Settings.subscribe` exist: feature-check + audio-element fallback.

## Verify

- `node --check index.js`, `python3 -m json.tool package.json`
- TEXT key coverage: every `text("x")` must exist in all 5 languages
- Cannot run under plain node (needs document/window). Real check is
  loading `plugin.zip` in Nuclear.

## Release

- `release.yml` triggers on tag `v*`, asserts
  `package.json` version == tag, zips `index.js` + `package.json`.
- Bump version, commit, tag `v<version>`, push tag.

Before finishing: pass docs/REVIEW.md.
