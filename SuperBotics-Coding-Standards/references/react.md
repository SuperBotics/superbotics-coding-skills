# React Coding Standards — Superbotics

Load `references/javascript.md` or `references/typescript.md` first, as applicable.
This file extends those rules with React-specific patterns.

---

## Directory Structure (React)

```
src/
├── components/
│   ├── common/             # Shared reusable components (Button, Modal, Input)
│   └── [FeatureName]/      # Feature-specific components grouped by domain
├── pages/                  # One file per route/page — composes components only
├── hooks/                  # Custom React hooks — one hook per file
├── context/                # React context providers — one file per context domain
├── services/               # API call functions — one file per API domain
├── store/                  # State management — one file per slice/reducer
├── types/                  # TypeScript interfaces and types — one file per domain
├── helpers/                # Pure utility functions — no React imports allowed
├── constants/              # Application-wide constants — one file per domain
└── assets/                 # Static files — images, fonts, icons
```

---

## Component Rules

- One component per file. File name must match the component name exactly.
- Only functional components — no class components.
- Component files export exactly one component as the **default export**.
- Helper functions used only by one component live in the same file, below the component.
- Shared helpers must be extracted to `helpers/`.

```jsx
/**
 * File: UserProfileCard.jsx
 * Purpose: Displays a user's profile summary including avatar, name, and role badge.
 *          Receives all data via props — contains no data-fetching logic.
 * Author: Developer Name
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */

import React from 'react';
import PropTypes from 'prop-types';

import RoleBadge from '../common/RoleBadge';
import UserAvatar from '../common/UserAvatar';

/**
 * Renders a card displaying core user profile information.
 *
 * @param {Object}  props               - Component props.
 * @param {string}  props.fullName      - The user's full display name.
 * @param {string}  props.emailAddress  - The user's email address.
 * @param {string}  props.roleName      - The user's role label (e.g., 'Admin', 'Editor').
 * @param {string}  props.avatarUrl     - URL to the user's profile avatar image.
 * @returns {JSX.Element}               - Rendered user profile card.
 */
function UserProfileCard({ fullName, emailAddress, roleName, avatarUrl }) {
    return (
        <div className="user-profile-card">
            <UserAvatar imageUrl={avatarUrl} altText={`${fullName} avatar`} />
            <div className="user-profile-card__details">
                <h2 className="user-profile-card__name">{fullName}</h2>
                <p className="user-profile-card__email">{emailAddress}</p>
                <RoleBadge roleName={roleName} />
            </div>
        </div>
    );
}

UserProfileCard.propTypes = {
    fullName:     PropTypes.string.isRequired,
    emailAddress: PropTypes.string.isRequired,
    roleName:     PropTypes.string.isRequired,
    avatarUrl:    PropTypes.string.isRequired,
};

export default UserProfileCard;
```

---

## Props

- Always define `PropTypes` for JavaScript components or TypeScript interfaces for TS components.
- Destructure props in the function signature — never access `props.value` inside the body.
- Never mutate props — they are read-only.
- Provide `defaultProps` or default parameter values for optional props.

---

## Hooks

- One custom hook per file, named `use[Purpose].js` (e.g., `useUserProfile.js`).
- Hooks must not contain JSX — they return data and handlers only.
- Never call hooks conditionally — always at the top level of the component.
- Extract all reusable hook logic from components into the `hooks/` directory.

```jsx
/**
 * File: useUserProfile.js
 * Purpose: Fetches and manages the state for a single user's profile data.
 *          Exposes loading, error, and resolved user profile data.
 * Author: Developer Name
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */

import { useState, useEffect } from 'react';
import { fetchUserProfileById } from '../services/user-api-service';

/**
 * Fetches user profile data for a given user ID.
 *
 * @param {number} userId  - The ID of the user whose profile is to be fetched.
 * @returns {{ userProfile: Object|null, isLoading: boolean, errorMessage: string|null }}
 */
function useUserProfile(userId) {
    const [userProfile, setUserProfile] = useState(null);
    const [isLoading, setIsLoading] = useState(true);
    const [errorMessage, setErrorMessage] = useState(null);

    useEffect(function fetchProfileOnMount() {
        setIsLoading(true);
        setErrorMessage(null);

        fetchUserProfileById(userId)
            .then(function handleFetchSuccess(profileData) {
                setUserProfile(profileData);
            })
            .catch(function handleFetchError(fetchError) {
                setErrorMessage('Failed to load user profile. Please try again.');
                console.error('User profile fetch error:', fetchError);
            })
            .finally(function clearLoadingState() {
                setIsLoading(false);
            });
    }, [userId]);

    return { userProfile, isLoading, errorMessage };
}

export default useUserProfile;
```

---

## State Management

- `useState` for local, component-scoped state only.
- `useContext` for shared state that does not need server sync.
- Redux (or equivalent) for global application state — one slice per domain.
- Never derive state that can be computed from existing state — use `useMemo`.

---

## Pages vs Components

| Page File (`/pages`)           | Component File (`/components`)         |
|--------------------------------|----------------------------------------|
| One per route                  | One per reusable UI element            |
| Composes components            | Receives data via props                |
| May call hooks for data        | Never fetches its own data             |
| No raw HTML layout logic       | Contains its own layout markup         |

---

## Event Handlers

- All event handler functions must be named `handle[EventAction]` (e.g., `handleFormSubmit`, `handleInputChange`).
- Never use inline arrow functions for event handlers on frequently-rendered elements.
- Always use `useCallback` when passing event handlers to memoised child components.

```jsx
// Correct
function handleFormSubmit(submitEvent) {
    submitEvent.preventDefault();
    processFormData(formValues);
}

// Incorrect — anonymous inline handler
<form onSubmit={(e) => { e.preventDefault(); processFormData(formValues); }}>
```

---

## Styling

- Use CSS Modules or BEM class names — no inline `style` props except for dynamic values.
- All class names must follow kebab-case.
- Never use `id` attributes for styling — only for accessibility.

---

## Forbidden Patterns

- No `dangerouslySetInnerHTML` without explicit sanitization.
- No direct DOM manipulation (`document.getElementById`) inside components — use refs.
- No `index` as a list `key` prop — always use a stable unique identifier.
- No business logic inside JSX markup — extract to functions above the return statement.
- No `console.log` in committed code.
