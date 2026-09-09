# Day 1 — Build a Laravel 12 REST API (Users & Profiles)

> *Part of **Practical Laravel Backend Integration & Automation** — REST & SOAP APIs, SFTP file
> transfers, and cron-scheduled jobs (Day 1 of 3).*

**Use case for the whole course:** a **User Management** API (Application Programming Interface).
Today you start from **nothing** — you create a brand-new Laravel 12 project, build a REST
(Representational State Transfer) **CRUD** (Create, Read, Update, Delete) API for users, **seed**
realistic data (a profile for every user), **consume** an external REST API (live currency rates),
add a related **`user_profiles`** table with a migration, and test the whole thing in **Postman**.

**Stack:** Laravel 12, PHP (Hypertext Preprocessor) 8.3, Composer, MySQL (the SQL — Structured
Query Language — database, via **Laragon**), Postman, VS Code.

**New to Laravel? A 2-minute primer.** Laravel is the most popular **PHP web framework** — a
toolkit of ready-made pieces (routing, database access, mail, and more) so you don't build a web
app from scratch. It's organised around **MVC (Model–View–Controller)**:

- **Model** — represents your data, usually one class per database table (e.g. `User`). Laravel
  models use **Eloquent**, its ORM (Object-Relational Mapper), so you read and write rows as PHP
  objects instead of writing raw SQL.
- **View** — the HTML shown to a person (Laravel's templates are called **Blade**). An **API**
  skips views and returns **JSON** instead.
- **Controller** — the code that handles a request: it takes the input, uses models to touch the
  database, and returns a response.

**The request lifecycle in one line:** a request hits a **route** → the route calls a
**controller** method → the controller uses **models** to read/write the database → it returns a
**response** (JSON, for an API).

**The folders you'll actually touch:**

| Folder / file | What lives there |
|---|---|
| `app/Models/` | Eloquent models (your tables as classes) |
| `app/Http/Controllers/` | Controllers (request handlers) |
| `routes/web.php` · `routes/api.php` | Browser routes · API routes |
| `database/migrations/` | Table definitions — your schema written as code |
| `database/factories/` · `database/seeders/` | Sample/test data generators |
| `.env` | Environment config — database, mail, secrets |
| `artisan` | The command-line tool: `php artisan ...` generates code, runs migrations, serves the app |

**Terms you'll meet today:** a **migration** builds or changes a table; a **factory/seeder** fills
tables with data; an **Eloquent relationship** links tables (a user *has one* profile); **route
model binding** lets Laravel fetch a record straight from the id in the URL.

**What you build today**
- A fresh **Laravel 12 project** running on Laragon
- `GET/POST/PUT/DELETE /api/users` — a Users **CRUD** API returning JSON (JavaScript Object
  Notation)
- A **`user_profiles`** table (one-to-one with users), created by a migration
- **Seed data:** ~10 users, each with a linked profile
- `GET/PUT /api/users/{id}/profile` — read and update a user's profile
- A controller that **consumes** an external REST API (live currency exchange, plus a POST) —
  calling *someone else's* API, in contrast to the one you build
- A **Postman collection** that exercises every endpoint
- **Mailpit** configured + a **simple test email** sent into a local inbox (the base for Day 2's
  alerts)

**What is NOT in scope today:** validation, custom error handling and authentication (Laravel
Sanctum) on the **Users API you build** — that API stays focused on **building, seeding and
testing**. (The REST *consumer* you write in Topic 7 *does* validate its input and fail gracefully,
because it calls an unreliable external service — a different concern from your own CRUD endpoints.
We also send one simple test email at the end; the *automatic failure alerts* built on it are Day 2.
SOAP — Simple Object Access Protocol — is Day 2; SFTP — SSH File Transfer Protocol — and cron are
Day 3.)

**How this day builds (each topic is a prerequisite for the next — easy first):**
1. Install the tools → 2. Create the blank project + database → 3. Learn the REST vocabulary
(concept) → 4. Tour the fresh project → 5. Build the **Users CRUD API** → 6. **Seed** the users
table → 7. **Consume** an external REST API (GET + POST) → 8. Add the related **`user_profiles`**
table (migration + relationship) → 9. **Seed profiles** + add the profile endpoints → 10. **Test
everything in Postman** → 11. **Send a test email** with Mailpit (a bridge to Day 2).

> We build **bottom-up**: a working project first, then the API, then the data, then the related
> table, then the tests. Nothing is "magic" later because you built each layer yourself.

> **This guide assumes you have installed nothing.** Topic 1 installs every tool from zero.
> If your machine is already set up, skim Topic 1 and jump to the verification at its end.

---

## Topic 1 — Install and verify your tools

You need: **Laragon** (PHP + MySQL, and it bundles Composer), **Composer** (the PHP package
manager — you'll use it to create the project and manage its libraries), **Postman** (to test the
API), and **VS Code** (to edit code). Do them in order.

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
php -v          # should say PHP 8.3.x (8.2.x is also fine)
```

> If `php -v` shows an old version, redo the "Make sure PHP is 8.3" step above and open a
> **new** terminal.

### 1.2 — Install / verify Composer (PHP package manager)

Composer creates Laravel projects and downloads the libraries they depend on. **Laragon already
bundles it**, so first just check — in the Laragon terminal:

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

## Topic 2 — Create a new Laravel 12 project

**Goal:** create a fresh Laravel 12 project with Composer, point it at MySQL, and get it running
in the browser.

**Step 1 — create the project.** In the Laragon terminal, go to Laragon's web root and let
Composer build a new Laravel 12 app called `training-app`:

```bash
cd C:\laragon\www
composer create-project laravel/laravel:^12.0 training-app
cd training-app
```

> `create-project` downloads Laravel 12 and its libraries into a new `training-app` folder, copies
> `.env.example` to **`.env`**, and generates your app key automatically. It takes a minute or two.

**Step 2 — create the database.** In the main Laragon window click **Database** — this opens
**HeidiSQL** already connected to MySQL. Right-click the connection name (left panel) →
**Create new → Database** → name it **`training`** → **OK**.

> Prefer the terminal? `mysql -u root -e "CREATE DATABASE training;"` does the same thing.

**Step 3 — point Laravel at MySQL.** A fresh Laravel 12 project uses **SQLite** by default, so you
must switch it to MySQL. Open the project in VS Code (`code .`), open **`.env`**, and set the
database lines to exactly this (uncomment the `DB_*` lines if they start with `#`):

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

**Step 4 — create the tables:**

```bash
php artisan migrate      # creates the default tables (users, cache, jobs) in the training DB
```

**Step 5 — open the app.** Because the project lives in `C:\laragon\www`, **Laragon serves it
automatically** at **`http://training-app.test`** — no `php artisan serve` needed. (Laragon
auto-creates that `.test` address and points it at the project's `public/` folder.) If the address
doesn't resolve the first time, click **Menu → Reload** in Laragon (or **Start All** again).

> **`training-app.test` won't load?** Make sure Laragon's **Menu → Preferences → Auto virtual
> hosts** is on, then **Reload**. As a fallback you can use `http://localhost/training-app/public`.
> Keep a Laragon terminal open for the `php artisan` commands in later topics.

**Checkpoint ✅**
- Visiting `http://training-app.test` shows the **Laravel welcome page**.
- In HeidiSQL, the `training` database now has a `users` table (empty for now).

**Common problems**
- *`could not find driver`* → in Laragon, **Menu → PHP → Extensions**, tick **pdo_mysql**, then
  reload Laragon and open a new terminal.
- *`Access denied for user 'root'`* → your MySQL has a password; put it in `.env` → `DB_PASSWORD`.
- *`Database 'training' doesn't exist`* → you skipped Step 2, or misspelled the name in `.env`.
- *Still hitting SQLite errors* → `DB_CONNECTION` is still `sqlite`; fix `.env` (Step 3) and run
  `php artisan config:clear`.

---

## Topic 3 — REST fundamentals (concept)

**Prerequisite:** the project is running (Topic 2). **Why this comes first:** you need the REST
vocabulary *before* you build endpoints, or Topic 5 won't mean much.

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

**Status codes you should know**
- **2xx success:** 200 OK, 201 Created, 204 No Content.
- **4xx you caused it (client):** 400 bad request, 404 not found.
- **5xx we broke it (server):** 500 internal error.

**Rules of thumb**
- Responses are **JSON**, always with a sensible status code.
- The URL names the *thing* (`/users`); the *verb* says what to do. Never `/getUsers`.

**Checkpoint ✅** You can say which verb + status you'd use to create a user (POST → 201).

---

## Topic 4 — Tour a fresh Laravel project

**Prerequisite:** the REST vocabulary from Topic 3.

**Goal:** see what a brand-new Laravel 12 project already gives you — so you know what you're
building **on top of**, and what you'll **add**.

**Open the project in VS Code** (`code .` from the project folder) and look at these:

| File / folder | What it is |
|---|---|
| `app/Models/User.php` | The **User model** (Eloquent — Laravel's database layer, one class per table). Ships ready to use. |
| `database/migrations/*_create_users_table.php` | Defines the `users` columns (you ran this in Topic 2). |
| `database/factories/UserFactory.php` | A **factory** — generates fake users for seeding/testing. |
| `database/seeders/DatabaseSeeder.php` | The **seeder** — code that fills tables with sample data. |
| `routes/web.php` | Browser (HTML) routes. The welcome page lives here. |
| `routes/api.php` | **API (JSON) routes — doesn't exist yet; you create it in Topic 5.** |

**Look at the User model.** Open `app/Models/User.php`. Note three things Laravel already set up
for you:

```php
protected $fillable = ['name', 'email', 'password'];   // fields that can be mass-assigned

protected $hidden = ['password', 'remember_token'];     // never included in JSON

protected function casts(): array
{
    return [
        'email_verified_at' => 'datetime',
        'password' => 'hashed',        // any password you set is auto-hashed
    ];
}
```

- `$fillable` = the only fields you can set in bulk with `User::create([...])`.
- `$hidden` = fields stripped out of any JSON conversion (so **passwords never leak** in a
  response — even without extra work).
- `'password' => 'hashed'` = when you save a password, Laravel **hashes it automatically**. You
  never store plain text.

**Key idea:** the User model, its table and its factory already exist. Today you **add an API
layer** on top (Topic 5), **seed data** (Topic 6), and **extend it with a related table**
(Topics 8–9).

**Checkpoint ✅** You can point to the User model, the users migration, the User factory, and
where the API routes will live.

---

## Topic 5 — Build the Users CRUD API

**Goal:** create the five CRUD endpoints for users, returning JSON.

**Step 1 — enable API routing.** A fresh Laravel 12 ships without `routes/api.php`. One command
creates it and wires up the `/api` prefix:

```bash
php artisan install:api
```

- When asked to run migrations, answer **yes**.
- You now have `routes/api.php`, and its routes are automatically prefixed with `/api`.

> `install:api` also installs **Laravel Sanctum** (token authentication). We're **not using auth
> today**, so just ignore the extra files — they're there if you add login later.

**This is what Laravel generates** — the fresh `routes/api.php` starts with a single default
route (which we'll replace in Step 3):

```php
<?php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

**Step 2 — create an API controller.** The `--api` flag scaffolds the 5 CRUD methods (no
`create`/`edit` form methods, since JSON APIs don't render forms):

```bash
php artisan make:controller Api/UserController --api
```

**This is what Laravel generates** — `app/Http/Controllers/Api/UserController.php` with five
**empty** methods (note the `//` bodies and the plain `string $id` parameters). You'll replace it
in Step 4:

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function index()
    {
        //
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(Request $request)
    {
        //
    }

    /**
     * Display the specified resource.
     */
    public function show(string $id)
    {
        //
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(Request $request, string $id)
    {
        //
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(string $id)
    {
        //
    }
}
```

**Step 3 — register the routes.** Open `routes/api.php` (the default file from Step 1) and add the
two **marked** lines, so the whole file reads:

```php
<?php

use App\Http\Controllers\Api\UserController;   // ← add this
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');

Route::apiResource('users', UserController::class);   // ← add this
```

Confirm the routes exist:

```bash
php artisan route:list --path=api
```

**Step 4 — fill in the controller.** Replace the whole scaffold from Step 2 with the version
below. Two things change from the generated file: the empty `//` bodies get real code, and the
`string $id` parameters become **`User $user`** (route model binding — Laravel fetches the record
for you):

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;

class UserController extends Controller
{
    // GET /api/users
    public function index()
    {
        return User::all();                 // Laravel auto-converts to JSON (200)
    }

    // POST /api/users
    public function store(Request $request)
    {
        // $fillable limits which fields are saved; the 'hashed' cast hashes the password for us
        $user = User::create($request->only(['name', 'email', 'password']));

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

**What "route model binding" does:** because the parameter is type-hinted `User $user`, Laravel
turns `/api/users/5` into `User::findOrFail(5)` automatically. If the user doesn't exist, an API
(JSON) request gets a clean **404** with no extra work.

**Step 5 — a quick smoke test in Postman.** (Full testing is Topic 10 — this is just to prove the
endpoint responds.)
1. New request: **POST** `http://training-app.test/api/users`.
2. **Body** tab → **raw** → set the dropdown to **JSON** → paste:
   ```json
   { "name": "Smoke Test", "email": "smoke@test.com", "password": "password123" }
   ```
3. **Send** → you get **201** and the new user as JSON. Notice **no password field** — that's
   `$hidden` working.
4. **GET** `http://training-app.test/api/users` → returns a JSON array containing that user.

**Checkpoint ✅** `POST /api/users` creates a user (201) and `GET /api/users` lists it — the CRUD
API works. (The table is nearly empty; we bulk-fill it next.)

> ⚠️ There's deliberately **no input validation** today — send well-formed JSON. Validation is a
> Day-2-and-beyond concern; today is about building and testing the shape of the API.

---

## Topic 6 — Seed the users table

**Goal:** fill the `users` table with realistic sample data using Laravel's **factory + seeder**,
so the API has plenty to return.

**Concept.** A **factory** describes how to build one fake record; a **seeder** decides how many
to create and with what data. The `UserFactory` already exists — you just tell the seeder to use
it.

**Step 1 — edit the seeder.** Open `database/seeders/DatabaseSeeder.php` and replace the `run`
method so it creates a known admin plus ten random users:

```php
use App\Models\User;

public function run(): void
{
    // A known account you'll reuse on Day 2
    User::factory()->create([
        'name'  => 'Admin',
        'email' => 'admin@test.com',
    ]);

    // Ten more random users
    User::factory(10)->create();
}
```

> The factory sets a default password of `password` (already hashed) and a random name/email for
> each user. `admin@test.com` gets a known email so later exercises can look it up.

**Step 2 — reset and seed the database:**

```bash
php artisan migrate:fresh --seed
```

> `migrate:fresh` drops every table and re-creates them, then `--seed` runs your seeder. Use it
> whenever you want a clean, known dataset. **It wipes all data** — which is exactly what we want
> here.

**Checkpoint ✅**
- In HeidiSQL, the `users` table has **11 rows** (Admin + 10).
- `GET http://training-app.test/api/users` returns all 11 as JSON, with `admin@test.com` among them.

**Common problems**
- *`Class "App\Models\User" not found`* → you forgot the `use App\Models\User;` line at the top of
  `DatabaseSeeder.php`.
- *Only 1 user appears* → you didn't replace the default `run()` body, or didn't re-run
  `migrate:fresh --seed`.

---

## Topic 7 — Consume a REST API: GET and POST

**Goal:** so far you *built* an API; now call **someone else's** REST API from Laravel — the
simplest kind of integration (an HTTP request and a JSON reply). We build it up **one step at a
time**: first the barest call, then inputs, then error handling — and finish by **sending** data
with a POST.

**The services we'll call** (both free, **no key, no limit**):
- **Frankfurter** — currency exchange rates (for the GET). Base URL `https://api.frankfurter.dev/v2`,
  docs at <https://frankfurter.dev>. We use `/rates?base=USD&quotes=MYR`, which replies with a
  **list of rows**, one per quoted currency:
  `[ { "date": "2026-01-05", "base": "USD", "quote": "MYR", "rate": 4.05 } ]`
- **JSONPlaceholder** — a fake API that pretends to save what you send and echoes it back with a
  new `id` (for the POST). `https://jsonplaceholder.typicode.com/posts`

**Laravel's HTTP client.** Laravel ships one (`Illuminate\Support\Facades\Http`) — nothing to
install. `Http::get()` / `Http::post()` make the call; you read the reply as an array.

---

### Part 1 — the simplest possible call

Make the call and hand back whatever comes out — no inputs, no error handling yet.

```bash
php artisan make:controller Api/ExchangeRateController
```

`app/Http/Controllers/Api/ExchangeRateController.php`:

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Http;

class ExchangeRateController extends Controller
{
    public function show()
    {
        // Call the API and return its JSON exactly as it came back
        $response = Http::get('https://api.frankfurter.dev/v2/rates?base=USD&quotes=MYR');

        return $response->json();
    }
}
```

Route it in `routes/api.php`:

```php
use App\Http\Controllers\Api\ExchangeRateController;

Route::get('/exchange', [ExchangeRateController::class, 'show']);
```

**Test.** A **GET** on `http://training-app.test/api/exchange` → you get the raw JSON back:
`[ { "date": "…", "base": "USD", "quote": "MYR", "rate": 4.05 } ]`. **That's a REST call:
one line, JSON in return.**

> **Note the shape.** `/v2/rates` always answers with a **JSON array** — one row per quoted
> currency — even when you ask for a single one. So the rate lives at `[0]['rate']`, not at a
> top-level key. (There is also `/v2/rate/USD/MYR`, which returns that same row **unwrapped** as a
> plain object. We stay on `/rates` because it takes query parameters, which is what we build next.)

---

### Part 2 — accept input and shape the response

Hard-coding `USD`/`MYR` isn't useful. Read `from`, `to` and `amount` from the query string, and
return a tidy result instead of the raw dump. Replace the method:

```php
use Illuminate\Http\Request;   // add this import at the top

public function show(Request $request)
{
    $from   = strtoupper($request->query('from', 'USD'));
    $to     = strtoupper($request->query('to', 'MYR'));
    $amount = (float) $request->query('amount', 1);

    // Pass the query as an array — Http builds "?base=…&quotes=…" for you
    $response = Http::get('https://api.frankfurter.dev/v2/rates', [
        'base'   => $from,
        'quotes' => $to,
    ]);

    // The reply is a list of rows — we asked for one currency, so take the first
    $row = $response->json()[0];

    return response()->json([
        'from'      => $from,
        'to'        => $to,
        'rate'      => $row['rate'],
        'amount'    => $amount,
        'converted' => round($amount * $row['rate'], 2),
        'as_of'     => $row['date'] ?? null,
    ]);
}
```

**Test.** `http://training-app.test/api/exchange?from=USD&to=MYR&amount=100` → a clean object with
`rate` and `converted` (amount × rate).

---

### Part 3 — validate input and handle failure

Right now a bad currency (`to=ZZZ`) or a dead service would throw an error — Frankfurter answers a
bad code with **422** and `{"status":422,"message":"invalid currency: ZZZ"}`, so `json()[0]` would
blow up on a missing index. Add **validation** and a **try/catch**, and check the rate actually came
back:

```php
public function show(Request $request)
{
    $data = $request->validate([
        'from'   => ['required', 'string', 'size:3'],
        'to'     => ['required', 'string', 'size:3'],
        'amount' => ['nullable', 'numeric', 'min:0'],
    ]);

    $from   = strtoupper($data['from']);
    $to     = strtoupper($data['to']);
    $amount = $data['amount'] ?? 1;

    try {
        $response = Http::timeout(10)->get('https://api.frankfurter.dev/v2/rates', [
            'base'   => $from,
            'quotes' => $to,
        ]);
    } catch (\Throwable $e) {
        report($e);   // log the real detail
        return response()->json(['message' => 'The exchange-rate service is unavailable.'], 503);
    }

    $row = $response->json()[0] ?? null;

    // No row back → bad currency code (Frankfurter answers 422), or an upstream problem
    if ($response->failed() || ! isset($row['rate'])) {
        return response()->json(['message' => "Could not get a rate for {$from} to {$to}."], 422);
    }

    $rate = $row['rate'];

    return response()->json([
        'from'      => $from,
        'to'        => $to,
        'rate'      => $rate,
        'amount'    => $amount,
        'converted' => round($amount * $rate, 2),
        'as_of'     => $row['date'] ?? null,
    ]);
}
```

**Test.** `?from=USD&to=MYR&amount=100` → **200**; `?from=USD&to=ZZZ` → a clean **422**, not a
crash. (This is the same "fail gracefully" habit you'll use for SOAP on Day 2.)

**Checkpoint ✅ (GET)** Each step returns more useful output than the last, ending with a validated,
crash-proof endpoint.

---

### Part 4 — send data with POST

`GET` **reads**; `POST` **sends**. Now build an endpoint that takes some data and posts it to an
external API. We'll use **JSONPlaceholder**, which pretends to create the record and echoes it back
with a new `id`.

```bash
php artisan make:controller Api/RemotePostController
```

`app/Http/Controllers/Api/RemotePostController.php`:

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

class RemotePostController extends Controller
{
    // POST /api/remote-posts   { "title": "...", "body": "..." }
    public function store(Request $request)
    {
        $data = $request->validate([
            'title' => ['required', 'string', 'max:255'],
            'body'  => ['required', 'string'],
        ]);

        try {
            // Http::post sends the array as a JSON body
            $response = Http::timeout(10)->post('https://jsonplaceholder.typicode.com/posts', [
                'title'  => $data['title'],
                'body'   => $data['body'],
                'userId' => 1,
            ]);
        } catch (\Throwable $e) {
            report($e);
            return response()->json(['message' => 'The remote API is unavailable.'], 503);
        }

        if ($response->failed()) {
            return response()->json(['message' => 'The remote API rejected the request.'], 502);
        }

        // JSONPlaceholder echoes the "created" record back with a new id
        return response()->json([
            'message' => 'Sent to the remote API.',
            'created' => $response->json(),
        ], 201);
    }
}
```

Route it:

```php
use App\Http\Controllers\Api\RemotePostController;

Route::post('/remote-posts', [RemotePostController::class, 'store']);
```

**Test in Postman.** A **POST** on `http://training-app.test/api/remote-posts`, Body → raw → JSON:
`{"title": "Hello", "body": "From Laravel"}` → **201** with `created` containing your data and a
new `id` (JSONPlaceholder returns `101`).

**Checkpoint ✅ (POST)** Your endpoint accepts JSON, forwards it to the external API with
`Http::post`, and returns the created record.

> **In the collection:** both calls are in the provided `Day1-User-API.postman_collection.json` —
> request **9** (GET exchange) and request **10** (POST remote-posts). Import it in Topic 10 to skip
> the typing.

**Common problems**
- *`Could not resolve host`* → no internet, or a proxy is blocking the API host.
- *GET gives 422 for a real currency* → check the 3-letter ISO code (`USD`, `EUR`, `MYR`, `SGD`, …).
- *POST body ignored* → in Postman set Body → **raw → JSON** (not Text/form-data).

> **Note:** consuming a REST API is the gentle version of integration. On **Day 2** you'll do the
> same job — call a remote service, map the answer — over **SOAP**, which adds an XML contract, an
> envelope and a `SOAPAction` header. Watch how much heavier it feels than these one-line REST calls.

---

## Topic 8 — A related table: user_profiles (one-to-one)

**Goal:** add a second table that **belongs to** a user, and wire up the Eloquent relationship. A
user **has one** profile (phone + bio).

**Concept.** A **one-to-one** relationship: each row in `users` has (at most) one matching row in
`user_profiles`, linked by a `user_id` column.

**Step 1 — create the migration:**

```bash
php artisan make:migration create_user_profiles_table
```

Edit the new file in `database/migrations/` — its `up()` method:

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

> `constrained()` adds the foreign key to `users`; `cascadeOnDelete()` removes a profile
> automatically when its user is deleted.

**Step 2 — create the model:**

```bash
php artisan make:model UserProfile
```

Edit `app/Models/UserProfile.php`:

```php
protected $fillable = ['user_id', 'phone', 'bio'];

public function user()
{
    return $this->belongsTo(User::class);   // a profile belongs to one user
}
```

**Step 3 — declare the other side of the relationship.** In `app/Models/User.php` add a method:

```php
public function profile()
{
    return $this->hasOne(UserProfile::class);   // a user has one profile
}
```

**Checkpoint ✅**
- The `user_profiles` table exists in HeidiSQL (empty for now).
- Both models compile: `php artisan tinker` then `App\Models\User::first()->profile` returns
  `null` (no profile yet) without error. Type `exit` to leave tinker.

---

## Topic 9 — Seed profiles + add the profile API

**Goal:** give **every** seeded user a profile, then expose endpoints to read and update it.

**Step 1 — a factory for profiles:**

```bash
php artisan make:factory UserProfileFactory
```

Edit `database/factories/UserProfileFactory.php` — its `definition()`:

```php
public function definition(): array
{
    return [
        'phone' => fake()->phoneNumber(),
        'bio'   => fake()->sentence(),
        // user_id is set automatically when we attach it to a user below
    ];
}
```

**Step 2 — seed a profile for each user.** Update `database/seeders/DatabaseSeeder.php` so every
user is created **with** a profile. Add the import and use `->has(...)`:

```php
use App\Models\User;
use App\Models\UserProfile;

public function run(): void
{
    // Admin, with a profile
    User::factory()
        ->has(UserProfile::factory(), 'profile')
        ->create(['name' => 'Admin', 'email' => 'admin@test.com']);

    // Ten more users, each with a profile
    User::factory(10)
        ->has(UserProfile::factory(), 'profile')
        ->create();
}
```

> `->has(UserProfile::factory(), 'profile')` tells Laravel: for each user, also create one linked
> `UserProfile` through the `profile()` relationship (it fills `user_id` for you).

Re-seed with the clean dataset:

```bash
php artisan migrate:fresh --seed
```

**Step 3 — a controller for the profile** (view + create/update):

```bash
php artisan make:controller Api/ProfileController
```

Edit `app/Http/Controllers/Api/ProfileController.php`:

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
        return $user->profile;   // null if none
    }

    // PUT /api/users/{user}/profile — creates it if missing, updates if it exists
    public function update(Request $request, User $user)
    {
        // updateOrCreate on the relationship sets user_id automatically
        $profile = $user->profile()->updateOrCreate(
            [],
            $request->only(['phone', 'bio'])
        );

        return response()->json($profile, 200);
    }
}
```

**Step 4 — add the routes.** In `routes/api.php` add:

```php
use App\Http\Controllers\Api\ProfileController;

Route::get('/users/{user}/profile', [ProfileController::class, 'show']);
Route::put('/users/{user}/profile', [ProfileController::class, 'update']);
```

**Checkpoint ✅**
- In HeidiSQL, `user_profiles` has **11 rows** (one per user).
- `GET http://training-app.test/api/users/1/profile` returns that user's profile (phone + bio).
- `PUT /api/users/1/profile` with `{"phone":"012-3456789","bio":"Team lead"}` returns **200** and
  the updated profile; the row changes in HeidiSQL.

**Common problems**
- *`Column not found: user_id`* → the profiles migration didn't run; re-run `php artisan migrate`
  (or `migrate:fresh --seed`).
- *A user gets two profiles* → you used `create()` instead of `updateOrCreate([], ...)` in the
  controller.
- *Profiles are empty after seeding* → you didn't pass `'profile'` as the relationship name in
  `->has(...)`, or forgot to re-run `migrate:fresh --seed`.

---

## Topic 10 — Test everything in Postman

**Goal:** exercise the full API in Postman and confirm each endpoint behaves. (First time using
Postman properly — follow closely.)

**Step 1 — import the ready-made collection.** A Postman **collection** (a saved folder of
requests) is provided so you don't type each one by hand:
**`Day1-User-API.postman_collection.json`** (in the course folder). In Postman click **Import**
(top-left) → drag the file in (or **Choose Files**) → **Import**. A collection named
**"Day 1 — User API (Laravel)"** appears in the left sidebar with all 10 requests ready to send.

**Step 2 — check the address.** The collection carries a `base_url` variable set to
`http://training-app.test`. If your app is on a different address, open the collection → the
**Variables** tab → edit `base_url` → **Save**. (There's no login/token to set — no auth today.)

**Step 3 — send each request, top to bottom**, and confirm the result. Here's what the collection
contains and what each should return:

| # | Method | URL | Body (raw → JSON) | Expect |
|---|---|---|---|---|
| 1 | GET | `{{base_url}}/api/users` | — | **200**, array of 11 users |
| 2 | POST | `{{base_url}}/api/users` | `{"name":"New Dev","email":"newdev@test.com","password":"password123"}` | **201**, the new user (no password field) |
| 3 | GET | `{{base_url}}/api/users/1` | — | **200**, one user |
| 4 | PUT | `{{base_url}}/api/users/1` | `{"name":"Renamed User"}` | **200**, updated user |
| 5 | GET | `{{base_url}}/api/users/1/profile` | — | **200**, that user's profile |
| 6 | PUT | `{{base_url}}/api/users/1/profile` | `{"phone":"012-3456789","bio":"Team lead"}` | **200**, updated profile |
| 7 | DELETE | `{{base_url}}/api/users/12` | — | **204**, empty body |
| 8 | GET | `{{base_url}}/api/users/99999` | — | **404**, "not found" JSON |
| 9 | GET | `{{base_url}}/api/exchange?from=USD&to=MYR&amount=100` | — | **200**, converted rate (Topic 7) |
| 10 | POST | `{{base_url}}/api/remote-posts` | `{"title":"Hello","body":"From Laravel"}` | **201**, created record with an `id` (Topic 7) |

> **Notes:** the imported requests already have their bodies set to **raw → JSON**. Request **2**
> creates the user that request **7** deletes — after sending **2**, copy the `id` from its
> response into request **7**'s URL (the sample uses `12`). Prefer to build the requests yourself?
> Recreate each row from the table (New request → method + `{{base_url}}` URL → Body → raw → JSON).

**Checkpoint ✅**
- Requests 1–6 return the expected **200/201** with correctly shaped JSON.
- Delete returns **204**; a missing user returns **404**.
- Passwords never appear in any response.

---

## Topic 11 — Send a test email with Mailpit (a bridge to Day 2)

**Goal:** get Laravel sending email, safely, into a local test inbox — and send your first
message. You'll build on this on Day 2 to fire **failure alerts** automatically.

**Concept.** Apps often need to send email (receipts, alerts, password resets). Testing against a
real mail server is risky — you might spam real people. **Mailpit** is a fake mail server bundled
with Laragon: your app "sends" mail and Mailpit **catches it** in a local web inbox. No account,
no external service, nothing leaves your machine.

**Step 1 — start Mailpit.** In Laragon, click **Menu → Tools → Mailpit** and choose **Start**
(some Laragon Full builds start it automatically with **Start All** — that's fine too). Open the
inbox in a browser: **`http://localhost:8025`** — you'll see an empty inbox. Mailpit receives mail
on SMTP (Simple Mail Transfer Protocol) port **1025**.

> **No Mailpit in the menu?** Make sure you installed **Laragon Full**. If your build lacks it,
> download `mailpit.exe` from `https://github.com/axllent/mailpit/releases`, drop it in
> `C:\laragon\bin\mailpit\`, and run it — same ports (1025 SMTP, 8025 web).

**Step 2 — point Laravel at Mailpit.** Open `.env` and set the mail lines (no username or
password — it's all local):

```dotenv
MAIL_MAILER=smtp
MAIL_HOST=127.0.0.1
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@training.test"
MAIL_FROM_NAME="Training App"
```

Reload the config so Laravel picks up the change:

```bash
php artisan config:clear
```

**Step 3 — add a temporary route that sends an email.** The simplest possible send is
`Mail::raw()` — a plain-text message, no template needed. Add this to `routes/web.php`:

```php
use Illuminate\Support\Facades\Mail;

Route::get('/send-test-email', function () {
    Mail::raw('Hello from your Laravel app on Day 1!', function ($message) {
        $message->to('someone@example.com')
                ->subject('Test email from Laravel');
    });

    return 'Email sent — check Mailpit at http://localhost:8025';
});
```

**Step 4 — send it.** Visit **`http://training-app.test/send-test-email`** in a browser. You'll see
the "Email sent" message.

**Step 5 — view the email in Mailpit.** Open (or refresh) the Mailpit inbox at
**`http://localhost:8025`**:
1. The new message appears at the **top of the message list** — from **Training App**
   (`hello@training.test`), to `someone@example.com`, subject **"Test email from Laravel"**.
2. **Click the message** to open it. The reading pane shows the headers (From / To / Subject) and
   the body text ("Hello from your Laravel app on Day 1!").
3. The tabs above the message let you inspect it different ways — **Text**, **HTML**, **Source**
   (the raw message), and **Raw**. Useful later for checking exactly what your app sent.
4. The **🗑 Delete** / **Delete all** button clears the inbox between tests.

**Checkpoint ✅** The email is visible in the **Mailpit inbox** (`http://localhost:8025`) within a
second or two — from "Training App", subject "Test email from Laravel", with your body text.

**Common problems**
- *Nothing arrives* → Mailpit isn't running (start it in Laragon), or you forgot
  `php artisan config:clear` after editing `.env`.
- *`Connection refused` on port 1025* → Mailpit isn't started; confirm the inbox at
  `http://localhost:8025` loads first.

> **Tidy up:** the `/send-test-email` route is only a demo — delete it now, or leave it as a
> reference.

> **Bridge to Day 2:** you've now proven Laravel can send mail. Tomorrow you turn this into an
> **automatic failure alert** — when a SOAP integration fails, Laravel emails the team, straight
> into this same Mailpit inbox.

---

## End-of-Day 1 — final working state

You should now have:
- A fresh **Laravel 12 project** at `C:\laragon\www\training-app`, connected to the `training`
  MySQL database.
- `routes/api.php` — the `users` resource plus the two `profile` routes.
- `app/Http/Controllers/Api/UserController.php` — Users CRUD.
- `app/Http/Controllers/Api/ProfileController.php` — read/update a user's profile.
- `app/Models/UserProfile.php` + the `user_profiles` table — a one-to-one relationship with users.
- `database/seeders/DatabaseSeeder.php` + `UserProfileFactory` — **11 users, each with a profile**.
- `app/Http/Controllers/Api/ExchangeRateController.php` + `RemotePostController.php` — **consuming**
  an external REST API (`GET /api/exchange`, `POST /api/remote-posts`).
- A Postman collection ("User API") that exercises every endpoint.
- **Mailpit** running, `.env` mail settings pointing at it, and a proven test-email send — ready
  for Day 2's automatic alerts.

**What ships "for free" from the framework (worth knowing)**
- Passwords are **hashed automatically** (the User model's `hashed` cast) and **never returned**
  (`$hidden`).
- `$fillable` limits mass assignment to `name`, `email`, `password`.
- Route model binding gives a clean **404** for a missing user on JSON requests.

**Stretch goals (if time remains)**
- Add input validation to `store`/`update` (Form Request classes) so bad input returns a clean
  **422** instead of a database error.
- Add token authentication with the **Sanctum** that `install:api` already installed.
- Return the profile inline with a user (eager-load `->load('profile')` and include it in the
  response).

**Tomorrow (Day 2):** consume and expose SOAP services against this same app, and email an alert
automatically when an integration fails.
