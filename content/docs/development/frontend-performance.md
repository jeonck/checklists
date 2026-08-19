---
title: "Frontend Performance"
description: "Verify a web page loads and responds fast enough on real devices and real networks before release."
icon: "speed"
weight: 460
toc: true
tags: ["frontend", "performance", "web-vitals", "browser"]
---

Frontend performance is decided by a small number of choices — how much JavaScript ships, when fonts and images load, and what blocks the main thread — and then eroded by a thousand small additions nobody measured. This checklist is organised around what the browser actually does with your page, and around the Core Web Vitals as the shared measurement, not as a scoring game. Measure on a mid-range Android device on a throttled connection, because that is what most of your users have.

{{< alert context="info" text="**Who runs this:** the frontend engineer shipping the change, with a reviewer looking at the measured numbers rather than the code. **When:** before a release that changes rendering, routing, third-party scripts, or the critical CSS or font path." />}}

## 1. Measurement and budgets

- [ ] **Field data from real users is collected, not just lab scores** — a synthetic run on a fast machine is a debugging tool, and the 75th percentile of real sessions is the number that matters.
- [ ] **Core Web Vitals are tracked per route, not site-wide** — a good average hides the one page that everyone lands on and nobody measured.
- [ ] **Lab testing uses mid-tier mobile CPU throttling and a throttled network** — a modern laptop on office wifi will pass tests that real users fail badly.
- [ ] **A performance budget exists with numeric thresholds** — total transferred JavaScript, request count, and target values for each vital, agreed with the product owner.
- [ ] **The budget is enforced in CI and a regression fails the build** — a budget that only produces a warning drifts upward with every release.
- [ ] **Interaction to Next Paint is measured against the real interactions users perform** — not just page load, since most perceived slowness happens after the page is up.
- [ ] **The measurement includes the first visit with an empty cache** — repeat-visit numbers flatter you and describe the wrong user.

## 2. Critical rendering path

- [ ] **The initial HTML response is fast and not blocked on slow server work** — every millisecond of time to first byte delays everything downstream, and no amount of frontend work recovers it.
- [ ] **Render-blocking CSS is limited to what the first viewport needs** — the browser cannot paint anything until it has parsed every blocking stylesheet.
- [ ] **Non-critical CSS is loaded without blocking render** — media-conditional loading or deferred injection, so a print stylesheet never delays the first paint.
- [ ] **Scripts in the document head are `defer` or `async` unless they genuinely must run first** — a synchronous script in the head stops HTML parsing while it downloads and executes.
- [ ] **Connections to critical third-party origins are pre-established** — `preconnect` for origins on the critical path removes a DNS lookup, TCP handshake, and TLS negotiation from the chain.
- [ ] **The Largest Contentful Paint element is identified and prioritised** — usually a hero image or heading; it should be in the initial HTML, discoverable by the preload scanner, and never lazy-loaded.
- [ ] **Compression and modern HTTP are enabled on all text assets** — Brotli or gzip on HTML, CSS, JavaScript, SVG, and JSON.

```html
<link rel="preconnect" href="https://cdn.example.com" crossorigin>
<link rel="preload" href="/fonts/inter-var.woff2" as="font" type="font/woff2" crossorigin>
<img src="/hero.avif" width="1200" height="630" fetchpriority="high" alt="">
```

## 3. JavaScript

- [ ] **The total JavaScript transferred on first load is measured against the budget** — parse, compile, and execution cost scale with bytes, and on a mid-range phone that cost is measured in seconds, not milliseconds.
- [ ] **Code is split by route and heavy components are loaded on demand** — shipping the settings page's dependencies to a visitor on the landing page is pure waste.
- [ ] **The bundle has been inspected with an analyser and every large dependency is justified** — a date library, an icon set, or a chart library imported in full is the usual culprit.
- [ ] **Tree shaking actually works for the way modules are imported** — a default import of an entire utility library defeats it silently.
- [ ] **Polyfills are served only to browsers that need them** — shipping legacy transpilation and polyfills to modern browsers inflates every bundle for everyone.
- [ ] **Long tasks are broken up so the main thread yields to input** — any task over 50ms delays the response to a tap, which is exactly what Interaction to Next Paint measures.
- [ ] **Expensive pure computation is moved off the main thread** — a web worker keeps parsing, sorting, or crypto from freezing the interface.
- [ ] **Event handlers do no layout-thrashing work** — reading a layout property after writing one in the same frame forces a synchronous reflow, and in a loop it is quadratic.

## 4. Images and media

- [ ] **Images are served in a modern format with fallbacks** — AVIF or WebP typically cut transfer size by half or more over JPEG at equivalent quality.
- [ ] **Responsive images serve appropriately sized variants** — `srcset` and `sizes`, so a phone does not download a 2000px-wide asset to display it at 375px.
- [ ] **Every image and video has explicit `width` and `height` or an aspect ratio** — without them the browser cannot reserve space, and the reflow on load is the most common cause of layout shift.
- [ ] **Below-the-fold images are lazy-loaded and above-the-fold images are not** — lazy-loading the LCP image delays its discovery and directly worsens the metric.
- [ ] **Images are compressed as part of the build, not by hand** — manual optimisation is skipped exactly when the release is rushed.
- [ ] **Video is not auto-played with a full preload on mobile** — use a poster image and `preload="none"` or `metadata` unless playback is the point of the page.
- [ ] **Icons are inline SVG or a sprite rather than an icon font** — icon fonts block on font loading, break with font-blocking failures, and are announced strangely by assistive technology.

## 5. Fonts

- [ ] **Web fonts are self-hosted or served from a preconnected origin** — a third-party font host adds a full connection setup to the critical path.
- [ ] **`font-display: swap` or `optional` is set** — the default blocks text rendering for up to three seconds, showing users a blank page rather than readable content.
- [ ] **Fonts are subset to the characters actually used** — a full multilingual variable font can be hundreds of kilobytes of glyphs nobody renders.
- [ ] **Only WOFF2 is served** — every browser that matters supports it, and older formats are dead weight.
- [ ] **The fallback font stack is metrically adjusted to the web font** — matching size and spacing with the font override descriptors prevents the reflow when the real font arrives.
- [ ] **The number of font families and weights is counted and minimised** — each additional weight is another request and another few tens of kilobytes.

## 6. Caching and delivery

- [ ] **Static assets are content-hashed and served with a long immutable cache lifetime** — a hashed filename can be cached for a year safely.
- [ ] **HTML is never cached long without revalidation** — otherwise a released fix cannot reach users who have the old document.
- [ ] **Assets are served from a CDN close to users, with compression applied at the edge** — round-trip latency dominates on mobile networks, and no amount of bundle tuning recovers a transatlantic hop per request.
- [ ] **A cache-busting deployment does not invalidate everything** — separating vendor and application bundles keeps unchanged dependencies cached across releases.
- [ ] **A service worker, if used, has a tested update path** — a stale service worker that never updates strands users on an old version, and it is a genuinely hard bug to diagnose remotely.
- [ ] **API responses on the critical path have deliberate cache headers** — a fast page waiting on an uncacheable request is still a slow page.

## 7. Runtime and interaction

- [ ] **Cumulative Layout Shift sources are hunted down on real page loads** — late-loading banners, injected consent dialogs, and asynchronously sized ads are the usual causes.
- [ ] **Content injected after load reserves its space in advance** — including any element inserted by a third-party script.
- [ ] **Animations are limited to `transform` and `opacity`** — animating layout-affecting properties forces reflow on every frame and drops frames on mid-range hardware.
- [ ] **Scroll and resize handlers are throttled or use passive listeners** — a non-passive scroll listener prevents the browser from scrolling on the compositor.
- [ ] **Long lists are virtualised** — rendering thousands of DOM nodes costs memory and makes every subsequent layout more expensive.
- [ ] **Interaction feedback appears within about 100ms even when the work takes longer** — perceived responsiveness is a separate problem from actual throughput.
- [ ] **Memory growth over a long session has been checked** — detached DOM nodes and uncleared listeners degrade single-page applications the longer they are open.

## 8. Third-party scripts

- [ ] **Every third-party script is inventoried with a named owner and a stated business reason** — the inventory is almost always longer than anyone expects.
- [ ] **The performance cost of each tag is measured individually** — block it and re-measure; several analytics and personalisation tags cost more than the whole application bundle.
- [ ] **Third-party scripts load asynchronously and after the critical path** — one synchronous tag can hold the entire page hostage to someone else's outage.
- [ ] **The page renders and functions when a third party is slow or unreachable** — test with the origin blocked, not just with it slow.
- [ ] **Tag manager containers have a review process** — an unreviewed container lets a marketing change ship arbitrary JavaScript straight to production.
- [ ] **Personalisation and experimentation scripts that hide content have a timeout** — an anti-flicker snippet with no fallback shows a blank page when its script fails.

{{< alert context="warning" text="**Common mistake:** optimising the bundle for weeks while a single synchronous third-party tag costs more than everything you saved. Measure the third parties first, and block them one at a time to attribute the cost." />}}

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Measurement and budgets | | | Pass / Pass with actions / Fail |
| Critical rendering path | | | Pass / Pass with actions / Fail |
| JavaScript | | | Pass / Pass with actions / Fail |
| Images and media | | | Pass / Pass with actions / Fail |
| Fonts | | | Pass / Pass with actions / Fail |
| Caching and delivery | | | Pass / Pass with actions / Fail |
| Runtime and interaction | | | Pass / Pass with actions / Fail |
| Third-party scripts | | | Pass / Pass with actions / Fail |

Record the measured value for each budgeted metric alongside the outcome, so the next reviewer can see the direction of travel rather than a bare pass.

## Related checklists

- [Web Accessibility (WCAG)](/docs/development/web-accessibility/)
- [Web Application Security](/docs/security/web-application-security/)
- [Load Balancer](/docs/networking/load-balancer/)
- [Observability](/docs/operations/observability/)
- [Capacity Planning](/docs/operations/capacity-planning/)

## References

- [web.dev — Core Web Vitals](https://web.dev/articles/vitals)
- [web.dev — Fast load times](https://web.dev/explore/fast)
- [MDN — Web Performance](https://developer.mozilla.org/en-US/docs/Web/Performance)
- [Chrome DevTools — Lighthouse](https://developer.chrome.com/docs/lighthouse/overview)
- [HTTP Archive — Web Almanac](https://almanac.httparchive.org/)
