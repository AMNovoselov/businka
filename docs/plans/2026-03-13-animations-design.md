# Animations Design — Businka Portfolio

**Date:** 2026-03-13
**Approach:** Vanilla CSS + Intersection Observer (no dependencies)
**Style:** Cinematic — slow, smooth, auteur-film feel

## Goals

- Hero feels alive via parallax mouse tracking on photo cards
- Page sections reveal cinematically as user scrolls
- No external libraries, no performance impact

## Animations

### 1. Hero — Mouse Parallax
- Three `.hero-photo-card` elements move on `mousemove` at different depths
- Depths: card 1 → 0.035, card 2 → 0.02, card 3 → 0.015
- CSS transition: `transform 0.8s cubic-bezier(0.25, 0.46, 0.45, 0.94)`

### 2. Hero — Entrance Sequence
Elements appear staggered on page load:
- `.hero-tag` — delay 0ms
- `h1` — delay 150ms
- `.hero-sub` — delay 300ms
- `.hero-cta` — delay 450ms
- `.hero-stats` — delay 600ms

All: `opacity: 0 → 1`, `translateY(20px) → 0`, duration `1.0s`

### 3. Scroll Reveal (Intersection Observer)
Classes added to elements in HTML, JS observer adds `.visible`:

| Class | Effect | Duration |
|---|---|---|
| `.fade-up` | opacity+translateY(30px) | 1.0s |
| `.fade-left` | opacity+translateX(-40px) | 1.0s |
| `.fade-right` | opacity+translateX(40px) | 1.0s |
| `.stagger-item` | fade-up with nth-child delay | 1.0s |

Easing for all: `cubic-bezier(0.22, 1, 0.36, 1)`
Observer threshold: `0.15` (triggers when 15% visible)

### 4. Applied to Sections
- Portfolio header → `.fade-up`
- Tab buttons → `.stagger-item` (stagger 80ms each)
- Masonry items → `.stagger-item` (stagger 60ms each, observer per tab)
- About photo → `.fade-left`
- About text block → `.fade-right`
- Contact section → `.fade-up`

## Implementation

Single `index.html` file changes:
1. Add CSS animation rules to `<style>`
2. Add `.fade-up` / `.fade-left` / `.fade-right` / `.stagger-item` classes to HTML elements
3. Add JS: Intersection Observer + mousemove parallax handler

No new files. No external dependencies.
