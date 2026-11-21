# jQuery in Angular — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Integrating jQuery into an Angular application is the practice of allowing legacy jQuery code — selectors, DOM manipulation, plugins, event handlers, and AJAX calls — to coexist with Angular’s component-based, declarative rendering model, typically during an incremental migration from a jQuery-based codebase to Angular or when a specific third-party plugin has no Angular-native equivalent.

**Technical Definition**
Angular and jQuery have fundamentally incompatible DOM ownership and change-detection models. Angular uses a hierarchical component tree, a virtual DOM-like change-detection system (Zone.js and `ChangeDetectorRef`), and a declarative data-binding paradigm where the model drives the view. jQuery operates directly on the real DOM, bypassing Angular’s change-detection entirely. When jQuery modifies a DOM node that Angular also manages, Angular’s internal state becomes inconsistent with the actual DOM, leading to lost updates, stale data, or `ExpressionChangedAfterItHasBeenCheckedError` exceptions. The integration therefore requires explicit **boundaries**: Angular owns the component tree and the model, jQuery owns isolated DOM islands created via `ElementRef`, and communication occurs through Angular inputs/outputs and lifecycle-managed initialisation and cleanup. The official Angular guidance and the broader Angular community recommend **not mixing jQuery with Angular** unless absolutely necessary, and when a jQuery plugin must be used, wrapping it in an Angular **directive** is the cleanest approach.

**Beginner-Friendly Explanation**
Angular and jQuery both want to control the page, but they do it in completely different ways. jQuery says: “I will find the element and change it directly.” Angular says: “I will describe what the page should look like based on my data, and I will handle all the changes myself.” If you let both change the same part of the page, they fight, and things break — Angular may overwrite jQuery’s changes, or jQuery may make Angular’s data out of sync with what the user sees. The solution is to give each one its own territory. Angular owns the overall page and the components. Inside a component or directive, you create a small container and tell jQuery: “This box is yours — do whatever you want inside it.” Angular promises not to touch that box. That way, jQuery plugins and legacy code can keep working while the rest of the application moves to Angular.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Conflicting DOM ownership** | Angular manages the DOM through data binding and change detection; jQuery mutates the real DOM directly. Concurrent control of the same nodes causes corruption or stale data. |
| **Change-detection bypass** | jQuery plugins and events run outside Angular’s Zone.js. Angular does not automatically detect changes made by jQuery; `NgZone` or `ChangeDetectorRef` must be used to trigger updates. |
| **Directive-based isolation** | The recommended pattern is to wrap jQuery plugins in an Angular directive that owns an `ElementRef` and manages the plugin’s lifecycle. |
| **Lifecycle-managed initialisation** | jQuery plugins must be initialised in `ngAfterViewInit` and destroyed in `ngOnDestroy` to prevent memory leaks. |
| **Separate event systems** | jQuery events do not trigger Angular bindings, and Angular events do not automatically invoke jQuery handlers. Bridging must be explicit. |
| **AJAX must use Angular’s HttpClient** | jQuery AJAX (`$.ajax`) bypasses Angular’s interceptors and change detection; `HttpClient` is the correct choice. |
| **Server-side rendering incompatibility** | jQuery expects a physical DOM; it must be guarded with `isPlatformBrowser` when using Angular Universal. |

---

### Prerequisites

- Working knowledge of Angular components, directives, `ElementRef`, and lifecycle hooks (`ngAfterViewInit`, `ngOnDestroy`).
- Working knowledge of jQuery selection, DOM manipulation, events, and plugins.
- Understanding of Angular’s change-detection system and Zone.js.
- Awareness that Angular’s `Renderer2` is the preferred abstraction for DOM manipulation.

---

### Related Programming Areas

- **DOM performance** — Angular’s change detection vs. jQuery’s direct DOM access.
- **Event performance** — Angular’s event bindings vs. jQuery’s event system.
- **Memory performance** — plugins must be destroyed to prevent leaks in Angular’s long-lived SPA context.
- **Security** — jQuery sinks (`.html()`) inside Angular components reintroduce XSS risk.
- **Dependency security** — legacy jQuery versions carry known CVEs.

---

### Core Concepts / Features (Index)

1. jQuery in Angular (General)
2. Angular ElementRef
3. Angular Lifecycle Hooks (`ngAfterViewInit`, `ngOnDestroy`)
4. DOM Manipulation Boundaries and `Renderer2`
5. jQuery Plugins in Angular (Directive Wrapper)
6. jQuery Events in Angular (`NgZone`)
7. jQuery AJAX in Angular (`HttpClient`)
8. Integration Challenges
9. Integration Best Practices
10. When to Avoid jQuery in Angular


## Core Concept 1: jQuery in Angular (General)

### Definitions

**Core Definition**
jQuery in Angular is the coexistence of the jQuery library with Angular components and directives, where jQuery is used for specific isolated tasks — typically legacy plugins or DOM operations that Angular does not cover — while Angular remains responsible for the component tree and the model-driven rendering.

**Technical Definition**
Angular’s change-detection system (powered by Zone.js) tracks asynchronous operations and decides when to update the view based on changes to the component’s model. jQuery operates outside this system: when jQuery modifies the DOM, Angular does not know about it, and when Angular re-renders, it may overwrite jQuery’s changes. The Angular community consistently advises against mixing the two, but when a jQuery plugin must be used, the recommended approach is to **wrap it in an Angular directive** that encapsulates the DOM access and manages the plugin’s lifecycle. The directive obtains the host element via `ElementRef`, initialises the plugin in `ngAfterViewInit`, and destroys it in `ngOnDestroy`. Communication between the plugin and Angular is handled through `@Input()` and `@Output()` properties.

**Beginner-Friendly Explanation**
jQuery in Angular is a compromise. You are saying: “Most of this page is Angular, but this one part is still jQuery.” That is fine as long as the two parts stay in their own lanes. The moment jQuery reaches into a part of the page that Angular is also changing, you get bugs that are hard to diagnose — Angular may wipe out jQuery’s changes, or jQuery may make Angular’s data out of sync with what the user sees. The general rule is: use jQuery only inside a dedicated directive or component that Angular does not touch, and only for things that Angular cannot easily do — usually wrapping an existing jQuery plugin.

---

### Purposes

- **To reuse existing jQuery plugins** without rewriting them from scratch.
- **To migrate incrementally** from a jQuery codebase to Angular, one component at a time.
- **To isolate legacy DOM manipulation** inside a boundary that Angular does not manage.
- **To bridge event and state changes** between Angular inputs/outputs and jQuery plugin APIs.
- **To avoid the cost of a full rewrite** while still moving toward Angular.

---

### Syntax Rules and Structure

#### General Isolation Pattern (Directive Wrapper)

```typescript
import { Directive, ElementRef, Input, OnDestroy, AfterViewInit, NgZone } from '@angular/core';
import * as $ from 'jquery';
import 'some-jquery-plugin';

@Directive({ selector: '[appSomePlugin]' })
export class SomePluginDirective implements AfterViewInit, OnDestroy {
  @Input() options: any = {};
  private $element: JQuery<HTMLElement>;

  constructor(private el: ElementRef, private ngZone: NgZone) {
    this.$element = $(this.el.nativeElement);
  }

  ngAfterViewInit(): void {
    this.ngZone.runOutsideAngular(() => {
      this.$element.somePlugin(this.options);
    });
  }

  ngOnDestroy(): void {
    this.$element.somePlugin('destroy');
  }
}
```

**Component Breakdown**

| Component | Description |
|---|---|
| `ElementRef` | Provides the host DOM element. |
| `ngAfterViewInit` | Runs after Angular has rendered the view; the correct place to initialise jQuery plugins. |
| `ngOnDestroy` | Runs before Angular destroys the directive; the correct place to destroy plugins. |
| `NgZone.runOutsideAngular` | Prevents the plugin from triggering unnecessary change-detection cycles. |
| `@Input()` | Receives configuration from the parent component. |

---

### Constraints and Limitations

- **Angular must not update the jQuery-managed subtree**; use a dedicated directive or an empty container.
- **jQuery plugins must be destroyed** to prevent memory leaks in Angular SPAs.
- **Angular and jQuery event systems are separate**; events do not automatically propagate between them.
- **jQuery version must be compatible**; legacy jQuery may carry known CVEs.

---

### Annotated Code Example

```typescript
// jquery-tooltip.directive.ts
import { Directive, ElementRef, Input, OnDestroy, AfterViewInit, NgZone } from '@angular/core';
import * as $ from 'jquery';

@Directive({ selector: '[appJqueryTooltip]' })
export class JqueryTooltipDirective implements AfterViewInit, OnDestroy {
  @Input() tooltipText = '';
  private $el: JQuery<HTMLElement>;

  constructor(private el: ElementRef, private ngZone: NgZone) {
    this.$el = $(this.el.nativeElement);
  }

  ngAfterViewInit(): void {
    // Step 1: Initialise the plugin outside Angular's zone
    this.ngZone.runOutsideAngular(() => {
      this.$el.tooltip({ title: this.tooltipText });
    });
  }

  ngOnDestroy(): void {
    // Step 2: Destroy the plugin before the directive is removed
    this.$el.tooltip('destroy');
  }
}
```

```html
<!-- app.component.html -->
<button appJqueryTooltip tooltipText="Hello from jQuery">Hover me</button>
```

**Expected Output**
A tooltip is initialised on the button. When the directive is destroyed, the tooltip is destroyed and its event listeners are removed.

**Why this output occurs**: `ngAfterViewInit` runs after Angular commits the button to the DOM. jQuery initialises the plugin on that node outside Angular’s zone, preventing unnecessary change detection. `ngOnDestroy` runs before the directive is removed, allowing the plugin to clean up.

---

### Real-World Cases

- **Legacy admin panels**: Wrapping a jQuery data table or date picker inside an Angular route during incremental migration.
- **Third-party widgets**: Embedding a jQuery-based chart or map library that has no Angular equivalent.
- **Gradual migration**: Moving one page at a time from a jQuery-rendered application to Angular.

---

## Core Concept 2: Angular ElementRef

### Definitions

**Core Definition**
`ElementRef` is an Angular dependency-injection token that provides a reference to the host DOM element of a component or directive. It is Angular’s supported escape hatch for accessing the native DOM element, which is necessary when integrating jQuery.

**Technical Definition**
`ElementRef` is a class in `@angular/core` whose `nativeElement` property holds the underlying DOM node. When injected into a directive or component, it refers to the host element (the element on which the directive is applied, or the component’s host element). `ElementRef` is populated after the view is initialised; accessing `nativeElement` before `ngAfterViewInit` may yield `null` or an empty node. For jQuery integration, `ElementRef` provides the real DOM node that jQuery needs for selection and plugin initialisation. However, Angular’s official guidance recommends using `Renderer2` instead of direct DOM manipulation whenever possible, because `Renderer2` works in server-side rendering and other non-browser contexts.

**Beginner-Friendly Explanation**
`ElementRef` is a way to get your hands on the actual HTML element that Angular created for a directive or component. Normally, Angular hides the real DOM from you and asks you to work through data binding and templates. But jQuery needs the real element — it cannot work with Angular’s model. `ElementRef` gives you that real element. You inject it into a directive, and after Angular renders the view, `nativeElement` points to the actual element in the page. You can then give that element to jQuery.

---

### Purposes

- **To obtain the real DOM node** that jQuery needs for selection and plugin initialisation.
- **To scope jQuery operations** to a directive’s host element rather than the entire document.
- **To avoid global selectors** (`$('.some-class')`) inside Angular components.
- **To pass the DOM node to a jQuery plugin** during initialisation.
- **To store the jQuery-wrapped element** for later plugin method calls.

---

### Syntax Rules and Structure

#### Injecting `ElementRef`

```typescript
import { Directive, ElementRef } from '@angular/core';

@Directive({ selector: '[appMyDirective]' })
export class MyDirective {
  constructor(private el: ElementRef) {
    // el.nativeElement is the host DOM element
  }
}
```

#### Wrapping with jQuery

```typescript
import * as $ from 'jquery';

const $element = $(this.el.nativeElement);
```

**Component Breakdown**

| Component | Description |
|---|---|
| `ElementRef` | The injected reference to the host element. |
| `nativeElement` | The raw DOM node. |
| `$(this.el.nativeElement)` | Wraps the DOM node in a jQuery object. |

---

### Constraints and Limitations

- **`nativeElement` is not available before `ngAfterViewInit`**; access it only after the view is initialised.
- **Direct DOM access bypasses Angular’s security and SSR support**; use `Renderer2` when possible.
- **`ElementRef` exposes the host element, not the template’s children**; use `@ViewChild` or query the DOM within the host element for descendants.

---

### Annotated Code Example

```typescript
import { Directive, ElementRef, AfterViewInit } from '@angular/core';
import * as $ from 'jquery';

@Directive({ selector: '[appDraggable]' })
export class DraggableDirective implements AfterViewInit {
  constructor(private el: ElementRef) {}

  ngAfterViewInit(): void {
    // Step 1: Wrap the host element
    const $el = $(this.el.nativeElement);

    // Step 2: Initialise the jQuery UI draggable plugin
    $el.draggable({ containment: 'parent' });
  }
}
```

**Expected Output**
The host element becomes draggable within its parent container.

**Why this output occurs**: `ElementRef.nativeElement` provides the real DOM node. jQuery UI’s `draggable` plugin is initialised on that node, attaching mouse event listeners and updating its position during dragging.

---

### Real-World Cases

- **Draggable widgets**: Wrapping jQuery UI draggable on a dashboard card.
- **Resizable panels**: Using jQuery UI resizable on a directive host.
- **Legacy plugin integration**: Providing the host element to any plugin that expects a DOM node.

---

## Core Concept 3: Angular Lifecycle Hooks (`ngAfterViewInit`, `ngOnDestroy`)

### Definitions

**Core Definition**
Angular lifecycle hooks are methods that Angular calls at specific stages of a directive or component’s existence. For jQuery integration, the relevant hooks are `ngAfterViewInit`, which runs after Angular has initialised the component’s view and child views, and `ngOnDestroy`, which runs just before Angular destroys the directive or component.

**Technical Definition**
`ngAfterViewInit` is the Angular equivalent of “the DOM is ready” for a component’s view. It fires once after Angular has fully initialised the component’s view and its child views. jQuery plugins should be initialised in this hook because the DOM node is not guaranteed to exist before this point. `ngOnDestroy` fires before Angular destroys the directive or component; it is the correct place to destroy jQuery plugins, remove event listeners, and cancel pending AJAX requests to prevent memory leaks. Unlike AngularJS’s `$scope.$apply()`, Angular’s `NgZone` is used to bring jQuery-initiated changes back into Angular’s change-detection system.

**Beginner-Friendly Explanation**
Angular lifecycle hooks are the “before” and “after” moments of a directive’s life. `ngAfterViewInit` means “the view is now on the page” — that is when jQuery can safely find and enhance its elements. `ngOnDestroy` means “the directive is about to be removed” — that is when jQuery must clean up. If you try to use jQuery before `ngAfterViewInit`, the elements may not exist yet. If you forget to clean up in `ngOnDestroy`, the plugin stays in memory after the directive is gone, which is especially bad in a single-page application where components are created and destroyed frequently.

---

### Purposes

- **To initialise jQuery plugins after the DOM is committed** (`ngAfterViewInit`).
- **To destroy plugins and remove handlers before the directive is removed** (`ngOnDestroy`).
- **To avoid accessing `ElementRef.nativeElement` before it is available**.
- **To prevent memory leaks** in Angular’s long-lived SPA context.
- **To coordinate with Angular’s change-detection system** when needed.

---

### Syntax Rules and Structure

#### Directive Lifecycle

```typescript
import { Directive, AfterViewInit, OnDestroy } from '@angular/core';

@Directive({ selector: '[appMyPlugin]' })
export class MyPluginDirective implements AfterViewInit, OnDestroy {
  ngAfterViewInit(): void {
    // initialise plugin
  }

  ngOnDestroy(): void {
    // destroy plugin
  }
}
```

#### Component Lifecycle

```typescript
import { Component, AfterViewInit, OnDestroy } from '@angular/core';

@Component({ selector: 'app-my-component', template: '...' })
export class MyComponent implements AfterViewInit, OnDestroy {
  ngAfterViewInit(): void {
    // initialise plugin on a child element via @ViewChild
  }

  ngOnDestroy(): void {
    // cleanup
  }
}
```

---

### Constraints and Limitations

- **`ngAfterViewInit` fires once**; if the view is re-rendered dynamically, the plugin may need to be re-initialised or updated.
- **`ngOnDestroy` is not called for services**; only for directives and components.
- **Angular’s lifecycle hooks run inside the Zone**; use `NgZone.runOutsideAngular` for plugin initialisation to avoid unnecessary change detection.

---

### Annotated Code Example (Date Picker Wrapper)

```typescript
import {
  Directive, ElementRef, Input, Output, EventEmitter,
  AfterViewInit, OnDestroy, NgZone
} from '@angular/core';
import * as $ from 'jquery';
import 'jquery-datepicker';

@Directive({ selector: '[appDatePicker]' })
export class DatePickerDirective implements AfterViewInit, OnDestroy {
  @Input() format = 'mm/dd/yyyy';
  @Output() dateSelected = new EventEmitter<string>();
  private $el: JQuery<HTMLElement>;

  constructor(private el: ElementRef, private ngZone: NgZone) {
    this.$el = $(this.el.nativeElement);
  }

  ngAfterViewInit(): void {
    this.ngZone.runOutsideAngular(() => {
      this.$el.datepicker({
        dateFormat: this.format,
        onSelect: (date: string) => {
          // Bridge jQuery event to Angular output
          this.ngZone.run(() => this.dateSelected.emit(date));
        }
      });
    });
  }

  ngOnDestroy(): void {
    this.$el.datepicker('destroy');
  }
}
```

**Expected Output**
The input is enhanced by the jQuery date picker. Selecting a date emits the `dateSelected` event with the selected date. When the directive is destroyed, the date picker is destroyed.

**Why this output occurs**: `ngAfterViewInit` initialises the plugin outside Angular’s zone. The `onSelect` callback uses `ngZone.run` to re-enter Angular’s zone and emit the output, ensuring change detection runs. `ngOnDestroy` destroys the plugin.

---

### Real-World Cases

- **Date pickers**: Initialising a jQuery date picker on an Angular directive host.
- **Rich text editors**: Wrapping a legacy jQuery-based WYSIWYG editor.
- **Data tables**: Initialising a jQuery DataTable on an Angular-rendered table.

---

## Core Concept 4: DOM Manipulation Boundaries and `Renderer2`

### Definitions

**Core Definition**
A DOM manipulation boundary is an explicit region of the page where either Angular or jQuery has exclusive control. `Renderer2` is Angular’s abstraction for safe DOM manipulation that works across platforms (browser, server, web worker).

**Technical Definition**
Angular provides `Renderer2` as a service that abstracts DOM operations (`addClass`, `setStyle`, `createElement`, `appendChild`, etc.) behind a platform-agnostic API. In a browser context, `Renderer2` is essentially a wrapper around native DOM APIs; its advantage is that it works in server-side rendering and other non-browser contexts where the physical DOM does not exist. jQuery, by contrast, directly accesses the browser DOM. The boundary pattern for jQuery integration is to render an **empty container** that Angular does not update, and let jQuery manage everything inside that container. `Renderer2` should be used for Angular-managed DOM changes (e.g., adding a class to the host element), while jQuery should be used only for the plugin’s API that requires direct DOM access. Mixing the two on the same nodes breaks Angular’s change detection and can cause hard-to-debug errors.

**Beginner-Friendly Explanation**
Imagine two chefs in one kitchen. One chef (Angular) insists on controlling every pot and pan. The other chef (jQuery) just grabs whatever is nearby and starts cooking. If they both grab the same pan, disaster. The solution is to give each chef their own counter. Angular gets the main kitchen; jQuery gets a small separate counter (the directive’s host element or an empty container). `Renderer2` is Angular’s own set of kitchen tools — safe, platform-independent, and designed for Angular’s workflow. Use `Renderer2` for Angular’s DOM changes and jQuery only for the plugin’s specific API.

---

### Purposes

- **To prevent Angular and jQuery from mutating the same nodes**.
- **To ensure Angular’s change detection remains consistent** with the real DOM.
- **To isolate jQuery plugins** inside a container that Angular does not update.
- **To use `Renderer2` for Angular-managed DOM changes** (classes, styles, attributes).
- **To maintain server-side rendering compatibility** by using `Renderer2` instead of direct DOM access.

---

### Syntax Rules and Structure

#### Using `Renderer2`

```typescript
import { Directive, ElementRef, Renderer2 } from '@angular/core';

@Directive({ selector: '[appHighlight]' })
export class HighlightDirective {
  constructor(private el: ElementRef, private renderer: Renderer2) {
    this.renderer.addClass(this.el.nativeElement, 'highlighted');
  }
}
```

#### Using jQuery for Plugin API

```typescript
this.$el.somePlugin({ option: value });   // plugin-specific API
```

**Rule**: Use `Renderer2` for Angular-managed DOM changes. Use jQuery only for the plugin’s API that requires direct DOM interaction.

---

### Constraints and Limitations

- **`Renderer2` does not support all DOM operations**; some advanced manipulations may require native DOM access.
- **`Renderer2` does not work in server-side rendering** for DOM manipulation.
- **The boundary must be absolute**; even a single Angular update to the jQuery container breaks the guarantee.

---

### Annotated Code Example

```typescript
import {
  Directive, ElementRef, Renderer2, AfterViewInit, OnDestroy, NgZone
} from '@angular/core';
import * as $ from 'jquery';

@Directive({ selector: '[appJqueryChart]' })
export class JqueryChartDirective implements AfterViewInit, OnDestroy {
  private $el: JQuery<HTMLElement>;

  constructor(
    private el: ElementRef,
    private renderer: Renderer2,
    private ngZone: NgZone
  ) {
    this.$el = $(this.el.nativeElement);
    // ✅ Use Renderer2 for Angular-managed styling
    this.renderer.setStyle(this.el.nativeElement, 'display', 'block');
  }

  ngAfterViewInit(): void {
    // ✅ Use jQuery only for the plugin's API
    this.ngZone.runOutsideAngular(() => {
      this.$el.someChart({ data: [] });
    });
  }

  ngOnDestroy(): void {
    this.$el.someChart('destroy');
  }
}
```

**Expected Output**
The host element receives a `display: block` style via `Renderer2`. jQuery initialises the chart plugin outside Angular’s zone. On destroy, the plugin is cleaned up.

**Why this output occurs**: `Renderer2` handles the Angular-managed style change safely. jQuery handles the plugin-specific initialisation and cleanup. The two operations are separated by concern.

---

### Real-World Cases

- **Dashboard widgets**: A `Renderer2`-styled container with a jQuery chart inside.
- **Form controls**: `Renderer2` for adding validation classes; jQuery for a plugin’s visual enhancement.
- **SSR-compatible components**: Using `Renderer2` for Angular-managed DOM changes to keep server rendering working.

---

## Core Concept 5: jQuery Plugins in Angular (Directive Wrapper)

### Definitions

**Core Definition**
Wrapping a jQuery plugin in an Angular directive means creating a directive that initialises the plugin on its host element, synchronises Angular inputs with the plugin’s options, bridges plugin events to Angular outputs, and destroys the plugin on directive destruction.

**Technical Definition**
jQuery plugins attach to DOM nodes and often add their own event listeners, timers, and DOM elements. The Angular directive wrapper pattern is: inject `ElementRef` and `NgZone`, wrap the host element with jQuery in the constructor, initialise the plugin in `ngAfterViewInit` (typically inside `ngZone.runOutsideAngular`), expose configuration via `@Input()` properties, emit events via `@Output()` properties, and destroy the plugin in `ngOnDestroy`. The directive should be applied as an attribute on the host element, and the plugin manages the DOM inside that element. This pattern keeps jQuery isolated from Angular’s change-detection system and provides a clean, reusable integration point.

**Beginner-Friendly Explanation**
A jQuery plugin is like a piece of furniture that assembles itself when you plug it in. It needs a real spot in the room (a DOM element) to stand on. Angular’s job is to build the room and the components. To bring the plugin in, you create a directive that says: “This element is the plugin’s spot.” The directive plugs the plugin into that element when the view is ready, passes configuration from Angular inputs, sends events back to Angular outputs, and tells the plugin to pack up when the directive is destroyed. If you do not tell the plugin to pack up, it stays in memory, still listening to events, and you get leaks.

---

### Purposes

- **To reuse existing jQuery plugins** without rewriting them in Angular.
- **To synchronise plugin state with Angular inputs** so the plugin reflects the component’s data.
- **To bridge plugin events to Angular outputs** so the component can respond to user interaction.
- **To clean up plugin resources** on directive destruction to prevent leaks.
- **To isolate plugin DOM changes** inside a boundary that Angular does not touch.

---

### Syntax Rules and Structure

#### Generic Directive Wrapper

```typescript
@Directive({ selector: '[appPluginName]' })
export class PluginNameDirective implements AfterViewInit, OnDestroy {
  @Input() options: PluginOptions = {};
  @Output() pluginEvent = new EventEmitter<any>();
  private $element: JQuery<HTMLElement>;

  constructor(private el: ElementRef, private ngZone: NgZone) {
    this.$element = $(this.el.nativeElement);
  }

  ngAfterViewInit(): void {
    this.ngZone.runOutsideAngular(() => {
      this.$element.pluginName({
        ...this.options,
        onEvent: (data: any) => {
          this.ngZone.run(() => this.pluginEvent.emit(data));
        }
      });
    });
  }

  ngOnDestroy(): void {
    this.$element.pluginName('destroy');
  }
}
```

---

### Constraints and Limitations

- **Plugins must provide a cleanup method** (`destroy`, `remove`, `unslick`, etc.); otherwise, manual listener removal is required.
- **Plugin options must be mapped carefully** from Angular inputs.
- **Plugin events must be bridged** to Angular outputs; they do not propagate automatically.
- **TypeScript typings** for the plugin may need to be declared manually if not provided.

---

### Annotated Code Example (Slick Carousel Wrapper)

```typescript
import {
  Directive, ElementRef, Input, AfterViewInit, OnDestroy, NgZone
} from '@angular/core';
import * as $ from 'jquery';
import 'slick-carousel';

@Directive({ selector: '[appSlickCarousel]' })
export class SlickCarouselDirective implements AfterViewInit, OnDestroy {
  @Input() slickOptions: any = {};
  private $element: JQuery<HTMLElement>;

  constructor(private el: ElementRef, private ngZone: NgZone) {
    this.$element = $(this.el.nativeElement);
  }

  ngAfterViewInit(): void {
    this.ngZone.runOutsideAngular(() => {
      this.$element.slick(this.slickOptions);
    });
  }

  ngOnDestroy(): void {
    this.$element.slick('unslick');
  }
}
```

```html
<div appSlickCarousel [slickOptions]="{ infinite: true, slidesToShow: 3 }">
  <div *ngFor="let img of images"><img [src]="img" /></div>
</div>
```

**Expected Output**
The div is enhanced by the Slick carousel plugin with the specified options. When the directive is destroyed, the carousel is unslicked and its resources are released.

**Why this output occurs**: `ngAfterViewInit` initialises Slick outside Angular’s zone. The `slickOptions` input is passed to the plugin. `ngOnDestroy` calls `unslick` to destroy the plugin.

---

### Real-World Cases

- **Admin dashboards**: Wrapping a jQuery DataTable, date picker, or autocomplete widget in an Angular directive.
- **Legacy editors**: Embedding a jQuery-based WYSIWYG editor in an Angular form.
- **Third-party widgets**: Integrating a jQuery carousel, modal, or chart library that has no Angular counterpart.

---

## Core Concept 6: jQuery Events in Angular (`NgZone`)

### Definitions

**Core Definition**
jQuery events in Angular refers to the challenge of handling events triggered by jQuery code within Angular’s change-detection system, which is powered by Zone.js. Because jQuery runs outside Angular’s zone, its events do not automatically trigger change detection.

**Technical Definition**
Angular uses Zone.js to monkey-patch asynchronous APIs (events, timers, AJAX) and to know when to run change detection. jQuery’s event system and plugins often use `setTimeout`, `setInterval`, or native event listeners that are not patched by Zone.js in the same way, or they run outside the Angular zone entirely. When a jQuery event updates data that Angular should render, the update may not be reflected in the view because change detection does not run. The bridge is to use `NgZone.run()` to re-enter Angular’s zone before updating Angular-bound data, or to use `ChangeDetectorRef.detectChanges()` to manually trigger change detection. For events that do not affect Angular’s view (e.g., purely visual plugin animations), `ngZone.runOutsideAngular()` should be used to prevent unnecessary change-detection cycles.

**Beginner-Friendly Explanation**
Angular and jQuery each have their own event system, and they do not talk to each other. If jQuery fires an event, Angular may not notice. If Angular handles an event, jQuery may not know. This is like two people in the same room using different languages — they can both hear sounds, but they do not understand each other. The bridge is `NgZone`: when jQuery needs Angular to know something, call `ngZone.run()` to step back into Angular’s world. When jQuery is doing something that should not trigger Angular updates (like a purely visual animation), call `ngZone.runOutsideAngular()` to keep Angular from wasting time on change detection.

---

### Purposes

- **To bridge jQuery plugin events to Angular outputs** so the component can respond to user interaction.
- **To trigger Angular change detection** after jQuery updates Angular-bound data.
- **To prevent unnecessary change-detection cycles** from animation-heavy plugins.
- **To avoid relying on automatic event propagation** between the two systems.
- **To maintain a single source of truth** for state, even when jQuery is involved.

---

### Syntax Rules and Structure

#### jQuery → Angular (Emit Output)

```typescript
this.$el.on("change.plugin", (e) => {
  this.ngZone.run(() => {
    this.valueChange.emit(e.target.value);
  });
});
```

#### jQuery Updates Angular Data (Trigger Change Detection)

```typescript
this.$el.on("someEvent", (data) => {
  this.ngZone.run(() => {
    this.someAngularProperty = data;   // change detection will run
  });
});
```

#### Run Outside Angular (Prevent Change Detection)

```typescript
this.ngZone.runOutsideAngular(() => {
  this.$el.someAnimationPlugin();
});
```

---

### Constraints and Limitations

- **`ngZone.run()` is asynchronous**; the callback runs inside the zone, but the surrounding code continues outside.
- **`ChangeDetectorRef.detectChanges()`** can be used for manual, synchronous change detection.
- **`runOutsideAngular` should not be used for events that affect Angular’s view**; the view will not update.

---

### Annotated Code Example

```typescript
import {
  Directive, ElementRef, Output, EventEmitter,
  AfterViewInit, OnDestroy, NgZone
} from '@angular/core';
import * as $ from 'jquery';

@Directive({ selector: '[appJquerySlider]' })
export class JquerySliderDirective implements AfterViewInit, OnDestroy {
  @Output() valueChange = new EventEmitter<number>();
  private $el: JQuery<HTMLElement>;

  constructor(private el: ElementRef, private ngZone: NgZone) {
    this.$el = $(this.el.nativeElement);
  }

  ngAfterViewInit(): void {
    this.ngZone.runOutsideAngular(() => {
      this.$el.slider({
        slide: (_event: any, ui: any) => {
          // Bridge jQuery event to Angular output inside the zone
          this.ngZone.run(() => this.valueChange.emit(ui.value));
        }
      });
    });
  }

  ngOnDestroy(): void {
    this.$el.slider('destroy');
  }
}
```

**Expected Output**
Dragging the slider emits `valueChange` with the new value, and Angular’s change detection runs because the emit happens inside `ngZone.run()`.

**Why this output occurs**: The slider plugin runs outside Angular’s zone to avoid unnecessary change detection during dragging. When the value changes, `ngZone.run()` re-enters the zone and emits the output, triggering change detection.

---

### Real-World Cases

- **Form validation**: A jQuery validation plugin firing events that must update Angular form state.
- **Custom controls**: A jQuery slider or colour picker whose value must be reflected in Angular.
- **Legacy widgets**: A jQuery autocomplete that must notify Angular when a selection is made.

---

## Core Concept 7: jQuery AJAX in Angular (`HttpClient`)

### Definitions

**Core Definition**
jQuery AJAX in Angular refers to the use of `$.ajax()`, `$.get()`, or `$.post()` inside an Angular application. While technically possible, it bypasses Angular’s `HttpClient`, interceptors, and change-detection system, and is therefore not recommended.

**Technical Definition**
Angular provides `HttpClient` (from `@angular/common/http`) as its native HTTP service. `HttpClient` returns RxJS `Observable`s, integrates with Angular’s change detection, supports HTTP interceptors for authentication and error handling, and provides type-safe responses. jQuery AJAX (`$.ajax`) returns a `jqXHR` object with callback-based success/error handling. When jQuery AJAX receives data, Angular does not automatically run change detection, so the view may not update unless `ngZone.run()` is called manually. Furthermore, jQuery AJAX does not participate in Angular’s HTTP interceptor chain, so authentication headers and error handling are not applied. The Angular community consensus is that `HttpClient` should be used for all HTTP communication in Angular applications, and jQuery AJAX should only be used temporarily during migration, with a plan to replace it.

**Beginner-Friendly Explanation**
AJAX is how your Angular component talks to the server. jQuery’s `$.ajax()` works, but Angular has its own way of doing AJAX — `HttpClient` — and it is designed to work seamlessly with the rest of Angular. When you use jQuery AJAX, Angular does not know the request happened, so the page may not update. jQuery AJAX also skips Angular’s security features, like interceptors that add authentication tokens. The rule is simple: use `HttpClient` for all HTTP calls in Angular. If you have legacy jQuery AJAX code, wrap it or replace it as part of the migration.

---

### Purposes

- **To understand why `HttpClient` is preferred over `$.ajax`** in Angular.
- **To migrate legacy jQuery AJAX calls to `HttpClient`**.
- **To ensure Angular’s change detection and interceptors work correctly**.
- **To use type-safe, Observable-based HTTP communication**.
- **To avoid callback-based AJAX** in an Observable-first framework.

---

### Syntax Rules and Structure

#### Angular `HttpClient` (Recommended)

```typescript
import { HttpClient } from '@angular/common/http';

constructor(private http: HttpClient) {}

ngOnInit(): void {
  this.http.get<Item[]>('/api/items').subscribe({
    next: (data) => (this.items = data),
    error: (err) => (this.error = err.message)
  });
}
```

#### jQuery AJAX (Not Recommended)

```typescript
$.ajax({
  url: '/api/items',
  success: (data) => {
    this.ngZone.run(() => (this.items = data));
  }
});
```

---

### Constraints and Limitations

- **`$.ajax` bypasses Angular interceptors**; authentication and error handling must be implemented manually.
- **`$.ajax` callbacks run outside Angular’s zone**; change detection may not run.
- **`HttpClient` returns Observables**; use `subscribe` or the `async` pipe.
- **`HttpClient` is type-safe**; use generics for response types.

---

### Annotated Code Example (Migration from jQuery AJAX to HttpClient)

```typescript
import { Component, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';

interface Profile {
  name: string;
  email: string;
}

@Component({ selector: 'app-profile', template: '...' })
export class ProfileComponent implements OnInit {
  profile: Profile | null = null;
  loading = true;
  error: string | null = null;

  constructor(private http: HttpClient) {}

  ngOnInit(): void {
    // ✅ Angular HttpClient: type-safe, Observable-based, interceptors work
    this.http.get<Profile>('/api/profile').subscribe({
      next: (data) => {
        this.profile = data;
        this.loading = false;
      },
      error: (err) => {
        this.error = 'Failed to load profile';
        this.loading = false;
      }
    });
  }
}
```

**Expected Output**
The component displays “Loading…” while the request is in flight. On success, it renders the profile. On failure, it displays an error message. Angular’s change detection runs automatically because `HttpClient` uses Observables inside the zone.

**Why this output occurs**: `HttpClient` integrates with Angular’s change detection and interceptors. The `subscribe` callback runs inside the zone, so assignments to `profile` and `loading` trigger a view update.

---

### Real-World Cases

- **Data-driven pages**: Fetching user profiles, product lists, or dashboards with `HttpClient`.
- **Form submission**: Sending form data via `HttpClient.post()` with automatic error handling.
- **Authenticated APIs**: Using interceptors to attach tokens and handle 401 responses.

---

## Core Concept 8: Integration Challenges

### Definitions

**Core Definition**
Integration challenges are the specific technical and architectural problems that arise when jQuery and Angular share the same runtime — including conflicting DOM ownership, change-detection bypass, separate event systems, AJAX incompatibility, and server-side rendering conflicts.

**Technical Definition**
The core challenge is **DOM ownership conflict**: Angular manages the DOM through its change-detection system and component tree, while jQuery mutates the DOM directly. When jQuery modifies a node that Angular also manages, Angular’s internal state becomes inconsistent with the actual DOM, leading to lost updates, stale data, or `ExpressionChangedAfterItHasBeenCheckedError`. Secondary challenges include: **change-detection bypass** (jQuery events run outside Angular’s zone; `NgZone.run()` or `ChangeDetectorRef.detectChanges()` is required); **event system separation** (jQuery events do not trigger Angular bindings, and vice versa); **AJAX incompatibility** (`$.ajax` bypasses `HttpClient` interceptors and change detection); **server-side rendering conflicts** (jQuery expects a physical DOM, which does not exist on the server); and **memory leaks** (plugins must be destroyed in `ngOnDestroy` to prevent listeners and timers from persisting).

**Beginner-Friendly Explanation**
The biggest challenge is that both libraries want to be the boss of the page. Angular says “I control this tree”; jQuery says “I can change anything I want.” When they disagree, bugs happen. There are smaller challenges too: Angular does not know when jQuery changes something, so the page may not update. jQuery and Angular have separate event systems, so an event in one does not automatically reach the other. jQuery AJAX skips Angular’s security and error-handling features. And jQuery does not work on the server, so server-side rendering breaks. Knowing these challenges is the first step to avoiding them — which is why the directive and boundary patterns exist.

---

### Purposes

- **To understand why jQuery and Angular conflict** at a technical level.
- **To anticipate the failure modes** of mixing the two libraries.
- **To design integration patterns** that avoid the conflicts.
- **To debug issues** that arise from DOM ownership, change detection, or event separation.
- **To make informed decisions** about migration strategy.

---

### Syntax Rules and Structure

| Challenge | Manifestation | Mitigation |
|---|---|---|
| **DOM ownership** | Angular drops updates, stale data, or `ExpressionChangedAfterItHasBeenCheckedError` | Empty container boundary; directive wrapper |
| **Change-detection bypass** | View does not update after jQuery changes data | `NgZone.run()` or `ChangeDetectorRef.detectChanges()` |
| **Event separation** | jQuery events do not trigger Angular bindings | Bridge with `EventEmitter` and `NgZone.run()` |
| **AJAX incompatibility** | Interceptors do not work; view does not update | Use `HttpClient` instead of `$.ajax` |
| **SSR conflict** | jQuery fails on the server (no DOM) | Guard with `isPlatformBrowser` |
| **Memory leaks** | Listeners and timers persist after navigation | Destroy plugins in `ngOnDestroy` |

---

### Constraints and Limitations

- **The directive wrapper adds a layer of indirection**; every plugin needs its own wrapper.
- **Change-detection bridging adds boilerplate**; every plugin event must be wired to Angular.
- **SSR guards add complexity**; `isPlatformBrowser` must be used wherever jQuery touches the DOM.

---

### Annotated Code Example (SSR Guard)

```typescript
import { Directive, ElementRef, AfterViewInit, Inject, PLATFORM_ID } from '@angular/core';
import { isPlatformBrowser } from '@angular/common';
import * as $ from 'jquery';

@Directive({ selector: '[appJqueryPlugin]' })
export class JqueryPluginDirective implements AfterViewInit {
  constructor(
    private el: ElementRef,
    @Inject(PLATFORM_ID) private platformId: Object
  ) {}

  ngAfterViewInit(): void {
    if (isPlatformBrowser(this.platformId)) {
      $(this.el.nativeElement).somePlugin();
    }
  }
}
```

**Expected Output**
The plugin is initialised only in the browser. On the server, the directive does nothing, preventing errors from the missing DOM.

**Why this output occurs**: `isPlatformBrowser` checks whether the code is running in a browser context. jQuery is skipped on the server, where the DOM does not exist.

---

### Real-World Cases

- **Angular Universal applications**: jQuery plugins must be guarded with `isPlatformBrowser`.
- **Incremental migration**: Legacy jQuery widgets coexist with new Angular components.
- **SPA route changes**: jQuery handlers and AJAX requests must be cleaned up when the route changes.

---

## Core Concept 9: Integration Best Practices

### Definitions

**Core Definition**
jQuery-in-Angular best practices are the consolidated conventions — wrap plugins in directives, initialise in `ngAfterViewInit`, destroy in `ngOnDestroy`, run outside Angular’s zone, bridge events with `NgZone`, use `HttpClient` for AJAX, guard SSR with `isPlatformBrowser`, and prefer Angular-native solutions — that make the integration safe, predictable, and maintainable.

**Technical Definition**
Best practices synthesise the individual patterns: (1) **wrap each jQuery plugin in an Angular directive** that owns an `ElementRef` and manages the plugin’s lifecycle; (2) **initialise the plugin in `ngAfterViewInit`** and **destroy it in `ngOnDestroy`**; (3) **run plugin initialisation inside `ngZone.runOutsideAngular()`** to prevent unnecessary change-detection cycles; (4) **bridge plugin events to Angular outputs** using `EventEmitter` and `NgZone.run()`; (5) **use Angular’s `HttpClient` for all AJAX** instead of `$.ajax`; (6) **guard jQuery calls with `isPlatformBrowser`** when using Angular Universal; (7) **use `Renderer2` for Angular-managed DOM changes** and jQuery only for the plugin’s API; and (8) **prefer Angular-native components** when they exist, reserving jQuery for legacy plugins with no equivalent.

**Beginner-Friendly Explanation**
Best practices are the rules that keep the integration from exploding. Give jQuery its own directive. Start it after Angular has built the view. Clean it up when the directive is removed. Run it outside Angular’s zone so Angular does not waste time checking for changes it did not cause. Wire its events to Angular deliberately. Use `HttpClient` for all HTTP calls. Guard against server-side rendering. And always ask: do I actually need jQuery here, or can Angular do this? If Angular can do it, use Angular.

---

### Purposes

- **To provide a single actionable checklist** for jQuery-in-Angular integration.
- **To prevent the most common integration failures** (DOM conflicts, leaks, change-detection mismatches).
- **To balance legacy compatibility with Angular’s reactive model**.
- **To establish a baseline for code reviews and migration planning**.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Wrapper** | Angular directive with `ElementRef` | jQuery in component constructors or `ngOnInit` |
| **Initialisation** | `ngAfterViewInit` | Before the view is rendered |
| **Cleanup** | `ngOnDestroy` | Leaving listeners and timers |
| **Zone** | `ngZone.runOutsideAngular()` for init | Running plugins inside Angular’s zone |
| **Events** | `NgZone.run()` + `EventEmitter` | Relying on automatic propagation |
| **AJAX** | `HttpClient` | `$.ajax` |
| **SSR** | `isPlatformBrowser` guard | jQuery on the server |
| **DOM changes** | `Renderer2` for Angular-managed changes | Direct jQuery for everything |

---

### Constraints and Limitations

- **Best practices are migration tools**, not a permanent architecture for new development.
- **Directive wrappers add boilerplate**; each plugin needs its own directive.
- **`NgZone` bridging requires discipline**; every plugin event must be accounted for.

---

### Annotated Code Example (Consolidated)

```typescript
import {
  Directive, ElementRef, Input, Output, EventEmitter,
  AfterViewInit, OnDestroy, NgZone, Inject, PLATFORM_ID
} from '@angular/core';
import { isPlatformBrowser } from '@angular/common';
import * as $ from 'jquery';

@Directive({ selector: '[appLegacyWidget]' })
export class LegacyWidgetDirective implements AfterViewInit, OnDestroy {
  @Input() config: any = {};
  @Output() widgetEvent = new EventEmitter<any>();
  private $el: JQuery<HTMLElement>;

  constructor(
    private el: ElementRef,
    private ngZone: NgZone,
    @Inject(PLATFORM_ID) private platformId: Object
  ) {
    this.$el = $(this.el.nativeElement);
  }

  ngAfterViewInit(): void {
    if (!isPlatformBrowser(this.platformId)) return;

    this.ngZone.runOutsideAngular(() => {
      this.$el.legacyWidget({
        ...this.config,
        onAction: (data: any) => {
          this.ngZone.run(() => this.widgetEvent.emit(data));
        }
      });
    });
  }

  ngOnDestroy(): void {
    if (isPlatformBrowser(this.platformId)) {
      this.$el.legacyWidget('destroy');
    }
  }
}
```

**Expected Output**
The widget is initialised only in the browser, outside Angular’s zone. Plugin events are bridged to Angular outputs. On destroy, the plugin is cleaned up.

**Why this output occurs**: Each best practice is applied: directive wrapper, `ngAfterViewInit`, `ngOnDestroy`, `NgZone.runOutsideAngular`, `NgZone.run` for events, `isPlatformBrowser` guard, and `EventEmitter` bridge.

---

### Real-World Cases

- **Code reviews**: A checklist ensures that new jQuery integrations follow safe patterns.
- **Migration planning**: Each wrapped plugin is a candidate for eventual replacement with an Angular component.
- **SSR applications**: Guarding jQuery with `isPlatformBrowser` keeps server rendering working.

---

## Core Concept 10: When to Avoid jQuery in Angular

### Definitions

**Core Definition**
Avoiding jQuery in Angular means recognising the situations in which using jQuery is unnecessary, counterproductive, or actively harmful, and choosing Angular’s native APIs or modern alternatives instead.

**Technical Definition**
The Angular community consensus is that jQuery should not be used in Angular applications unless absolutely necessary. The primary reason is the **change-detection and DOM ownership conflict**: jQuery manipulates the real DOM directly, while Angular manages the DOM through its component tree and change-detection system; mixing the two can break Angular’s rendering and produce hard-to-debug errors. Specific cases to avoid include: **AJAX** — use `HttpClient` instead of `$.ajax()`; **DOM manipulation** — use Angular directives (`*ngIf`, `*ngFor`), bindings, and `Renderer2` instead of `$(".selector").hide()`; **event handling** — use Angular’s `(event)` bindings instead of `$(element).on()`; **animation** — use Angular’s `@angular/animations` or CSS transitions instead of `$.animate()`; and **utility functions** — use native `Array.prototype` methods instead of `$.each()` or `$.map()`. jQuery should be reserved for **legacy plugins that have no Angular equivalent** and **incremental migration scenarios** where rewriting is not feasible. Even then, the plugin must be wrapped in a directive and confined to a boundary.

**Beginner-Friendly Explanation**
Most of what people use jQuery for in 2026 is already built into Angular or the browser. Angular handles DOM updates, events, and state. `HttpClient` handles AJAX. CSS handles animations. Native JavaScript handles array iteration. jQuery adds a second, conflicting system for things Angular already does well. The only good reason to use jQuery in an Angular app is when you have a legacy plugin that you cannot easily replace. For everything else, avoid it. As the Angular community says: “Forget about jQuery” — with Angular, you update the model, and the page updates automatically.

---

### Purposes

- **To prevent unnecessary DOM ownership conflicts**.
- **To keep the application maintainable** by using one change-detection model.
- **To reduce bundle size** by avoiding jQuery when it is not needed.
- **To use modern, purpose-built APIs** (`HttpClient`, `Renderer2`, Angular bindings).
- **To reserve jQuery for genuine legacy compatibility** rather than habit.

---

### Syntax Rules and Structure

| jQuery Use Case | Angular Alternative |
|---|---|
| `$.ajax()`, `$.get()` | `HttpClient` |
| `$(".el").hide()` | `*ngIf` / `[hidden]` / `Renderer2` |
| `$(el).on("click", fn)` | `(click)="fn()"` |
| `$.animate()` | `@angular/animations` / CSS transitions |
| `$.each()` / `$.map()` | `Array.prototype.forEach` / `.map()` |
| `$.extend()` | Spread operator / `Object.assign()` |

---

### Constraints and Limitations

- **Legacy plugins may have no Angular equivalent**; in that case, wrapping is justified.
- **Incremental migration may require temporary coexistence**; avoid jQuery for new code within the migration.
- **Some jQuery utilities have no direct native equivalent**; evaluate case by case.

---

### Annotated Code Example

```typescript
// ❌ AVOID: jQuery for AJAX in Angular
$.ajax({ url: '/api/data', success: (data) => { this.items = data; } });

// ✅ USE: HttpClient
this.http.get<Item[]>('/api/data').subscribe(data => this.items = data);

// ❌ AVOID: jQuery for DOM manipulation
$('#myDiv').hide();

// ✅ USE: Angular binding
// <div *ngIf="isVisible">…</div>

// ❌ AVOID: jQuery for events
$(element).on('click', handler);

// ✅ USE: Angular event binding
// <button (click)="handler()">Click</button>
```

**Expected Output**
The `HttpClient` version loads data into Angular’s model and triggers change detection automatically. The Angular binding hides the div through the model. The `(click)` binding handles the click through Angular’s event system.

**Why this output occurs**: Each alternative uses Angular’s native mechanism, avoiding the DOM ownership conflict and keeping the application consistent.

---

### Real-World Cases

- **New Angular development**: No jQuery at all; use Angular and modern APIs.
- **Migration projects**: New features are built in Angular; jQuery is confined to legacy directive wrappers.
- **Code reviews**: Flags jQuery usage for AJAX, events, or DOM manipulation as a candidate for replacement.

---

## References

- Using DOM APIs | Angular v18 — https://v18.angular.dev
- Angular ElementRef and jQuery integration | Stack Overflow — https://stackoverflow.com/revisions/d85aecfd-33e0-40ce-b772-6a30867f373e/view-source
- How call jQuery method in component? | Stack Overflow — https://stackoverflow.com/feeds/question/42857895
- Angular lifecycle hooks and jQuery | Stack Overflow — https://stackoverflow.com/revisions/13b809eb-5195-4c68-8ab8-ce8d29647ca9/view-source
- How to Seamlessly Integrate jQuery Plugins into Angular 16 – A Step-by-Step Guide | Plusiune Developers — https://plusiunedevelopers.com/blog/how-to-seamlessly-integrate-jquery-plugins-into-angular-16-a-stepbystep-guide
- Why not use jQuery for DOM manipulation in React and Angular | Tencent Cloud — https://cloud.tencent.cn/developer/information/为什么不推荐在react，Angular中通过jQuery等外部库进行DOM操作-video
- Do you understand that with Angular you read and write to the model... never to the page? | SSW Rules — https://github.com/sswconsulting/ssw.rules.content/blob/b6085589a185b8b90aa3728d420653f9c4b798cb/public/uploads/rules/angular-read-and-write-to-the-model-never-to-the-page/rule.mdx
- Angular NgZone and jQuery events | Stack Overflow — https://stackoverflow.com/revisions/d06378dc-178b-441b-bbd2-a8377b1f27b2/view-source
- Angular wrapper component jQuery plugin example | Stack Overflow — https://stackoverflow.com/revisions/f232679e-fce4-42a7-ba7c-b5b7df8b23b0/view-source
- ng-select2 – Angular wrapper for Select2 | npm — https://www.npmjs.com/package/ng-select2
- Angular HttpClient vs jQuery AJAX | Stack Overflow — https://stackoverflow.com/revisions/1a8212e9-7c91-4b7f-b452-2415623e27f8/view-source
- Angular wrapper component jQuery plugin best practices | Stack Overflow — https://stackoverflow.com/revisions/4d8a38cf-da36-49db-aacc-21dc44107f25/view-source
- jQuery in Angular integration challenges | Stack Overflow — https://stackoverflow.com/revisions/a74da2f8-6dde-4a7a-a82e-3d842abc9a82/view-source
- Angular wrapper component jQuery plugin | Stack Overflow — https://stackoverflow.com/revisions/f232679e-fce4-42a7-ba7c-b5b7df8b23b0/view-source