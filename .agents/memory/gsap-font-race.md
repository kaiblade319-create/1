---
name: GSAP ScrollTrigger font loading race
description: Why elements overlap on first page load and how to fix it permanently
---

# GSAP ScrollTrigger font loading race

**The rule:** Always refresh ScrollTrigger after `document.fonts.ready` resolves, and pre-hide GSAP-animated elements in CSS.

**Why:** On first load, web fonts aren't cached. The browser renders HTML with system fallback fonts (shorter/differently sized), GSAP ScrollTrigger calculates all start/end trigger positions, *then* the real fonts load and cause a reflow — shifting every element's height. Now all ScrollTrigger trigger points are wrong, pinned sections overlap content below them, and elements "eat each other." After reload, fonts are cached → layout is stable before ScrollTrigger runs → no problem. That's why it takes 3-4 reloads to look normal.

**How to apply:**
1. `document.fonts.ready.then(() => ScrollTrigger.refresh())` — inside both `load` listeners that set up ScrollTrigger
2. `setTimeout(() => ScrollTrigger.refresh(), 600)` — catches image/late-load shifts
3. Debounced `resize` → `ScrollTrigger.refresh(250ms)`
4. CSS `opacity: 0` on elements GSAP animates via `fromTo` — prevents visible flash before GSAP applies its initial `from` state. Add `opacity: 1` to the corresponding `gsap.set()` or `gsap.to()` target state.
5. Guard `arcRing.offsetWidth / 2 > 0` before using it — if layout hasn't settled, all cards stack at 0,0.
