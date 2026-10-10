# CSS Comprehensive, Structured, and Progressive Learning Roadmap

## From Styling Foundations to Advanced Layout Systems, Responsive Design, Animation, and Production CSS Architecture

CSS is best learned as more than "making things look pretty." The progression should cover **syntax → selectors → cascade → specificity → inheritance → box model → colors → typography → backgrounds → borders → positioning → flexbox → grid → responsive design → transitions → animations → transforms → custom properties → preprocessors → methodologies → architecture → performance → production engineering**.

---

# I. CSS Foundations

- **1. What CSS Is**
  - CSS
  - Cascading Style Sheets
  - CSS history
  - Håkon Wium Lie
  - CSS1
  - CSS2
  - CSS2.1
  - CSS3
  - CSS4 (modular)
  - CSS philosophy
    - Separation of concerns
    - Separation of content and presentation
    - Cascading
    - Inheritance
    - Specificity
  - CSS vs HTML
  - CSS vs JavaScript
  - CSS vs Sass
  - CSS vs Tailwind
  - CSS vs CSS-in-JS
  - CSS use cases
    - Web design
    - Web applications
    - Responsive design
    - Animations
    - Theming
    - Print styles
    - Accessibility
  - CSS in modern web development
  - CSS ecosystem
  - CSS specifications
  - W3C
  - CSS Working Group
  - CSS modules

- **2. How CSS Works**
  - CSS syntax
  - Rulesets
  - Selectors
  - Declarations
  - Properties
  - Values
  - At-rules
  - Comments
  - Whitespace
  - Case sensitivity
  - Parsing
  - Rendering
  - Rendering pipeline
  - Critical rendering path
  - CSSOM
  - CSS and DOM
  - CSS and accessibility
  - CSS best practices

- **3. Adding CSS to HTML**
  - External stylesheets
    - `<link rel="stylesheet">`
  - Internal stylesheets
    - `<style>`
  - Inline styles
    - `style` attribute
  - `@import`
  - Preload
  - Prefetch
  - Media queries
  - Conditional loading
  - CSS delivery best practices
  - CSS performance
  - Render-blocking CSS
  - Critical CSS
  - CSS best practices

- **4. CSS Syntax**
  - Rulesets
  - Selectors
  - Declaration blocks
  - Declarations
  - Properties
  - Values
  - Semicolons
  - Braces
  - Comments
    - `/* */`
  - At-rules
    - `@media`
    - `@supports`
    - `@import`
    - `@font-face`
    - `@keyframes`
    - `@layer`
    - `@container`
    - `@scope`
    - `@property`
  - Vendor prefixes
  - CSS syntax best practices

- **5. CSS Units**
  - Absolute units
    - `px`
    - `cm`
    - `mm`
    - `in`
    - `pt`
    - `pc`
    - `Q`
  - Relative units
    - `em`
    - `rem`
    - `ex`
    - `ch`
    - `cap`
    - `ic`
    - `lh`
    - `rlh`
    - `vw`
    - `vh`
    - `vmin`
    - `vmax`
    - `vb`
    - `vi`
    - `svw`
    - `svh`
    - `lvw`
    - `lvh`
    - `dvw`
    - `dvh`
    - `cqw`
    - `cqh`
    - `cqi`
    - `cqb`
    - `cqmin`
    - `cqmax`
  - Unit conversion
  - Unit selection
  - Unit best practices

- **6. CSS Colors**
  - Color keywords
    - Named colors
    - `transparent`
    - `currentColor`
  - Hex colors
    - `#fff`
    - `#ffffff`
    - `#ffffffff`
  - RGB
    - `rgb()`
    - `rgba()`
  - HSL
    - `hsl()`
    - `hsla()`
  - HWB
    - `hwb()`
  - LAB
    - `lab()`
  - LCH
    - `lch()`
  - OKLAB
    - `oklab()`
  - OKLCH
    - `oklch()`
  - `color()`
  - `color-mix()`
  - `light-dark()`
  - Relative colors
  - Color functions
  - Color spaces
  - Color gamut
  - Color contrast
  - Color accessibility
  - Color best practices

- **7. CSS Values**
  - Length values
  - Percentage values
  - Number values
  - Angle values
  - Time values
  - Frequency values
  - Resolution values
  - String values
  - URL values
  - Function values
  - `calc()`
  - `min()`
  - `max()`
  - `clamp()`
  - `round()`
  - `mod()`
  - `rem()`
  - `sin()`
  - `cos()`
  - `tan()`
  - `asin()`
  - `acos()`
  - `atan()`
  - `atan2()`
  - `pow()`
  - `sqrt()`
  - `hypot()`
  - `log()`
  - `exp()`
  - `abs()`
  - `sign()`
  - Value best practices

---

# II. Selectors

- **8. Selector Fundamentals**
  - Selectors
  - Selector types
  - Selector lists
  - Selector specificity
  - Selector matching
  - Selector performance
  - Selector best practices

- **9. Basic Selectors**
  - Universal selector
    - `*`
  - Type selectors
    - `div`
    - `p`
    - `a`
  - Class selectors
    - `.class`
  - ID selectors
    - `#id`
  - Attribute selectors
    - `[attr]`
    - `[attr="value"]`
    - `[attr~="value"]`
    - `[attr|="value"]`
    - `[attr^="value"]`
    - `[attr$="value"]`
    - `[attr*="value"]`
    - `[attr="value" i]`
    - `[attr="value" s]`
  - Basic selector best practices

- **10. Combinators**
  - Descendant combinator
    - ` `
  - Child combinator
    - `>`
  - Adjacent sibling combinator
    - `+`
  - General sibling combinator
    - `~`
  - Column combinator
    - `||`
  - Combinator best practices

- **11. Pseudo-Classes**
  - User action pseudo-classes
    - `:hover`
    - `:active`
    - `:focus`
    - `:focus-within`
    - `:focus-visible`
  - Location pseudo-classes
    - `:link`
    - `:visited`
    - `:any-link`
    - `:target`
    - `:target-within`
    - `:scope`
  - Tree-structural pseudo-classes
    - `:root`
    - `:empty`
    - `:first-child`
    - `:last-child`
    - `:only-child`
    - `:first-of-type`
    - `:last-of-type`
    - `:only-of-type`
    - `:nth-child()`
    - `:nth-last-child()`
    - `:nth-of-type()`
    - `:nth-last-of-type()`
  - Logical pseudo-classes
    - `:not()`
    - `:is()`
    - `:where()`
    - `:has()`
  - Input pseudo-classes
    - `:enabled`
    - `:disabled`
    - `:checked`
    - `:indeterminate`
    - `:default`
    - `:valid`
    - `:invalid`
    - `:in-range`
    - `:out-of-range`
    - `:required`
    - `:optional`
    - `:read-only`
    - `:read-write`
    - `:placeholder-shown`
    - `:autofill`
    - `:user-invalid`
    - `:user-valid`
  - Linguistic pseudo-classes
    - `:lang()`
    - `:dir()`
  - Other pseudo-classes
    - `:fullscreen`
    - `:modal`
    - `:picture-in-picture`
    - `:defined`
    - `:host`
    - `:host()`
    - `:host-context()`
    - `:playing`
    - `:paused`
    - `:seeking`
    - `:buffering`
    - `:stalled`
    - `:muted`
    - `:volume-locked`
  - Pseudo-class best practices

- **12. Pseudo-Elements**
  - `::before`
  - `::after`
  - `::first-line`
  - `::first-letter`
  - `::selection`
  - `::target-text`
  - `::spelling-error`
  - `::grammar-error`
  - `::marker`
  - `::placeholder`
  - `::file-selector-button`
  - `::backdrop`
  - `::part()`
  - `::slotted()`
  - `::cue`
  - `::cue-region`
  - Pseudo-element best practices

- **13. Selector Specificity**
  - Specificity
  - Specificity calculation
    - Inline styles
    - IDs
    - Classes
    - Attributes
    - Pseudo-classes
    - Elements
    - Pseudo-elements
  - Specificity rules
  - `!important`
  - Specificity best practices
  - Specificity pitfalls
  - Specificity management

- **14. Advanced Selectors**
  - `:has()`
  - `:is()`
  - `:where()`
  - `:not()`
  - `:nth-child()` with `of`
  - Selector nesting
  - Nesting best practices
  - Selector performance
  - Selector best practices

---

# III. Cascade, Inheritance, and Specificity

- **15. Cascade**
  - Cascade
  - Cascade origins
    - User agent
    - User
    - Author
  - Cascade layers
  - Cascade order
  - Cascade best practices

- **16. Inheritance**
  - Inheritance
  - Inherited properties
  - Non-inherited properties
  - `inherit`
  - `initial`
  - `unset`
  - `revert`
  - `revert-layer`
  - Inheritance best practices

- **17. Specificity**
  - Specificity
  - Specificity calculation
  - Specificity rules
  - Specificity best practices
  - Specificity management

- **18. `!important`**
  - `!important`
  - `!important` usage
  - `!important` pitfalls
  - `!important` best practices

- **19. Cascade Layers**
  - `@layer`
  - Layer declaration
  - Layer order
  - Layer nesting
  - Layer best practices

- **20. CSS Reset**
  - CSS reset
  - Normalize.css
  - Reset.css
  - Modern reset
  - Reset best practices
  - Reset vs normalize

- **21. CSS Custom Properties**
  - Custom properties
  - `--variable`
  - `var()`
  - Fallback values
  - Custom property scope
  - Custom property inheritance
  - `@property`
  - Custom property best practices

---

# IV. Box Model

- **22. Box Model Fundamentals**
  - Box model
  - Content box
  - Padding box
  - Border box
  - Margin box
  - Box model diagram
  - Box model best practices

- **23. Width and Height**
  - `width`
  - `height`
  - `min-width`
  - `max-width`
  - `min-height`
  - `max-height`
  - `box-sizing`
    - `content-box`
    - `border-box`
  - `aspect-ratio`
  - Width and height best practices

- **24. Padding**
  - `padding`
  - `padding-top`
  - `padding-right`
  - `padding-bottom`
  - `padding-left`
  - `padding-block`
  - `padding-inline`
  - `padding-block-start`
  - `padding-block-end`
  - `padding-inline-start`
  - `padding-inline-end`
  - Padding best practices

- **25. Margin**
  - `margin`
  - `margin-top`
  - `margin-right`
  - `margin-bottom`
  - `margin-left`
  - `margin-block`
  - `margin-inline`
  - `margin-block-start`
  - `margin-block-end`
  - `margin-inline-start`
  - `margin-inline-end`
  - Margin collapsing
  - Negative margins
  - Margin best practices

- **26. Border**
  - `border`
  - `border-width`
  - `border-style`
  - `border-color`
  - `border-top`
  - `border-right`
  - `border-bottom`
  - `border-left`
  - `border-block`
  - `border-inline`
  - `border-radius`
  - `border-image`
  - Border best practices

- **27. Overflow**
  - `overflow`
  - `overflow-x`
  - `overflow-y`
  - `overflow-wrap`
  - `overflow-clip-margin`
  - `text-overflow`
  - Overflow best practices

- **28. Display**
  - `display`
    - `block`
    - `inline`
    - `inline-block`
    - `none`
    - `flex`
    - `grid`
    - `inline-flex`
    - `inline-grid`
    - `table`
    - `table-row`
    - `table-cell`
    - `contents`
    - `flow-root`
    - `list-item`
    - `run-in`
  - Display best practices

- **29. Visibility**
  - `visibility`
    - `visible`
    - `hidden`
    - `collapse`
  - `opacity`
  - `display: none` vs `visibility: hidden` vs `opacity: 0`
  - Visibility best practices

- **30. Box Shadow**
  - `box-shadow`
  - Inset shadows
  - Multiple shadows
  - Shadow best practices

- **31. Outline**
  - `outline`
  - `outline-width`
  - `outline-style`
  - `outline-color`
  - `outline-offset`
  - Outline vs border
  - Outline best practices
  - Focus indicators

---

# V. Typography

- **32. Typography Fundamentals**
  - Typography
  - Font families
  - Font stacks
  - Font sizes
  - Font weights
  - Font styles
  - Line heights
  - Letter spacing
  - Word spacing
  - Text alignment
  - Text decoration
  - Text transform
  - Typography best practices

- **33. Font Properties**
  - `font-family`
  - `font-size`
  - `font-weight`
  - `font-style`
  - `font-variant`
  - `font-stretch`
  - `font-size-adjust`
  - `font-kerning`
  - `font-feature-settings`
  - `font-variation-settings`
  - `font-optical-sizing`
  - `font-synthesis`
  - `font-display`
  - Font property best practices

- **34. Text Properties**
  - `color`
  - `line-height`
  - `letter-spacing`
  - `word-spacing`
  - `text-align`
  - `text-align-last`
  - `text-indent`
  - `text-decoration`
  - `text-decoration-line`
  - `text-decoration-style`
  - `text-decoration-color`
  - `text-decoration-thickness`
  - `text-underline-offset`
  - `text-underline-position`
  - `text-transform`
  - `text-shadow`
  - `text-overflow`
  - `text-wrap`
  - `text-wrap-mode`
  - `text-wrap-style`
  - `text-emphasis`
  - `text-emphasis-style`
  - `text-emphasis-color`
  - `text-emphasis-position`
  - `text-orientation`
  - `text-combine-upright`
  - `text-rendering`
  - `text-size-adjust`
  - Text property best practices

- **35. Web Fonts**
  - `@font-face`
  - Font formats
    - WOFF
    - WOFF2
    - TTF
    - OTF
    - EOT
    - SVG
  - Font loading
  - Font display
  - Font preloading
  - Font subsetting
  - Font optimization
  - Variable fonts
  - Web font best practices

- **36. Text Layout**
  - Writing modes
  - `writing-mode`
  - `direction`
  - `unicode-bidi`
  - Text orientation
  - Vertical text
  - RTL support
  - Text layout best practices

- **37. Lists**
  - `list-style`
  - `list-style-type`
  - `list-style-position`
  - `list-style-image`
  - `marker`
  - `::marker`
  - List best practices

- **38. Links**
  - Link styling
  - Link states
    - `:link`
    - `:visited`
    - `:hover`
    - `:active`
    - `:focus`
  - Link best practices

---

# VI. Backgrounds and Borders

- **39. Backgrounds**
  - `background`
  - `background-color`
  - `background-image`
  - `background-repeat`
  - `background-position`
  - `background-size`
  - `background-attachment`
  - `background-clip`
  - `background-origin`
  - `background-blend-mode`
  - Multiple backgrounds
  - Gradient backgrounds
  - Background best practices

- **40. Gradients**
  - Linear gradients
    - `linear-gradient()`
  - Radial gradients
    - `radial-gradient()`
  - Conic gradients
    - `conic-gradient()`
  - Repeating gradients
  - Gradient color stops
  - Gradient interpolation
  - Gradient best practices

- **41. Borders**
  - `border`
  - `border-radius`
  - `border-image`
  - `border-image-source`
  - `border-image-slice`
  - `border-image-width`
  - `border-image-outset`
  - `border-image-repeat`
  - Border best practices

- **42. Border Radius**
  - `border-radius`
  - Individual corners
  - Elliptical corners
  - Border radius best practices

- **43. Box Shadow**
  - `box-shadow`
  - Inset shadows
  - Multiple shadows
  - Shadow best practices

- **44. Filters and Effects**
  - `filter`
  - `blur()`
  - `brightness()`
  - `contrast()`
  - `drop-shadow()`
  - `grayscale()`
  - `hue-rotate()`
  - `invert()`
  - `opacity()`
  - `saturate()`
  - `sepia()`
  - `backdrop-filter`
  - `mix-blend-mode`
  - `isolation`
  - Filter best practices

---

# VII. Layout

- **45. Layout Fundamentals**
  - Layout
  - Normal flow
  - Block formatting context
  - Inline formatting context
  - Layout modes
  - Layout best practices

- **46. Positioning**
  - `position`
    - `static`
    - `relative`
    - `absolute`
    - `fixed`
    - `sticky`
  - `top`
  - `right`
  - `bottom`
  - `left`
  - `inset`
  - `inset-block`
  - `inset-inline`
  - `z-index`
  - Stacking context
  - Positioning best practices
  - Positioning pitfalls

- **47. Floating**
  - `float`
  - `clear`
  - `clearfix`
  - Float best practices
  - Float pitfalls
  - Float alternatives

- **48. Flexbox**
  - Flexbox
  - `display: flex`
  - Flex container
  - Flex items
  - Flex direction
    - `flex-direction`
  - Flex wrap
    - `flex-wrap`
  - Flex flow
    - `flex-flow`
  - Justify content
    - `justify-content`
  - Align items
    - `align-items`
  - Align content
    - `align-content`
  - Align self
    - `align-self`
  - Flex grow
    - `flex-grow`
  - Flex shrink
    - `flex-shrink`
  - Flex basis
    - `flex-basis`
  - Flex
    - `flex`
  - Gap
    - `gap`
    - `row-gap`
    - `column-gap`
  - Order
    - `order`
  - Flexbox best practices
  - Flexbox patterns

- **49. Grid**
  - CSS Grid
  - `display: grid`
  - Grid container
  - Grid items
  - Grid template columns
    - `grid-template-columns`
  - Grid template rows
    - `grid-template-rows`
  - Grid template areas
    - `grid-template-areas`
  - Grid template
    - `grid-template`
  - Grid auto columns
    - `grid-auto-columns`
  - Grid auto rows
    - `grid-auto-rows`
  - Grid auto flow
    - `grid-auto-flow`
  - Grid column
    - `grid-column`
  - Grid row
    - `grid-row`
  - Grid area
    - `grid-area`
  - Gap
    - `gap`
  - Justify items
    - `justify-items`
  - Align items
    - `align-items`
  - Justify content
    - `justify-content`
  - Align content
    - `align-content`
  - Justify self
    - `justify-self`
  - Align self
    - `align-self`
  - Grid functions
    - `repeat()`
    - `minmax()`
    - `fit-content()`
  - Grid units
    - `fr`
  - Named grid lines
  - Named grid areas
  - Subgrid
  - Grid best practices
  - Grid patterns

- **50. Multi-Column Layout**
  - `columns`
  - `column-count`
  - `column-width`
  - `column-gap`
  - `column-rule`
  - `column-span`
  - `column-fill`
  - `break-inside`
  - Multi-column best practices

- **51. Tables**
  - Table layout
  - `table-layout`
  - `border-collapse`
  - `border-spacing`
  - `caption-side`
  - `empty-cells`
  - Table best practices

- **52. Container Queries**
  - Container queries
  - `container-type`
  - `container-name`
  - `@container`
  - Container query units
  - Container query best practices

- **53. Logical Properties**
  - Logical properties
  - `margin-block`
  - `margin-inline`
  - `padding-block`
  - `padding-inline`
  - `border-block`
  - `border-inline`
  - `inset-block`
  - `inset-inline`
  - `block-size`
  - `inline-size`
  - `min-block-size`
  - `min-inline-size`
  - `max-block-size`
  - `max-inline-size`
  - Logical property best practices

---

# VIII. Responsive Design

- **54. Responsive Design Fundamentals**
  - Responsive design
  - Adaptive design
  - Fluid design
  - Mobile-first
  - Desktop-first
  - Breakpoints
  - Responsive best practices

- **55. Media Queries**
  - `@media`
  - Media types
    - `all`
    - `screen`
    - `print`
    - `speech`
  - Media features
    - `width`
    - `min-width`
    - `max-width`
    - `height`
    - `min-height`
    - `max-height`
    - `orientation`
    - `aspect-ratio`
    - `resolution`
    - `hover`
    - `any-hover`
    - `pointer`
    - `any-pointer`
    - `prefers-color-scheme`
    - `prefers-reduced-motion`
    - `prefers-contrast`
    - `prefers-reduced-transparency`
    - `prefers-reduced-data`
    - `color-gamut`
    - `dynamic-range`
    - `video-dynamic-range`
    - `forced-colors`
    - `inverted-colors`
    - `scripting`
    - `display-mode`
  - Media query operators
    - `and`
    - `or`
    - `not`
    - `only`
  - Media query ranges
  - Media query best practices

- **56. Breakpoints**
  - Breakpoints
  - Common breakpoints
    - 320px
    - 480px
    - 768px
    - 1024px
    - 1280px
    - 1440px
    - 1920px
  - Breakpoint selection
  - Breakpoint best practices
  - Breakpoint pitfalls

- **57. Fluid Typography**
  - Fluid typography
  - `clamp()`
  - `min()`
  - `max()`
  - Viewport units
  - Fluid typography best practices

- **58. Fluid Layouts**
  - Fluid layouts
  - Percentage widths
  - `fr` units
  - `minmax()`
  - Fluid layout best practices

- **59. Responsive Images**
  - Responsive images
  - `srcset`
  - `sizes`
  - `<picture>`
  - Art direction
  - Resolution switching
  - Responsive image best practices

- **60. Responsive Design Patterns**
  - Column drop
  - Mostly fluid
  - Layout shifter
  - Tiny tweaks
  - Off-canvas
  - Responsive pattern best practices

---

# IX. Transitions and Animations

- **61. Transitions**
  - `transition`
  - `transition-property`
  - `transition-duration`
  - `transition-timing-function`
  - `transition-delay`
  - Transition best practices
  - Transition performance

- **62. Timing Functions**
  - `ease`
  - `linear`
  - `ease-in`
  - `ease-out`
  - `ease-in-out`
  - `cubic-bezier()`
  - `steps()`
  - `linear()`
  - Timing function best practices

- **63. Transforms**
  - `transform`
  - `translate()`
  - `translateX()`
  - `translateY()`
  - `translateZ()`
  - `translate3d()`
  - `scale()`
  - `scaleX()`
  - `scaleY()`
  - `scaleZ()`
  - `scale3d()`
  - `rotate()`
  - `rotateX()`
  - `rotateY()`
  - `rotateZ()`
  - `rotate3d()`
  - `skew()`
  - `skewX()`
  - `skewY()`
  - `matrix()`
  - `matrix3d()`
  - `perspective()`
  - `transform-origin`
  - `transform-style`
  - `perspective`
  - `perspective-origin`
  - `backface-visibility`
  - Transform best practices

- **64. Animations**
  - `@keyframes`
  - `animation`
  - `animation-name`
  - `animation-duration`
  - `animation-timing-function`
  - `animation-delay`
  - `animation-iteration-count`
  - `animation-direction`
  - `animation-fill-mode`
  - `animation-play-state`
  - Animation best practices
  - Animation performance

- **65. Animation Patterns**
  - Loading animations
  - Hover animations
  - Entrance animations
  - Exit animations
  - Attention animations
  - Scroll animations
  - Animation pattern best practices

- **66. View Transitions**
  - View Transitions API
  - `view-transition-name`
  - `::view-transition`
  - `::view-transition-group()`
  - `::view-transition-image-pair()`
  - `::view-transition-old()`
  - `::view-transition-new()`
  - View transition best practices

- **67. Scroll-Driven Animations**
  - Scroll-driven animations
  - `animation-timeline`
  - `scroll()`
  - `view()`
  - Scroll timeline
  - View timeline
  - Scroll-driven best practices

---

# X. Advanced CSS

- **68. CSS Functions**
  - `calc()`
  - `min()`
  - `max()`
  - `clamp()`
  - `var()`
  - `env()`
  - `attr()`
  - `url()`
  - `image-set()`
  - `cross-fade()`
  - `element()`
  - `paint()`
  - `counter()`
  - `counters()`
  - `symbols()`
  - `toggle()`
  - `repeat()`
  - `minmax()`
  - `fit-content()`
  - `rgb()`
  - `hsl()`
  - `color-mix()`
  - CSS function best practices

- **69. CSS Counters**
  - CSS counters
  - `counter-reset`
  - `counter-increment`
  - `counter-set`
  - `counter()`
  - `counters()`
  - Counter best practices

- **70. CSS Shapes**
  - CSS shapes
  - `shape-outside`
  - `shape-margin`
  - `shape-image-threshold`
  - `clip-path`
  - `mask`
  - `mask-image`
  - `mask-mode`
  - `mask-repeat`
  - `mask-position`
  - `mask-clip`
  - `mask-origin`
  - `mask-size`
  - `mask-composite`
  - Shape best practices

- **71. CSS Clipping and Masking**
  - `clip-path`
  - `mask`
  - `mask-image`
  - `mask-border`
  - Clipping and masking best practices

- **72. CSS Filters**
  - `filter`
  - `backdrop-filter`
  - Filter best practices

- **73. CSS Blend Modes**
  - `mix-blend-mode`
  - `background-blend-mode`
  - Blend mode values
  - Blend mode best practices

- **74. CSS Scroll Snap**
  - Scroll snap
  - `scroll-snap-type`
  - `scroll-snap-align`
  - `scroll-snap-stop`
  - `scroll-padding`
  - `scroll-margin`
  - Scroll snap best practices

- **75. CSS Overscroll**
  - `overscroll-behavior`
  - `overscroll-behavior-x`
  - `overscroll-behavior-y`
  - Overscroll best practices

- **76. CSS Scrollbars**
  - `scrollbar-width`
  - `scrollbar-color`
  - `scrollbar-gutter`
  - `::-webkit-scrollbar`
  - Scrollbar styling best practices

- **77. CSS Writing Modes**
  - `writing-mode`
  - `direction`
  - `text-orientation`
  - Writing mode best practices

- **78. CSS Tables**
  - Table layout
  - `table-layout`
  - `border-collapse`
  - `border-spacing`
  - Table best practices

- **79. CSS Lists**
  - `list-style`
  - `::marker`
  - `counter-reset`
  - List best practices

- **80. CSS Forms**
  - Form styling
  - Input styling
  - `appearance`
  - `accent-color`
  - `caret-color`
  - Form best practices

- **81. CSS Accessibility**
  - Accessibility
  - Focus indicators
  - Color contrast
  - `prefers-reduced-motion`
  - `prefers-color-scheme`
  - `forced-colors`
  - Screen reader considerations
  - Accessibility best practices

- **82. CSS Print Styles**
  - `@media print`
  - Page breaks
  - `break-before`
  - `break-after`
  - `break-inside`
  - `orphans`
  - `widows`
  - Print best practices

---

# XI. CSS Architecture

- **83. CSS Methodologies**
  - BEM
    - Block
    - Element
    - Modifier
  - OOCSS
    - Object-Oriented CSS
  - SMACSS
    - Scalable and Modular Architecture for CSS
  - Atomic CSS
  - ITCSS
    - Inverted Triangle CSS
  - ACSS
    - Atomic CSS
  - Methodology comparison
  - Methodology best practices

- **84. CSS Naming Conventions**
  - BEM naming
  - Kebab-case
  - CamelCase
  - Snake_case
  - Naming best practices

- **85. CSS Organization**
  - File organization
  - Folder structure
  - Import order
  - Layer order
  - Organization best practices

- **86. CSS Architecture Patterns**
  - Layered architecture
  - Component-based architecture
  - Utility-first architecture
  - Atomic design
  - Design systems
  - Architecture best practices

- **87. CSS Custom Properties for Theming**
  - Theming
  - Custom properties
  - Theme switching
  - Dark mode
  - Light mode
  - Theming best practices

- **88. CSS Preprocessors**
  - Sass
    - Variables
    - Nesting
    - Mixins
    - Functions
    - Partials
    - Imports
    - Extends
    - Placeholders
    - Operators
    - Control directives
  - Less
  - Stylus
  - Preprocessor comparison
  - Preprocessor best practices

- **89. CSS Postprocessors**
  - PostCSS
  - Autoprefixer
  - CSSNano
  - Stylelint
  - PostCSS plugins
  - Postprocessor best practices

- **90. CSS-in-JS**
  - CSS-in-JS
  - Styled Components
  - Emotion
  - Linaria
  - Stitches
  - Vanilla Extract
  - CSS-in-JS best practices
  - CSS-in-JS trade-offs

- **91. Utility-First CSS**
  - Utility-first CSS
  - Tailwind CSS
  - UnoCSS
  - Windi CSS
  - Utility-first best practices

- **92. Atomic CSS**
  - Atomic CSS
  - Atomizer
  - Utility-first vs atomic CSS
  - Atomic CSS best practices

- **93. Design Systems**
  - Design systems
  - Design tokens
  - Component libraries
  - Style guides
  - Design system best practices

---

# XII. CSS Performance

- **94. Performance Fundamentals**
  - Performance
  - Rendering performance
  - Paint performance
  - Layout performance
  - Composite performance
  - Performance metrics
  - Performance best practices

- **95. Render-Blocking CSS**
  - Render-blocking CSS
  - Critical CSS
  - Inline critical CSS
  - Deferred CSS
  - Async CSS
  - CSS loading best practices

- **96. CSS Optimization**
  - Minification
  - Compression
  - Concatenation
  - Tree shaking
  - Dead code elimination
  - CSS optimization best practices

- **97. CSS Selectors Performance**
  - Selector performance
  - Selector matching
  - Selector complexity
  - Selector best practices

- **98. CSS Animations Performance**
  - Animation performance
  - Composited properties
  - `transform`
  - `opacity`
  - `will-change`
  - Animation performance best practices

- **99. CSS Containment**
  - CSS containment
  - `contain`
  - `contain-intrinsic-size`
  - `content-visibility`
  - Containment best practices

- **100. CSS `content-visibility`**
  - `content-visibility`
  - `content-visibility: auto`
  - `content-visibility: hidden`
  - `contain-intrinsic-size`
  - Content visibility best practices

- **101. CSS Layered Rendering**
  - Layered rendering
  - Composite layers
  - `will-change`
  - `transform: translateZ(0)`
  - `backface-visibility`
  - Layered rendering best practices

- **102. CSS Profiling**
  - Chrome DevTools
  - Performance panel
  - Rendering panel
  - Layers panel
  - CSS profiling best practices

---

# XIII. CSS Tools

- **103. CSS Preprocessors**
  - Sass
  - Less
  - Stylus
  - Preprocessor best practices

- **104. CSS Postprocessors**
  - PostCSS
  - Autoprefixer
  - CSSNano
  - Postprocessor best practices

- **105. CSS Frameworks**
  - Bootstrap
  - Tailwind CSS
  - Bulma
  - Foundation
  - Materialize
  - UIkit
  - Semantic UI
  - Framework comparison
  - Framework selection

- **106. CSS Linters**
  - Stylelint
  - CSSLint
  - Linter best practices

- **107. CSS Formatters**
  - Prettier
  - CSScomb
  - Formatter best practices

- **108. CSS Debugging Tools**
  - Chrome DevTools
  - Firefox Developer Tools
  - Safari Web Inspector
  - Edge DevTools
  - Debugging best practices

- **109. CSS Documentation Tools**
  - Storybook
  - Styleguidist
  - Pattern Lab
  - Documentation best practices

- **110. CSS Testing Tools**
  - BackstopJS
  - Percy
  - Chromatic
  - Visual regression testing
  - Testing best practices

---

# XIV. CSS Projects by Difficulty

## Beginner Projects

- **1. Personal Profile Card**
  - HTML structure
  - CSS styling
  - Typography
  - Colors
  - Layout

- **2. Navigation Bar**
  - Flexbox
  - Links
  - Hover effects
  - Responsive design

- **3. Button Collection**
  - Button styles
  - Hover states
  - Focus states
  - Transitions

- **4. Pricing Table**
  - Layout
  - Typography
  - Colors
  - Responsive design

- **5. Photo Gallery**
  - Grid
  - Images
  - Hover effects
  - Responsive design

---

## Intermediate Projects

- **6. Responsive Landing Page**
  - Layout
  - Typography
  - Colors
  - Responsive design
  - Animations

- **7. Dashboard UI**
  - Grid
  - Flexbox
  - Cards
  - Charts
  - Responsive design

- **8. E-Commerce Product Page**
  - Layout
  - Typography
  - Colors
  - Responsive design
  - Animations

- **9. Blog Layout**
  - Grid
  - Typography
  - Cards
  - Responsive design

- **10. Form Design**
  - Form styling
  - Input styling
  - Validation styles
  - Responsive design

---

## Advanced Projects

- **11. Design System**
  - Design tokens
  - Components
  - Documentation
  - Storybook
  - Accessibility

- **12. Animated Landing Page**
  - Scroll animations
  - View transitions
  - Keyframes
  - Performance

- **13. CSS Grid Layout System**
  - Grid
  - Subgrid
  - Container queries
  - Responsive design

- **14. Theming System**
  - Custom properties
  - Dark mode
  - Light mode
  - Theme switching

- **15. CSS Architecture**
  - BEM
  - ITCSS
  - Utility-first
  - Design tokens

---

## Expert Projects

- **16. CSS Framework**
  - Utility classes
  - Components
  - Theming
  - Documentation
  - Performance

- **17. CSS-in-JS Library**
  - Runtime
  - Static extraction
  - Theming
  - Performance

- **18. High-Performance CSS**
  - Critical CSS
  - Code splitting
  - Containment
  - Content visibility
  - Performance optimization

- **19. Accessible Design System**
  - WCAG compliance
  - Color contrast
  - Focus management
  - Screen reader support
  - Keyboard navigation

- **20. Production CSS Architecture**
  - Layered architecture
  - Design tokens
  - Component library
  - Documentation
  - Testing
  - Performance
  - Accessibility

---

# XV. Progressive CSS Learning Sequence

## Level 1 — CSS Fundamentals

- Master:
  - CSS syntax
  - Adding CSS to HTML
  - Selectors
  - Properties
  - Values
  - Units
  - Colors

## Level 2 — Selectors and Specificity

- Master:
  - Basic selectors
  - Combinators
  - Pseudo-classes
  - Pseudo-elements
  - Specificity
  - Cascade
  - Inheritance

## Level 3 — Box Model

- Master:
  - Box model
  - Width and height
  - Padding
  - Margin
  - Border
  - Overflow
  - Display
  - Visibility

## Level 4 — Typography

- Master:
  - Typography fundamentals
  - Font properties
  - Text properties
  - Web fonts
  - Text layout
  - Lists
  - Links

## Level 5 — Backgrounds and Borders

- Master:
  - Backgrounds
  - Gradients
  - Borders
  - Border radius
  - Box shadow
  - Filters and effects

## Level 6 — Layout

- Master:
  - Layout fundamentals
  - Positioning
  - Floating
  - Flexbox
  - Grid
  - Multi-column layout
  - Tables
  - Container queries
  - Logical properties

## Level 7 — Responsive Design

- Master:
  - Responsive design fundamentals
  - Media queries
  - Breakpoints
  - Fluid typography
  - Fluid layouts
  - Responsive images
  - Responsive patterns

## Level 8 — Transitions and Animations

- Master:
  - Transitions
  - Timing functions
  - Transforms
  - Animations
  - Animation patterns
  - View transitions
  - Scroll-driven animations

## Level 9 — Advanced CSS

- Master:
  - CSS functions
  - CSS counters
  - CSS shapes
  - CSS clipping and masking
  - CSS filters
  - CSS blend modes
  - CSS scroll snap
  - CSS overscroll
  - CSS scrollbars
  - CSS writing modes
  - CSS tables
  - CSS lists
  - CSS forms
  - CSS accessibility
  - CSS print styles

## Level 10 — CSS Architecture

- Master:
  - CSS methodologies
  - CSS naming conventions
  - CSS organization
  - CSS architecture patterns
  - CSS custom properties for theming
  - CSS preprocessors
  - CSS postprocessors
  - CSS-in-JS
  - Utility-first CSS
  - Atomic CSS
  - Design systems

## Level 11 — CSS Performance

- Master:
  - Performance fundamentals
  - Render-blocking CSS
  - CSS optimization
  - Selector performance
  - Animation performance
  - CSS containment
  - `content-visibility`
  - Layered rendering
  - CSS profiling

## Level 12 — CSS Tools

- Master:
  - CSS preprocessors
  - CSS postprocessors
  - CSS frameworks
  - CSS linters
  - CSS formatters
  - CSS debugging tools
  - CSS documentation tools
  - CSS testing tools

## Level 13 — Production Engineering

- Master:
  - Design systems
  - Component libraries
  - Theming
  - Accessibility
  - Performance
  - Testing
  - Documentation
  - Production best practices

---

# XVI. Final CSS Competency Map

- **Foundations**

  - CSS syntax
  - Adding CSS to HTML
  - Units
  - Colors
  - Values

- **Selectors**

  - Basic selectors
  - Combinators
  - Pseudo-classes
  - Pseudo-elements
  - Specificity
  - Advanced selectors

- **Cascade**

  - Cascade
  - Inheritance
  - Specificity
  - `!important`
  - Cascade layers
  - CSS reset
  - Custom properties

- **Box Model**

  - Box model
  - Width and height
  - Padding
  - Margin
  - Border
  - Overflow
  - Display
  - Visibility
  - Box shadow
  - Outline

- **Typography**

  - Typography fundamentals
  - Font properties
  - Text properties
  - Web fonts
  - Text layout
  - Lists
  - Links

- **Backgrounds and Borders**

  - Backgrounds
  - Gradients
  - Borders
  - Border radius
  - Box shadow
  - Filters and effects

- **Layout**

  - Layout fundamentals
  - Positioning
  - Floating
  - Flexbox
  - Grid
  - Multi-column layout
  - Tables
  - Container queries
  - Logical properties

- **Responsive Design**

  - Responsive design fundamentals
  - Media queries
  - Breakpoints
  - Fluid typography
  - Fluid layouts
  - Responsive images
  - Responsive patterns

- **Transitions and Animations**

  - Transitions
  - Timing functions
  - Transforms
  - Animations
  - Animation patterns
  - View transitions
  - Scroll-driven animations

- **Advanced CSS**

  - CSS functions
  - CSS counters
  - CSS shapes
  - CSS clipping and masking
  - CSS filters
  - CSS blend modes
  - CSS scroll snap
  - CSS overscroll
  - CSS scrollbars
  - CSS writing modes
  - CSS tables
  - CSS lists
  - CSS forms
  - CSS accessibility
  - CSS print styles

- **Architecture**

  - CSS methodologies
  - CSS naming conventions
  - CSS organization
  - CSS architecture patterns
  - CSS custom properties for theming
  - CSS preprocessors
  - CSS postprocessors
  - CSS-in-JS
  - Utility-first CSS
  - Atomic CSS
  - Design systems

- **Performance**

  - Performance fundamentals
  - Render-blocking CSS
  - CSS optimization
  - Selector performance
  - Animation performance
  - CSS containment
  - `content-visibility`
  - Layered rendering
  - CSS profiling

- **Tools**

  - CSS preprocessors
  - CSS postprocessors
  - CSS frameworks
  - CSS linters
  - CSS formatters
  - CSS debugging tools
  - CSS documentation tools
  - CSS testing tools

- **Production**

  - Design systems
  - Component libraries
  - Theming
  - Accessibility
  - Performance
  - Testing
  - Documentation

---

## Recommended Overall Progression

**CSS Fundamentals → Selectors and Specificity → Box Model → Typography → Backgrounds and Borders → Layout → Responsive Design → Transitions and Animations → Advanced CSS → CSS Architecture → CSS Performance → CSS Tools → Production Engineering**
