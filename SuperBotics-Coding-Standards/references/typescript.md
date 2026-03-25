# TypeScript Coding Standards — Superbotics

Load `references/javascript.md` first. This file extends those rules with TypeScript-specific patterns.

---

## Core Rules

- `strict` mode must be enabled in `tsconfig.json` — no exceptions.
- Never use `any` — always define a proper type or interface.
- Never use type assertions (`as SomeType`) unless interfacing with a third-party library that requires it — always document why.
- Prefer `interface` for object shapes; use `type` for unions, intersections, and computed types.
- All exported functions and class methods must have explicit return types.

---

## tsconfig Requirements

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "strictNullChecks": true,
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler"
  }
}
```

---

## File Header

```typescript
/**
 * File: user-api-service.ts
 * Purpose: Provides typed functions for all HTTP API calls related to the User resource.
 *          Handles request construction and response parsing for the user API endpoints.
 * Author: Developer Name
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */
```

---

## Types and Interfaces

- One type/interface file per domain in `types/`.
- Interfaces named `I[PascalCase]` (e.g., `IUserProfile`).
- Type aliases named `T[PascalCase]` (e.g., `TApiResponse`).
- Enum names in PascalCase; enum values in UPPER_SNAKE_CASE.

```typescript
/**
 * File: types/user-types.ts
 * Purpose: Defines all TypeScript interfaces and types for the User domain.
 * Author: Developer Name
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */

/**
 * Represents a user account record as returned by the API.
 */
interface IUserProfile {
    readonly id: number;
    fullName: string;
    emailAddress: string;
    roleName: string;
    isActive: boolean;
    createdAt: string;
}

/**
 * Represents the structure of a standard API response wrapper.
 *
 * @template TData - The type of the data payload contained in the response.
 */
interface IApiResponse<TData> {
    message: string;
    data: TData;
    success: boolean;
}

/**
 * Enumeration of supported user role types.
 */
enum UserRole {
    ADMIN = 'ADMIN',
    EDITOR = 'EDITOR',
    VIEWER = 'VIEWER',
}

export { IUserProfile, IApiResponse, UserRole };
```

---

## Functions

- Always declare parameter types and return types explicitly.
- Generic functions must name their type parameters descriptively (e.g., `TItem`, `TResponse`).

```typescript
/**
 * Fetches a user profile from the API by their ID.
 *
 * @param {number} userId  - The unique identifier of the user to retrieve.
 * @returns {Promise<IUserProfile | null>} - Resolved user profile or null if not found.
 */
async function fetchUserProfileById(userId: number): Promise<IUserProfile | null> {
    try {
        const apiResponse = await fetch(`/api/v1/users/${userId}`);

        if (!apiResponse.ok) {
            return null;
        }

        const responseBody: IApiResponse<IUserProfile> = await apiResponse.json();
        return responseBody.data;
    } catch (fetchError: unknown) {
        console.error('Failed to fetch user profile:', { userId, fetchError });
        return null;
    }
}

export { fetchUserProfileById };
```

---

## Class Patterns

- Fully type all class properties — no implicit `any`.
- Use `private`, `protected`, `public` access modifiers on all members.
- Use `readonly` for properties that should not change after construction.

```typescript
/**
 * File: UserRepository.ts
 * Purpose: Handles all database operations for the User entity.
 * Author: Developer Name
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */

import type { DatabaseConnection } from '../types/database-types';
import type { IUserProfile } from '../types/user-types';

class UserRepository {
    private readonly databaseConnection: DatabaseConnection;
    private readonly tableName: string = 'users';

    /**
     * Initialises the repository with a database connection.
     *
     * @param {DatabaseConnection} databaseConnection - Active database connection.
     */
    public constructor(databaseConnection: DatabaseConnection) {
        this.databaseConnection = databaseConnection;
    }

    /**
     * Retrieves all active user records from the database.
     *
     * @returns {Promise<IUserProfile[]>} - Array of active user profile objects.
     */
    public async findAllActiveUsers(): Promise<IUserProfile[]> {
        const queryResult = await this.databaseConnection.query<IUserProfile>(
            `SELECT * FROM ${this.tableName} WHERE is_active = true ORDER BY created_at DESC`
        );

        return queryResult.rows;
    }
}

export { UserRepository };
```

---

## Generics

- Use generics to avoid code duplication — not to add complexity.
- Always constrain generics with `extends` when possible.

```typescript
/**
 * Wraps any value in the standard API response shape.
 *
 * @template TData - The type of the data to wrap.
 * @param {string} message  - Human-readable response message.
 * @param {TData}  data     - The payload to include in the response.
 * @returns {IApiResponse<TData>} - Typed API response object.
 */
function buildApiResponse<TData>(message: string, data: TData): IApiResponse<TData> {
    return {
        message: message,
        data: data,
        success: true,
    };
}
```

---

## Forbidden Patterns

- No `any` — use `unknown` and narrow the type explicitly.
- No non-null assertion operator (`!`) unless third-party library forces it — document why.
- No type assertions on data from external APIs — always validate and narrow.
- No `@ts-ignore` without a comment explaining the reason and a plan to remove it.
- No implicit `any` from missing types — always install `@types/` packages for third-party libraries.
