# PHP Coding Standards — Superbotics

## Core Rules

- Follow **PSR-12** as the baseline standard.
- Use `declare(strict_types=1);` at the top of every PHP file.
- Never use the closing `?>` tag in PHP-only files.
- One class per file. File name must match the class name exactly.
- All files must use UTF-8 encoding without BOM.

---

## File Header

```php
<?php

declare(strict_types=1);

/**
 * File: UserRepository.php
 * Purpose: Handles all database read/write operations for the User entity.
 *          Provides an abstraction layer between the data source and the application.
 * Author: Developer Name
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */
```

---

## Class Structure

- Classes must be named in PascalCase.
- Properties must be declared with full visibility (`public`, `protected`, `private`).
- Constructor property promotion is permitted only when it does not reduce readability.
- Always use typed properties — never declare a property without a type.

```php
class UserRepository
{
    private DatabaseConnection $databaseConnection;

    private string $tableName = 'users';

    /**
     * Initialises the repository with a database connection dependency.
     *
     * @param DatabaseConnection $databaseConnection  Active database connection instance.
     */
    public function __construct(DatabaseConnection $databaseConnection)
    {
        $this->databaseConnection = $databaseConnection;
    }
}
```

---

## Methods

- All methods must have full return type declarations.
- All parameters must have type declarations.
- Use named arguments when calling functions with more than 3 parameters.
- Never exceed ~20 lines per method — extract into private helper methods.

```php
/**
 * Retrieves a single user record by their unique identifier.
 *
 * @param  int    $userId   The unique ID of the user to retrieve.
 * @return array|null       Associative array of user data, or null if not found.
 */
public function findUserById(int $userId): array|null
{
    $query = "SELECT * FROM {$this->tableName} WHERE id = :userId LIMIT 1";

    $statement = $this->databaseConnection->prepare($query);
    $statement->bindValue(':userId', $userId, PDO::PARAM_INT);
    $statement->execute();

    $userRecord = $statement->fetch(PDO::FETCH_ASSOC);

    if ($userRecord === false) {
        return null;
    }

    return $userRecord;
}
```

---

## Variables and Constants

- All variables must be in camelCase and fully descriptive.
- Constants must be in UPPER_SNAKE_CASE and defined with `const` inside classes, or `define()` only in global scope when unavoidable.
- Never use `var` for property declarations.

```php
// Correct
private const MAX_LOGIN_ATTEMPTS = 5;
$userEmailAddress = 'user@example.com';
$isAccountActive = true;

// Incorrect — never do this
$e = 'user@example.com';
$active = true;
```

---

## Control Structures

- Always use curly braces `{}`, even for single-line `if` statements.
- No shorthand `if`: never use `if (condition) statement;` on one line without braces.
- Use early returns to avoid deep nesting.

```php
// Correct — early return pattern
public function processUserActivation(int $userId): bool
{
    $userRecord = $this->findUserById($userId);

    if ($userRecord === null) {
        return false;
    }

    if ($userRecord['is_active'] === true) {
        return false;
    }

    return $this->activateUserRecord($userId);
}

// Incorrect — avoid deep nesting
public function processUserActivation(int $userId): bool
{
    $userRecord = $this->findUserById($userId);
    if ($userRecord !== null) {
        if ($userRecord['is_active'] !== true) {
            // logic deeply nested
        }
    }
}
```

---

## Error Handling

- Always use typed exceptions — never throw the base `\Exception` directly.
- Catch specific exception types — never use a bare `catch (\Exception $e)` unless re-throwing.
- Log exceptions with full context before re-throwing or handling.

```php
/**
 * Saves a new user record to the database.
 *
 * @param  array  $userData   Associative array of validated user field values.
 * @return int                The auto-generated ID of the newly created user.
 * @throws DatabaseWriteException  When the insert operation fails.
 */
public function createUser(array $userData): int
{
    try {
        $insertId = $this->databaseConnection->insert($this->tableName, $userData);
        return $insertId;
    } catch (\PDOException $pdoException) {
        throw new DatabaseWriteException(
            'Failed to insert user record into database.',
            previous: $pdoException
        );
    }
}
```

---

## Separation of Concerns

| File Type         | Allowed Content                                              |
|-------------------|--------------------------------------------------------------|
| Model             | Property definitions, relationships, casting rules only      |
| Repository        | All database queries for one entity                         |
| Service           | Business logic, orchestration — no direct DB calls           |
| Controller        | HTTP input/output only — delegates to Service                |
| Helper            | Stateless pure functions with no dependencies               |
| Config            | Configuration arrays only — no logic                        |

---

## Forbidden Patterns

- No `eval()` — ever.
- No `extract()` — it creates invisible variables.
- No `compact()` — it is implicit and error-prone.
- No raw SQL strings inside controllers or services — use repositories.
- No `$_GET`, `$_POST`, `$_REQUEST` directly in business logic — always sanitize via a request layer.
- No supressed errors with `@` operator.
