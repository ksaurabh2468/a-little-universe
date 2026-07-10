# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

A single-page interactive birthday website — a "Birthday Universe" gift site built for Momo
(Abhilasha), created as a surprise. Dark, cinematic, playful (not overly romantic) tone.
One self-contained file, no build step, no framework.

## Repository boundary — important

This is a **completely separate project from `D:\infinity_project\lma_infinity`** (the LMA
trading platform). They share no code, no config, no git history.

**Never create, edit, or copy files into `lma_infinity` for this project's sake** — not even
temporarily for local testing. If a Claude Code session working on this project has its working
directory set to `lma_infinity`, do local testing entirely within *this* repo (or via a
standalone background server + browser tooling) instead of writing anything into `lma_infinity`.

## Structure

```
index.html        — the entire site: HTML + CSS + JS in one file
photos/            — polaroid-wall images
.claude/launch.json — local dev server config (see below)
.vercel/            — Vercel project link (do not edit by hand)
```

Everything — markup, styles, and script — lives in `index.html`. There is no bundler, no
package.json, no dependencies. Edit the file directly.

### Content lives in one place

Near the top of the `<script>` block is a `CONFIG` object — name, birthday/unlock timestamp,
access passcode, Wrapped slides, compliments, cake/scratch/easter-egg reveal text, mixtape
tracklist, polaroid captions, and the finale message. Personalize the experience by editing
values there, not by hunting through the DOM-building code below it.

## Access model

- The countdown/hourglass teaser screen is **always public** — it reveals nothing and needs no
  passcode. Anyone with the link just sees a "something's brewing" countdown.
- Tapping the gift box opens a passcode modal (`CONFIG.accessCode`, currently `MOMO11`). Correct
  code unlocks the full experience and is remembered via `localStorage` on that browser/device.
- The countdown reaching zero only changes the box's *visual* state (glow vs. dim) — it does
  **not** bypass the passcode. Access is controlled entirely by the code, independent of time.
- This is a **client-side soft lock only** — not real security. Anyone who views page source, or
  browses the public GitHub repo directly, can read the passcode and all content in plain text.
  Good enough to stop a casual visitor from spoiling the surprise; not good enough against anyone
  who goes looking on purpose. (Discussed with the user — accepted as-is for this use case.)

## Deployment

Two live mirrors, both served from the same `index.html`:

| Target | URL | Update mechanism |
|---|---|---|
| Vercel | https://a-little-universe-smoky.vercel.app | `vercel deploy --prod --yes` (run from this repo) |
| GitHub Pages | https://ksaurabh2468.github.io/a-little-universe/ | Automatic on `git push origin main` (repo is public; Pages builds off `main` root) |

GitHub Pages/CDN caches responses (`Cache-Control: max-age=600`) and takes ~30–40s to rebuild
after a push — don't mistake a stale cache hit for a failed deploy. Vercel invalidates instantly
on each new deployment alias.

After any content or behavior change: commit + push (updates GitHub Pages) **and** run
`vercel deploy --prod --yes` (updates Vercel) — the two are independent, neither triggers the
other.

## Local dev server

`.claude/launch.json` defines a `birthday-universe` server (`python -m http.server 5588`). This
only appears in a session's Server list when that session's working directory is this repo. If a
session is rooted elsewhere (e.g. `lma_infinity`), start a plain background server instead:

```
cd D:\projects\birthday-universe && python -m http.server 5588
```

## Verifying changes before shipping

This page runs continuous animation loops (starfield background, confetti, hourglass canvas) via
`requestAnimationFrame` that never idle. Screenshot tools that wait for a render-idle state will
hang/time out on this page — if that happens, neutralize animation first:

```js
window.requestAnimationFrame = () => 0;   // then screenshot; reload after to restore
```

**Always get an actual visual screenshot before considering a change verified** — DOM/accessibility
snapshots and computed-style checks confirm *logic* (a class got added, an opacity changed) but
will not catch pure rendering bugs. (`clip-path` + `border` on the same element is a known trap —
clip-path silently clips away borders; prefer `<canvas>` for anything geometrically fiddly, matching
the pattern already used for the hourglass, cake, and wheel-of-plans.)

Test locally (screenshot + click-through) before pushing/deploying — production is a byte-for-byte
copy of this file, so anything broken here is equally broken there.
