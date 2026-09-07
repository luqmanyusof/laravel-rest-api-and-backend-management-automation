# Day 2 — SOAP Integration & Failure Notifications

> *Part of **Practical Laravel Backend Integration & Automation** — REST & SOAP APIs, SFTP file
> transfers, and cron-scheduled jobs (Day 2 of 3).*

Today you integrate with **SOAP** (Simple Object Access Protocol) — the older, contract-based
web-service style still used by banks, government, telcos and legacy enterprise systems. You will
**consume** an external SOAP service, **expose** your own SOAP endpoint, handle **faults** safely,
and send an **email alert automatically** whenever an integration fails.

**You also start tomorrow's downloads in the background today.** The VirtualBox installer and
the Ubuntu Server ISO (the disc-image installer file) are large (~3 GB total), so you begin
downloading them early (Topic 1) so the files are ready on disk for Day 3. **No installing today
— just downloading.**

**Stack:** Laravel 12, PHP (Hypertext Preprocessor) 8.3 with `ext-soap` (PHP's built-in SOAP
extension), MySQL (Laragon), Postman (used for both REST — Representational State Transfer — and
SOAP), Mailpit (a local test email inbox built into Laragon), VirtualBox + Ubuntu 24.04 Server
(downloads for Day 3).

**What you build today**
- A service that calls an external SOAP API (Application Programming Interface) and stores the
  result in your database
- Safe fault handling: `SoapFault`, timeouts, and a simple retry
- Your own SOAP endpoint (`getUserByEmail`) exposed from Laravel, tested in Postman
- An **email alert** that fires automatically on any integration failure
- **The VirtualBox installer + Ubuntu 24.04 Server ISO downloaded** — ready to install on Day 3

**What is NOT in scope today:** REST (Day 1), SFTP (SSH File Transfer Protocol) / scheduling
(Day 3), production mail providers (we use Mailpit, a local test inbox that ships with Laragon —
no signup needed).

**How this day builds (prerequisites first, easy first):**
1. Enable SOAP + tools (setup) → 2. Learn **what SOAP is** vs REST (the vocabulary) →
3. **Consume** a SOAP service — the easy direction, and you first try it **by hand in Postman**
before writing code → 4. Make that call **robust** (faults, timeouts, retry) → 5. **Expose** your
own SOAP service — the harder direction, tackled once you understand calls → 6. Set up **email**
(a new prerequisite for alerts) → 7. Finale: **auto-email** when an integration fails.

> **Consume before expose, by-hand before code:** calling someone else's service is easier than
> building your own, so we do that first — and we always poke a service manually in Postman
> before automating it, so the code never feels like magic.

> **Carry-over:** you need Day 1's Laravel app running in Laragon — continue in the **same
> `C:\laragon\www\training-app`** you built on Day 1. If you're starting fresh on a new machine,
> redo **Day 1 Topics 2 + 5–8** (`composer create-project` → set `.env` for MySQL →
> `php artisan install:api` → `php artisan migrate:fresh --seed`) — Laragon then serves it at
> `http://training-app.test` with a seeded `admin@test.com` user, which today's SOAP examples use.

---

## Topic 1 — Enable SOAP, ready your tools & start the VM download

Three setup jobs. **Start job 1.3 (the big downloads) first so they run while you work through
the rest.** You already have Postman from Day 1 — that's your SOAP testing tool too, so there's
nothing new to install for testing.

### 1.1 — Enable the PHP SOAP extension (`ext-soap`)

SOAP ships with PHP but is often switched off. Turn it on in Laragon:

1. In the Laragon window, click **Menu → PHP → Extensions**.
2. Find **soap** (or `php_soap`) in the list and **click it so it's ticked/enabled**.
3. Click **Menu → Apache → Reload** (or **Stop All** then **Start All**).
4. **Close and reopen** the Laragon terminal.

Verify:

```bash
php -r "echo class_exists('SoapClient') ? 'SOAP OK' : 'SOAP MISSING';"
```

- If it still says `SOAP MISSING`: find the active config with `php --ini` (look at "Loaded
  Configuration File"), open that `php.ini` in VS Code, find the line `;extension=soap`, remove
  the leading semicolon so it reads `extension=soap`, save, reload Apache, and open a new
  terminal.

> **Reload Laragon after enabling the extension.** Apache (which serves `training-app.test`) only
> picks up a newly enabled extension after a reload — **Menu → Apache → Reload**, or **Stop All**
> then **Start All**. Skip this and Topic 3 fails with *Class "SoapClient" not found* even though
> the terminal check above prints `SOAP OK`.

### 1.2 — Start Mailpit (safe test email, built into Laragon)

Mailpit is a fake email server bundled with Laragon: your app "sends" mail and Mailpit catches
it in a local web inbox, so nothing reaches real people. No signup, no external service.

> **Recap:** you set Mailpit up on **Day 1 (Topic 10)** and already sent a test email through it.
> If it's still running and your `.env` mail settings are in place, just confirm the inbox loads
> and move on. The steps below are the quick version, in case you're starting fresh.

1. In Laragon, click **Menu → Tools → Mailpit** and choose **Start** (some Laragon Full builds
   start it automatically with **Start All** — that's fine too).
2. Open the Mailpit inbox in a browser: **`http://localhost:8025`**. You'll see an empty inbox.
3. Keep this tab open. Mailpit listens for mail on SMTP (Simple Mail Transfer Protocol) port
   **1025** — you'll point Laravel at
   it in Topic 6 (no username or password required).

> **Don't see Mailpit in the menu?** Make sure you installed **Laragon Full** (recent versions
> include it). If your build lacks it, download `mailpit.exe` from
> `https://github.com/axllent/mailpit/releases`, drop it in `C:\laragon\bin\mailpit\`, and run it
> — it serves the same ports (1025 SMTP, 8025 web).

### 1.3 — Download VirtualBox + the Ubuntu 24.04 Server ISO (download only — do this now)

Tomorrow's SFTP work runs on a Linux VM (Virtual Machine). The files are big (~3 GB together),
so **start these
two downloads now** and let them finish in the background while you do the SOAP topics.
**Do NOT install or run anything yet — you only download today.** All installing and VM setup
happens on Day 3.

1. **Download the VirtualBox installer:** go to **`https://www.virtualbox.org/wiki/Downloads`**
   → **Windows hosts** → save the `.exe`. Leave it in your Downloads folder — **don't run it**.
2. **Download the Ubuntu Server ISO:** go to **`https://ubuntu.com/download/server`** → click
   **Download Ubuntu Server 24.04 LTS** (Long-Term Support). You get a file named like
   `ubuntu-24.04.x-live-server-amd64.iso` (~2.7 GB). Leave it downloading.
   > Get the **Server** image, not Desktop — it's smaller and matches a real SFTP host. Pick the
   > **LTS** version (24.04), not an interim release.

Once both files are on disk, you're done with VM prep for today.

**Checkpoint ✅ (Topic 1 complete)**
- The terminal prints `SOAP OK`.
- The Mailpit inbox (`http://localhost:8025`) is open in a browser tab.
- The VirtualBox installer and the Ubuntu Server ISO are both downloading (or finished) — saved
  on disk, not installed.

---

## Topic 2 — SOAP vs REST (concept)

**Goal:** understand what SOAP is and why you'd use it. Read this — no coding yet.

**REST (yesterday)** is a *style*: URLs (Uniform Resource Locators) + HTTP (Hypertext Transfer
Protocol) verbs + JSON (JavaScript Object Notation), loosely defined. **SOAP** is a *protocol*:
strict rules, XML (Extensible Markup Language) only, with a formal contract.

**The three things that define SOAP**

1. **WSDL (Web Services Description Language) — the contract** — an XML file describing every
   operation: names, input parameters, output shape, and the endpoint URL. A machine-readable API
   manual. Tools read the WSDL and know exactly how to call the service.

2. **The SOAP Envelope (the message)** — every request/response is an XML document with a fixed
   structure:
   ```xml
   <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
     <soap:Body>
       <getUserByEmail>
         <email>admin@test.com</email>
       </getUserByEmail>
     </soap:Body>
   </soap:Envelope>
   ```
   The `<Body>` holds the actual call.

3. **SOAP Faults (the errors)** — SOAP doesn't use HTTP status codes for business errors; it
   returns a `<soap:Fault>` element. In PHP this arrives as a `SoapFault` exception (Topic 4).

**When you'll meet SOAP:** bank/payment gateways, insurance, government tax portals, telco
provisioning, older ERP (Enterprise Resource Planning) / CRM (Customer Relationship Management)
systems. Anywhere with a formal contract and long-lived integrations.

**REST vs SOAP at a glance**

| | REST | SOAP |
|---|---|---|
| Data format | usually JSON | always XML |
| Contract | optional | required (WSDL) |
| Errors | HTTP status codes | `<soap:Fault>` |
| Feel | lightweight, flexible | strict, verbose, formal |

**PHP's role:** the built-in `ext-soap` gives you `SoapClient` (to *consume* a service) and
`SoapServer` (to *expose* one). You rarely write raw XML — the extension builds and parses
envelopes for you from the WSDL.

**Checkpoint ✅** You can explain, in one sentence each: what a WSDL is, what an envelope is,
and how a SOAP error is reported.

> ⏳ **Background task:** just let the VirtualBox + Ubuntu downloads from Topic 1.3 keep running.
> Nothing to install today — you'll set up the VM on Day 3.

---

## Topic 3 — Consume an external SOAP service and map it to the database

**Goal:** call a real external SOAP operation and store the result in MySQL. We do this in **two
phases, easy first**: (1) call the service **by hand in Postman** so you *see* exactly what a
SOAP request and response look like, then (2) **automate that same call in Laravel**. Doing it
by hand first means the code in phase 2 holds no surprises.

**The service we'll call.** A public, no-auth SOAP service that converts a number to English
words — reliable and needs no keys:

- Endpoint: `https://www.dataaccess.com/webservicesserver/NumberConversion.wso`
- Contract (WSDL): the same URL with `?WSDL` on the end.
- Operation: `NumberToWords(ubiNum)` → the number spelled out.

---

### Phase 1 — call the SOAP service by hand in Postman (before any code)

> **Ready-made requests:** a Postman collection for all of today's SOAP calls is provided —
> **`Day2-SOAP-API.postman_collection.json`** (in the course folder). **Import** it and you'll have
> folder **A. By hand** (the external calls in this phase) and folder **B. Your app** (used in
> Topics 3–5). You can follow the steps below by hand, or just send the matching request.

**Step 1a — look at the contract (optional but useful).** In Postman, do a **GET** on
`https://www.dataaccess.com/webservicesserver/NumberConversion.wso?WSDL`. The response is the
**WSDL** — the XML manual. Skim it and you'll spot the operation `NumberToWords`, its input part
`ubiNum`, and the response part `NumberToWordsResult`. Those three names are what phase 2's code
will use — you're reading them straight off the contract.

**Step 1b — actually call the operation.** In Postman:
1. Method **POST**, URL `https://www.dataaccess.com/webservicesserver/NumberConversion.wso`.
2. **Headers** tab — add two:
   - `Content-Type` = `text/xml; charset=utf-8`
   - `SOAPAction` = `""`  *(this service uses an empty SOAP action — keep the two quotes)*
3. **Body** tab → **raw** → set the dropdown to **XML** → paste this envelope:
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:web="http://www.dataaccess.com/webservicesserver/">
     <soap:Body>
       <web:NumberToWords>
         <web:ubiNum>1234</web:ubiNum>
       </web:NumberToWords>
     </soap:Body>
   </soap:Envelope>
   ```
4. Click **Send**.

**Step 1c — read the response.** You get XML back containing:
```xml
<m:NumberToWordsResult>one thousand two hundred and thirty four </m:NumberToWordsResult>
```

**What you just learned (the prerequisites for phase 2):**
- A SOAP call is a **POST of an XML envelope** to the endpoint URL.
- The `<web:NumberToWords>` element = the **operation**; `<web:ubiNum>` = the **input**.
- The answer lives in `<...NumberToWordsResult>` = the **result** you want.

Laravel's `SoapClient` (phase 2) does all of this envelope-building and parsing for you — but now
you know what it's doing under the hood.

**Checkpoint ✅ (phase 1)** Postman returns the number in words. You can point to the operation
name, the input `ubiNum`, and the result `NumberToWordsResult` in the XML.

---

### Phase 2 — automate the same call in Laravel and save the result

**Step 2 — a table to store results.** Create a migration:

```bash
php artisan make:migration create_conversions_table
```

Edit the new file's `up()` method (in `database/migrations/`):

```php
public function up(): void
{
    Schema::create('conversions', function (Blueprint $table) {
        $table->id();
        $table->unsignedBigInteger('number');   // the input
        $table->text('words');                   // the SOAP result
        $table->timestamps();
    });
}
```

```bash
php artisan migrate
```

**Step 3 — a model:**

```bash
php artisan make:model Conversion
```

In `app/Models/Conversion.php`:

```php
protected $fillable = ['number', 'words'];
```

**Step 4 — a service class** (keeps SOAP logic out of the controller). Create
`app/Services/NumberSoapService.php`:

```php
<?php

namespace App\Services;

use SoapClient;

class NumberSoapService
{
    private string $wsdl = 'https://www.dataaccess.com/webservicesserver/NumberConversion.wso?WSDL';

    public function numberToWords(int $number): string
    {
        $client = new SoapClient($this->wsdl, [
            'cache_wsdl'         => WSDL_CACHE_NONE, // avoid stale-contract issues while learning
            'connection_timeout' => 10,             // seconds to establish the connection
            'exceptions'         => true,           // throw SoapFault instead of returning it
        ]);

        // The operation takes a parameter named 'ubiNum' (per the WSDL)
        $response = $client->NumberToWords(['ubiNum' => $number]);

        // Result comes back on ->NumberToWordsResult
        return trim($response->NumberToWordsResult);
    }
}
```

> **Recognise those names?** `ubiNum` and `NumberToWordsResult` are exactly what you saw in
> Postman in phase 1 — the code just fills in the same envelope. To list any WSDL's operations
> from the terminal instead: `php -r "print_r((new SoapClient('PASTE_WSDL_URL'))->__getFunctions());"`

**Step 5 — controller + route:**

```bash
php artisan make:controller Api/ConversionController
```

`app/Http/Controllers/Api/ConversionController.php`:

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Conversion;
use App\Services\NumberSoapService;
use Illuminate\Http\Request;

class ConversionController extends Controller
{
    public function store(Request $request, NumberSoapService $soap)
    {
        $data = $request->validate(['number' => ['required', 'integer', 'min:0']]);

        $words = $soap->numberToWords($data['number']);   // SOAP call

        $conversion = Conversion::create([                // map result → DB
            'number' => $data['number'],
            'words'  => $words,
        ]);

        return response()->json($conversion, 201);
    }
}
```

In `routes/api.php`:

```php
use App\Http\Controllers\Api\ConversionController;

Route::post('/conversions', [ConversionController::class, 'store']);
```

**Step 6 — test your Laravel endpoint in Postman.** This time you call *your* API (which calls
the SOAP service for you): `POST http://training-app.test/api/conversions`, Body → raw → JSON:
`{"number": 1234}`.

**Checkpoint ✅**
- Response **201** with `words` = `"one thousand two hundred and thirty four"` — the same answer
  you got by hand in phase 1, now produced by your own endpoint.
- A new row exists in the `conversions` table (check in HeidiSQL via Laragon's Database button).

**Common problems**
- *"Could not connect to host" / SSL error* → firewall or SSL (Secure Sockets Layer) inspection.
  For learning only,
  pass a relaxed context to `SoapClient`:
  `['stream_context' => stream_context_create(['ssl' => ['verify_peer' => false, 'verify_peer_name' => false]])]`.
- *"SOAP-ERROR: Parsing WSDL"* → wrong URL, or a proxy is blocking the request.
- *Result is `null`* → wrong result property; run the `__getFunctions()` command above.

---

## Topic 4 — Fault handling: SoapFault, timeouts & a simple retry

**Goal:** make the integration robust. External services fail — your app must fail *gracefully*
and predictably, not crash.

**Three failure modes to handle:**
1. **SoapFault** — the service returned an error.
2. **Timeout** — the service is slow/unreachable and never answers.
3. **Transient failure** — a blip that succeeds on a second try → **retry**.

**Step 1 — set a socket timeout.** `connection_timeout` only covers *connecting*; the read
timeout is PHP's `default_socket_timeout`. Set both:

```php
public function numberToWords(int $number): string
{
    ini_set('default_socket_timeout', '15');   // max seconds to wait for a reply

    $client = new SoapClient($this->wsdl, [
        'cache_wsdl'         => WSDL_CACHE_NONE,
        'connection_timeout' => 10,
        'exceptions'         => true,
    ]);

    $response = $client->NumberToWords(['ubiNum' => $number]);
    return trim($response->NumberToWordsResult);
}
```

**Step 2 — wrap the call with try/catch + retry.** Put the two `use` lines at the **top** of the
file (with the existing `use SoapClient;`, under `<?php` — not inside the class, or PHP reads
`use SoapFault;` as a *trait* and errors with "Trait SoapFault not found"), then add this method
to the service class:

```php
use SoapFault;
use Illuminate\Support\Facades\Log;

public function numberToWordsWithRetry(int $number, int $attempts = 3): string
{
    $lastError = null;

    for ($try = 1; $try <= $attempts; $try++) {
        try {
            return $this->numberToWords($number);        // success → return immediately
        } catch (SoapFault $e) {
            $lastError = $e;
            Log::warning("SOAP attempt {$try} failed: {$e->getMessage()}");
            sleep(1);                                     // brief pause before retrying
        }
    }

    throw new \RuntimeException(
        "SOAP call failed after {$attempts} attempts: " . $lastError->getMessage(),
        0, $lastError
    );
}
```

> The final `RuntimeException` gives the rest of the app one predictable exception type to catch
> (Topic 7 emails on it), instead of leaking SOAP internals everywhere.

**Step 3 — use the safe method** in the controller and return a clean error:

```php
public function store(Request $request, NumberSoapService $soap)
{
    $data = $request->validate(['number' => ['required', 'integer', 'min:0']]);

    try {
        $words = $soap->numberToWordsWithRetry($data['number']);
    } catch (\RuntimeException $e) {
        return response()->json([
            'message' => 'The number-conversion service is unavailable. Please try later.',
        ], 503);   // 503 Service Unavailable — an upstream dependency failed
    }

    $conversion = Conversion::create(['number' => $data['number'], 'words' => $words]);
    return response()->json($conversion, 201);
}
```

**Step 4 — force a failure to prove it.** Temporarily add `-BROKEN` to the WSDL filename in the
service, send the request, and watch:
- The response is a clean **503** (no stack trace).
- `storage/logs/laravel.log` shows three "SOAP attempt failed" warnings.

Then restore the correct URL.

**Checkpoint ✅** A broken service produces a graceful 503 + logged warnings, not a 500 crash.
A working service still returns 201.

**Security note:** never return the raw `SoapFault` message to the client — it can leak internal
endpoints and stack details. Log the detail, return a generic message.

> ⏳ **Background task:** check that both downloads from Topic 1.3 have finished and the two files
> (VirtualBox installer + Ubuntu ISO) are saved on disk. That's all — installation is a Day 3 job.

---

## Topic 5 — Expose your own SOAP endpoint (SoapServer)

**Goal:** turn your Laravel app *into* a SOAP service. Other systems call `getUserByEmail` and
get user details back. Test it with Postman by sending a raw SOAP envelope.

**Concept.** `SoapServer` is the mirror of `SoapClient`: give it a WSDL and a PHP class, and it
answers SOAP requests by calling your class's methods. **Writing a correct WSDL by hand is
fiddly, so use the ready-made one below** — just drop it in.

**Step 1 — add the WSDL.** In VS Code, create `public/userinfo.wsdl` with exactly this content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions name="UserInfo"
  targetNamespace="urn:UserInfo"
  xmlns:tns="urn:UserInfo"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
  xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
  xmlns="http://schemas.xmlsoap.org/wsdl/">

  <message name="getUserByEmailRequest">
    <part name="email" type="xsd:string"/>
  </message>
  <message name="getUserByEmailResponse">
    <part name="name" type="xsd:string"/>
    <part name="email" type="xsd:string"/>
    <part name="createdAt" type="xsd:string"/>
  </message>

  <portType name="UserInfoPortType">
    <operation name="getUserByEmail">
      <input message="tns:getUserByEmailRequest"/>
      <output message="tns:getUserByEmailResponse"/>
    </operation>
  </portType>

  <binding name="UserInfoBinding" type="tns:UserInfoPortType">
    <soap:binding style="rpc" transport="http://schemas.xmlsoap.org/soap/http"/>
    <operation name="getUserByEmail">
      <soap:operation soapAction="urn:UserInfo#getUserByEmail"/>
      <input><soap:body use="encoded" namespace="urn:UserInfo"
        encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"/></input>
      <output><soap:body use="encoded" namespace="urn:UserInfo"
        encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"/></output>
    </operation>
  </binding>

  <service name="UserInfoService">
    <port name="UserInfoPort" binding="tns:UserInfoBinding">
      <soap:address location="http://training-app.test/api/soap"/>
    </port>
  </service>
</definitions>
```

> Two things link the WSDL to your code: the operation name `getUserByEmail` (must match a PHP
> method) and `<soap:address location>` (must match your route URL).

**Step 2 — the service class.** Create `app/Soap/UserInfoSoapService.php`:

```php
<?php

namespace App\Soap;

use App\Models\User;
use SoapFault;

class UserInfoSoapService
{
    public function getUserByEmail($email)
    {
        $user = User::where('email', $email)->first();

        if (! $user) {
            throw new SoapFault('Server', "No user found for {$email}");  // becomes <soap:Fault>
        }

        // Keys match the response <part> names in the WSDL
        return [
            'name'      => $user->name,
            'email'     => $user->email,
            'createdAt' => $user->created_at->toDateTimeString(),
        ];
    }
}
```

**Step 3 — a controller to host the server:**

```bash
php artisan make:controller SoapServerController
```

`app/Http/Controllers/SoapServerController.php`:

```php
<?php

namespace App\Http\Controllers;

use App\Soap\UserInfoSoapService;
use Illuminate\Http\Request;
use SoapServer;

class SoapServerController extends Controller
{
    public function handle(Request $request)
    {
        $wsdlPath = public_path('userinfo.wsdl');

        // Clients fetch the contract via ...?wsdl
        if ($request->has('wsdl')) {
            return response()->file($wsdlPath, ['Content-Type' => 'text/xml']);
        }

        $server = new SoapServer($wsdlPath);
        $server->setClass(UserInfoSoapService::class);

        // SoapServer writes to output; capture it and return as the HTTP response
        ob_start();
        $server->handle();
        $xml = ob_get_clean();

        return response($xml, 200, ['Content-Type' => 'text/xml; charset=utf-8']);
    }
}
```

**Step 4 — the route.** In `routes/api.php` (the `/api` prefix means **no CSRF (Cross-Site
Request Forgery) token** is required — important, since SOAP clients can't send Laravel's CSRF
token):

```php
use App\Http\Controllers\SoapServerController;

Route::match(['get', 'post'], '/soap', [SoapServerController::class, 'handle']);
```

**Step 5 — test in Postman** by sending a raw SOAP request. *(This is requests **5** and **6** in
folder **B. Your app** of the provided `Day2-SOAP-API.postman_collection.json` — import it to skip
the typing.)*
1. New request: method **POST**, URL `http://training-app.test/api/soap`.
2. **Headers** tab — add two:
   - `Content-Type` = `text/xml; charset=utf-8`
   - `SOAPAction` = `"urn:UserInfo#getUserByEmail"` (keep the quotes)
3. **Body** tab → **raw** → change the dropdown to **XML** → paste the SOAP envelope below.
4. Click **Send**.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:urn="urn:UserInfo">
  <soapenv:Body>
    <urn:getUserByEmail><email>admin@test.com</email></urn:getUserByEmail>
  </soapenv:Body>
</soapenv:Envelope>
```

> **Want to see the contract first?** In Postman, do a **GET** on
> `http://training-app.test/api/soap?wsdl` — it returns the raw WSDL XML your service publishes.

**Checkpoint ✅**
- A valid email returns a SOAP XML response containing `name`, `email`, `createdAt`.
- An unknown email returns a `<soap:Fault>` with your "No user found" message.

**Common problems**
- *"looks like we got no XML document"* → an error/HTML leaked before the SOAP output; check
  `storage/logs/laravel.log` (usually a typo in the service class).
- *Empty or "actor" fault* → the `SOAPAction` header is missing or its quotes were dropped.
- *419 Page Expired* → you put the route in `web.php` (CSRF). Keep it in `api.php`.
- *Body sent as Text, not XML* → set Body → raw → **XML** so the envelope isn't mangled.

---

## Topic 6 — Email setup with Mailpit

**Goal:** send email from Laravel into your local Mailpit inbox (started in Topic 1.2).

**Concept.** You configure Laravel's mail settings, then build a **Notification** — a Laravel
class representing "a message to send someone" that goes out by email.
Because Mailpit runs on your own machine, there are **no credentials** — just a host and port.

**Step 1 — point `.env` at Mailpit** (SMTP on `127.0.0.1:1025`, no username/password/encryption).
*You already did this on Day 1 (Topic 10) — confirm the lines are present, or set them if you're
starting fresh:*

```dotenv
MAIL_MAILER=smtp
MAIL_HOST=127.0.0.1
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="alerts@training.test"
MAIL_FROM_NAME="Integration Monitor"
```

Reload config:

```bash
php artisan config:clear
```

**Step 2 — create a notification:**

```bash
php artisan make:notification IntegrationFailedNotification
```

Edit `app/Notifications/IntegrationFailedNotification.php`:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Notification;

class IntegrationFailedNotification extends Notification
{
    use Queueable;

    public function __construct(
        public string $integration,   // e.g. "Number SOAP service"
        public string $reason,        // the error message
    ) {}

    public function via(object $notifiable): array
    {
        return ['mail'];
    }

    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
            ->error()
            ->subject("⚠️ Integration failed: {$this->integration}")
            ->line("The integration \"{$this->integration}\" failed.")
            ->line("Reason: {$this->reason}")
            ->line('Time: ' . now()->toDateTimeString())
            ->line('Please investigate.');
    }
}
```

**Step 3 — send a test email** with an on-demand notification (no user needed). Add a temporary
route in `routes/api.php`:

```php
use Illuminate\Support\Facades\Notification;
use App\Notifications\IntegrationFailedNotification;

Route::get('/test-mail', function () {
    Notification::route('mail', 'admin@test.com')
        ->notify(new IntegrationFailedNotification('Test Integration', 'This is a test.'));

    return 'sent';
});
```

Visit `http://training-app.test/api/test-mail`.

**Checkpoint ✅** The email appears in your **Mailpit inbox** (`http://localhost:8025`) within a
second or two, with red "error" styling and your subject line. **Click it to open** and read the
headers and body — the same way you viewed the Day 1 test email (Day 1, Topic 10).

**Common problems**
- *Nothing arrives* → Mailpit isn't running (start it in Laragon), or you forgot
  `php artisan config:clear` after editing `.env`.
- *`Connection refused` on port 1025* → Mailpit isn't started, or another tool took the port.
  Confirm the web UI at `http://localhost:8025` loads first.
- *Old Mailtrap settings still used* → a cached config; run `php artisan config:clear` (and
  `php artisan optimize:clear` if needed).

---

## Topic 7 — Auto-email on integration failure (finale)

**Goal:** tie it together — when a SOAP integration fails after retries, automatically send the
alert email you built in Topic 6. That's it: one line in the failure path.

**The whole idea in one sentence:** in the `catch` block, before returning the error, send the
notification. No queues, no workers — the email goes out right there.

**Step 1 — fire the alert from the failure path.** In `ConversionController::store` (Topic 4),
add the two `use` lines at the **top** of the controller file (with the others, under `<?php`),
then send the notification inside the `catch` block:

```php
use Illuminate\Support\Facades\Notification;
use App\Notifications\IntegrationFailedNotification;

public function store(Request $request, NumberSoapService $soap)
{
    $data = $request->validate(['number' => ['required', 'integer', 'min:0']]);

    try {
        $words = $soap->numberToWordsWithRetry($data['number']);
    } catch (\RuntimeException $e) {
        // Integration failed → email the team, then return a clean error
        Notification::route('mail', 'admin@test.com')
            ->notify(new IntegrationFailedNotification('Number SOAP service', $e->getMessage()));

        return response()->json([
            'message' => 'The number-conversion service is unavailable. Please try later.',
        ], 503);
    }

    $conversion = Conversion::create(['number' => $data['number'], 'words' => $words]);
    return response()->json($conversion, 201);
}
```

**Step 2 — trigger a failure end-to-end.**
1. Add `-BROKEN` to the WSDL URL in `NumberSoapService`.
2. `POST /api/conversions` with `{"number": 99}`.
3. Observe:
   - The API responds **503** with the friendly message.
   - The **alert email** lands in Mailpit (`http://localhost:8025`).
4. Restore the correct WSDL URL.

**Checkpoint ✅ (end-of-day goal)**
- A working call → **201** + row saved, no email.
- A failing call → **503** + an alert email in Mailpit.

**Common problems**
- *No email* → Mailpit isn't running, or the `.env` mail settings are wrong (Topic 6). Re-run
  `php artisan config:clear`.
- *No 503, you got a 500* → the alert line threw; check `storage/logs/laravel.log`.

> **Later, in production:** sending email inside a request slows that request a little. Real apps
> push email to a background **queue** so the response stays fast. We skip that here to keep the
> focus on the integration itself — but it's a one-line upgrade (`implements ShouldQueue` on the
> notification) when you need it.

---

## End-of-Day 2 — final working state

You should now have:
- `app/Services/NumberSoapService.php` — consumes external SOAP with timeout + retry, throwing a
  clean `RuntimeException` on total failure.
- `app/Models/Conversion.php` + `conversions` table — where SOAP results are stored.
- `app/Http/Controllers/Api/ConversionController.php` — calls SOAP, saves the result, emails on
  failure.
- `public/userinfo.wsdl` + `app/Soap/UserInfoSoapService.php` +
  `app/Http/Controllers/SoapServerController.php` — your **exposed** SOAP endpoint at `/api/soap`.
- `app/Notifications/IntegrationFailedNotification.php` — the **email alert**.
- `.env` — Mailpit SMTP (`127.0.0.1:1025`).
- **The VirtualBox installer + Ubuntu 24.04 Server ISO downloaded to disk** — ready to install on
  Day 3 (nothing installed yet).

**Remove the temporary `/api/test-mail` route and restore the WSDL URL** before moving on.

**Security / robustness recap**
- Raw `SoapFault` details are **logged, not returned** — no internal leakage.
- Failures degrade to a friendly **503**, never a 500 crash or stack trace.
- Retries absorb transient blips before we give up and alert.
- Your exposed SOAP service throws a controlled fault for unknown users (no data leak).

**Stretch goals (if time remains)**
- Add a second WSDL operation (e.g. `getUserCount`) and implement it.
- Send alerts to multiple recipients: `Notification::route('mail', ['a@x.test','b@x.test'])`.

**Tomorrow (Day 3):** install VirtualBox and build the Ubuntu VM from today's downloads, then set
up an **SFTP** server, transfer files by GUI and terminal, and schedule a routine task with
**cron** — a
self-contained Linux module (no Laravel).
