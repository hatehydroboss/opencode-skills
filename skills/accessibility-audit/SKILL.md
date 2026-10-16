---
name: accessibility-audit
description: Audit and fix web accessibility issues for WCAG 2.1 AA compliance. Use when checking screen reader support, keyboard navigation, color contrast, ARIA labels, alt text, form accessibility, or inclusive design. Trigger on "accessibility", "a11y", "WCAG", "screen reader", "keyboard navigation", "ARIA", "alt text", "color contrast", or "inclusive design".
---

# Accessibility Audit

Accessibility is not a feature — it's a quality attribute. Aim for WCAG 2.1 AA.

## The Four POUR Principles

1. **Perceivable** — content available to all senses
2. **Operable** — navigable by all input methods
3. **Understandable** — clear and predictable
4. **Robust** — works across assistive technologies

## Quick Audit Checklist

### Images & Media
- [ ] All `<img>` have meaningful `alt=""` (empty for decorative)
- [ ] Complex images (charts, diagrams) have long descriptions
- [ ] Videos have captions; audio has transcripts
- [ ] No text baked into images (if unavoidable, provide description)

### Forms
- [ ] Every input has a `<label>` (use `for`/`id`, not just placeholder)
- [ ] Required fields marked with text + `aria-required`, not just `*`
- [ ] Error messages linked to inputs: `aria-describedby` + `role="alert"`
- [ ] No positive tabindex (use DOM order)

### Keyboard
- [ ] All interactive elements reachable via Tab
- [ ] Visible focus indicator (don't remove outlines without replacement)
- [ ] Logical tab order (no keyboard traps)
- [ ] Skip-to-content link as first focusable element
- [ ] Enter/Space activate buttons and links

### Semantics
- [ ] One `<h1>` per page, heading levels don't skip
- [ ] Landmarks: `<header>`, `<nav>`, `<main>`, `<footer>`
- [ ] Lists use `<ul>`/`<ol>`, not div soup
- [ ] Tables use `<th>` with `scope`
- [ ] Links say where they go ("Read more about pricing" not "Click here")

### Color & Contrast
- [ ] Text contrast ratio ≥ 4.5:1 (normal), ≥ 3:1 (large text ≥ 24px)
- [ ] UI components and graphical objects ≥ 3:1
- [ ] Information not conveyed by color alone (errors have icon + text)
- [ ] Works in high contrast mode

### ARIA
- [ ] No ARIA when HTML semantics suffice (`<button>` > `<div role="button">`)
- [ ] `aria-label` or `aria-labelledby` on icon-only buttons
- [ ] `aria-live="polite"` for dynamic content updates
- [ ] `aria-expanded` on collapsible sections

## Screen Reader Testing

Test with at least one screen reader:
- **Windows**: NVDA + Firefox (free)
- **Mac**: VoiceOver + Safari
- **Mobile**: TalkBack (Android) / VoiceOver (iOS)

What to verify:
- Page title announces page purpose
- Images read alt text or are skipped
- Form fields announce labels and errors
- Dynamic content changes are announced

## Common Fixes

| Issue | Fix |
|-------|-----|
| Icon button with no label | `aria-label="Close dialog"` |
| Modal doesn't trap focus | Implement focus trap, return focus on close |
| Dropdown not keyboard accessible | Use native `<select>` or implement full keyboard support |
| Loading spinner invisible to SR | Add `role="status"` + `aria-live="polite"` |
| Table data confusing | Add `<caption>`, `scope` on `<th>` |

## Automated Testing

Run in CI:
```bash
# axe-core (browser extension or CI)
npx @axe-core/cli https://yoursite.com

# Lighthouse CI
npx lhci autorun

# Pa11y
npx pa11y https://yoursite.com
```

Automated tools catch ~30% of issues. Manual testing is essential.

## Checklist

- [ ] All images have appropriate alt text
- [ ] All form inputs have associated labels
- [ ] Full keyboard navigation works
- [ ] Focus indicators visible
- [ ] Color contrast meets AA (4.5:1 text, 3:1 UI)
- [ ] Heading hierarchy is logical
- [ ] Screen reader tested on key pages
- [ ] Automated accessibility tests in CI
- [ ] Dynamic content changes announced
