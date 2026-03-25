# HTML Coding Standards — Superbotics

---

## Core Rules

- All HTML files must be valid HTML5 — start with `<!DOCTYPE html>`.
- Always specify `lang` attribute on `<html>`.
- Always include `<meta charset="UTF-8">` and a viewport meta tag.
- One HTML file = one page or one template partial. Never combine multiple pages.
- No inline styles (`style="..."`) — all styling must be in external CSS files.
- No inline event handlers (`onclick="..."`) — all behaviour must be in external JS files.

---

## File Header Comment

```html
<!--
  File: user-profile-page.html
  Purpose: Renders the full user profile page including personal details,
           account settings, and activity history sections.
  Author: Developer Name
  Created: YYYY-MM-DD
  Last Modified: YYYY-MM-DD
-->
```

---

## Document Structure

Every HTML file must follow this structure exactly:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Page description for SEO and accessibility.">
    <title>Page Title — Superbotics</title>

    <!-- External stylesheets — order: reset, base, component, page-specific -->
    <link rel="stylesheet" href="/assets/css/reset.css">
    <link rel="stylesheet" href="/assets/css/base.css">
    <link rel="stylesheet" href="/assets/css/components/navigation.css">
    <link rel="stylesheet" href="/assets/css/pages/user-profile.css">
</head>
<body>

    <header class="site-header" role="banner">
        <!-- Header content -->
    </header>

    <main class="main-content" id="main-content" role="main">
        <!-- Primary page content -->
    </main>

    <footer class="site-footer" role="contentinfo">
        <!-- Footer content -->
    </footer>

    <!-- External scripts at the end of body — defer loading where possible -->
    <script src="/assets/js/main.js" defer></script>

</body>
</html>
```

---

## Semantics

- Always use the most semantically appropriate element — never use `<div>` when a semantic element exists.
- Use `<nav>`, `<header>`, `<main>`, `<footer>`, `<article>`, `<section>`, `<aside>` appropriately.
- Use `<button>` for interactive actions — never `<div>` or `<span>` with click handlers.
- Use `<a>` only for navigation — not for buttons or form actions.
- Use `<table>` only for tabular data — never for layout.

---

## Accessibility (required, not optional)

- Every `<img>` must have a descriptive `alt` attribute. Decorative images use `alt=""`.
- Every form input must have an associated `<label>` with a matching `for` attribute.
- Interactive elements must be keyboard accessible.
- Use ARIA roles and attributes only when native HTML semantics are insufficient.
- Colour contrast must meet WCAG 2.1 AA minimum (4.5:1 for normal text).

```html
<!-- Correct — fully accessible form field -->
<div class="form-field">
    <label for="user-email-input" class="form-field__label">
        Email Address
    </label>
    <input
        type="email"
        id="user-email-input"
        name="email_address"
        class="form-field__input"
        placeholder="you@example.com"
        autocomplete="email"
        required
        aria-describedby="user-email-help"
    >
    <p id="user-email-help" class="form-field__help-text">
        We will send your account confirmation to this address.
    </p>
</div>

<!-- Incorrect — input without label -->
<input type="email" placeholder="Email">
```

---

## Class Naming

- All class names must be in kebab-case.
- Use BEM naming convention: `block__element--modifier`.
- Never use `id` attributes for styling — only for accessibility and JS anchors.

```html
<!-- BEM class structure -->
<div class="user-card">
    <div class="user-card__avatar">
        <img src="..." alt="User avatar" class="user-card__avatar-image">
    </div>
    <div class="user-card__body">
        <h2 class="user-card__name">John Doe</h2>
        <p class="user-card__role user-card__role--admin">Administrator</p>
    </div>
</div>
```

---

## Attributes

- Always use double quotes for attribute values.
- Boolean attributes must be written without a value: `required`, `disabled`, `checked`.
- Attribute order convention: `id` → `class` → `type` → `name` → `value` → `aria-*` → `data-*`.

---

## Forbidden Patterns

- No `<table>` layout.
- No inline styles.
- No inline scripts (`<script>` tags with code inside HTML body).
- No deprecated elements: `<center>`, `<font>`, `<b>` (use `<strong>`), `<i>` (use `<em>`).
- No `<br>` for spacing — use CSS margin/padding.
- No deeply nested `<div>` soup — always restructure with semantic elements.
