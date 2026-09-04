# CLAUDE.md

## Project

**Semina 2026** — landing site for BWAVE's "2026 마음결 세미나" (a mental-health seminar, 2026-09-17). Static, no framework, no build step. Two pages:

- `index.html` — main landing page with cinematic pinned-scroll UX.
- `archive.html` — past-seminar recap page (simple reveal-on-scroll).

Deploy is plain static hosting; open the HTML files directly or serve the folder (`python3 -m http.server`). There is no package manager, bundler, or test suite.

## Stack (all via CDN, no local deps)

- **Tailwind CSS** (`cdn.tailwindcss.com`) with an inline `tailwind.config` in each file. Custom palette: `brand` (`#0077FA`/cyan `#34C1FD`), `ink`, `paper`; custom shadows `soft`/`lift`/`glow`.
- **Pretendard** font (jsDelivr).
- **Iconify** (`iconify-icon` web component) for icons.
- Language is Korean (`lang="ko"`). Body uses `.kr { word-break: keep-all; }` for Korean line breaking.

## index.html architecture

The page is one long document divided into **pinned "pin" sections** that play like scroll-driven scenes on desktop, and fall back to normal stacked layout on mobile / touch / reduced-motion.

- Desktop detection: `matchMedia('(min-width:1024px) and (pointer:fine)')` adds `.desk` to `<html>` early (before paint) to avoid content flash.
- Three scroll controllers (`introCtl`, `progCtl`, `actCtl`), each an IIFE with `enable()`/`disable()`/`render()`. `applyMode()` toggles them based on the desktop media query; `onScroll()` rAF-throttles all three `render()` calls.
- A pinned section gets `.on` + an explicit `style.height` in `dvh`; its `.pin-stage` becomes `position: sticky`. Scroll progress inside that height drives the scene. When off (mobile), `.pin-stage { display: contents }` and it lays out normally.
- **`PACE` object (~line 334)** controls scroll length (in `dvh`) of each pinned frame: `P1` intro, `program` array (P2–P6), `act` array (P7 program / P8 product / P9 apply). Tuning these changes how long each scene stays pinned. Keep them modest — they were deliberately shortened (see git history).
- Section ids used as anchors: `#top`, `#change`, `#program`, `#product`, `#apply`. Nav and footer links point to these; `archive.html` links back to `index.html#<id>`.
- Apply CTAs (`a[href="#apply"]`) are intercepted and smooth-scroll to the P9 apply frame; the Google Form link lives only on the apply-screen button.
- `history.scrollRestoration = 'manual'` + `scrollTo(0,0)` on load — refresh always starts at top so pinned frames don't stick (past bug; don't remove).
- Other UI: top scroll-progress bar, auto-hiding nav on scroll-down, IntersectionObserver `.reveal`/`.in` fade-ins, mouse-scroll hint that hides after 60px.

## Editing conventions

- Match the existing inline style: Tailwind utility classes in markup, one-off effects in the `<style>` block, all JS in the single trailing `<script>`. No external JS/CSS files.
- Images live in `assets/` (Korean filenames). Keep them there.
- When touching scroll behavior, test both desktop (pinned) and a narrow / touch viewport (fallback) — the two code paths diverge via `.desk` and `.on`.
- `.claude/` and `.context/` are gitignored working dirs, not shipped.
