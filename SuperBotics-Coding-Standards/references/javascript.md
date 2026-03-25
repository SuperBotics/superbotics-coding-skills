# JavaScript Coding Standards — Superbotics

Applies to all plain `.js` files not covered by a framework-specific reference (React, Node).
For React, load `references/react.md`. For Node/Express, load `references/nodejs.md`.

---

## Core Rules

- Use `'use strict';` at the top of every non-module JS file.
- Prefer ES Modules (`import`/`export`) over CommonJS (`require`/`module.exports`) in browser code.
- Never use `var` — use `const` for values that do not change, `let` for values that do.
- No implicit globals — every variable must be declared.
- All files must have a single, clear purpose — utility helpers, event handlers, or data formatters — never mixed.

---

## File Header

```javascript
/**
 * File: date-formatter.js
 * Purpose: Provides pure utility functions for formatting date values into
 *          human-readable strings for display in the UI layer.
 * Author: Developer Name
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */
```

---

## Variables

- All variables must be descriptive — communicate type intent through naming.
- Boolean variables must start with `is`, `has`, `can`, or `should`.
- Arrays must use plural names.
- Never reuse a variable for a different purpose than its declaration intent.

```javascript
// Correct
const userEmailAddress = 'user@example.com';
const isAccountVerified = true;
const orderItemList = [];
const MAX_RETRY_COUNT = 5;

// Incorrect
const e = 'user@example.com';
let flag = true;
let arr = [];
```

---

## Functions

- Every function must be declared with a full `function` keyword or an assigned named `const` — avoid anonymous functions as top-level exports.
- Arrow functions are permitted for callbacks and short inline transformations only.
- Functions must not exceed ~20 lines — extract into helpers.
- Functions must do one thing — if the name contains "and", split it.

```javascript
/**
 * Formats a raw ISO date string into a localised display string.
 *
 * @param {string} isoDateString  - ISO 8601 date string (e.g., '2024-01-15T10:30:00Z').
 * @param {string} localeCode    - BCP 47 locale code (e.g., 'en-IN', 'fr-FR').
 * @returns {string}             - Human-readable date string formatted for the given locale.
 */
function formatDateForDisplay(isoDateString, localeCode) {
    const parsedDate = new Date(isoDateString);

    if (isNaN(parsedDate.getTime())) {
        return 'Invalid date';
    }

    const formatOptions = {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
    };

    return parsedDate.toLocaleDateString(localeCode, formatOptions);
}

export { formatDateForDisplay };
```

---

## No Shorthand Constructs

The following shorthand patterns are **forbidden**:

```javascript
// Forbidden — shorthand property
const name = 'Sudarshan';
const user = { name };          // Wrong
const user = { name: name };    // Correct

// Forbidden — implicit return arrow
const double = x => x * 2;                   // Wrong
const double = function double(x) {           // Correct
    return x * 2;
};

// Forbidden — optional chaining in complex expressions
const city = user?.address?.city;            // Permitted only for simple null guard
const total = cart?.items?.reduce(...);      // Wrong — use explicit null checks

// Forbidden — nullish coalescing for non-obvious defaults
const label = value ?? 'default';           // Permitted only when clearly intentional
```

---

## Control Flow

- Always use `===` and `!==` — never `==` or `!=`.
- Always use curly braces for `if`, `for`, `while` blocks, even single-line.
- Prefer `for...of` for iterating arrays. Avoid `forEach` when the loop contains complex logic.
- Use early returns to avoid nesting beyond 3 levels.

```javascript
// Correct — early return
function getUserDisplayName(userObject) {
    if (userObject === null || userObject === undefined) {
        return 'Unknown User';
    }

    if (typeof userObject.fullName !== 'string') {
        return 'Unnamed User';
    }

    return userObject.fullName;
}
```

---

## Modules

- One module = one responsibility (e.g., `date-formatter.js`, `currency-converter.js`).
- Named exports are preferred over default exports for utility modules.
- Import statements are always grouped: browser APIs → third-party → internal.

```javascript
// Third-party
import { format } from 'date-fns';

// Internal modules
import { getUserById } from './user-repository.js';
import { formatCurrency } from '../helpers/currency-formatter.js';
```

---

## Error Handling

- Always wrap async operations in try/catch.
- Never swallow errors silently — always log or rethrow.
- Use `console.error()` with context — never `console.log()` for errors.

```javascript
/**
 * Fetches a user record from the API by ID.
 *
 * @param {number} userId  - The numeric ID of the user to retrieve.
 * @returns {Promise<Object|null>} - Resolved user object or null on failure.
 */
async function fetchUserById(userId) {
    try {
        const apiResponse = await fetch(`/api/v1/users/${userId}`);
        const responseData = await apiResponse.json();
        return responseData;
    } catch (fetchError) {
        console.error('Failed to fetch user by ID:', { userId, fetchError });
        return null;
    }
}
```

---

## Forbidden Patterns

- No `eval()` — ever.
- No `with` statement.
- No `document.write()`.
- No inline event handlers in HTML (`onclick="..."`) — always attach via `addEventListener`.
- No global variable pollution — wrap module code in a module scope.
- No `console.log` left in committed production code.
