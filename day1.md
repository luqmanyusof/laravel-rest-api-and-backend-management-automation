# Day 1 — Laravel REST API: Validation, Error Handling & Security

> *Part of **Practical Laravel Backend Integration & Automation** — REST & SOAP APIs, SFTP file
> transfers, and cron-scheduled jobs (Day 1 of 3).*

**Use case for the whole course:** a **User Management** API (Application Programming Interface).
Today you build a full REST (Representational State Transfer) API to create, read, update and
delete users — with validation, consistent JSON (JavaScript Object Notation) errors, and
token-based login using Laravel Sanctum. You test everything in Postman.

**Stack:** Laravel 12, PHP (Hypertext Preprocessor) 8.3, Composer, MySQL (the SQL — Structured
Query Language — database, via **Laragon**), Postman, VS Code. *(A provided app is set up with
`composer install`.)*

**What you build today**
- `GET/POST/PUT/DELETE /api/users` — a validated, secured **CRUD** (Create, Read, Update, Delete)
  API
- `POST /api/register` and `POST /api/login` — issue an API token
- Clean JSON errors for 401 / 403 / 404 / 422 / 500

**What is NOT in scope today:** SOAP (Simple Object Access Protocol), email, SFTP (SSH File
Transfer Protocol), scheduling (Days 2–3).

**How this day builds (each topic is a prerequisite for the next — easy first):**
1. Install the tools → 2. Get the app running → 3. Learn the REST vocabulary (concept) →
4. Tour the app to see where REST fits → 5. Build a **plain CRUD API that just works** (no rules
yet — the easy win) → 6. Add validation → 7. Shape the JSON + handle errors cleanly →
8. Lock it down with token auth → 9. Extend it with a second, related table.

> We deliberately build **easy first**: get a working (but unprotected) API in Topic 5, then
> layer on validation, clean errors and security one topic at a time. Nothing is "magic" later
> because you built the plain version first.

> **This guide assumes you have installed nothing.** Topic 1 installs every tool from zero.
> If your machine is already set up, skim Topic 1 and jump to the verification at its end.

---

## Topic 1 — Install and verify your tools

You need: **Laragon** (PHP + MySQL, and it bundles Composer), **Composer** (the PHP package
manager — you'll use it to install the app's dependencies), **Postman** (to test the API), and
**VS Code** (to edit code). Do them in order.

### 1.1 — Install Laragon (PHP + Composer + MySQL)

Laragon is a free local development environment for Windows. Installing it gives you PHP,
Composer and a MySQL database together, so you don't install them separately.

1. Open a browser and go to **`https://laragon.org/download/`**.
2. Download **"Laragon Full"** (the big download — it includes PHP, MySQL and more).
3. Run the downloaded `.exe`. Click **Next** through the installer, keep the default install
   folder (`C:\laragon`), and click **Install**, then **Finish**. Let it launch.
4. In the Laragon window, click the big **Start All** button. Two services start —
   **Apache** and **MySQL**. When they're running you'll see them listed in green.

**Make sure PHP is version 8.2 or newer** (Laravel 12 requires it):
1. In Laragon, click **Menu → PHP → Version**. If **8.3** (or 8.2) is available, select it.
2. If only an older version is listed: **Menu → PHP → Quick add** → pick **php 8.3**, wait for
   it to download, then select it under **Menu → PHP → Version**.

**Open the Laragon terminal** (this terminal already knows where PHP, Composer and MySQL are —
always use it for this course): in the Laragon window click the **Terminal** button (or
**Menu → Terminal**).

Verify PHP in that terminal:

```bash
php -v          # must say PHP 8.3.x
```

> If `php -v` shows an old version, redo the "Make sure PHP is 8.3" step above and open a
> **new** terminal.

### 1.2 — Install / verify Composer (PHP package manager)

Composer downloads the libraries a Laravel app depends on. **Laragon already bundles it**, so
first just check — in the Laragon terminal:

```bash
composer -V     # should print "Composer version 2.x"
```

- **If you see a version, you're done — skip to 1.3.**
- **If it says "not recognised"** (you're not using the Laragon terminal, or Laragon didn't add
  it), install it standalone:
  1. Go to **`https://getcomposer.org/download/`** and download **Composer-Setup.exe**.
  2. Run it → it auto-detects your PHP (point it at Laragon's PHP if asked, e.g.
     `C:\laragon\bin\php\php-8.3.x\php.exe`) → **Next** through the defaults → **Install → Finish**.
  3. Open a **new** terminal and run `composer -V` again to confirm.

### 1.3 — Install Postman (API testing)

1. Go to **`https://www.postman.com/downloads/`** and download **Windows 64-bit**.
2. Run the installer — it installs and opens automatically (no clicking through options).
3. On first launch it asks you to sign in. You can **create a free account** (recommended, it
   saves your work) or click **"Continue without an account"** at the bottom to skip.
4. You'll land on the Postman workspace. That's all for now — you'll use it in Topic 5 onward.

### 1.4 — Install VS Code (code editor)

1. Go to **`https://code.visualstudio.com/`** and click **Download for Windows**.
2. Run the installer. On the "Select Additional Tasks" screen, **tick "Open with Code"** for
   files and folders (handy later), then **Next → Install → Finish**.
3. Open VS Code. Click the **Extensions** icon on the left (four squares), search
   **"PHP Intelephense"**, and click **Install** — this gives you PHP autocomplete and error
   highlighting.

**Checkpoint ✅ (Topic 1 complete)**
- The Laragon terminal prints a **PHP 8.3** version and a **Composer 2.x** version.
- Postman and VS Code both open.
- In Laragon, Apache and MySQL show green.

**Common problems**
- *`php` is "not recognised"* → you're using the normal Windows terminal. Use **Laragon's**
  terminal (the Terminal button), which has PHP on its PATH.
- *MySQL won't start (port 3306 in use)* → you likely have XAMPP or another MySQL running. Stop
  it, then click **Start All** in Laragon again.

---

## Topic 2 — Get the Laravel application running

**Goal:** set up the **provided app**, install its dependencies with Composer, connect it to the
database, and run it.

**What the provided app already contains (so you don't build it):** a Laravel 12 project with the
built-in `User` model + `users` migration + a seeder (a class that fills tables with sample data), and a small read-only **`/users`** web page
(a `UserWebController`, a Blade view — Laravel's HTML template format — and a `web.php` route).
You'll **add the API layer** on top
— you won't create the app from scratch. *(No provided app? See the fallback at the end of this
topic.)*

**Step 1 — put the app in place.** Unzip the provided **`training-app.zip`** into `C:\laragon\www\`
so the path is **`C:\laragon\www\training-app`**. Then, in the Laragon terminal:

```bash
cd C:\laragon\www\training-app
```

**Step 2 — install the PHP dependencies with Composer** (this downloads Laravel's libraries into
a `vendor/` folder — the app can't run without it):

```bash
composer install
```

**Step 3 — set up the environment file:**

```bash
copy .env.example .env      # create your local config
php artisan key:generate    # generate the app encryption key
```

**Step 4 — create the database.** In the main Laragon window click **Database** — this opens
**HeidiSQL** already connected to MySQL. Right-click the connection name (left panel) →
**Create new → Database** → name it **`training`** → **OK**.

> Prefer the terminal? `mysql -u root -e "CREATE DATABASE training;"` does the same thing.

**Step 5 — point Laravel at the database.** Open `.env` in VS Code (`code .`) and set:

```dotenv
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=training
DB_USERNAME=root
DB_PASSWORD=
```

> In Laragon, MySQL's user is `root` with an **empty** password. If yours has a password, put it
> in `DB_PASSWORD`.

**Step 6 — create the tables, seed users, and run:**

```bash
php artisan migrate      # creates the users table
php artisan db:seed      # adds ~10 test users + an admin (login: admin@test.com)
php artisan serve        # starts the app — leave this terminal running
```

Open a **second** Laragon terminal for the remaining commands today.

**Checkpoint ✅**
- Visiting `http://127.0.0.1:8000/users` shows the provided read-only list of seeded users.
- In HeidiSQL, the `training` database has a `users` table with rows.

**Common problems**
- *`composer install` fails* → make sure `composer -V` works (Topic 1.2) and you're **inside**
  `C:\laragon\www\training-app` (where `composer.json` lives).
- *"could not find driver"* → in Laragon, **Menu → PHP → Extensions**, tick **pdo_mysql**, then
  reload Laragon and open a new terminal.
- *"Access denied for user root"* → your MySQL has a password; put it in `.env` → `DB_PASSWORD`.
- *"No application encryption key"* → you skipped `php artisan key:generate` (Step 3).

**No provided app? (fallback)** Create a fresh Laravel 12 project instead of Steps 1–2 — it ships
the same `User` model and `users` migration (you'll just lack the read-only `/users` page):
`composer create-project laravel/laravel:^12.0 training-app`, then continue from Step 3.

---

## Topic 3 — REST fundamentals (concept)

**Prerequisite:** the app is running (Topic 2). **Why this comes first:** you need the REST
vocabulary *before* you look at the code, or the tour in Topic 4 won't mean much.

**Goal:** speak REST before writing it. Short section — read, then we code.

**A "resource"** is a thing your API exposes. Ours is **users**. REST maps HTTP (Hypertext
Transfer Protocol) verbs to actions on that resource, each at a URL (Uniform Resource Locator):

| Verb | URL | Action | Success status |
|---|---|---|---|
| GET | `/api/users` | list all users | 200 OK |
| GET | `/api/users/5` | show one user | 200 OK |
| POST | `/api/users` | create a user | 201 Created |
| PUT/PATCH | `/api/users/5` | update user 5 | 200 OK |
| DELETE | `/api/users/5` | delete user 5 | 204 No Content |

**Status codes you must know**
- **2xx success:** 200 OK, 201 Created, 204 No Content.
- **4xx you caused it (client):** 400 bad request, 401 not logged in, 403 forbidden,
  404 not found, 422 validation failed.
- **5xx we broke it (server):** 500 internal error.

**Rules of thumb**
- APIs are **stateless** — every request carries its own auth (a token), no sessions.
- Responses are **JSON**, always with a sensible status code.
- The URL names the *thing* (`/users`); the *verb* says what to do. Never `/getUsers`.

**Checkpoint ✅** You can say which verb + status you'd use to create a user (POST → 201).

---

## Topic 4 — Tour the app: where the API layer fits

**Prerequisite:** the REST vocabulary from Topic 3.

**Goal:** understand the pieces you'll touch today. You are **adding** an API next to the
existing app — not rewriting it.

**Open the project in VS Code** (`code .` from the project folder) so you can see these files.

| File / folder | What it is |
|---|---|
| `app/Models/User.php` | The User model (Eloquent — Laravel's database layer, one class per table). Maps to the `users` table. |
| `database/migrations/*_create_users_table.php` | Defines the `users` columns. |
| `routes/web.php` | Browser (HTML) routes. |
| `routes/api.php` | **API (JSON) routes — we create this in Topic 5.** |
| `app/Http/Controllers/` | Where controller classes live. |

**Look at the User model.** Open `app/Models/User.php`. Note two security-relevant lines:

```php
protected $fillable = ['name', 'email', 'password'];   // mass-assignment allow-list

protected $hidden = ['password', 'remember_token'];     // never sent in JSON
```

- `$fillable` = the only fields that can be set in bulk (stops someone injecting
  `is_admin=true`).
- `$hidden` = fields stripped out of any JSON conversion (so passwords never leak).

**See the contrast for real.** The provided app already includes a tiny **web** feature: a route
in `routes/web.php`, a `UserWebController`, and `resources/views/users.blade.php` that render the
`/users` page you saw in Topic 2. Open `http://127.0.0.1:8000/users` — that's the **same User
data returned as HTML**. Today you'll build the **API** version of the same data, returned as
**JSON**.

**Key idea:** **web routes return HTML; API routes return JSON.** They share the same models and
database. The web side is already built (reuse it as your reference); today you work almost
entirely in `routes/api.php` and a new API controller.

**Checkpoint ✅** You can point to the User model, the users migration, the existing `/users`
web page, and where the API routes will live.

---

## Topic 5 — API routes, resource controller & route model binding

**Goal:** create the five CRUD endpoints returning JSON.

**Step 1 — enable API routing + Sanctum.** Laravel 12 ships without `routes/api.php`. One command
creates it *and* installs Sanctum via Composer (you'll use Sanctum for auth in Topic 8):

```bash
php artisan install:api
```

- When asked to run migrations, answer **yes** (it adds the `personal_access_tokens` table).
- You now have `routes/api.php`, and its routes are automatically prefixed with `/api`.

**Step 2 — create an API controller.** The `--api` flag scaffolds the 5 CRUD methods (no
`create`/`edit` form methods, since JSON APIs don't render forms):

```bash
php artisan make:controller Api/UserController --api
```

**Step 3 — register the routes.** Open `routes/api.php` and add:

```php
use App\Http\Controllers\Api\UserController;

Route::apiResource('users', UserController::class);
```

Confirm the routes exist:

```bash
php artisan route:list --path=api
```

**Step 4 — fill in the controller.** Open `app/Http/Controllers/Api/UserController.php` and
replace it with:

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

class UserController extends Controller
{
    // GET /api/users
    public function index()
    {
        return User::all();               // Laravel auto-converts to JSON (200)
    }

    // POST /api/users
    public function store(Request $request)
    {
        $user = User::create([
            'name'     => $request->name,
            'email'    => $request->email,
            'password' => Hash::make($request->password),  // NEVER store plain text
        ]);

        return response()->json($user, 201);   // 201 Created
    }

    // GET /api/users/{user}  — route model binding fetches the user for us
    public function show(User $user)
    {
        return $user;
    }

    // PUT/PATCH /api/users/{user}
    public function update(Request $request, User $user)
    {
        $user->update($request->only(['name', 'email']));
        return $user;
    }

    // DELETE /api/users/{user}
    public function destroy(User $user)
    {
        $user->delete();
        return response()->json(null, 204);    // 204 No Content
    }
}
```

**What "route model binding" does:** because the parameter is type-hinted `User $user`,
Laravel turns `/api/users/5` into `User::findOrFail(5)` automatically. If the user doesn't
exist it throws a "not found" exception — which we turn into clean JSON in Topic 7.

**Checkpoint ✅** In Postman (or a browser for GET), `GET http://127.0.0.1:8000/api/users`
returns a JSON array of users. Notice **passwords are absent** — that's `$hidden` working.

> ⚠️ Right now `store` has **no validation** and **no auth**. That's intentional — we fix
> validation in Topic 6 and lock it down in Topic 8.

---

## Topic 6 — Validation with Form Requests

**Goal:** reject bad input with automatic, consistent 422 responses — without cluttering the
controller.

**Concept (30 sec):** A **Form Request** is a dedicated class that holds validation rules.
Type-hint it in the controller and Laravel validates *before* your code runs. On an API
request, a failure automatically returns **422** with a JSON list of errors.

**Step 1 — create two request classes:**

```bash
php artisan make:request StoreUserRequest
php artisan make:request UpdateUserRequest
```

**Step 2 — `app/Http/Requests/StoreUserRequest.php`:**

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;   // authorisation handled by Sanctum middleware (Topic 8)
    }

    public function rules(): array
    {
        return [
            'name'     => ['required', 'string', 'max:255'],
            'email'    => ['required', 'email', 'unique:users,email'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
        ];
    }

    public function messages(): array
    {
        return [
            'email.unique'       => 'That email address is already registered.',
            'password.confirmed' => 'The password confirmation does not match.',
        ];
    }
}
```

> `confirmed` means the request must also include a `password_confirmation` field that matches.
> `messages()` overrides the default wording.

**Step 3 — `app/Http/Requests/UpdateUserRequest.php`** (email must ignore the current user so
an unchanged email doesn't fail the unique rule):

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class UpdateUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        $userId = $this->route('user')->id;

        return [
            'name'  => ['sometimes', 'required', 'string', 'max:255'],
            'email' => ['sometimes', 'required', 'email', Rule::unique('users')->ignore($userId)],
        ];
    }
}
```

> `sometimes` = only validate the field if it's present (allows partial updates).

**Step 4 — use them in the controller.** Add the two `use` lines at the **top** of the file
(alongside the existing `use` statements, under `<?php`), then swap the two type-hints and read
the validated data:

```php
use App\Http\Requests\StoreUserRequest;
use App\Http\Requests\UpdateUserRequest;

public function store(StoreUserRequest $request)
{
    $data = $request->validated();               // only the rule'd fields, all clean
    $data['password'] = Hash::make($data['password']);

    $user = User::create($data);
    return response()->json($user, 201);
}

public function update(UpdateUserRequest $request, User $user)
{
    $user->update($request->validated());
    return $user;
}
```

**Checkpoint ✅** In Postman, `POST /api/users` with an empty body returns **422** and a JSON
`errors` object listing each missing field. A valid body returns **201** with the new user.

**Security note:** `$request->validated()` (not `$request->all()`) means only approved fields
ever reach the database — a second layer on top of `$fillable`.

---

## Topic 7 — API Resources & consistent error handling

**Goal:** (a) control exactly what JSON a user looks like, and (b) make **every** error come
back as clean JSON — never an HTML error page.

### 7a — Shape the output with an API Resource

**Concept:** an **API Resource** is a class that formats a model into JSON. It guarantees a
stable shape and lets you hide/rename fields.

```bash
php artisan make:resource UserResource
```

Edit `app/Http/Resources/UserResource.php`:

```php
public function toArray(Request $request): array
{
    return [
        'id'         => $this->id,
        'name'       => $this->name,
        'email'      => $this->email,
        'created_at' => $this->created_at->toDateTimeString(),
        // note: no password field — it can never leak from here
    ];
}
```

Use it in the controller:

```php
use App\Http\Resources\UserResource;

public function index()
{
    return UserResource::collection(User::all());
}

public function show(User $user)
{
    return new UserResource($user);
}

public function store(StoreUserRequest $request)
{
    $data = $request->validated();
    $data['password'] = Hash::make($data['password']);
    $user = User::create($data);

    return (new UserResource($user))->response()->setStatusCode(201);
}
```

### 7b — Global JSON error handling

**The problem:** by default, hitting a missing user or an unauthenticated route can return an
HTML page. API clients need JSON.

**In Laravel 12, exception handling lives in `bootstrap/app.php`.** Open it. Put the `use` lines
below at the **very top** of the file (right after `<?php`, with any existing `use` statements);
the `->withExceptions(...)` block then replaces the **existing** empty `->withExceptions(...)`
call already in that file — do not paste the `use` lines inside it:

```php
use Illuminate\Auth\AuthenticationException;
use Illuminate\Database\Eloquent\ModelNotFoundException;
use Illuminate\Http\Request;
use Illuminate\Validation\ValidationException;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Throwable;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (Throwable $e, Request $request) {
        // Only intervene for API/JSON requests; let the web app show HTML errors
        if (! $request->is('api/*') && ! $request->expectsJson()) {
            return null;
        }

        if ($e instanceof ValidationException) {
            return response()->json([
                'message' => 'The given data was invalid.',
                'errors'  => $e->errors(),
            ], 422);
        }

        if ($e instanceof AuthenticationException) {
            return response()->json(['message' => 'Unauthenticated.'], 401);
        }

        if ($e instanceof ModelNotFoundException || $e instanceof NotFoundHttpException) {
            return response()->json(['message' => 'Resource not found.'], 404);
        }

        // Catch-all: hide the real error in production, show it while developing
        return response()->json([
            'message' => config('app.debug') ? $e->getMessage() : 'Server error.',
        ], 500);
    });
})
```

**Security note:** the catch-all only reveals the real message when `APP_DEBUG=true`
(development). In production you set `APP_DEBUG=false` so internal details never leak.

**Checkpoint ✅**
- `GET /api/users/99999` (missing) → **404** `{"message":"Resource not found."}`
- `POST /api/users` with bad data → **422** with an `errors` object.
- No request ever returns an HTML error page.

---

## Topic 8 — Sanctum token authentication + Postman (hands-on finale)

**Goal:** require a valid token to manage users. Register/login to get a token, then use it on
every protected request. Prove the whole flow in Postman.

**Concept:** **Sanctum** issues a long random **token** to a logged-in user. The client sends
it on each request as `Authorization: Bearer <token>`. No token = 401.

**Step 1 — enable tokens on the User model.** In `app/Models/User.php`, add the `HasApiTokens`
trait (a trait is a bundle of reusable methods mixed into a class; running `install:api` in
Topic 5 may already have added it — if so, just confirm it's there):

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, /* ...existing traits... */;
}
```

**Step 2 — create an auth controller:**

```bash
php artisan make:controller Api/AuthController
```

`app/Http/Controllers/Api/AuthController.php`:

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\StoreUserRequest;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    // POST /api/register
    public function register(StoreUserRequest $request)
    {
        $data = $request->validated();
        $data['password'] = Hash::make($data['password']);
        $user = User::create($data);

        $token = $user->createToken('api-token')->plainTextToken;

        return response()->json(['user' => $user, 'token' => $token], 201);
    }

    // POST /api/login
    public function login(Request $request)
    {
        $request->validate([
            'email'    => ['required', 'email'],
            'password' => ['required'],
        ]);

        $user = User::where('email', $request->email)->first();

        // Same generic message whether email or password is wrong (don't leak which)
        if (! $user || ! Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        $token = $user->createToken('api-token')->plainTextToken;

        return response()->json(['user' => $user, 'token' => $token], 200);
    }

    // POST /api/logout — revoke the token used for this request
    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();
        return response()->json(['message' => 'Logged out.'], 200);
    }
}
```

**Step 3 — split public vs protected routes.** Rewrite `routes/api.php`:

```php
use App\Http\Controllers\Api\AuthController;
use App\Http\Controllers\Api\UserController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

// Public — no token needed
Route::post('/register', [AuthController::class, 'register']);
Route::post('/login',    [AuthController::class, 'login']);

// Protected — valid Sanctum token required
// (middleware = a checkpoint that runs before the controller; here it rejects requests with no valid token)
Route::middleware('auth:sanctum')->group(function () {
    Route::post('/logout', [AuthController::class, 'logout']);
    Route::get('/me', fn (Request $request) => $request->user());
    Route::apiResource('users', UserController::class);
});
```

**Step 4 — test the full flow in Postman.** (First time using Postman — follow closely.)

1. **Create a Collection** (left sidebar → **Collections → +** → name it "User API"). A
   collection is just a folder for your saved requests.
2. **Create an Environment** so you don't retype the URL and token: top-right, click the
   **Environments** icon (or **Environments** in the left sidebar) → **+** → name it "Local".
   Add two variables:
   - `base_url` with value `http://127.0.0.1:8000`
   - `token` (leave the value blank)
   Click **Save**, then **select "Local"** in the environment dropdown (top-right).
3. **Register a user.** New request (**+** tab): method **POST**, URL
   `{{base_url}}/api/register`. Go to the **Body** tab → choose **raw** → in the dropdown on
   the right change **Text** to **JSON**. Paste:
   ```json
   {
     "name": "New Dev",
     "email": "newdev@test.com",
     "password": "password123",
     "password_confirmation": "password123"
   }
   ```
   Click **Send**. You get back JSON with a `token`. *(Register a **new** email like
   `newdev@test.com` — `admin@test.com` is already seeded, so registering that one fails the
   `unique` rule with a **422**.)*
4. **Auto-save the token.** On that same request, open the **Scripts** tab → **Post-response**,
   and paste:
   ```javascript
   pm.environment.set("token", pm.response.json().token);
   ```
   Now every login/register automatically stores the token in your `{{token}}` variable.
5. **Prove protection works.** New request: **GET** `{{base_url}}/api/users`. Send it with **no
   auth** → you get **401 Unauthenticated**.
6. **Add the token.** On that request open the **Authorization** tab → **Auth Type: Bearer
   Token** → in the Token box type `{{token}}`. Send again → **200** with the user list.
7. **Exercise the rest** (all with the Bearer token set): create (POST `/api/users`) — note the
   `id` in the response — then show/update/delete **that** id (e.g. GET/PUT/DELETE
   `/api/users/12`). Don't delete the user you logged in as, or your token stops working.

> **Postman quick reference:** Body must be **raw → JSON** (not Text). Bearer token goes in the
> **Authorization** tab. A 419 error means you hit a web route by mistake — API routes under
> `/api` don't use CSRF (Cross-Site Request Forgery protection). A parse error means the body
> isn't set to JSON.

**Checkpoint ✅ (end-of-day goal)**
- Without a token, protected routes return **401**.
- With a token, full CRUD works and returns correctly shaped JSON.
- Validation errors return **422**; missing records return **404**.

---

## Topic 9 — A second table: user profiles (relationships)

**Goal:** add a simple related table to show how one resource links to another. A user **has
one** profile (phone + bio). This reinforces everything from Topics 5–8 and introduces Eloquent
**relationships**.

**Concept.** A **one-to-one** relationship: each row in `users` has (at most) one matching row
in `user_profiles`, linked by a `user_id` column.

**Step 1 — the table.** Create a migration:

```bash
php artisan make:migration create_user_profiles_table
```

Edit its `up()` method:

```php
public function up(): void
{
    Schema::create('user_profiles', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained()->cascadeOnDelete();  // links to users.id
        $table->string('phone')->nullable();
        $table->string('bio')->nullable();
        $table->timestamps();
    });
}
```

```bash
php artisan migrate
```

> `constrained()` adds the foreign key to `users`; `cascadeOnDelete()` deletes the profile
> automatically when its user is deleted.

**Step 2 — the model:**

```bash
php artisan make:model UserProfile
```

`app/Models/UserProfile.php`:

```php
protected $fillable = ['user_id', 'phone', 'bio'];

public function user()
{
    return $this->belongsTo(User::class);   // a profile belongs to one user
}
```

**Step 3 — declare the relationship on User.** In `app/Models/User.php` add:

```php
public function profile()
{
    return $this->hasOne(UserProfile::class);   // a user has one profile
}
```

**Step 4 — a controller with two endpoints** (view + create/update the profile):

```bash
php artisan make:controller Api/ProfileController
```

`app/Http/Controllers/Api/ProfileController.php`:

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;

class ProfileController extends Controller
{
    // GET /api/users/{user}/profile
    public function show(User $user)
    {
        return $user->profile;   // null if none yet
    }

    // PUT /api/users/{user}/profile  — creates it if missing, updates if it exists
    public function update(Request $request, User $user)
    {
        $data = $request->validate([
            'phone' => ['nullable', 'string', 'max:30'],
            'bio'   => ['nullable', 'string', 'max:255'],
        ]);

        // updateOrCreate on the relationship sets user_id automatically
        $profile = $user->profile()->updateOrCreate([], $data);

        return response()->json($profile, 200);
    }
}
```

**Step 5 — routes.** Add these **inside** the `auth:sanctum` group in `routes/api.php`:

```php
use App\Http\Controllers\Api\ProfileController;

Route::get('/users/{user}/profile',  [ProfileController::class, 'show']);
Route::put('/users/{user}/profile',  [ProfileController::class, 'update']);
```

**Step 6 — (nice touch) include the profile with the user.** In `UserResource` (Topic 7) add a
line so a user's JSON carries its profile when it's loaded:

```php
'profile' => $this->whenLoaded('profile'),   // only included when eager-loaded
```

Then eager-load it in `UserController::show`: `$user->load('profile');`. (Skip this step if
you're short on time — the two endpoints above are enough.)

**Checkpoint ✅**
- `PUT /api/users/1/profile` with `{"phone":"012-3456789","bio":"Team lead"}` (Bearer token
  set) returns **200** with the saved profile, and a row appears in `user_profiles`.
- `GET /api/users/1/profile` returns that profile; for a user with none it returns `null`.

**Common problems**
- *`Column not found: user_id`* → the migration didn't run; re-run `php artisan migrate`.
- *A user gets two profiles* → you used `create()` instead of `updateOrCreate([], $data)`.

---

## End-of-Day 1 — final working state

You should now have:
- `routes/api.php` — public `register`/`login`, protected `users` resource + `logout`/`me`.
- `app/Http/Controllers/Api/UserController.php` — CRUD using validated requests + resources.
- `app/Http/Controllers/Api/AuthController.php` — register/login/logout with tokens.
- `app/Http/Requests/StoreUserRequest.php`, `UpdateUserRequest.php` — validation rules.
- `app/Http/Resources/UserResource.php` — JSON shaping (no password).
- `bootstrap/app.php` — global JSON error handling for 401/404/422/500.
- `app/Models/User.php` — `HasApiTokens`, `$fillable`, `$hidden`.
- A Postman environment that stores and reuses the token.

**Security recap (what protects this API)**
- Passwords **hashed** with `Hash::make`; never stored or returned in plain text.
- `$fillable` + `$request->validated()` block mass-assignment attacks.
- `$hidden` + API Resource guarantee passwords never appear in responses.
- **Token auth** on every management route; generic login error avoids user enumeration.
- Errors return **safe JSON**; real messages hidden when `APP_DEBUG=false`.

**Stretch goals (if time remains)**
- Rate limiting: `Route::middleware(['auth:sanctum','throttle:60,1'])`.
- Paginate `index()` with `User::paginate(15)`.
- Add a `role` column and block non-admins from deleting users.

**Tomorrow (Day 2):** consume and expose SOAP services, and email an alert automatically when
an integration fails.
