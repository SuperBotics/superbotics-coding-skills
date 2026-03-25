# CSS Coding Standards — Superbotics

Applies to `.css` and `.scss` files. SCSS is preferred for all non-trivial projects.

---

## File Structure

- One CSS/SCSS file per concern or component.
- Never mix component styles with page-level layout styles in the same file.
- File naming in kebab-case, matching the component or context: `user-profile-card.css`.

```
assets/css/
├── reset.css               # CSS reset — modified once only
├── base.css                # Global variables, typography, body defaults
├── layout/
│   ├── grid.css            # Application grid system
│   └── container.css       # Max-width wrappers
├── components/
│   ├── button.css          # All button variants
│   ├── form-field.css      # Input, label, help text styles
│   └── navigation.css      # Navigation bar styles
└── pages/
    ├── dashboard.css       # Dashboard page overrides
    └── user-profile.css    # Profile page overrides
```

---

## File Header

```css
/**
 * File: button.css
 * Purpose: Defines all visual styles for button elements across the application,
 *          including primary, secondary, danger, and disabled variants.
 * Author: Developer Name
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */
```

---

## Custom Properties (Variables)

- All design tokens must be defined as CSS custom properties in `base.css`.
- Never hardcode colour values, font sizes, or spacing values inline — always reference a variable.

```css
/* base.css */
:root {
    --color-primary:          #2563EB;
    --color-primary-hover:    #1D4ED8;
    --color-secondary:        #6B7280;
    --color-danger:           #DC2626;
    --color-text-primary:     #111827;
    --color-text-secondary:   #6B7280;
    --color-background:       #FFFFFF;
    --color-surface:          #F9FAFB;
    --color-border:           #E5E7EB;

    --font-family-base:       'Inter', sans-serif;
    --font-size-base:         16px;
    --font-size-small:        14px;
    --font-size-large:        18px;
    --font-weight-regular:    400;
    --font-weight-medium:     500;
    --font-weight-bold:       700;

    --spacing-xs:    4px;
    --spacing-sm:    8px;
    --spacing-md:    16px;
    --spacing-lg:    24px;
    --spacing-xl:    40px;

    --border-radius-sm:   4px;
    --border-radius-md:   8px;
    --border-radius-lg:   12px;

    --shadow-sm:   0 1px 2px rgba(0, 0, 0, 0.05);
    --shadow-md:   0 4px 6px rgba(0, 0, 0, 0.07);
}
```

---

## Class Naming — BEM

Use BEM: `block__element--modifier`

```css
/* Block */
.user-card { }

/* Element */
.user-card__avatar { }
.user-card__name { }
.user-card__role { }

/* Modifier */
.user-card--highlighted { }
.user-card__role--admin { }
.user-card__role--viewer { }
```

---

## Property Declaration Order

Declare CSS properties in this order within every ruleset:

1. Layout / positioning: `display`, `position`, `top`, `right`, `bottom`, `left`, `z-index`, `float`
2. Box model: `width`, `height`, `min-width`, `max-width`, `margin`, `padding`, `border`, `box-sizing`
3. Typography: `font-family`, `font-size`, `font-weight`, `line-height`, `text-align`, `color`
4. Visual: `background`, `box-shadow`, `opacity`, `border-radius`
5. Transitions / animations: `transition`, `animation`
6. Miscellaneous: `cursor`, `overflow`, `pointer-events`

```css
/* Correct property order */
.primary-button {
    /* 1. Layout */
    display: inline-flex;
    position: relative;

    /* 2. Box model */
    width: auto;
    padding: var(--spacing-sm) var(--spacing-md);
    border: 2px solid var(--color-primary);
    box-sizing: border-box;

    /* 3. Typography */
    font-family: var(--font-family-base);
    font-size: var(--font-size-base);
    font-weight: var(--font-weight-medium);
    color: var(--color-background);

    /* 4. Visual */
    background-color: var(--color-primary);
    border-radius: var(--border-radius-md);

    /* 5. Transition */
    transition: background-color 150ms ease-in-out, border-color 150ms ease-in-out;

    /* 6. Misc */
    cursor: pointer;
}

.primary-button:hover {
    background-color: var(--color-primary-hover);
    border-color: var(--color-primary-hover);
}

.primary-button:disabled {
    background-color: var(--color-secondary);
    border-color: var(--color-secondary);
    cursor: not-allowed;
    opacity: 0.6;
}
```

---

## SCSS Rules (when using SCSS)

- Nesting must not exceed 3 levels deep.
- All SCSS variables must map to or reference CSS custom properties.
- Use `@mixin` for repeated patterns. One mixin per file in `scss/mixins/`.
- Use `@extend` sparingly — prefer mixins for explicit composition.

```scss
/* Correct SCSS nesting — 2 levels max in most cases */
.navigation-bar {
    display: flex;
    background-color: var(--color-primary);

    .navigation-bar__link {
        color: var(--color-background);
        text-decoration: none;

        &:hover {
            text-decoration: underline;
        }
    }
}
```

---

## Responsive Design

- Use mobile-first media queries.
- All breakpoints must be defined as variables in `base.css`.
- Never use pixel values directly in media queries — reference breakpoint variables (or SCSS variables).

```css
/* Mobile-first breakpoints */
:root {
    --breakpoint-sm:  576px;
    --breakpoint-md:  768px;
    --breakpoint-lg:  1024px;
    --breakpoint-xl:  1280px;
}

/* Base style (mobile) */
.user-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: var(--spacing-md);
}

/* Tablet and above */
@media (min-width: 768px) {
    .user-grid {
        grid-template-columns: repeat(2, 1fr);
    }
}

/* Desktop and above */
@media (min-width: 1024px) {
    .user-grid {
        grid-template-columns: repeat(3, 1fr);
    }
}
```

---

## Forbidden Patterns

- No `!important` — ever, unless overriding a third-party library with documented reason.
- No inline styles on HTML elements — all styles in CSS files.
- No hardcoded hex, RGB, or named colour values outside of `base.css` variables.
- No IDs (`#id`) as CSS selectors — only classes and elements.
- No deeply nested selectors beyond 3 levels.
- No use of `float` for layout — use CSS Grid or Flexbox.
