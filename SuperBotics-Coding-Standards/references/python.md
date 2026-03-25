# Python Coding Standards — Superbotics

---

## Core Rules

- Follow **PEP 8** as the baseline standard.
- All projects must use Python 3.10 or higher.
- Use type hints on all function signatures and class properties.
- One class or one cohesive set of related pure functions per file.
- File names in snake_case: `user_repository.py`, `date_formatter.py`.

---

## File Header

```python
"""
File: user_repository.py
Purpose: Handles all database read/write operations for the User entity.
         Provides an abstraction layer between the ORM and the service layer.
Author: Developer Name
Created: YYYY-MM-DD
Last Modified: YYYY-MM-DD
"""
```

---

## Imports

Always order imports in three groups separated by blank lines:

```python
# 1. Standard library
import os
import datetime
from typing import Optional

# 2. Third-party packages
import sqlalchemy
from fastapi import HTTPException

# 3. Internal modules
from app.models.user_model import UserModel
from app.helpers.date_formatter import format_date_for_display
```

---

## Variables and Constants

- Variables in `snake_case`, descriptive and fully named.
- Module-level constants in `UPPER_SNAKE_CASE`.
- Boolean variables must start with `is_`, `has_`, `can_`, or `should_`.
- Never reuse a variable name for a different purpose.

```python
# Correct
MAX_LOGIN_ATTEMPTS: int = 5
user_email_address: str = 'user@example.com'
is_account_active: bool = True
order_item_list: list = []

# Incorrect
e = 'user@example.com'
flag = True
```

---

## Functions

- All function signatures must include type hints for every parameter and the return type.
- Functions must not exceed ~20 lines — extract into helpers.
- Functions must do one thing.
- Use descriptive verb-noun names: `fetch_user_by_id()`, `calculate_order_total()`.

```python
def fetch_user_by_id(
    database_session: DatabaseSession,
    user_id: int
) -> Optional[UserModel]:
    """
    Retrieves a single user record from the database by their unique ID.

    Args:
        database_session (DatabaseSession): The active SQLAlchemy database session.
        user_id (int): The unique identifier of the user to retrieve.

    Returns:
        Optional[UserModel]: The user model instance if found, or None.
    """
    user_record = (
        database_session
        .query(UserModel)
        .filter(UserModel.id == user_id)
        .first()
    )

    return user_record
```

---

## Classes

- One class per file.
- Use dataclasses or Pydantic models for data containers — not plain dicts.
- Always define `__init__` with type-annotated parameters.
- Class names in PascalCase.

```python
"""
File: user_service.py
Purpose: Contains all business logic for user account operations.
         Orchestrates between UserRepository and external services.
Author: Developer Name
Created: YYYY-MM-DD
Last Modified: YYYY-MM-DD
"""

from app.repositories.user_repository import UserRepository
from app.models.user_model import UserModel
from app.exceptions.user_exceptions import UserNotFoundException


class UserService:
    """
    Manages all business logic for user account operations.

    Attributes:
        user_repository (UserRepository): Repository for user data access.
    """

    def __init__(self, user_repository: UserRepository) -> None:
        """
        Initialises the UserService with required dependencies.

        Args:
            user_repository (UserRepository): Injected user data repository.
        """
        self.user_repository = user_repository

    def get_active_user_or_raise(self, user_id: int) -> UserModel:
        """
        Fetches a user and raises an exception if not found or inactive.

        Args:
            user_id (int): The unique identifier of the user.

        Returns:
            UserModel: The resolved and active user model.

        Raises:
            UserNotFoundException: If the user is not found.
        """
        user_record = self.user_repository.find_by_id(user_id)

        if user_record is None:
            raise UserNotFoundException(f'User with ID {user_id} was not found.')

        return user_record
```

---

## Error Handling

- Always define custom exception classes — one per file in `exceptions/`.
- Never use a bare `except:` clause — always catch a specific exception type.
- Always log the exception with context before re-raising.

```python
# exceptions/user_exceptions.py
class UserNotFoundException(Exception):
    """Raised when a requested user record does not exist in the database."""
    pass


class UserAccountInactiveException(Exception):
    """Raised when an operation is attempted on an inactive user account."""
    pass
```

---

## Project Structure (Python/FastAPI or Django)

```
project/
├── app/
│   ├── config/           # One config file per concern
│   ├── models/           # ORM models — one file per entity
│   ├── schemas/          # Pydantic request/response schemas — one file per domain
│   ├── repositories/     # DB queries — one file per model
│   ├── services/         # Business logic — one file per domain
│   ├── controllers/      # Route handlers — one file per resource
│   ├── middleware/        # Middleware — one file per concern
│   ├── helpers/          # Pure utility functions
│   ├── exceptions/       # Custom exception classes — one per file
│   └── constants/        # App-wide constants — one file per domain
├── tests/                # Mirrors app/ structure
├── requirements.txt
└── main.py               # App entry point only
```

---

## Forbidden Patterns

- No `print()` in production code — use the `logging` module.
- No mutable default arguments: `def func(items=[])` is wrong — use `None` and initialise inside.
- No wildcard imports: `from module import *`.
- No global state mutations inside functions.
- No `exec()` or `eval()`.
- No accessing `os.environ` directly in business logic — always use a config module.
