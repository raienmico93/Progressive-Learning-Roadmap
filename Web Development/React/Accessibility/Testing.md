# Accessibility Testing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Accessibility testing is the process of verifying that a website or application can be used by people with disabilities — including those who rely on keyboards, screen readers, screen magnification, voice control, or other assistive technologies.

**Technical Definition:** Accessibility testing is the systematic evaluation of a digital product against accessibility standards (primarily WCAG 2.2) through two complementary layers: **automated scanning**, which runs rule-based tools (axe-core, Lighthouse) over the DOM to detect machine-checkable violations, and **manual testing**, which exercises the interface with a keyboard, screen reader, and visual inspection to evaluate the human experience that rules cannot reach. No single method is sufficient: automated tools detect roughly 30–40% of WCAG issues, while approximately 42% of WCAG 2.2 AA criteria cannot be automated at all. A complete accessibility testing strategy combines both layers, tests each state of the interface (not just the initial page load), and validates across real browsers and assistive technology combinations.

**Beginner-Friendly Explanation:** Accessibility testing means checking that everyone can use your app — including people who cannot see, cannot use a mouse, or need larger text. You run automated tools to catch obvious problems fast, then you put your mouse away, turn on a screen reader, and actually try to use the app yourself. If you can complete the task without sight or a mouse, you are on the right track.

### Key Characteristics

- **Two-layer approach:** Automated scanning catches mechanical failures in seconds; manual testing catches meaning, order, and real-world usability issues that tools cannot see.
- **State-aware:** Automated scanners see one state of one page — you must scan dialogs, error states, and dynamic content explicitly.
- **Keyboard-first manual testing:** The keyboard pass finds more genuine defects than any other manual technique, with no software or training required.
- **Screen reader validation:** Testing with NVDA, JAWS, or VoiceOver confirms that the accessibility tree is exposed correctly and that dynamic updates are announced.
- **Continuous integration:** Automated checks should run in CI/CD to prevent regressions on every release.

### Prerequisites

- A basic understanding of semantic HTML and the accessibility tree.
- A keyboard and a screen reader (NVDA or VoiceOver are free; JAWS has a demo mode).
- Automated tooling: axe-core, Lighthouse, `eslint-plugin-jsx-a11y`, and `jest-axe`.
- A color contrast checker (Vispero CCA or a browser extension).

### Related Programming Areas

- **Semantic HTML** — the foundation of accessible markup.
- **ARIA** — supplements native semantics for custom widgets.
- **Keyboard navigation** — the primary manual testing interface.
- **Screen readers** — the primary assistive technology for blind and low-vision users.
- **CI/CD pipelines** — where automated accessibility checks live.

### Core Concepts / Features

---

## 1. Accessibility Testing

### Definitions

**Core Definition:** Accessibility testing is the practice of verifying that a digital product can be used by people with disabilities.

**Technical Definition:** Accessibility testing evaluates conformance with WCAG 2.2 success criteria and the real-world usability of the interface with assistive technologies. It is performed through automated scans (axe-core, Lighthouse) that detect machine-checkable violations, and manual passes (keyboard, screen reader, visual, focus) that evaluate meaning, reading order, alternative text quality, and navigation consistency. The QA accessibility test pass follows a defined sequence: automated sweep → keyboard → screen reader → visual checks → defect filing.

**Beginner-Friendly Explanation:** Accessibility testing is how you find out whether your app works for people with disabilities — before they do. You run tools, then you test it yourself with a keyboard and a screen reader.

### Purposes

- To identify barriers that prevent people with disabilities from using the product.
- To comply with legal requirements (ADA, EAA, Section 508) and WCAG 2.2 AA.
- To catch regressions before they reach production through CI-integrated checks.
- To improve the experience for all users — accessible design benefits everyone.

### Syntax Rules and Structure

**The five-pass QA sequence:**

| Pass | What it catches | Time |
|------|----------------|------|
| **1. Automated sweep** | Missing labels, contrast failures, ARIA errors, heading hierarchy | Seconds per state |
| **2. Keyboard** | Focus traps, invisible focus, broken tab order, non-operable controls | Minutes |
| **3. Screen reader** | Incorrect announcements, missing names, unannounced dynamic content | Minutes |
| **4. Visual checks** | Contrast over images, zoom/reflow, motion | Minutes |
| **5. Defect filing** | Reproduction steps and expected vs. actual behavior | — |

**Constraints and Limitations:**

- A clean automated scan is not a pass — 10 of 356 W3C techniques are fully machine-checkable, and meaning, reading order, and alt-text quality are invisible to automation.
- Contrast over images produces false positives — the tool guesses at a variable background.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: A Complete QA Test Pass

```markdown
## QA Accessibility Test Pass (critical journey)
### 1. Automated sweep
- Run axe-core on the page.
- Open every dialog, submit the form incorrectly, and scan the error state.

### 2. Keyboard pass (unplug the mouse)
- Tab from the top. Note where focus goes and whether it is visible.
- Test Shift+Tab, Enter, Space, Escape, and Arrow keys on every component.
- Check: Can you get out of every dialog? Does focus return?

### 3. Screen reader pass
- Start NVDA (Ctrl+Alt+N) or VoiceOver (Cmd+F5).
- Navigate by headings (H / Insert+F7). Are they in logical order?
- Tab through forms. Are labels announced? Are errors announced?

### 4. Visual checks
- Zoom to 200% and 400%. Does content reflow?
- Check contrast on text over images.
```

**Expected Output:** A structured, repeatable verification of the critical user journey that catches both mechanical and human-experience barriers.

### Real-World Cases

- **Release gates:** The QA test pass runs on every critical journey before release.
- **Design systems:** Every component is tested with automated scans and keyboard interaction before publication.
- **Legal compliance:** Documented test passes provide evidence of WCAG conformance.

### References

- WCAGrules — The QA Accessibility Test Pass — https://wcagrules.com/checklists/qa-accessibility-testing
- BrowserStack — Accessibility Testing Techniques — https://www.browserstack.com/accessibility-testing/techniques

---

## 2. Automated Testing

### Definitions

**Core Definition:** Automated accessibility testing uses software tools to scan the rendered DOM against accessibility rules and report violations without manual intervention.

**Technical Definition:** Automated scanners (axe-core, Lighthouse, Pa11y, WAVE) parse the DOM, apply rule sets mapped to WCAG success criteria, and flag elements that fail machine-checkable conditions — missing `alt` attributes, insufficient contrast, invalid ARIA, duplicate IDs, missing form labels, and empty links or buttons. These tools run in CI/CD pipelines and browser extensions, catching mechanical failures in seconds and freeing manual testing time for issues that require human judgment.

**Beginner-Friendly Explanation:** Automated testing is a robot that checks your code for common accessibility mistakes — like images without alt text or buttons without names. It is fast and catches a lot, but it cannot tell whether your alt text is actually meaningful.

### Purposes

- To catch mechanical, rule-based violations quickly and consistently.
- To prevent accessibility regressions by running on every commit or PR.
- To free manual testing time for issues that require human judgment.
- To provide a baseline scan of every page state before deeper manual testing.

### Syntax Rules and Structure

**What automation can and cannot check:**

| Fully automatable | Cannot be fully automated |
|-------------------|---------------------------|
| HTML structure and semantics | Cognitive accessibility (readability) |
| ARIA attributes, roles, states | Contextual appropriateness of alt text |
| Form labels and error messages | Real keyboard workflows |
| Focus capability and accessible names | Screen reader UX |
| Color contrast (WCAG AA/AAA) | Content quality and clarity |
| Image alt text presence | Meaningful link text |
| Table structure and headers | — |

**Constraints and Limitations:**

- Automated tools detect roughly **30–40% of WCAG issues** — a clean scan is not a pass.
- Scanners see one state of one page — dialogs, error states, and dynamic content must be scanned explicitly.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: axe-core Scan in CI

```javascript
// accessibility.spec.ts — Playwright + axe-core
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('page has no accessibility violations', async ({ page }) => {
  await page.goto('/');
  const results = await new AxeBuilder({ page }).analyze();
  expect(results.violations).toEqual([]);
});
```

**Expected Output:** The test fails if axe-core detects any WCAG 2.1 AA violations on the page.

### Real-World Cases

- **CI gates:** axe-core runs on every pull request and blocks merges on new violations.
- **Storybook:** Component stories are scanned automatically before publication.
- **Large applications:** Scoped scans (`include('#element')`) target specific components for faster feedback.

### References

- GitHub — Automated Accessibility Scan Testing Guidelines — https://github.com/mattermost/mattermost/blob/b350edef791c93725ebb5b854867ac12469ce07d/e2e-tests/playwright/docs/accessibility/automated_scan_testing.md
- WCAGrules — The QA Accessibility Test Pass — https://wcagrules.com/checklists/qa-accessibility-testing

---

## 3. Manual Testing

### Definitions

**Core Definition:** Manual accessibility testing is the process of a human tester operating the interface with assistive technologies and keyboard input to evaluate usability and catch issues that automated tools cannot detect.

**Technical Definition:** Manual testing covers content review, keyboard testing, and screen reader verification — the three areas where automated tools are blind. It evaluates whether focus order is logical, whether focus indicators are visible, whether alt text and link text are meaningful, whether dynamic content is announced, and whether the interface is operable without a mouse. Approximately 42% of WCAG 2.2 AA criteria cannot be automated, with focus order and alt-text quality as canonical examples.

**Beginner-Friendly Explanation:** Manual testing is you, using the app like a real person with a disability would — keyboard only, screen reader on, no mouse. It catches the things a robot cannot judge: whether the order makes sense, whether the labels are clear, whether you can actually finish the task.

### Purposes

- To evaluate meaning, reading order, and content quality that automation cannot judge.
- To verify that keyboard and screen reader users can complete real tasks.
- To catch focus traps, invisible indicators, and confusing navigation.
- To validate that dynamic content and error states are announced correctly.

### Syntax Rules and Structure

**Manual testing protocol:**

| Step | Action | What to check |
|------|--------|---------------|
| 1 | Unplug the mouse | Can you complete the journey with Tab, Enter, Space, Escape, and Arrow keys? |
| 2 | Tab from the top | Does focus follow logical reading order? Is it always visible? |
| 3 | Test every key | Arrow keys in menus, Enter on buttons, Space on checkboxes, Escape on dialogs. |
| 4 | Screen reader pass | Are headings, labels, and errors announced? Are dynamic updates announced? |
| 5 | Document defects | Exact keys pressed, what happened, what should have happened. |

**Constraints and Limitations:**

- Manual testing is time-consuming — it complements, not replaces, automated scanning.
- Different screen readers and browsers behave differently — test the combinations your users actually use.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Keyboard-Only Testing Protocol

```markdown
## Keyboard-Only Test
1. Close all other tabs. Navigate to the homepage.
2. Press Tab repeatedly from the top of the page.
   - Focus must be visible at every stop.
   - Focus must follow left-to-right, top-to-bottom visual order.
   - No interactive element should be skipped.
3. Press Shift+Tab to move backward. The order must reverse predictably.
4. Open a modal. Check: Does focus move into it? Can you Tab out? Does Escape close it?
5. After closing the modal, check: Does focus return to the trigger?
```

**Expected Output:** Every interactive element is reachable, focus is always visible, and no component traps the user.

### Real-World Cases

- **Critical journey validation:** QA testers manually verify login, checkout, and core workflows.
- **Design system QA:** Every component is keyboard-tested before release.
- **Defect reproduction:** Manual testing produces the reproduction steps that make defects actionable.

### References

- Siteimprove — The Keyboard-Only User: Navigating Without a Mouse — https://www.siteimprove.com/blog/keyboard-navigation-accessibility-testing/
- BrowserStack — Accessibility Testing Techniques — https://www.browserstack.com/accessibility-testing/techniques

---

## 4. Keyboard Testing

### Definitions

**Core Definition:** Keyboard testing verifies that every interactive element and function can be operated using only a keyboard, without a mouse.

**Technical Definition:** Keyboard testing validates WCAG SC 2.1.1 (Keyboard) and SC 2.1.2 (No Keyboard Trap). It exercises Tab/Shift+Tab for sequential focus navigation, Enter/Space for activation, Arrow keys for composite widgets, Escape for dismissing dialogs, and checks that focus is always visible (SC 2.4.7). Any point where the tester gets stuck is a defect with a clear reproduction.

**Beginner-Friendly Explanation:** Keyboard testing means putting your mouse away and trying to use the entire app with just Tab, Enter, Space, and the arrow keys. If you cannot reach something, activate it, or get stuck in a dialog, that is a bug.

### Purposes

- To ensure people with motor disabilities who cannot use a mouse can operate the interface.
- To catch focus traps, invisible focus indicators, and broken tab order.
- To verify that custom widgets respond to the keys users expect.
- To satisfy WCAG SC 2.1.1 and SC 2.1.2.

### Syntax Rules and Structure

**Keyboard test matrix:**

| Key | What to test |
|-----|-------------|
| `Tab` | Focus moves forward in logical order. |
| `Shift + Tab` | Focus moves backward predictably. |
| `Enter` | Activates links and buttons. |
| `Space` | Toggles checkboxes and activates buttons. |
| `Arrow keys` | Navigate within dropdowns, radio groups, menus, and tabs. |
| `Escape` | Closes modals, menus, and tooltips. |

**Constraints and Limitations:**

- Tabbing once and looking for a skip link is not a valid test — landmarks and a heading on main content satisfy SC 2.4.1 without a skip link.
- Focus indicators must be visible in both normal mode and Windows High Contrast mode.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Keyboard Test for a Modal

```markdown
1. Tab to the "Open settings" button. Press Enter.
2. Check: Does focus move into the dialog?
3. Press Tab repeatedly. Check: Does focus stay inside the dialog (focus trap)?
4. Press Escape. Check: Does the dialog close?
5. Check: Does focus return to the "Open settings" button?
```

**Expected Output:** The dialog traps focus while open, closes on Escape, and restores focus to the trigger.

### Real-World Cases

- **Forms:** Tab through fields, Space to toggle checkboxes, Enter to submit.
- **Menus and dropdowns:** Arrow keys navigate options, Escape closes.
- **Data grids:** Arrow keys navigate cells, Enter activates editing.

### References

- Siteimprove — The Keyboard-Only User — https://www.siteimprove.com/blog/keyboard-navigation-accessibility-testing/
- Microsoft Learn — Test keyboard accessibility — https://learn.microsoft.com/en-us/training/modules/develop-keyboard-accessible-products/7-test-keyboard-accessibility

---

## 5. Screen-Reader Testing

### Definitions

**Core Definition:** Screen-reader testing is the manual process of operating the interface with a screen reader to verify that content, controls, and dynamic updates are announced correctly.

**Technical Definition:** Screen-reader testing validates the **accessibility tree** as rendered by a specific screen reader and browser combination. It verifies accessible names, roles, states, document structure (headings, landmarks), form labels and error announcements, and dynamic content via live regions. The minimum recommended coverage is NVDA + Firefox (Windows), VoiceOver + Safari (macOS), and VoiceOver + Safari (iOS); comprehensive coverage adds JAWS + Chrome and TalkBack + Chrome.

**Beginner-Friendly Explanation:** Screen-reader testing means turning on NVDA or VoiceOver and listening to your app. You check that every button, heading, form field, and error message is announced clearly and in a logical order.

### Purposes

- To verify that blind and low-vision users can perceive and operate all functionality.
- To catch missing accessible names, incorrect roles, and unannounced dynamic content.
- To validate heading hierarchy and landmark navigation.
- To satisfy WCAG SC 4.1.2 (Name, Role, Value).

### Syntax Rules and Structure

**Minimum screen reader test:**

| Step | NVDA (Windows) | VoiceOver (macOS) |
|------|----------------|-------------------|
| Start | Ctrl+Alt+N | Cmd+F5 |
| Stop speech | Ctrl | Ctrl |
| Next item | Down Arrow | Ctrl+Option+Right |
| List headings | Insert+F7 → Headings | Ctrl+Option+U → Headings |
| Next heading | H | Ctrl+Option+Cmd+H |
| List links | Insert+F7 → Links | Ctrl+Option+U → Links |

**Constraints and Limitations:**

- A screen reader does not read your HTML directly — it reads what the browser hands to the OS accessibility layer, and mismatched pairings produce odd behavior that is not your site's fault.
- You can learn enough in twenty minutes to find real problems, but this is not the same as a daily user's experience.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: NVDA Test Script

```markdown
## NVDA Test Script
1. Start NVDA (Ctrl+Alt+N). Navigate to the page.
2. Press Insert+Down to read from the top. Note the page title and main landmark.
3. Press H repeatedly. Are headings in logical order (H1 → H2 → H3)?
4. Press D to navigate landmarks. Are header, nav, main, and footer reachable?
5. Tab to the first form field. Is its label announced? Is the required state announced?
6. Submit the form with invalid data. Is the error announced? Does focus move to the error?
7. Trigger a dynamic update. Is it announced?
```

**Expected Output:** All content is announced correctly, headings and landmarks are navigable, and dynamic updates are communicated.

### Real-World Cases

- **Form validation:** Error messages announced via `role="alert"` and focus moved to the first error.
- **Single-page applications:** Route changes move focus to the new page heading and announce the change.
- **Modals:** Focus trapped inside, Escape closes, and focus returns to the trigger.

### References

- WCAGrules — How to Test With a Screen Reader — https://wcagrules.com/guides/screen-reader-testing
- GitHub — Screen Reader Testing Implementation Playbook — https://github.com/aiskillstore/marketplace/blob/main/skills/sickn33/screen-reader-testing/resources/implementation-playbook.md

---

## 6. Focus-Order Validation

### Definitions

**Core Definition:** Focus-order validation verifies that keyboard focus moves through interactive elements in a sequence that is logical, meaningful, and preserves operability.

**Technical Definition:** Focus order validation implements WCAG SC 2.4.3 (Focus Order). The tester determines the logical order of interactive elements, tabs through the content from the first focusable element, and checks that the focus order matches the logical order. The default focus order is DOM source order; positive `tabindex` values that override it are strongly discouraged. Common keyboard navigation techniques (Tab, Shift+Tab, Arrow keys) are considered logical by default.

**Beginner-Friendly Explanation:** Focus-order validation means checking that when you press Tab, you move through the page in a sensible order — left to right, top to bottom, following the visual layout. If focus jumps around unpredictably, the order needs fixing.

### Purposes

- To ensure keyboard users encounter content in a predictable, meaningful sequence.
- To prevent disorientation caused by focus jumping between unrelated regions.
- To satisfy WCAG SC 2.4.3 (Focus Order).
- To catch positive `tabindex` abuse and DOM-order mismatches.

### Syntax Rules and Structure

**Focus-order test procedure:**

| Step | Action |
|------|--------|
| 1 | Determine the logical order of interactive elements (visual reading order). |
| 2 | Tab from the first focusable element through the content. |
| 3 | Check that the focus order matches the logical order. |
| 4 | Press Shift+Tab to verify the reverse order is predictable. |
| 5 | Check that focus never lands on hidden or off-screen elements. |

**Constraints and Limitations:**

- A strictly linear order is not required — non-linear techniques are acceptable if they are common and predictable.
- Positive `tabindex` values override natural order and should be removed.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Validating Focus Order

```markdown
1. On the checkout page, the visual order is: Email → Password → Remember me → Sign in.
2. Press Tab from the top.
3. Expected focus sequence: Email → Password → Remember me checkbox → Sign in button.
4. If focus jumps to the footer before the form, the DOM order is wrong — fix the source order, not `tabindex`.
```

**Expected Output:** The focus sequence matches the visual reading order, and no interactive element is skipped.

### Real-World Cases

- **Checkout flows:** Focus must move through shipping, payment, and confirmation in order.
- **Single-page applications:** Route changes must reset focus to the main heading, not the top of the page.
- **Modals:** Focus must move into the dialog and stay trapped until closed.

### References

- W3C — Focus Order Meaningful (WCAG 3 Draft) — https://w3c.github.io/wcag3/informative/interactive-components/navigating-content/focus-order-meaningful/
- Chrome for Developers — Manually test tab order — https://developer.chrome.com/docs/lighthouse/accessibility/tabindex

---

## 7. Color-Contrast Verification

### Definitions

**Core Definition:** Color-contrast verification checks that text and UI components have sufficient luminance difference from their backgrounds to be legible for users with low vision or color blindness.

**Technical Definition:** WCAG defines minimum contrast ratios under SC 1.4.3 (Contrast, Level AA): **4.5:1 for normal text** and **3:1 for large text**. SC 1.4.11 (Non-text Contrast, Level AA) requires **3:1 for UI components and graphical objects**. Level AAA requires 7:1 for normal text and 4.5:1 for large text. Tools like Vispero's Colour Contrast Analyser calculate ratios from hex, RGB, HSV, or HSL values and display pass/fail results for WCAG 2.0, 2.1, and 2.2.

**Beginner-Friendly Explanation:** Color contrast is about whether text is easy to read against its background. Dark gray on white is fine; light gray on white is not. WCAG sets minimum ratios — 4.5:1 for normal text, 3:1 for large text and UI components.

### Purposes

- To ensure text is readable by people with low vision, color blindness, or in poor lighting.
- To make UI components and focus indicators distinguishable.
- To comply with WCAG SC 1.4.3 and SC 1.4.11.
- To catch contrast failures that automated scanners may miss over images or gradients.

### Syntax Rules and Structure

**Contrast requirements:**

| Content | Minimum ratio | WCAG SC |
|---------|--------------|---------|
| Normal text | 4.5:1 | 1.4.3 AA |
| Large text (≥18pt, or ≥14pt bold) | 3:1 | 1.4.3 AA |
| UI components and graphics | 3:1 | 1.4.11 AA |
| Normal text (AAA) | 7:1 | 1.4.6 AAA |

**Constraints and Limitations:**

- Contrast over images produces false positives in automated tools — the tool guesses at a variable background.
- Focus indicators must also meet contrast requirements (3:1 against adjacent colors).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Checking Contrast with Vispero CCA

```markdown
1. Open the Colour Contrast Analyser.
2. Use the eyedropper to pick the foreground text color from the page.
3. Pick the background color behind the text.
4. Read the ratio: if it is ≥ 4.5:1 for normal text, it passes AA.
5. For a button border, check that the ratio is ≥ 3:1 against the adjacent background.
```

**Expected Output:** A pass/fail result for WCAG 2.0, 2.1, and 2.2 AA and AAA.

### Real-World Cases

- **Form error messages:** Red text must meet 4.5:1 against the background and must not rely on color alone.
- **Buttons:** Text and borders must meet 4.5:1 (text) and 3:1 (border/background).
- **Focus indicators:** The focus ring must meet 3:1 against adjacent colors.

### References

- Vispero — Colour Contrast Analyser (CCA) — https://vispero.com/lp/color-contrast-checker/
- W3C — Understanding SC 1.4.3 Contrast (Minimum) — https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html

---

## 8. Lighthouse

### Definitions

**Core Definition:** Lighthouse is an open-source, automated tool from Google that audits web pages for performance, accessibility, best practices, SEO, and PWA compliance, producing a weighted score and actionable diagnostics.

**Technical Definition:** Lighthouse's accessibility score is a weighted average of all accessibility audits, with weights based on axe-core's user-impact analysis. Unlike performance audits, accessibility audits are binary — a page receives full credit or zero for each audit, with no partial credit. Lighthouse runs in Chrome DevTools, from the command line, or in CI, and covers WCAG A and AA rules.

**Beginner-Friendly Explanation:** Lighthouse is a free audit tool built into Chrome. You click a button and it gives you a score out of 100 for accessibility, along with a list of what is wrong and how to fix it.

### Purposes

- To provide a fast, free baseline accessibility score for any page.
- To surface actionable diagnostics for common failures (missing names, contrast, ARIA errors).
- To track accessibility trends over time in CI.
- To identify high-impact issues through weighted scoring.

### Syntax Rules and Structure

**Running Lighthouse:**

| Method | Command / Action |
|--------|-----------------|
| Chrome DevTools | Lighthouse tab → select "Accessibility" → Analyze |
| CLI | `lighthouse https://example.com --only-categories=accessibility` |
| CI | `lighthouse-ci` with assertions on score thresholds |

**Scoring:** The accessibility score is a weighted average. Heavy-weight audits include `aria-allowed-attr` (4.1%), `aria-hidden-body` (4.1%), and `button-name` (7%).

**Constraints and Limitations:**

- Lighthouse only scans the initial page state — dialogs, error states, and dynamic content are not covered.
- A high Lighthouse score does not guarantee accessibility — it covers only machine-checkable rules.
- Manual audits are not included in the score.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Lighthouse CI Assertion

```yaml
# .lighthouserc.yml
ci:
  assert:
    assertions:
      categories:accessibility: ['error', { minScore: 0.95 }]
```

**Expected Output:** The CI job fails if the accessibility score drops below 95%.

### Real-World Cases

- **Pre-deploy checks:** Lighthouse runs on staging before release.
- **Performance monitoring:** Accessibility score trends are tracked alongside performance.
- **Developer education:** Diagnostic details explain why each audit failed.

### References

- Chrome for Developers — Lighthouse Accessibility Scoring — https://developer.chrome.com/docs/lighthouse/accessibility/scoring
- Chrome for Developers — Lighthouse Accessibility Audits — https://developer.chrome.com/docs/lighthouse/accessibility/

---

## 9. axe (axe-core)

### Definitions

**Core Definition:** axe-core is an open-source accessibility engine that runs rule-based tests against the DOM to detect WCAG violations and best-practice issues.

**Technical Definition:** axe-core is the accessibility testing library behind Lighthouse, jest-axe, Cypress Accessibility, and many other tools. It provides a comprehensive default rule set covering WCAG 2.0, 2.1, and 2.2 at levels A, AA, and AAA, plus best-practice rules. Each rule is tagged with the WCAG success criteria it maps to. axe-core detects roughly **30–40% of WCAG issues** automatically.

**Beginner-Friendly Explanation:** axe-core is the engine that powers most automated accessibility checkers. It runs a set of rules against your page and tells you exactly which elements fail and why.

### Purposes

- To provide a comprehensive, standards-mapped rule set for automated scanning.
- To integrate into browser testing frameworks (Playwright, Cypress, Jest).
- To scope scans to specific components or states.
- To produce detailed violation reports with selectors and fix guidance.

### Syntax Rules and Structure

```typescript
// Playwright + axe-core
import AxeBuilder from '@axe-core/playwright';

const results = await new AxeBuilder({ page })
  .include('#component')          // Scope to element
  .disableRules(['color-contrast']) // Only with documented justification
  .analyze();

expect(results.violations).toEqual([]);
```

**Constraints and Limitations:**

- Use axe-core's default rule set — maintaining custom rule lists adds maintenance overhead.
- Disable rules only with documented justification (e.g., known framework limitation).
- axe-core cannot evaluate meaning, reading order, or alt-text quality.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Scoped Scan of a Modal

```typescript
await page.getByRole('button', { name: 'Open modal' }).click();
const results = await new AxeBuilder({ page })
  .include('[role="dialog"]')
  .analyze();
expect(results.violations).toHaveLength(0);
```

**Expected Output:** The modal is scanned in its open state for accessibility violations.

### Real-World Cases

- **Playwright E2E:** axe-core runs after navigation to catch rendered DOM issues.
- **Cypress:** Cypress Accessibility runs axe-core against every snapshot.
- **Component libraries:** Each component story is scanned in isolation.

### References

- GitHub — Automated Accessibility Scan Testing Guidelines — https://github.com/mattermost/mattermost/blob/b350edef791c93725ebb5b854867ac12469ce07d/e2e-tests/playwright/docs/accessibility/automated_scan_testing.md
- axe-core — Rule Descriptions — https://github.com/dequelabs/axe-core/blob/develop/doc/rule-descriptions.md

---

## 10. jest-axe

### Definitions

**Core Definition:** jest-axe is a Jest matcher library that runs axe-core against rendered DOM containers and asserts that no accessibility violations are found.

**Technical Definition:** jest-axe provides the `axe(container)` function and the `toHaveNoViolations()` matcher. It is used with React Testing Library to scan a component's rendered output for axe-core violations. Because it runs at the unit/component test level, it catches regressions before they reach integration or E2E tests.

**Beginner-Friendly Explanation:** jest-axe brings axe-core into your Jest unit tests. You render a component, run `axe(container)`, and assert that it has no violations — so accessibility is checked every time you run your tests.

### Purposes

- To catch accessibility violations at the component level during unit testing.
- To prevent regressions in component accessibility.
- To integrate accessibility into the existing Jest test workflow.
- To provide fast feedback without requiring a full browser.

### Syntax Rules and Structure

```javascript
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('component has no accessibility violations', async () => {
  const { container } = render(<Button>Click me</Button>);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

**Constraints and Limitations:**

- jest-axe runs in jsdom, which does not render CSS — contrast checks are limited.
- It scans the component's container only — page-level context (landmarks, heading hierarchy) is not evaluated.
- Use it alongside `eslint-plugin-jsx-a11y` for static checks and browser-based axe for full-page scans.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing Multiple Component States

```javascript
describe('Button Accessibility', () => {
  test('default button has no violations', async () => {
    const { container } = render(<Button>Click me</Button>);
    expect(await axe(container)).toHaveNoViolations();
  });

  test('icon button with aria-label has no violations', async () => {
    const { container } = render(
      <Button aria-label="Close"><CloseIcon /></Button>
    );
    expect(await axe(container)).toHaveNoViolations();
  });

  test('disabled button is still accessible', async () => {
    const { container } = render(<Button disabled>Disabled</Button>);
    expect(await axe(container)).toHaveNoViolations();
  });
});
```

**Expected Output:** All three states pass the axe-core scan.

### Real-World Cases

- **Component libraries:** Every component test includes a jest-axe scan.
- **Design system CI:** PRs fail if a component introduces a new violation.
- **Form components:** Tests verify that labels, errors, and hints are correctly associated.

### References

- GitHub — Jest with jest-axe — https://github.com/aj-geddes/useful-ai-prompts/blob/3f5182cfd739fc113f4af5244a1cf342ad7f7911/skills/accessibility-testing/references/jest-with-jest-axe.md
- jest-axe — npm — https://www.npmjs.com/package/jest-axe

---

## 11. eslint-plugin-jsx-a11y

### Definitions

**Core Definition:** `eslint-plugin-jsx-a11y` is an ESLint plugin that performs static analysis on JSX elements to catch accessibility issues before runtime.

**Technical Definition:** The plugin evaluates JSX abstract syntax trees (ASTs) against a set of accessibility rules — `alt-text`, `anchor-is-valid`, `label-has-associated-control`, `click-events-have-key-events`, `no-autofocus`, and others. It catches issues in static code during development, and must be combined with runtime tools like `@axe-core/react` or jest-axe to test the rendered DOM.

**Beginner-Friendly Explanation:** This ESLint plugin warns you about accessibility mistakes while you are writing React code — like putting a click handler on a div without a keyboard handler, or an image without alt text. It catches the problem before you even run the app.

### Purposes

- To catch accessibility issues at authoring time, before the code runs.
- To enforce consistent accessible patterns across the codebase.
- To complement runtime tools by catching static-code issues they cannot see.
- To educate developers about accessibility rules through immediate feedback.

### Syntax Rules and Structure

**Installation and configuration:**

```json
// .eslintrc.json
{
  "plugins": ["jsx-a11y"],
  "extends": ["plugin:jsx-a11y/recommended"]
}
```

**Key rules:**

| Rule | What it enforces |
|------|------------------|
| `alt-text` | Images must have alternative text. |
| `anchor-is-valid` | Anchors must point to real destinations. |
| `label-has-associated-control` | Form inputs must have labels. |
| `click-events-have-key-events` | Click handlers require keyboard handlers. |
| `no-autofocus` | Avoid autofocus on page load. |
| `no-static-element-interactions` | Static elements should not have interaction handlers. |

**Constraints and Limitations:**

- Static analysis only — it cannot see rendered DOM, dynamic content, or CSS-based issues.
- Must be combined with runtime tools (axe-core, jest-axe) for full coverage.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Catching a Missing Keyboard Handler

```jsx
// ❌ ESLint error: click-events-have-key-events
<div onClick={handleClick}>Submit</div>

// ✅ No error: native button provides keyboard support
<button type="button" onClick={handleClick}>Submit</button>
```

**Expected Output:** ESLint flags the `<div>` click handler because it lacks a keyboard event handler, guiding the developer to use a `<button>`.

### Real-World Cases

- **React codebases:** The plugin is included in Airbnb's ESLint config and is standard in React projects.
- **CI linting:** Accessibility lint rules run on every commit alongside other ESLint rules.
- **Developer education:** Immediate feedback teaches accessible patterns as developers write code.

### References

- GitHub — jsx-eslint/eslint-plugin-jsx-a11y — https://github.com/jsx-eslint/eslint-plugin-jsx-a11y
- npm — eslint-plugin-jsx-a11y — https://www.npmjs.com/package/eslint-plugin-jsx-a11y

---

## 12. Accessibility Testing Best Practices

### Definitions

**Core Definition:** Accessibility testing best practices are the guidelines that make testing thorough, efficient, and integrated into the development lifecycle.

**Technical Definition:** Best practices combine automated scans with manual keyboard and screen reader passes, run on every state (not just the initial page load), test across real browsers and assistive technology combinations, and integrate checks into CI/CD. They emphasize that a clean automated scan is not a pass, and that defects must be reported with exact reproduction steps.

**Beginner-Friendly Explanation:** Best practices are the habits that make accessibility testing actually work — automate what you can, test by hand what you must, check every state, and never trust a clean scan as proof of accessibility.

### Purposes

- To catch both mechanical and human-experience barriers.
- To prevent regressions through CI integration.
- To make defect reports actionable and reproducible.
- To build accessibility into the development process rather than bolting it on at the end.

### Best Practices Checklist

| Practice | Description |
|----------|-------------|
| **Automate first** | Run axe-core or Lighthouse to clear mechanical failures quickly. |
| **Test every state** | Open dialogs, submit forms wrong, scan error states — scanners see one state at a time. |
| **Keyboard pass** | Unplug the mouse and complete the journey with Tab, Enter, Space, Escape, and Arrow keys. |
| **Screen reader pass** | Test with NVDA or VoiceOver — listen for correct announcements and logical order. |
| **Check focus order** | Tab order must match logical reading order; no positive `tabindex`. |
| **Verify contrast** | 4.5:1 normal text, 3:1 large text and UI components. |
| **Integrate in CI** | Run automated checks on every PR to catch regressions. |
| **Report defects precisely** | Exact keys pressed, what happened, what should have happened. |
| **Never equate a clean scan with compliance** | Automated tools catch only 30–40% of issues. |

**Constraints and Limitations:**

- Accessibility testing is not a one-time audit — it must run on every release.
- Manual testing requires practice — learn six screen reader commands and start listening.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Integrated Accessibility Testing Workflow

```markdown
## Per-PR (automated)
- ESLint jsx-a11y runs on commit.
- jest-axe scans every component test.

## Per-release (manual)
1. Run axe-core on every critical journey state.
2. Keyboard pass: unplug the mouse, complete the journey.
3. Screen reader pass: NVDA or VoiceOver, check headings, labels, errors, dynamic updates.
4. Contrast check: CCA on text over images and custom UI.

## Defect filing
- "On the checkout page, pressing Tab from the email field skips the password field and jumps to the footer. Expected: focus moves to the password field."
```

**Expected Output:** A repeatable process that catches mechanical issues early and human-experience issues before release.

### Real-World Cases

- **Release gates:** Automated checks block merges on new violations; manual passes block releases on critical journey failures.
- **Design systems:** Components are tested with jest-axe and keyboard interaction before publication.
- **Legal compliance:** Documented test passes and defect fixes provide evidence of WCAG conformance.

### References

- WCAGrules — The QA Accessibility Test Pass — https://wcagrules.com/checklists/qa-accessibility-testing
- BrowserStack — Accessibility Testing Techniques — https://www.browserstack.com/accessibility-testing/techniques
- Siteimprove — The Keyboard-Only User — https://www.siteimprove.com/blog/keyboard-navigation-accessibility-testing/

---

## References

- WCAGrules — The QA Accessibility Test Pass — https://wcagrules.com/checklists/qa-accessibility-testing
- WCAGrules — How to Test With a Screen Reader — https://wcagrules.com/guides/screen-reader-testing
- BrowserStack — Accessibility Testing Techniques — https://www.browserstack.com/accessibility-testing/techniques
- Siteimprove — The Keyboard-Only User: Navigating Without a Mouse — https://www.siteimprove.com/blog/keyboard-navigation-accessibility-testing/
- W3C — Focus Order Meaningful (WCAG 3 Draft) — https://w3c.github.io/wcag3/informative/interactive-components/navigating-content/focus-order-meaningful/
- W3C — Understanding SC 1.4.3 Contrast (Minimum) — https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html
- Vispero — Colour Contrast Analyser (CCA) — https://vispero.com/lp/color-contrast-checker/
- Chrome for Developers — Lighthouse Accessibility Scoring — https://developer.chrome.com/docs/lighthouse/accessibility/scoring
- Chrome for Developers — Lighthouse Accessibility Audits — https://developer.chrome.com/docs/lighthouse/accessibility/
- axe-core — Rule Descriptions — https://github.com/dequelabs/axe-core/blob/develop/doc/rule-descriptions.md
- GitHub — Automated Accessibility Scan Testing Guidelines — https://github.com/mattermost/mattermost/blob/b350edef791c93725ebb5b854867ac12469ce07d/e2e-tests/playwright/docs/accessibility/automated_scan_testing.md
- GitHub — jest-axe Examples — https://github.com/aj-geddes/useful-ai-prompts/blob/3f5182cfd739fc113f4af5244a1cf342ad7f7911/skills/accessibility-testing/references/jest-with-jest-axe.md
- GitHub — jsx-eslint/eslint-plugin-jsx-a11y — https://github.com/jsx-eslint/eslint-plugin-jsx-a11y
- npm — eslint-plugin-jsx-a11y — https://www.npmjs.com/package/eslint-plugin-jsx-a11y
- Microsoft Learn — Test keyboard accessibility — https://learn.microsoft.com/en-us/training/modules/develop-keyboard-accessible-products/7-test-keyboard-accessibility
- Chrome for Developers — Manually test tab order — https://developer.chrome.com/docs/lighthouse/accessibility/tabindex
- GitHub — Screen Reader Testing Implementation Playbook — https://github.com/aiskillstore/marketplace/blob/main/skills/sickn33/screen-reader-testing/resources/implementation-playbook.md