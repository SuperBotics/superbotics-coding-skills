# Laravel Coding Standards — Superbotics

Load `references/php.md` first. This file extends those rules with Laravel-specific patterns.

---

## Directory Structure (Laravel)

```
app/
├── Console/Commands/       # One command class per file
├── Exceptions/             # Custom exception classes — one per file
├── Http/
│   ├── Controllers/        # One controller per resource
│   ├── Middleware/         # One middleware per concern
│   └── Requests/           # One FormRequest per action (StoreUserRequest, UpdateUserRequest)
├── Models/                 # One Eloquent model per table
├── Repositories/           # One repository per model
├── Services/               # One service class per business domain
├── Events/                 # One event class per event
├── Listeners/              # One listener per event-listener pair
└── Policies/               # One policy per model
config/                     # One config file per domain
database/
├── migrations/             # One migration per schema change
└── seeders/                # One seeder per table or dataset
resources/
├── views/                  # Blade templates — one file per screen
└── lang/                   # Language files — one file per section
routes/
├── web.php                 # Web routes only
├── api.php                 # API routes only
└── console.php             # Artisan command routes
tests/
├── Feature/                # Feature tests mirror controller structure
└── Unit/                   # Unit tests mirror service/repository structure
```

---

## Models

- One model per Eloquent table.
- Never write query logic inside a model — use a Repository.
- Declare all fillable fields explicitly — never use `$guarded = []`.
- All casts must be explicitly typed.
- Relationships are the only methods permitted in a model file.

```php
<?php

declare(strict_types=1);

/**
 * File: User.php
 * Purpose: Eloquent model representing a user record in the users table.
 *          Defines fillable fields, casts, and Eloquent relationships only.
 * Author: Developer Name
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * The database table associated with this model.
     *
     * @var string
     */
    protected $table = 'users';

    /**
     * Fields that are mass-assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'full_name',
        'email_address',
        'hashed_password',
        'is_active',
    ];

    /**
     * Field type casts for automatic conversion.
     *
     * @var array<string, string>
     */
    protected $casts = [
        'is_active'  => 'boolean',
        'created_at' => 'datetime',
        'updated_at' => 'datetime',
    ];

    /**
     * Returns all orders that belong to this user.
     *
     * @return HasMany
     */
    public function orders(): HasMany
    {
        return $this->hasMany(Order::class, 'user_id', 'id');
    }
}
```

---

## Controllers

- One controller per resource (e.g., `UserController`, `OrderController`).
- Only HTTP-layer logic: receive request, call service, return response.
- Always use Form Requests for validation — no `$request->validate()` inside the controller.
- Always return typed responses (`JsonResponse`, `RedirectResponse`, `View`).

```php
<?php

declare(strict_types=1);

/**
 * File: UserController.php
 * Purpose: Handles HTTP requests for the User resource.
 *          Delegates all business logic to UserService.
 * Author: Developer Name
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */

namespace App\Http\Controllers;

use App\Http\Requests\StoreUserRequest;
use App\Services\UserService;
use Illuminate\Http\JsonResponse;

class UserController extends Controller
{
    private UserService $userService;

    /**
     * Injects the UserService dependency.
     *
     * @param UserService $userService  Service class handling user business logic.
     */
    public function __construct(UserService $userService)
    {
        $this->userService = $userService;
    }

    /**
     * Creates a new user account from validated request data.
     *
     * @param  StoreUserRequest $request  Validated and authorized HTTP request.
     * @return JsonResponse               JSON response with the newly created user data.
     */
    public function store(StoreUserRequest $request): JsonResponse
    {
        $validatedUserData = $request->validated();
        $createdUser = $this->userService->createNewUser($validatedUserData);

        return response()->json([
            'message' => 'User account created successfully.',
            'user'    => $createdUser,
        ], 201);
    }
}
```

---

## Services

- One service per business domain (e.g., `UserService`, `InvoiceService`).
- Services may call repositories, other services, events, and jobs.
- Never access `$_REQUEST`, `Request`, or any HTTP layer directly inside a service.
- Services must be fully testable without an HTTP context.

---

## Repositories

- One repository per Eloquent model.
- All Eloquent queries live here — never in controllers or services.
- Return plain arrays or model instances — never return query builder objects to callers.

```php
/**
 * Retrieves all active users ordered by most recently created.
 *
 * @return \Illuminate\Support\Collection  Collection of active User model instances.
 */
public function getAllActiveUsers(): \Illuminate\Support\Collection
{
    return User::query()
        ->where('is_active', true)
        ->orderBy('created_at', 'desc')
        ->get();
}
```

---

## Routes

- Group routes by domain using `Route::prefix()` and `Route::middleware()`.
- Always assign route names with `->name()`.
- API routes must always be versioned: `api/v1/users`.
- No closures in route files — always reference a controller method.

```php
// routes/api.php
Route::prefix('v1')->middleware(['auth:sanctum'])->group(function () {
    Route::prefix('users')->name('users.')->group(function () {
        Route::get('/', [UserController::class, 'index'])->name('index');
        Route::post('/', [UserController::class, 'store'])->name('store');
        Route::get('/{userId}', [UserController::class, 'show'])->name('show');
        Route::put('/{userId}', [UserController::class, 'update'])->name('update');
        Route::delete('/{userId}', [UserController::class, 'destroy'])->name('destroy');
    });
});
```

---

## Form Requests

- One FormRequest per form action — `StoreUserRequest` and `UpdateUserRequest` are separate files.
- All validation rules and authorization logic live here.
- Never duplicate validation rules — DRY applies here.

---

## Migrations

- One migration per schema change — never combine unrelated table changes.
- Always implement both `up()` and `down()` methods.
- Migration file names must describe the action: `create_users_table`, `add_email_verified_at_to_users_table`.

---

## Forbidden Patterns in Laravel

- No `DB::statement()` raw SQL in controllers or services.
- No `Model::all()` without a column filter and a limit in production queries.
- No logic in Blade templates — prepare all data in the controller.
- No `dd()`, `dump()`, or `var_dump()` in committed code.
- No hardcoded environment values — always use `config()` or `env()` in config files only.
