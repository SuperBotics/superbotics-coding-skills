---
name: superbotics-coding-standards
description: >
  Enforce and apply Superbotics organization coding standards whenever writing, reviewing, refactoring,
  or generating any code. Use this skill for ALL code generation tasks across PHP, Laravel, JavaScript,
  React, Node.js, HTML, CSS, Python, TypeScript, and SQL. Trigger whenever a developer asks for code,
  a code review, a scaffold, a module, a component, a query, a migration, a model, a controller, a view,
  an API, a utility, a test, or any code-related output. Also trigger when onboarding new developers,
  creating documentation, or auditing existing code for standards compliance. This skill must be consulted
  before writing any line of code — it is not optional.
---

# Superbotics Coding Standards Skill

This skill defines and enforces the Superbotics organization's universal coding standards.
Before writing any code, read this file fully. For language-specific rules, load the
relevant reference file from `references/`.

---

## Table of Contents

1. [Universal Rules](#universal-rules) — Apply to ALL languages and frameworks
2. [File & Module Structure](#file--module-structure)
3. [Naming Conventions](#naming-conventions)
4. [Documentation & Comments](#documentation--comments)
5. [Language Reference Files](#language-reference-files) — Load the right one for your task

---

## Universal Rules

These rules apply regardless of language, framework, or file type.
They are non-negotiable and must never be bypassed.

### File Size
- Every file must contain **at most ~200 lines of code** (excluding blank lines and comments).
- If a file approaches 200 lines, split it into smaller, focused modules before continuing.
- Never place more than one class, model, component, or major construct in a single file.

### Function Size
- Every function or method must be **approximately 20 lines or fewer**.
- If a function grows beyond 20 lines, extract sub-tasks into helper functions with descriptive names.
- A function must do exactly one thing. If you can describe it with "and", split it.

### Variable Naming
- All variable names must be **descriptive and self-explanatory**.
- Never use single-letter variables except in standard loop counters (`i`, `j`) where context is obvious.
- Never use abbreviations unless they are universally understood (e.g., `id`, `url`, `http`).
- The name must communicate both purpose and data type intent (e.g., `userEmailAddress`, `isPaymentVerified`, `orderItemList`).

### Syntax Format
- **No shorthand syntax.** Always use the full, explicit form of any language construct.
- No ternary operators for anything more than a simple value assignment.
- No implicit type coercions or language "magic" — be explicit at all times.
- No arrow-function-only style when a full function declaration adds clarity.
- Destructuring is permitted only when it does not reduce readability.

### Code Comments
- Every file must have a **file-level header comment** describing its purpose, author, and date.
- Every function/method must have a **documentation block** describing purpose, parameters, and return value.
- Inline comments are **not required** on individual lines, but are encouraged for non-obvious logic.
- Comments must describe *why*, not *what* — the code itself must be readable enough to explain the what.

### Separation of Concerns
- One file = one purpose. Never mix models, views, controllers, utilities, or configuration in one file.
- All code must follow a clear MVC or equivalent layered architecture.
- Business logic must never appear in view/template files.
- Database queries must never appear directly in controllers — always delegate to a model or repository.
- Configuration values must live in dedicated config files, never hardcoded inline.

### Code Maintainability
- Prefer clarity over cleverness. A junior developer must be able to read and understand any file.
- Avoid deep nesting — maximum 3 levels of nesting per function. Refactor with early returns.
- Magic numbers and magic strings must always be extracted into named constants.
- No dead code — remove commented-out blocks before committing.

---

## File & Module Structure

### Directory Layout (applies to all languages)

```
project-root/
├── config/         # All configuration files — one file per domain (database, mail, app, etc.)
├── models/         # Data layer — one file per entity/model
├── controllers/    # Request handling — one file per resource/domain
├── views/          # Templates/components — one file per screen or reusable component
├── services/       # Business logic — one file per service domain
├── repositories/   # Data access abstraction — one file per model
├── helpers/        # Pure utility functions — one file per utility domain
├── middleware/     # Request/response pipeline — one file per concern
├── routes/         # Route definitions — one file per route group
├── tests/          # Tests mirror the source directory structure
└── types/          # (TypeScript/typed languages) Type definitions — one file per domain
```

### Rules
- Every directory must have a `README.md` explaining its purpose and contents.
- Test files must mirror the source path: `src/services/PaymentService.js` → `tests/services/PaymentService.test.js`.
- Imports at the top of every file, grouped: standard library → third-party → internal modules.

---

## Naming Conventions

| Construct              | Convention        | Example                          |
|------------------------|-------------------|----------------------------------|
| Files (classes)        | PascalCase        | `UserController.php`             |
| Files (utilities)      | kebab-case        | `date-formatter.js`              |
| Classes                | PascalCase        | `InvoiceService`                 |
| Methods / Functions    | camelCase         | `calculateOrderTotal()`          |
| Variables              | camelCase         | `userEmailAddress`               |
| Constants              | UPPER_SNAKE_CASE  | `MAX_RETRY_COUNT`                |
| Database tables        | snake_case plural | `order_items`                    |
| Database columns       | snake_case        | `created_at`, `user_id`          |
| CSS classes            | kebab-case        | `primary-button`, `nav-header`   |
| React components       | PascalCase        | `InvoiceTable`                   |
| TypeScript interfaces  | PascalCase + `I`  | `IUserProfile`                   |
| TypeScript types       | PascalCase + `T`  | `TApiResponse`                   |
| Environment variables  | UPPER_SNAKE_CASE  | `DATABASE_HOST`                  |

---

## Documentation & Comments

### File Header Block (required in every file)

```
/**
 * File: UserController.php
 * Purpose: Handles all HTTP request routing and response logic for the User resource.
 *          Delegates business logic to UserService and data access to UserRepository.
 * Author: [Developer Name]
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */
```

### Function/Method Documentation Block (required for every function)

```
/**
 * Calculates the total price of an order including taxes and discounts.
 *
 * @param  array  $orderItems        List of order item objects with price and quantity.
 * @param  float  $discountRate      Decimal discount rate between 0 and 1 (e.g., 0.1 for 10%).
 * @param  float  $taxRate           Decimal tax rate to apply after discount.
 * @return float                     Final order total after discount and tax.
 */
```

### Inline Comments
- Use inline comments to explain *why* a specific decision was made, not to translate code into English.
- Do not comment every line — only lines where intent is non-obvious.

---

## Language Reference Files

Load the appropriate reference file before generating any code.
Each file contains syntax rules, structural patterns, and annotated examples.

| Language / Framework | Reference File                        | When to Load                                      |
|----------------------|----------------------------------------|---------------------------------------------------|
| PHP                  | `references/php.md`                   | Any `.php` file                                   |
| Laravel              | `references/laravel.md`               | Any Laravel models, controllers, routes, jobs     |
| JavaScript           | `references/javascript.md`            | Any `.js` file outside a framework context        |
| React                | `references/react.md`                 | Any `.jsx` or React component file                |
| Node.js              | `references/nodejs.md`                | Any server-side `.js` with Express or Node APIs   |
| HTML                 | `references/html.md`                  | Any `.html` or template file                      |
| CSS                  | `references/css.md`                   | Any `.css` or `.scss` file                        |
| Python               | `references/python.md`                | Any `.py` file                                    |
| TypeScript           | `references/typescript.md`            | Any `.ts` or `.tsx` file                          |
| SQL                  | `references/sql.md`                   | Any `.sql`, migration, or raw query               |

**Always load the reference file before producing any code output.**
If a task spans multiple languages (e.g., a Laravel controller + a React component), load both files.
