# Accessibility Testing: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Accessibility testing is the systematic process of verifying that a web application can be perceived, operated, and understood by people with disabilities, using a combination of automated tools, manual keyboard audits, screen reader testing, contrast verification, zoom evaluation, and preference-based checks.

**Technical Definition:** Accessibility testing is the practice of evaluating a web application against the Web Content Accessibility Guidelines (WCAG) 2.2 success criteria and the WAI-ARIA Authoring Practices using a multi-layered approach: (1) **automated static analysis** via `eslint-plugin-jsx-a11y` to catch JSX-level issues during development; (2) **automated runtime audits** via `axe-core` integrated into unit tests (`jest-axe`) and CI pipelines; (3) **manual keyboard audits** to verify tab order, focus visibility, and keyboard traps; (4) **assistive technology testing** with screen readers (NVDA, JAWS, VoiceOver) to validate real-world announcements; (5) **visual and contrast verification** against WCAG AA/AAA ratios (4.5:1 for normal text, 3:1 for large text and non-text); (6) **zoom and reflow testing** at up to 400% magnification (WCAG 1.4.10); and (7) **user-preference testing** for `prefers-reduced-motion`, `prefers-color-scheme`, and `prefers-contrast`.

**Beginner-Friendly Explanation:** You cannot just "add accessibility" and assume it works. You have to test it—the same way you test any other feature. Automated tools catch some problems (like missing alt text), but they only catch about 30–50% of issues. The rest requires a human: tabbing through the app with a keyboard, listening to it with a screen reader, checking colour contrast, zooming in to 400%, and respecting the user's motion and colour preferences. Accessibility testing is the difference between "we wrote accessible code" and "we know it works for real users."

### Key Characteristics

- **Multi-Layered Defence:** No single tool catches all issues; automated, manual, and assistive-technology testing are complementary.
- **Automated Tools Catch 30–50%:** `axe-core` and `eslint-plugin-jsx-a11y` are valuable but insufficient on their own.
- **Shift-Left Testing:** Linting catches issues in the editor; unit tests catch them in CI; manual testing catches them before release.
- **Real Assistive Technology Required:** Testing with actual screen readers (NVDA, JAWS, VoiceOver) is non-negotiable for critical flows.
- **WCAG 2.2 AA is the Baseline:** Most organisations target AA; AAA is aspirational for high-stakes content.
- **Keyboard-First Mindset:** Unplug your mouse and try to complete every workflow.
- **Zoom and Reflow Matter:** 400% zoom is a WCAG requirement, not a nice-to-have.
- **User Preferences Are Mandatory:** `prefers-reduced-motion` must be respected to prevent vestibular harm.

### Prerequisites

- Solid understanding of HTML semantics and WAI-ARIA.
- Working knowledge of React function components and Hooks.
- Familiarity with a testing framework (Jest, Vitest) and Testing Library.
- Access to a screen reader (NVDA on Windows, VoiceOver on macOS/iOS, JAWS on Windows).
- Awareness of WCAG 2.2 success criteria and the WAI-ARIA Authoring Practices.

### Related Programming Areas

- **Web Accessibility (a11y):** WCAG, ARIA, and assistive technology.
- **Testing:** Unit, integration, and end-to-end testing.
- **CI/CD:** Automated quality gates and regression prevention.
- **Design Systems:** Contrast, typography, and motion tokens.
- **Performance:** Zoom, reflow, and layout stability.

### Core Concepts / Features

1. Automated Audit Pipelines
2. Keyboard Navigation Audits
3. Assistive Technology Screen Readers
4. Visual & Contrast Verification
5. Zoom & Reflow Evaluation
6. User-Driven Preferences

---

## Core Concept 1: Automated Audit Pipelines

### Definitions

**Core Definition:** Automated audit pipelines are the integration of accessibility linters (`eslint-plugin-jsx-a11y`), unit testing utilities (`axe-core`, `@testing-library/jest-dom`), and continuous integration scanners to catch accessibility violations automatically before they reach production.

**Technical Definition:** Automated accessibility testing operates at three layers. **Static analysis** (`eslint-plugin-jsx-a11y`) runs in the editor and CI, flagging JSX-level issues such as missing `alt` attributes, invalid ARIA roles, and click handlers on non-interactive elements. **Runtime unit testing** (`jest-axe` + `@testing-library/react`) renders components in JSDOM and runs `axe-core` against the rendered output, catching contrast, ARIA, and structural violations. **CI scanners** (Lighthouse CI, Pa11y CI, axe DevTools) run against a deployed preview or production build, catching regressions across the full application. Together, these layers form a "shift-left" strategy that prevents most accessibility bugs from reaching users.

**Beginner-Friendly Explanation:** Automated testing is like spell-check for accessibility. It catches the obvious mistakes—missing labels, broken ARIA, low contrast—before anyone reviews the code. It is fast, runs on every commit, and never gets tired. But like spell-check, it does not catch everything: it cannot tell you whether a screen reader announces something correctly or whether keyboard navigation feels logical. That is why it is the first layer, not the only one.

### Purposes

- To catch accessibility violations at the earliest possible stage (editor, commit, CI).
- To prevent regressions when components are refactored.
- To enforce consistent standards across a codebase.
- To free human reviewers to focus on issues that automated tools cannot detect.
- To provide documentation of known accessibility patterns in the codebase.

### Syntax Rules and Structure

**1. ESLint Plugin (`eslint-plugin-jsx-a11y`):**
```bash
npm install -D eslint eslint-plugin-jsx-a11y
```

```json
// .eslintrc.json
{
  "extends": [
    "plugin:jsx-a11y/recommended"
  ],
  "plugins": ["jsx-a11y"],
  "rules": {
    "jsx-a11y/alt-text": "error",
    "jsx-a11y/aria-props": "error",
    "jsx-a11y/aria-role": "error",
    "jsx-a11y/click-events-have-key-events": "error",
    "jsx-a11y/no-noninteractive-element-interactions": "warn"
  }
}
```

**Component Breakdown:**
- `plugin:jsx-a11y/recommended`: Enables the recommended rule set.
- `alt-text`: Requires `alt` on `<img>`, `<area>`, `<input type="image">`.
- `aria-props`: Validates ARIA attributes against known names.
- `aria-role`: Validates ARIA roles against known values.
- `click-events-have-key-events`: Requires keyboard handlers alongside click handlers.
- `no-noninteractive-element-interactions`: Prevents event handlers on non-interactive elements.

**2. Unit Testing with `jest-axe`:**
```bash
npm install -D jest-axe @testing-library/react @testing-library/jest-dom
```

```jsx
// Button.test.jsx
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import { Button } from './Button';

expect.extend(toHaveNoViolations);

describe('Button', () => {
  it('has no accessibility violations', async () => {
    const { container } = render(<Button>Click me</Button>);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  it('has an accessible name', () => {
    const { getByRole } = render(<Button>Click me</Button>);
    expect(getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  it('is keyboard focusable', () => {
    const { getByRole } = render(<Button>Click me</Button>);
    const button = getByRole('button');
    button.focus();
    expect(button).toHaveFocus();
  });
});
```

**Component Breakdown:**
- `expect.extend(toHaveNoViolations)`: Adds the `toHaveNoViolations` matcher.
- `axe(container)`: Runs axe-core against the rendered output.
- `getByRole('button', { name })`: Queries by accessible role and name.
- `toHaveFocus()`: Asserts keyboard focus.

**3. CI Scanner (`Lighthouse CI`):**
```yaml
# .github/workflows/accessibility.yml
name: Accessibility Audit
on: [push, pull_request]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run build
      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v12
        with:
          urls: |
            https://preview.example.com/
            https://preview.example.com/checkout
          budgetPath: ./lighthouse-budget.json
          uploadArtifacts: true
```

```json
// lighthouse-budget.json
[
  {
    "path": "/*",
    "audits": [
      { "id": "accessibility", "weight": 1 }
    ],
    "assertions": {
      "categories:accessibility": ["error", { "minScore": 0.95 }]
    }
  }
]
```

**Component Breakdown:**
- `treosh/lighthouse-ci-action`: Runs Lighthouse against preview URLs.
- `budgetPath`: Defines accessibility score thresholds.
- `categories:accessibility`: Asserts a minimum score (0.95 = 95%).

**Syntax Rules:**
- Install `eslint-plugin-jsx-a11y` and extend the recommended configuration.
- Add `jest-axe` to unit tests for every component and critical workflow.
- Use `@testing-library/jest-dom` matchers (`toHaveFocus`, `toHaveAccessibleName`, `toBeInTheDocument`).
- Run Lighthouse CI (or Pa11y CI, axe DevTools CI) against preview deployments.
- Set accessibility score thresholds as CI gates (fail the build below the threshold).
- Test both individual components and composed pages.

**Constraints and Limitations:**
- Automated tools catch only 30–50% of accessibility issues.
- `axe-core` in JSDOM cannot test colour contrast reliably (no layout engine).
- Lighthouse CI requires a deployed preview URL; it does not work on local-only builds.
- False positives and false negatives exist; investigate every report.
- Automated tests cannot verify that a screen reader announces something correctly.

### Annotated Code Example: Complete Automated Pipeline

```jsx
// TextField.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { axe, toHaveNoViolations } from 'jest-axe';
import { TextField } from './TextField';

expect.extend(toHaveNoViolations);

describe('TextField', () => {
  it('renders with a label and no axe violations', async () => {
    const { container } = render(<TextField label="Email" />);

    // 1. Automated axe audit
    const results = await axe(container);
    expect(results).toHaveNoViolations();

    // 2. Accessible name check
    expect(screen.getByLabelText('Email')).toBeInTheDocument();
  });

  it('associates error messages with the input', async () => {
    render(<TextField label="Email" error="Email is required" />);

    const input = screen.getByLabelText('Email');
    expect(input).toHaveAttribute('aria-invalid', 'true');
    expect(input).toHaveAccessibleDescription('Email is required');
  });

  it('is operable with the keyboard', async () => {
    const user = userEvent.setup();
    render(<TextField label="Email" />);

    await user.tab();
    expect(screen.getByLabelText('Email')).toHaveFocus();
  });
});
```

**Expected Output:** All tests pass if the component is accessible. If the label is missing, the axe audit fails. If `aria-invalid` is missing, the second test fails. If the input cannot receive keyboard focus, the third test fails.

**Why This Output Occurs:** `axe(container)` runs the full axe-core rule set against the rendered DOM. `getByLabelText` verifies the label is associated. `toHaveAttribute('aria-invalid', 'true')` verifies the error state. `toHaveAccessibleDescription` verifies the description is linked. `userEvent.tab()` simulates keyboard navigation, and `toHaveFocus()` verifies the input received focus.

### Real-World Cases

- **Component libraries:** Every component has a `.test.tsx` with `jest-axe` and role-based queries.
- **CI/CD pipelines:** Lighthouse CI runs on every pull request and fails if the accessibility score drops below 95%.
- **Design systems:** ESLint rules are shared across all packages to enforce consistent patterns.
- **Marketing sites:** Pa11y CI runs against all key pages to catch regressions.
- **Enterprise apps:** axe DevTools CI runs against a full user journey.

### References

- eslint-plugin-jsx-a11y - https://github.com/jsx-eslint/eslint-plugin-jsx-a11y
- jest-axe - https://github.com/nickcolley/jest-axe
- Testing Library - https://testing-library.com/docs/
- Testing Library Jest DOM - https://github.com/testing-library/jest-dom
- axe-core - https://github.com/dequelabs/axe-core
- Lighthouse CI - https://github.com/GoogleChrome/lighthouse-ci
- Pa11y CI - https://github.com/pa11y/pa11y-ci
- Deque axe DevTools - https://www.deque.com/axe/devtools/

---

## Core Concept 2: Keyboard Navigation Audits

### Definitions

**Core Definition:** Keyboard navigation audits are the manual process of navigating an application using only the keyboard (Tab, Shift+Tab, Enter, Space, Arrow keys, Escape) to detect keyboard traps, verify logical tab order, and check visible focus indicators.

**Technical Definition:** Keyboard navigation testing verifies compliance with WCAG 2.2 SC 2.1.1 (Keyboard), SC 2.1.2 (No Keyboard Trap), SC 2.4.3 (Focus Order), SC 2.4.7 (Focus Visible), and SC 2.4.11 (Focus Not Obscured). The audit process involves: (1) unplugging the mouse; (2) tabbing through every interactive element on the page; (3) verifying that the tab order matches the visual order; (4) confirming that focus is always visible; (5) checking that focus is not trapped in any component (except intentionally in modals); (6) testing all keyboard interactions defined by the WAI-ARIA Authoring Practices (arrow keys for tabs, Enter/Space for buttons, Escape for overlays); and (7) verifying that focus is restored after overlays close.

**Beginner-Friendly Explanation:** Keyboard navigation is the most fundamental accessibility test. If you cannot use your app with just a keyboard, you cannot use it with a screen reader, voice control, or switch device. The test is simple: put your mouse away, close your eyes if you can, and try to complete every task using only Tab, Shift+Tab, Enter, Space, and the arrow keys. Can you see where focus is? Does the order make sense? Do you ever get stuck? These are the questions the audit answers.

### Purposes

- To verify that every interactive element is reachable and operable via keyboard.
- To detect keyboard traps where focus cannot escape.
- To confirm that tab order matches the visual order.
- To ensure focus indicators are visible on every focused element.
- To verify that focus is restored after overlays close.
- To test keyboard interactions defined by the WAI-ARIA Authoring Practices.

### Syntax Rules and Structure

**Keyboard Audit Checklist:**
```
1. Unplug the mouse (or hide it).
2. Tab through every page from top to bottom.
3. Verify the tab order matches the visual order.
4. Verify every interactive element receives visible focus.
5. Verify no element traps focus (except modals).
6. Test Enter/Space on buttons and links.
7. Test Arrow keys on tabs, menus, and radio groups.
8. Test Escape on modals, dropdowns, and popovers.
9. Test Home/End on long lists.
10. Verify focus returns to the trigger after overlays close.
```

**Visible Focus Indicator (CSS):**
```css
/* ✅ Good: visible focus ring for keyboard users only */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}

/* ❌ Bad: removes focus indicator entirely */
/* *:focus { outline: none; } */

/* ⚠️ Acceptable: custom focus styles that are clearly visible */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
  box-shadow: 0 0 0 4px rgba(0, 95, 204, 0.3);
}
```

**Component Breakdown:**
- `:focus-visible`: Applies the focus ring only for keyboard navigation.
- `outline: 2px solid #005fcc`: High-contrast visible outline.
- `outline-offset: 2px`: Separates the outline from the element.
- Never use `outline: none` without a replacement.

**Testing Focus Trap in a Modal:**
```jsx
// The modal should trap focus, so Tab cycles within it.
// Test manually: open the modal, press Tab repeatedly,
// verify focus never leaves the modal. Press Escape,
// verify focus returns to the trigger.
```

**Syntax Rules:**
- Test every page and every interactive workflow with the keyboard.
- Use `:focus-visible` for focus rings; do not remove them without a replacement.
- Ensure tab order matches visual order; avoid positive `tabIndex` values.
- Trap focus only in modals; everywhere else, focus must flow naturally.
- Restore focus to the trigger when modals and dropdowns close.
- Test the keyboard interactions specified by the WAI-ARIA APG for each widget type.
- Use browser DevTools to inspect `document.activeElement` if focus is unclear.

**Constraints and Limitations:**
- Manual testing is time-consuming and requires discipline.
- Different browsers have different focus behaviours; test in at least Chrome and Firefox.
- Testing on macOS Safari requires enabling "Press Tab to highlight each item on a webpage" in Safari preferences.
- Keyboard traps can be subtle (e.g., a component that swallows Tab but does not handle Shift+Tab).
- Focus visibility can be obscured by sticky headers or overlapping content (WCAG 2.4.11).

### Annotated Code Example: Keyboard Trap Detection

```jsx
// A component that accidentally traps focus
function BadDropdown({ items }) {
  const [isOpen, setIsOpen] = useState(false);

  function handleKeyDown(e) {
    if (e.key === 'Tab') {
      e.preventDefault(); // ❌ Traps focus — Tab does nothing
    }
  }

  return (
    <div onKeyDown={handleKeyDown}>
      <button onClick={() => setIsOpen((o) => !o)}>Open</button>
      {isOpen && <ul>{items.map((i) => <li key={i.id}>{i.label}</li>)}</ul>}
    </div>
  );
}

// ✅ Fixed: let Tab close the menu and move on
function GoodDropdown({ items }) {
  const [isOpen, setIsOpen] = useState(false);
  const buttonRef = useRef(null);

  function handleKeyDown(e) {
    if (e.key === 'Escape') {
      setIsOpen(false);
      buttonRef.current?.focus();
    } else if (e.key === 'Tab') {
      setIsOpen(false); // Let Tab move naturally, but close the menu
    }
  }

  return (
    <div onKeyDown={handleKeyDown}>
      <button ref={buttonRef} onClick={() => setIsOpen((o) => !o)}>Open</button>
      {isOpen && <ul>{items.map((i) => <li key={i.id}>{i.label}</li>)}</ul>}
    </div>
  );
}
```

**Expected Output:** The bad dropdown traps focus—pressing Tab does nothing. The good dropdown closes on Tab and allows focus to move naturally. Pressing Escape closes the menu and returns focus to the button.

**Why This Output Occurs:** The bad dropdown calls `e.preventDefault()` on Tab, which prevents the browser from moving focus. The good dropdown does not prevent Tab; it closes the menu and lets the browser move focus naturally. Escape closes the menu and restores focus to the trigger.

### Real-World Cases

- **E-commerce checkout:** Tabbing through the entire checkout flow without a mouse.
- **Dashboard navigation:** Navigating between widgets and actions.
- **Modal dialogs:** Verifying focus trap and restoration.
- **Tabs and accordions:** Verifying arrow-key navigation.
- **Form wizards:** Verifying that Back/Next are keyboard accessible.

### References

- WCAG 2.2 - Keyboard (2.1.1) - https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html
- WCAG 2.2 - No Keyboard Trap (2.1.2) - https://www.w3.org/WAI/WCAG22/Understanding/no-keyboard-trap.html
- WCAG 2.2 - Focus Order (2.4.3) - https://www.w3.org/WAI/WCAG22/Understanding/focus-order.html
- WCAG 2.2 - Focus Visible (2.4.7) - https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html
- WCAG 2.2 - Focus Not Obscured (2.4.11) - https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum.html
- W3C WAI-ARIA APG - Keyboard Interface - https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/
- WebAIM - Keyboard Accessibility - https://webaim.org/techniques/keyboard/

---

## Core Concept 3: Assistive Technology Screen Readers

### Definitions

**Core Definition:** Assistive technology screen reader testing is the process of using a real screen reader (NVDA, JAWS, VoiceOver) on a real browser to verify that content is announced correctly, in the right order, and with the right semantics.

**Technical Definition:** Screen readers convert the DOM's accessibility tree into speech and braille output. They announce element roles, names, states, and descriptions as the user navigates. Testing requires pairing a screen reader with a specific browser (NVDA/Firefox on Windows, JAWS/Chrome on Windows, VoiceOver/Safari on macOS and iOS, TalkBack/Chrome on Android) because each combination interprets the accessibility tree differently. The audit verifies: (1) page structure (landmarks and headings), (2) interactive element names and roles, (3) state changes (expanded, selected, checked), (4) dynamic content announcements (`aria-live`), (5) focus order and announcements, and (6) form labels, errors, and help text.

**Beginner-Friendly Explanation:** A screen reader is software that reads the screen aloud for blind and low-vision users. Testing with a screen reader is like testing your app with a user who cannot see it—you hear what they hear. You navigate with keyboard shortcuts (H for headings, K for links, F for form fields), and you listen for the announcements. If the screen reader says "button" with no label, or reads content in a confusing order, or misses an error message, you have found a real bug that automated tools cannot detect.

### Purposes

- To verify that content is announced correctly and in a logical order.
- To test that interactive elements have correct roles, names, and states.
- To verify that dynamic content changes are announced via live regions.
- To test form labels, error messages, and help text.
- To verify that landmarks and headings provide useful navigation.
- To catch issues that automated tools miss (context, order, announcements).

### Syntax Rules and Structure

**Screen Reader and Browser Pairings:**

| Screen Reader | Browser | Platform |
|---|---|---|
| NVDA | Firefox | Windows |
| NVDA | Chrome | Windows |
| JAWS | Chrome | Windows |
| VoiceOver | Safari | macOS |
| VoiceOver | Safari | iOS |
| TalkBack | Chrome | Android |

**Essential Screen Reader Shortcuts:**

| Shortcut | NVDA (Windows) | VoiceOver (macOS) |
|---|---|---|
| Next heading | H | VO + Cmd + H |
| Next landmark | D | VO + U, then arrow |
| Next link | K | VO + Cmd + L |
| Next form field | F | VO + Cmd + J |
| Next button | B | VO + Cmd + B |
| Read from cursor | NVDA + Down | VO + A |
| Stop reading | Ctrl | Ctrl |

**Testing Dynamic Announcements (`aria-live`):**
```jsx
function SaveStatus({ status }) {
  return (
    <div role="status" aria-live="polite" aria-atomic="true">
      {status === 'saving' && 'Saving changes...'}
      {status === 'saved' && 'Changes saved successfully.'}
      {status === 'error' && 'Failed to save. Please try again.'}
    </div>
  );
}
```

**Component Breakdown:**
- `role="status"`: Equivalent to `aria-live="polite"`.
- `aria-atomic="true"`: Announces the entire message, not just the changed node.
- Screen reader announces the message when `status` changes.

**Testing Form Errors:**
```jsx
function EmailField({ error }) {
  const id = useId();
  const errorId = `${id}-error`;

  return (
    <div>
      <label htmlFor={id}>Email</label>
      <input
        id={id}
        type="email"
        aria-invalid={!!error}
        aria-describedby={error ? errorId : undefined}
      />
      {error && (
        <p id={errorId} role="alert">
          {error}
        </p>
      )}
    </div>
  );
}
```

**Component Breakdown:**
- `aria-invalid="true"`: Screen reader announces "invalid entry."
- `aria-describedby`: Links the input to its error message.
- `role="alert"`: Announces the error immediately when it appears.

**Syntax Rules:**
- Test with at least one screen reader and browser combination per platform.
- Navigate using headings, landmarks, links, and form fields—not just arrow keys.
- Verify that every interactive element has a name (not just "button" or "link").
- Verify that state changes (expanded, selected, checked) are announced.
- Verify that dynamic content is announced via `aria-live` regions.
- Verify that form errors are announced and linked to their inputs.
- Test with the screen reader's speech at a comfortable rate; do not rush.
- Record findings with the exact announcement and the expected announcement.

**Constraints and Limitations:**
- Each screen reader/browser pairing behaves differently; test the pairings your users use.
- Screen readers do not announce visual-only information (colour, position, size).
- Testing requires practice; screen reader navigation is a skill.
- Automated tools cannot replace screen reader testing.
- Some screen readers have bugs or quirks that are not your app's fault; document them.

### Annotated Code Example: Screen Reader Test Plan

```jsx
// A component with multiple accessibility features to verify
function ProductCard({ product, onAddToCart }) {
  return (
    <article aria-labelledby={`product-${product.id}-title`}>
      <img src={product.image} alt={product.imageAlt} />
      <h3 id={`product-${product.id}-title`}>{product.name}</h3>
      <p>{product.description}</p>
      <p aria-label={`Price: ${product.price} dollars`}>
        ${product.price}
      </p>
      <button
        onClick={onAddToCart}
        aria-label={`Add ${product.name} to cart`}
      >
        Add to Cart
      </button>
    </article>
  );
}
```

**Expected Screen Reader Announcements:**
- Navigating by headings: "Product Name, heading level 3."
- Navigating to the image: "Product Name, image."
- Navigating to the price: "Price: 29.99 dollars."
- Navigating to the button: "Add Product Name to cart, button."

**Why This Output Occurs:** The `<article>` with `aria-labelledby` creates a labelled region. The `<h3>` provides a heading. The image has descriptive alt text. The price has an `aria-label` for clearer pronunciation. The button has an `aria-label` that includes the product name for context.

### Real-World Cases

- **E-commerce:** Testing the full purchase flow with NVDA + Firefox.
- **SaaS dashboards:** Testing navigation and data tables with VoiceOver + Safari.
- **Government services:** Testing forms and multi-step processes with JAWS + Chrome.
- **Mobile apps:** Testing touch navigation with VoiceOver + Safari on iOS.
- **Documentation:** Testing code examples and navigation with NVDA.

### References

- NVDA Screen Reader - https://www.nvaccess.org/
- JAWS Screen Reader - https://www.freedomscientific.com/products/software/jaws/
- VoiceOver (macOS) - https://www.apple.com/accessibility/vision/
- VoiceOver (iOS) - https://www.apple.com/accessibility/vision/
- TalkBack (Android) - https://support.google.com/accessibility/android/answer/6283677
- WebAIM - Screen Reader User Survey - https://webaim.org/projects/screenreadersurvey/
- W3C WAI - Testing with Screen Readers - https://www.w3.org/WAI/test-evaluate/tools/

---

## Core Concept 4: Visual & Contrast Verification

### Definitions

**Core Definition:** Visual and contrast verification is the process of auditing text and non-text contrast ratios against WCAG AA and AAA requirements, using contrast checkers and colour-blindness simulators.

**Technical Definition:** WCAG 2.2 defines minimum contrast ratios for text and non-text content. **SC 1.4.3 (Contrast Minimum, AA)** requires a contrast ratio of at least 4.5:1 for normal text and 3:1 for large text (18pt or 14pt bold). **SC 1.4.6 (Contrast Enhanced, AAA)** requires 7:1 for normal text and 4.5:1 for large text. **SC 1.4.11 (Non-text Contrast, AA)** requires 3:1 for UI components (borders, icons, form controls) and graphical objects (charts, diagrams). Contrast is calculated using the relative luminance formula defined in WCAG. Colour-blindness simulators (deuteranopia, protanopia, tritanopia) verify that information is not conveyed by colour alone (SC 1.4.1).

**Beginner-Friendly Explanation:** Imagine reading light grey text on a white background—it is hard, and for some people, impossible. Contrast verification checks that text and UI elements have enough contrast to be readable. The WCAG ratio is a number between 1:1 (no contrast) and 21:1 (black on white). Normal text needs 4.5:1; large text needs 3:1. Non-text elements like icons and borders also need 3:1. Colour-blindness simulators check that you are not relying on colour alone (e.g., a red border for errors without a text message).

### Purposes

- To verify that text is readable by users with low vision.
- To verify that UI components and graphical objects have sufficient contrast.
- To confirm that information is not conveyed by colour alone.
- To test against WCAG AA and AAA requirements.
- To catch contrast issues that automated tools may miss (e.g., contrast on hover states).

### Syntax Rules and Structure

**WCAG Contrast Requirements:**

| Content Type | AA Minimum | AAA Minimum |
|---|---|---|
| Normal text (< 18pt) | 4.5:1 | 7:1 |
| Large text (≥ 18pt or 14pt bold) | 3:1 | 4.5:1 |
| UI components (borders, icons) | 3:1 | 3:1 |
| Graphical objects (charts) | 3:1 | 3:1 |

**Contrast Checker Tools:**
- WebAIM Contrast Checker: https://webaim.org/resources/contrastchecker/
- Deque Color Contrast Analyzer: https://www.deque.com/axe/devtools/
- Polypane Contrast Checker: https://polypane.app/color-contrast-checker/
- Chrome DevTools: Inspect element → Styles → colour swatch shows contrast ratio.

**Testing Colour-Blindness:**
- Chrome DevTools → Rendering → Emulate vision deficiencies (Deuteranopia, Protanopia, Tritanopia, Achromatopsia).
- Sim Daltonism (macOS).
- Coblis Color Blindness Simulator: https://www.color-blindness.com/coblis-color-blindness-simulator/

**CSS with Sufficient Contrast:**
```css
/* ✅ Good: dark text on light background */
.text-primary {
  color: #1a1a1a; /* Contrast with #ffffff: 17.4:1 (AAA) */
  background: #ffffff;
}

/* ❌ Bad: low contrast */
.text-muted-bad {
  color: #aaaaaa; /* Contrast with #ffffff: 2.3:1 (fails AA) */
  background: #ffffff;
}

/* ✅ Good: error state with icon and text, not just colour */
.error-message {
  color: #b00020; /* Contrast with #ffffff: 8.3:1 (AAA) */
}
.error-message::before {
  content: '⚠️ ';
}
```

**Component Breakdown:**
- `#1a1a1a` on `#ffffff`: 17.4:1 — passes AAA for all text sizes.
- `#aaaaaa` on `#ffffff`: 2.3:1 — fails AA for normal text.
- `#b00020` on `#ffffff`: 8.3:1 — passes AAA.
- The `::before` icon ensures the error is not conveyed by colour alone.

**Syntax Rules:**
- Check text contrast against its actual background (including gradients and images).
- Check hover, focus, active, and disabled states.
- Check UI component contrast (borders, icons, form controls) against adjacent colours.
- Check graphical object contrast (chart lines, data points) against their background.
- Use colour-blindness simulators to verify that information is not colour-dependent.
- Provide text labels or icons alongside colour indicators (errors, status, required fields).
- Test in both light and dark modes.

**Constraints and Limitations:**
- Automated contrast checkers cannot evaluate text over images or gradients reliably.
- Contrast ratios assume sRGB; wide-gamut colours (P3) may need manual verification.
- The 4.5:1 ratio is a minimum, not a target; higher contrast is better.
- Colour-blindness simulators approximate; real users' experiences vary.
- Disabled states are exempt from WCAG contrast requirements, but should still be distinguishable.

### Annotated Code Example: Accessible Status Indicator

```jsx
function StatusBadge({ status }) {
  const config = {
    success: { label: 'Success', color: '#0f5132', bg: '#d1e7dd', icon: '✓' },
    warning: { label: 'Warning', color: '#664d03', bg: '#fff3cd', icon: '⚠' },
    error: { label: 'Error', color: '#842029', bg: '#f8d7da', icon: '✕' },
  };

  const { label, color, bg, icon } = config[status];

  return (
    <span
      style={{
        color,
        background: bg,
        padding: '4px 8px',
        borderRadius: 4,
        display: 'inline-flex',
        alignItems: 'center',
        gap: 4,
      }}
    >
      <span aria-hidden="true">{icon}</span>
      {label}
    </span>
  );
}
```

**Expected Output:** A status badge with an icon, a text label, and sufficient contrast. The icon is hidden from screen readers (`aria-hidden="true"`) because the text label already communicates the status. Colour-blind users can rely on the icon and text, not just the colour.

**Why This Output Occurs:** Each status has a distinct icon and text label in addition to colour. The colour combinations are chosen for sufficient contrast (dark text on light background). `aria-hidden="true"` on the icon prevents redundant announcements. The badge is understandable without colour perception.

### Real-World Cases

- **E-commerce:** Product prices, sale badges, and availability indicators.
- **Dashboards:** Chart colours, status indicators, and data labels.
- **Forms:** Error states, required field markers, and validation messages.
- **Navigation:** Active link indicators, breadcrumbs, and focus rings.
- **Dark mode:** Verifying contrast in both light and dark themes.

### References

- WCAG 2.2 - Contrast Minimum (1.4.3) - https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html
- WCAG 2.2 - Contrast Enhanced (1.4.6) - https://www.w3.org/WAI/WCAG22/Understanding/contrast-enhanced.html
- WCAG 2.2 - Non-text Contrast (1.4.11) - https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html
- WCAG 2.2 - Use of Color (1.4.1) - https://www.w3.org/WAI/WCAG22/Understanding/use-of-color.html
- WebAIM Contrast Checker - https://webaim.org/resources/contrastchecker/
- Coblis Color Blindness Simulator - https://www.color-blindness.com/coblis-color-blindness-simulator/
- Chrome DevTools - Emulate Vision Deficiencies - https://developer.chrome.com/docs/devtools/accessibility/reference/#emulate-vision-deficiencies

---

## Core Concept 5: Zoom & Reflow Evaluation

### Definitions

**Core Definition:** Zoom and reflow evaluation is the process of testing an application's layout and functionality at up to 400% browser magnification, verifying that content does not overlap, is not cut off, and remains readable and operable.

**Technical Definition:** WCAG 2.2 SC 1.4.4 (Resize Text, AA) requires that text can be resized up to 200% without loss of content or functionality. SC 1.4.10 (Reflow, AA) requires that content can be presented without loss of information or functionality, and without requiring scrolling in two dimensions, at a viewport width of 320 CSS pixels (equivalent to 400% zoom on a 1280px viewport). This means: (1) no horizontal scrolling for content that is not a data table, map, or diagram; (2) text wraps rather than being cut off; (3) interactive elements remain operable; (4) content does not overlap; and (5) no content is clipped or hidden. Testing is done by zooming the browser to 400% (or setting the viewport to 320px) and navigating all key workflows.

**Beginner-Friendly Explanation:** Imagine you have low vision and need to zoom in to 400% to read anything. If your app breaks—text overlaps, buttons disappear, you have to scroll horizontally to read a paragraph—you cannot use it. Zoom and reflow testing checks that your app still works when someone zooms in a lot. The rule is simple: at 400% zoom (or 320px wide), content should stack vertically, text should wrap, and nothing should be cut off or overlapping.

### Purposes

- To verify that text can be enlarged to 200% without loss of content.
- To verify that content reflows at 400% zoom (320px viewport) without horizontal scrolling.
- To ensure that interactive elements remain operable at high zoom.
- To detect content overlap, clipping, and truncation.
- To verify that sticky headers and footers do not obscure focused content.
- To comply with WCAG 2.2 SC 1.4.4 and SC 1.4.10.

### Syntax Rules and Structure

**Testing Procedure:**
```
1. Open the page in a desktop browser (Chrome, Firefox).
2. Set zoom to 400% (Ctrl/Cmd + '+' repeatedly, or use the zoom menu).
   - Alternative: set the viewport to 320px wide in DevTools.
3. Navigate through every key workflow.
4. Verify:
   - No horizontal scrolling (except for data tables, maps, diagrams).
   - Text wraps and remains readable.
   - No content is cut off, clipped, or overlapped.
   - Buttons and links remain clickable.
   - Focus indicators remain visible.
   - Sticky headers/footers do not obscure focused content.
5. Test in both portrait and landscape orientations.
```

**CSS That Reflows Well:**
```css
/* ✅ Good: fluid layout with max-width and wrapping */
.container {
  max-width: 1200px;
  width: 100%;
  margin: 0 auto;
  padding: 0 16px;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 24px;
}

/* ✅ Good: text wraps naturally */
.text {
  word-wrap: break-word;
  overflow-wrap: break-word;
}

/* ❌ Bad: fixed width causes horizontal scroll at high zoom */
.fixed-width {
  width: 1200px;
}

/* ❌ Bad: fixed height causes content clipping */
.fixed-height {
  height: 200px;
  overflow: hidden;
}
```

**Component Breakdown:**
- `max-width: 1200px; width: 100%`: Fluid container that shrinks at high zoom.
- `auto-fit` + `minmax(280px, 1fr)`: Grid columns collapse to one column when space is tight.
- `word-wrap: break-word`: Long words break rather than overflowing.
- `width: 1200px`: Fixed width forces horizontal scrolling at 400% zoom.
- `height: 200px; overflow: hidden`: Fixed height clips content.

**Sticky Header That Respects Zoom:**
```css
.sticky-header {
  position: sticky;
  top: 0;
  z-index: 100;
}

/* Ensure focused elements scroll into view below the sticky header */
:target,
:focus {
  scroll-margin-top: 80px; /* Height of the sticky header */
}
```

**Component Breakdown:**
- `scroll-margin-top`: Adds space above the target element when scrolled into view.
- Prevents the sticky header from obscuring focused or targeted content (WCAG 2.4.11).

**Syntax Rules:**
- Use relative units (`rem`, `em`, `%`, `vw`) instead of fixed `px` where possible.
- Use `max-width` and `min-width` instead of fixed `width`.
- Avoid fixed `height` on content containers; let content determine height.
- Use `flex-wrap: wrap` and `grid-template-columns: repeat(auto-fit, minmax(...))`.
- Use `overflow-wrap: break-word` for long words and URLs.
- Use `scroll-margin-top` to prevent sticky headers from obscuring focused content.
- Test at 400% zoom on a 1280px viewport (equivalent to 320px) and at 200% text-only zoom.

**Constraints and Limitations:**
- Data tables, maps, and diagrams are exempt from the reflow requirement (they may scroll horizontally).
- Text-only zoom (browser setting) differs from full-page zoom; test both.
- High zoom can trigger different CSS media queries; verify the layout at each breakpoint.
- Mobile browsers handle zoom differently; test on real devices.
- Content that relies on hover (tooltips, dropdowns) may be inaccessible at high zoom.

### Annotated Code Example: Reflow-Friendly Layout

```jsx
function ProductPage({ product }) {
  return (
    <div style={{ maxWidth: 1200, margin: '0 auto', padding: '0 16px' }}>
      <div
        style={{
          display: 'grid',
          gridTemplateColumns: 'repeat(auto-fit, minmax(280px, 1fr))',
          gap: 24,
        }}
      >
        <img
          src={product.image}
          alt={product.imageAlt}
          style={{ width: '100%', height: 'auto', borderRadius: 8 }}
        />
        <div>
          <h1 style={{ fontSize: 'clamp(1.5rem, 4vw, 2rem)' }}>
            {product.name}
          </h1>
          <p style={{ fontSize: 'clamp(1rem, 2vw, 1.125rem)' }}>
            {product.description}
          </p>
          <p style={{ fontSize: '1.25rem', fontWeight: 700 }}>
            ${product.price}
          </p>
          <button style={{ padding: '12px 24px', fontSize: '1rem' }}>
            Add to Cart
          </button>
        </div>
      </div>
    </div>
  );
}
```

**Expected Output:** At normal zoom, the image and text appear side by side. At 400% zoom (320px viewport), the grid collapses to a single column: image on top, text below. The heading uses `clamp()` to scale fluidly. The button remains fully visible and clickable. No horizontal scrolling is required.

**Why This Output Occurs:** `repeat(auto-fit, minmax(280px, 1fr))` creates a responsive grid that collapses to one column when the container is narrower than 280px. `maxWidth: 1200` with `width: 100%` (implied by the block element) keeps the container fluid. `clamp()` scales font sizes between a minimum and maximum. `height: 'auto'` on the image preserves its aspect ratio. No fixed widths or heights cause overflow.

### Real-World Cases

- **E-commerce:** Product pages, checkout flows, and cart views at 400% zoom.
- **Dashboards:** Data tables (which may scroll horizontally, but controls must remain usable).
- **Documentation:** Long-form articles and code examples at high zoom.
- **Forms:** Multi-field forms that must stack vertically at high zoom.
- **Navigation:** Menus and sidebars that collapse gracefully at high zoom.

### References

- WCAG 2.2 - Resize Text (1.4.4) - https://www.w3.org/WAI/WCAG22/Understanding/resize-text.html
- WCAG 2.2 - Reflow (1.4.10) - https://www.w3.org/WAI/WCAG22/Understanding/reflow.html
- WCAG 2.2 - Focus Not Obscured (2.4.11) - https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum.html
- WebAIM - Zoom and Reflow - https://webaim.org/articles/zoom/
- W3C WAI - Text Resizing and Zoom - https://www.w3.org/WAI/perspective-videos/resize/

---

## Core Concept 6: User-Driven Preferences

### Definitions

**Core Definition:** User-driven preferences are the operating-system-level settings that users choose—reduced motion, dark mode, high contrast, reduced transparency—that web applications should respect via CSS media queries (`prefers-reduced-motion`, `prefers-color-scheme`, `prefers-contrast`).

**Technical Definition:** CSS media queries expose user preferences from the operating system. `prefers-reduced-motion` (WCAG 2.3.3) indicates that the user wants minimal animation; it can be `no-preference` or `reduce`. `prefers-color-scheme` indicates light or dark mode preference. `prefers-contrast` indicates increased or reduced contrast preference. `forced-colors` indicates Windows High Contrast Mode. `prefers-reduced-transparency` indicates reduced transparency. Applications must respect these preferences to avoid harming users with vestibular disorders (motion), low vision (contrast), or photosensitivity. Testing verifies that the application adapts correctly and that disabling motion does not break functionality.

**Beginner-Friendly Explanation:** Your phone or computer has settings for motion, colour, and contrast. If you get motion sickness from animations, you can turn on "Reduce Motion." If you prefer dark mode, you set that. If you need high contrast, you set that too. Your website should respect these settings—not force animations, not force light mode, not ignore high contrast. Testing means changing those settings and verifying that your app adapts correctly.

### Purposes

- To respect the user's motion preferences and prevent vestibular harm.
- To respect light/dark mode preferences.
- To respect high-contrast and forced-colors preferences.
- To verify that disabling motion does not break functionality.
- To verify that the app is usable in Windows High Contrast Mode.
- To comply with WCAG 2.2 SC 1.4.12, SC 1.4.13, and SC 2.3.3.

### Syntax Rules and Structure

**Reduced Motion (`prefers-reduced-motion`):**
```css
/* Default: animations enabled */
.fade-in {
  animation: fadeIn 300ms ease-in-out;
}

/* Respect user preference: no animations */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

**Component Breakdown:**
- `@media (prefers-reduced-motion: reduce)`: Applies when the user has enabled reduced motion.
- `animation-duration: 0.01ms`: Effectively disables animations without removing them (preserving `animationend` events).
- `scroll-behavior: auto`: Disables smooth scrolling.
- The universal selector ensures all animations and transitions are covered.

**React Hook for Reduced Motion:**
```jsx
function usePrefersReducedMotion() {
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(
    () => window.matchMedia('(prefers-reduced-motion: reduce)').matches
  );

  useEffect(() => {
    const query = window.matchMedia('(prefers-reduced-motion: reduce)');
    const handleChange = (e) => setPrefersReducedMotion(e.matches);
    query.addEventListener('change', handleChange);
    return () => query.removeEventListener('change', handleChange);
  }, []);

  return prefersReducedMotion;
}

function AnimatedComponent() {
  const prefersReducedMotion = usePrefersReducedMotion();

  return (
    <div
      style={{
        transition: prefersReducedMotion ? 'none' : 'transform 300ms ease',
        transform: 'scale(1.1)',
      }}
    >
      Content
    </div>
  );
}
```

**Component Breakdown:**
- `window.matchMedia('(prefers-reduced-motion: reduce)')`: Reads the preference.
- `query.addEventListener('change', ...)`: Listens for changes.
- `prefersReducedMotion ? 'none' : 'transform 300ms ease'`: Conditionally applies the transition.

**Dark Mode (`prefers-color-scheme`):**
```css
:root {
  --bg: #ffffff;
  --text: #1a1a1a;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg: #1a1a1a;
    --text: #f5f5f5;
  }
}

body {
  background: var(--bg);
  color: var(--text);
}
```

**High Contrast (`prefers-contrast`):**
```css
@media (prefers-contrast: more) {
  :root {
    --border-color: #000000;
    --text-secondary: #333333; /* Darker than usual */
  }
}
```

**Forced Colors (`forced-colors`):**
```css
@media (forced-colors: active) {
  .custom-button {
    border: 1px solid ButtonText;
    color: ButtonText;
    background: ButtonFace;
  }
}
```

**Syntax Rules:**
- Always respect `prefers-reduced-motion: reduce`; disable or reduce animations.
- Use `prefers-color-scheme` for light/dark mode; support both.
- Use `prefers-contrast: more` to enhance contrast when requested.
- Use `forced-colors: active` to support Windows High Contrast Mode.
- Test with the OS setting enabled, not just the browser DevTools emulation.
- Ensure that disabling motion does not hide content or break functionality.
- Provide a manual toggle for users who want to override the OS setting.

**Constraints and Limitations:**
- `prefers-reduced-motion` applies to CSS animations and transitions, but not to JavaScript-driven animations (e.g., `requestAnimationFrame`, GSAP). These must be handled separately.
- `prefers-color-scheme` does not indicate the user's preference for your app specifically; provide a manual toggle.
- `forced-colors` overrides author colours; test with Windows High Contrast Mode.
- DevTools emulation is convenient but may not match the real OS behaviour.
- Some users enable reduced motion but still expect some feedback (e.g., a subtle fade); test with real users.

### Annotated Code Example: Preference-Aware Modal

```jsx
function AccessibleModal({ isOpen, onClose, children }) {
  const prefersReducedMotion = usePrefersReducedMotion();

  useEffect(() => {
    if (!isOpen) return;
    function handleKey(e) {
      if (e.key === 'Escape') onClose();
    }
    document.addEventListener('keydown', handleKey);
    return () => document.removeEventListener('keydown', handleKey);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div
      style={{
        position: 'fixed',
        inset: 0,
        background: 'rgba(0, 0, 0, 0.5)',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        animation: prefersReducedMotion ? 'none' : 'fadeIn 200ms ease',
      }}
    >
      <div
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        style={{
          background: 'var(--surface)',
          padding: 24,
          borderRadius: 12,
          maxWidth: 400,
          animation: prefersReducedMotion ? 'none' : 'slideUp 200ms ease',
        }}
      >
        <h2 id="modal-title">Modal Title</h2>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  );
}
```

**Expected Output:** The modal fades in and slides up when opened. If the user has enabled "Reduce Motion" in their OS, the animations are disabled and the modal appears instantly. All functionality (Escape to close, focus management) remains intact.

**Why This Output Occurs:** The `usePrefersReducedMotion` hook reads the OS preference and updates when it changes. The `animation: prefersReducedMotion ? 'none' : 'fadeIn 200ms ease'` conditionally applies the animation. The modal's functionality does not depend on the animation, so disabling it does not break anything.

### Real-World Cases

- **Marketing sites:** Respect reduced motion for hero animations and scroll effects.
- **Dashboards:** Respect dark mode for chart colours and backgrounds.
- **E-commerce:** Respect reduced motion for product carousels and image zooms.
- **SaaS applications:** Respect high contrast for users with low vision.
- **Any application:** Support Windows High Contrast Mode for government and enterprise users.

### References

- WCAG 2.2 - Animation from Interactions (2.3.3) - https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html
- WCAG 2.2 - Content on Hover or Focus (1.4.13) - https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus.html
- MDN Web Docs - prefers-reduced-motion - https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- MDN Web Docs - prefers-color-scheme - https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme
- MDN Web Docs - prefers-contrast - https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-contrast
- MDN Web Docs - forced-colors - https://developer.mozilla.org/en-US/docs/Web/CSS/@media/forced-colors
- WebAIM - Reduced Motion - https://webaim.org/articles/motion/
- Microsoft Edge - Forced Colors Mode - https://learn.microsoft.com/en-us/microsoft-edge/devtools-guide-chromium/accessibility/emulate-forced-colors

---

## Comparison and Decision Guidance

| Testing Layer | Tool | Catches | Misses | When to Run |
|---|---|---|---|---|
| **Static analysis** | `eslint-plugin-jsx-a11y` | Missing alts, invalid ARIA, non-interactive handlers | Runtime issues, contrast | Editor + CI |
| **Unit testing** | `jest-axe`, Testing Library | ARIA, roles, names, focus | Contrast in JSDOM, screen reader output | CI |
| **CI scanner** | Lighthouse CI, Pa11y CI | Full-page issues, regressions | Dynamic content, complex interactions | Every PR |
| **Keyboard audit** | Manual | Traps, tab order, focus visibility | Screen reader announcements | Before release |
| **Screen reader** | NVDA, JAWS, VoiceOver | Announcements, order, context | Visual issues, contrast | Before release |
| **Contrast** | WebAIM, DevTools | Text/UI contrast ratios | Colour-blindness context | Design + review |
| **Zoom/reflow** | Browser zoom, DevTools | Layout breaks, overflow, overlap | Screen reader behaviour | Before release |
| **Preferences** | OS settings, DevTools | Motion, dark mode, high contrast | JavaScript animations | Before release |

**Decision Guidance:**
- **Automate what you can:** ESLint, `jest-axe`, and Lighthouse CI catch the majority of common issues.
- **Manual keyboard audit is mandatory** before release; it catches traps and order issues that tools miss.
- **Screen reader testing is mandatory** for critical flows; no tool replaces it.
- **Contrast checking is mandatory** for all text and UI components; automated tools help but manual review catches gradients and images.
- **Zoom and reflow testing is mandatory** for WCAG 2.2 AA compliance.
- **Respect user preferences** in CSS and JavaScript; test with real OS settings.
- **Combine layers:** No single tool or technique is sufficient; accessibility requires a multi-layered approach.

---

## References

- W3C WAI-ARIA Authoring Practices Guide - https://www.w3.org/WAI/ARIA/apg/
- WCAG 2.2 Quick Reference - https://www.w3.org/WAI/WCAG22/quickref/
- W3C WAI - Easy Checks - https://www.w3.org/WAI/test-evaluate/easy-checks/
- W3C WAI - Testing with Screen Readers - https://www.w3.org/WAI/test-evaluate/tools/
- eslint-plugin-jsx-a11y - https://github.com/jsx-eslint/eslint-plugin-jsx-a11y
- jest-axe - https://github.com/nickcolley/jest-axe
- axe-core - https://github.com/dequelabs/axe-core
- Testing Library - https://testing-library.com/docs/
- Testing Library Jest DOM - https://github.com/testing-library/jest-dom
- Lighthouse CI - https://github.com/GoogleChrome/lighthouse-ci
- Pa11y CI - https://github.com/pa11y/pa11y-ci
- WebAIM Contrast Checker - https://webaim.org/resources/contrastchecker/
- Coblis Color Blindness Simulator - https://www.color-blindness.com/coblis-color-blindness-simulator/
- NVDA Screen Reader - https://www.nvaccess.org/
- JAWS Screen Reader - https://www.freedomscientific.com/products/software/jaws/
- VoiceOver (macOS) - https://www.apple.com/accessibility/vision/
- TalkBack (Android) - https://support.google.com/accessibility/android/answer/6283677
- MDN Web Docs - prefers-reduced-motion - https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- MDN Web Docs - prefers-color-scheme - https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme
- MDN Web Docs - prefers-contrast - https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-contrast
- MDN Web Docs - forced-colors - https://developer.mozilla.org/en-US/docs/Web/CSS/@media/forced-colors
- WebAIM - Keyboard Accessibility - https://webaim.org/techniques/keyboard/
- WebAIM - Zoom and Reflow - https://webaim.org/articles/zoom/
- WebAIM - Reduced Motion - https://webaim.org/articles/motion/
- Deque axe DevTools - https://www.deque.com/axe/devtools/
- Microsoft Edge - Forced Colors Mode - https://learn.microsoft.com/en-us/microsoft-edge/devtools-guide-chromium/accessibility/emulate-forced-colors