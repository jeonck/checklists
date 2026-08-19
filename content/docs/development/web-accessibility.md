---
title: "Web Accessibility (WCAG)"
description: "Verify a web interface meets WCAG 2.2 level AA in practice, tested with a keyboard and a screen reader."
icon: "accessibility_new"
weight: 470
toc: true
tags: ["accessibility", "wcag", "frontend", "inclusive-design"]
---

Automated tools catch roughly a third of accessibility defects, and almost none of the ones that actually stop somebody completing a task. This checklist is organised around how the interface is used — keyboard, screen reader, zoom, forms — rather than around the numbering of the specification, and it covers the success criteria added in WCAG 2.2 that most existing components fail. Target level AA unless a legal obligation says otherwise.

{{< alert context="info" text="**Who runs this:** the engineer or designer shipping the interface, ideally with a colleague who uses a screen reader daily. **When:** during component development and again before release of any new page, flow, or design system component." />}}

## 1. Semantics and structure

- [ ] **Native HTML elements are used before ARIA** — a `button` element is focusable, activatable by Enter and Space, and announced correctly for free; a `div` with a click handler is none of those things.
- [ ] **Headings describe the structure and nest without skipping levels** — screen reader users navigate by heading, and a page whose headings were chosen for their font size is unnavigable.
- [ ] **Landmark regions wrap the main areas of the page** — `header`, `nav`, `main`, and `footer`, with exactly one `main` per page.
- [ ] **Lists, tables, and figures use the elements that mean those things** — a data table needs `th` with a correct `scope`, or the relationship between a cell and its header is lost.
- [ ] **The page has a unique, descriptive `title` that leads with the page-specific part** — it is the first thing announced and the first thing shown in a tab list.
- [ ] **`lang` is set on the `html` element and on any inline passage in another language** — otherwise the screen reader pronounces the content with the wrong phoneme set.
- [ ] **ARIA is not applied where it contradicts the element** — a wrong role is worse than no role, because it overrides what the browser already knew.
- [ ] **Every custom widget follows an established ARIA pattern completely** — the roles, states, and keyboard interactions of the APG pattern are a package, and implementing half of one produces a control that announces itself as something it does not behave like.

## 2. Keyboard operation

- [ ] **Every interactive element can be reached and operated with the keyboard alone** — unplug the mouse and complete the primary user journey; this single test finds more real defects than any scanner.
- [ ] **Focus order follows the visual reading order** — a positioned layout that reorders content visually without reordering the DOM produces a focus path that jumps around the screen.
- [ ] **There is no keyboard trap** — focus must be able to leave every component, including embedded media players and third-party iframes.
- [ ] **A modal moves focus in on open, keeps focus inside while open, and returns focus to the trigger on close** — and Escape closes it.
- [ ] **A skip link to the main content is the first focusable element and becomes visible on focus** — a keyboard user should not have to tab through fifty navigation links on every page.
- [ ] **Custom controls implement the expected keys for their role** — arrow keys within a tab list, radio group, or menu; Enter and Space to activate; Escape to dismiss.
- [ ] **No functionality requires a specific pointer gesture** — WCAG 2.2 requires that anything achievable by dragging also be achievable by a single tap or click, which rules out drag-only reordering and sliders with no keyboard equivalent.
- [ ] **Custom keyboard shortcuts using single characters can be turned off or remapped** — they collide with screen reader navigation keys.

## 3. Focus visibility

- [ ] **A visible focus indicator is present on every focusable element** — removing the default outline without replacing it is the most common accessibility regression in any design refresh.
- [ ] **The focus indicator has sufficient contrast against both the component and the background** — a subtle indicator that meets nobody's needs is functionally the same as none.
- [ ] **The focused element is never fully hidden by sticky headers, footers, or cookie banners** — this is WCAG 2.2 criterion 2.4.11, and a sticky header with no scroll offset fails it on every page.
- [ ] **Focus styles are applied through `:focus-visible` where mouse users should not see them** — this removes the usual business objection to visible focus rings without harming keyboard users.
- [ ] **Focus is moved deliberately after actions that change context** — after a route change in a single-page application, move focus to the new heading, or the keyboard user is left at the top of a page they cannot tell has changed.
- [ ] **Focus is never moved automatically in a way that interrupts the user** — auto-focusing a field mid-interaction disorients screen reader users.

## 4. Colour, contrast, and visual presentation

- [ ] **Text meets a 4.5:1 contrast ratio, or 3:1 for large text** — measure the computed values including any overlay or gradient behind the text.
- [ ] **Non-text elements meet 3:1** — icons that carry meaning, form field borders, focus indicators, and chart series boundaries all count.
- [ ] **Colour is never the only way information is conveyed** — an error state needs an icon or text, and a chart needs labels or patterns, not just a legend keyed by hue.
- [ ] **The interface remains usable at 200% zoom and at 400% with a 320px-wide viewport** — content must reflow into one column without horizontal scrolling and without clipping.
- [ ] **Text spacing overrides do not break the layout** — increased line height, letter spacing, and word spacing must not cause clipped or overlapping text, which fixed-height containers usually do.
- [ ] **`prefers-reduced-motion` is honoured** — remove parallax, large transitions, and auto-playing animation for users who have asked for it, because motion can cause genuine nausea.
- [ ] **Nothing flashes more than three times per second** — this criterion exists to prevent seizures and has no exceptions worth negotiating.

## 5. Forms and error handling

- [ ] **Every input has a programmatically associated label** — a placeholder is not a label; it disappears on typing and is often ignored by assistive technology.
- [ ] **Required fields are marked in the accessible name or with `aria-required`, not only with a coloured asterisk** — a visual convention explained once at the top of the form is not available to someone tabbing straight into the field.
- [ ] **Errors are announced, identified in text, and describe how to fix the problem** — "invalid input" tells nobody anything; "date must be in the past" is actionable.
- [ ] **Error messages are associated with their field via `aria-describedby`** — so the error is read when the field receives focus rather than only existing visually above it.
- [ ] **Validation errors move focus or announce via a live region** — a message rendered silently is invisible to a screen reader user who has already moved on.
- [ ] **Fields collecting the user's own information use the correct `autocomplete` token** — this drives browser autofill and is a WCAG requirement, not a convenience.
- [ ] **Information the user already entered is not requested again in the same process** — WCAG 2.2 criterion 3.3.7; either carry it forward or offer it for selection.
- [ ] **Authentication does not depend on a cognitive function test** — WCAG 2.2 criterion 3.3.8 means allowing password managers to paste, and not requiring the user to transcribe, remember, or solve a puzzle with no alternative.
- [ ] **Submissions that are legal, financial, or irreversible are reversible, checked, or confirmed** — a confirmation step is the accepted mechanism.

## 6. Content, media, and non-text alternatives

- [ ] **Every image has an alternative that serves its purpose** — describe the function for a functional image, the information for an informative one, and use an empty `alt` for a purely decorative one so it is skipped rather than announced as a filename.
- [ ] **Complex images have a longer description available in text** — charts, diagrams, and infographics cannot be served by a short alternative.
- [ ] **Link text makes sense out of context** — screen reader users list links; a page of "read more" links is a page of identical, useless entries.
- [ ] **Buttons and icon-only controls have accessible names** — check the computed accessible name in the browser's accessibility inspector, not just that an `aria-label` attribute exists.
- [ ] **Pre-recorded video has accurate captions and audio description where visual information is not in the soundtrack** — auto-generated captions without review routinely fail on names and technical terms.
- [ ] **Audio-only content has a transcript, and nothing auto-plays sound for more than three seconds without a control to stop it** — unexpected audio competes directly with a screen reader's speech output.
- [ ] **Help mechanisms appear in a consistent place across pages** — WCAG 2.2 criterion 3.2.6, which matters most for users who rely on finding help in the same location every time.

## 7. Dynamic behaviour and timing

- [ ] **Content that updates without a page load is announced appropriately** — use a polite live region for status messages and reserve assertive for genuinely urgent ones.
- [ ] **Live regions exist in the DOM before the content is inserted** — a region added and populated in the same tick is frequently not announced at all.
- [ ] **Loading states are conveyed to assistive technology** — a spinner with no accessible status leaves a screen reader user with silence and no idea whether anything is happening.
- [ ] **Time limits can be turned off, adjusted, or extended** — including session timeouts, with a warning and at least twenty seconds to respond.
- [ ] **Auto-updating or moving content can be paused, stopped, or hidden** — carousels, tickers, and live feeds all qualify.
- [ ] **Focus and context do not change on input alone** — selecting a value in a dropdown must not navigate the page; require an explicit action.
- [ ] **Disabled controls are used sparingly, and a disabled submit button is not the only feedback** — a disabled element is typically skipped by keyboard navigation and offers no explanation of what is missing.

## 8. Touch and target size

- [ ] **Interactive targets are at least 24 by 24 CSS pixels, or have equivalent spacing** — WCAG 2.2 criterion 2.5.8; the common failures are icon-only buttons, table row actions, and close buttons.
- [ ] **Targets that sit close together have adequate spacing** — adjacent small controls cause mis-taps for anyone with a motor impairment or a moving vehicle.
- [ ] **Pointer actions complete on release, not on press** — so a user who lands on the wrong target can slide off to cancel.
- [ ] **Anything triggered by a multi-point or path-based gesture has a single-pointer alternative** — pinch-to-zoom-only maps and swipe-only carousels fail this.
- [ ] **A visible label's text is contained in the accessible name** — otherwise a speech-input user saying the visible label cannot activate the control.
- [ ] **Hover-triggered content can also be triggered by keyboard, remains visible while the pointer moves onto it, and is dismissible with Escape.**

## 9. Testing and governance

- [ ] **An automated scanner runs in CI on key pages and on component stories** — it catches the mechanical third cheaply and prevents regressions.
- [ ] **The critical journey has been completed with a screen reader** — NVDA or JAWS with a desktop browser, and VoiceOver on iOS, since mobile behaviour differs materially.
- [ ] **The keyboard-only pass is done by someone who did not build the feature** — authors unconsciously avoid the paths they know are broken.
- [ ] **Testing includes 400% zoom, forced colours or high contrast mode, and reduced motion** — forced colours in particular breaks interfaces that convey state through background colour alone.
- [ ] **Known gaps are recorded with a remediation date rather than quietly carried** — an accessibility statement that claims full conformance while known defects exist is a legal liability.
- [ ] **Design system components are audited once and reused** — fixing a component fixes every page that uses it, which is the only affordable way to maintain accessibility at scale.
- [ ] **Accessibility acceptance criteria are written into the ticket** — anything added at the end of a sprint is the thing dropped when the sprint runs late.

{{< alert context="warning" text="**Common mistake:** treating a clean automated scan as a pass. Scanners cannot judge whether alternative text is meaningful, whether focus order matches the visual order, or whether a custom widget behaves the way it announces itself. The keyboard-only pass and the screen reader pass are not optional." />}}

## Sign-off

| Area | Reviewer | Date | Outcome |
|---|---|---|---|
| Semantics and structure | | | Pass / Pass with actions / Fail |
| Keyboard operation | | | Pass / Pass with actions / Fail |
| Focus visibility | | | Pass / Pass with actions / Fail |
| Colour, contrast, and visual presentation | | | Pass / Pass with actions / Fail |
| Forms and error handling | | | Pass / Pass with actions / Fail |
| Content, media, and non-text alternatives | | | Pass / Pass with actions / Fail |
| Dynamic behaviour and timing | | | Pass / Pass with actions / Fail |
| Touch and target size | | | Pass / Pass with actions / Fail |
| Testing and governance | | | Pass / Pass with actions / Fail |

Record every failure against the specific success criterion it breaches, so the accessibility statement and any remediation plan can be written from this table directly.

## Related checklists

- [Frontend Performance](/docs/development/frontend-performance/)
- [Code Review](/docs/development/code-review/)
- [REST API Design](/docs/development/rest-api-design/)
- [Web Application Security](/docs/security/web-application-security/)

## References

- [W3C — Web Content Accessibility Guidelines (WCAG) 2.2](https://www.w3.org/TR/WCAG22/)
- [W3C — How to Meet WCAG (Quick Reference)](https://www.w3.org/WAI/WCAG22/quickref/)
- [W3C — ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
- [W3C WAI — Evaluating Web Accessibility](https://www.w3.org/WAI/test-evaluate/)
- [MDN — Accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility)
