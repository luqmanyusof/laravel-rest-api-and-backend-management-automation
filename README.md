# Practical Laravel Backend Integration & Automation

*Building REST & SOAP APIs, SFTP file transfers, and cron-scheduled jobs — a hands-on 3-day
course.*

**Table of contents.** Each day is one Markdown file of step-by-step lab notes for
beginner-to-intermediate participants, sized for ~5–5.5 hours. The flow is
**REST → SOAP → SFTP → Cron**.

**Acronyms used throughout** (each is also expanded on first use inside each day's file):
**API** = Application Programming Interface · **REST** = Representational State Transfer ·
**SOAP** = Simple Object Access Protocol · **WSDL** = Web Services Description Language ·
**XML** = Extensible Markup Language · **JSON** = JavaScript Object Notation ·
**HTTP** = Hypertext Transfer Protocol · **CRUD** = Create, Read, Update, Delete ·
**SMTP** = Simple Mail Transfer Protocol · **SSH** = Secure Shell · **SFTP** = SSH File Transfer
Protocol · **FTP/FTPS** = File Transfer Protocol / FTP Secure · **GUI** = Graphical User
Interface · **VM** = Virtual Machine · **CSV** = Comma-Separated Values · **SCP** = Secure Copy
(file copy over SSH) · **UFW** = Uncomplicated Firewall (Ubuntu's firewall tool) · **cron** = the
Linux job scheduler.

**The three days**
- **[Day 1 — Laravel REST API: Validation, Error Handling & Security](day1.md)**
- **[Day 2 — SOAP Integration & Failure Notifications](day2.md)**
- **[Day 3 — SFTP, Cron & Scheduled File Transfers](day3.md)**

**How to read this:** every topic follows the same shape — a short **Objective** (what you set
out to do), the **steps**, and an **Outcome** you can verify (the topic's checkpoint). Topics are
ordered *prerequisites first, easy first* — each one builds on the last.

**Course-level learning outcomes.** By the end, participants can: build validated REST endpoints
with consistent error handling; secure APIs with token auth (Sanctum); test APIs in Postman;
consume and expose SOAP services and handle faults; send automatic failure-alert emails; use SFTP
by terminal and GUI; and automate a routine task with cron.

---

## Day 1 — [Laravel REST API: Validation, Error Handling & Security](day1.md)

*Build a secured User-Management REST API, layering complexity easy-first: a plain CRUD API that
works, then validation, clean errors, token auth, and a related table.*

| # | Topic | Objective | Outcome |
|---|---|---|---|
| 1 | [Install & verify your tools](day1.md#topic-1--install-and-verify-your-tools) | Install Laragon (PHP 8.3, Composer, MySQL), Postman and VS Code from zero, and verify them | A working local stack: PHP/Composer/MySQL running, Postman and VS Code open |
| 2 | [Get the Laravel app running](day1.md#topic-2--get-the-laravel-application-running) | Set up the provided app (`composer install`) and connect it to MySQL | The `/users` page loads at `127.0.0.1:8000`; the `training` DB has a seeded `users` table |
| 3 | [REST fundamentals (concept)](day1.md#topic-3--rest-fundamentals-concept) | Learn the REST vocabulary: resources, HTTP verbs, status codes, JSON | Can pick the right verb + status for an action (e.g. create → POST → 201) |
| 4 | [Tour the app: where the API fits](day1.md#topic-4--tour-the-app-where-the-api-layer-fits) | See the models, migrations and routes you'll build on | Can point to the User model, its migration, and where API routes live |
| 5 | [API routes, controller & route model binding](day1.md#topic-5--api-routes-resource-controller--route-model-binding) | Build the five CRUD endpoints returning JSON (the easy win — no rules yet) | `GET /api/users` returns JSON; full CRUD works (still unvalidated/unsecured) |
| 6 | [Validation with Form Requests](day1.md#topic-6--validation-with-form-requests) | Reject bad input with Form Request classes and custom messages | Invalid body → **422** with a JSON `errors` list; valid body → **201** |
| 7 | [API Resources & error handling](day1.md#topic-7--api-resources--consistent-error-handling) | Shape the JSON output and make every error return clean JSON | Stable JSON (no password leak); 401/404/422/500 all return JSON, never HTML |
| 8 | [Sanctum auth + Postman (finale)](day1.md#topic-8--sanctum-token-authentication--postman-hands-on-finale) | Secure the API with token auth and prove the full flow in Postman | No token → **401**; with token → full CRUD; token auto-saved in Postman |
| 9 | [A second table: user profiles (relationships)](day1.md#topic-9--a-second-table-user-profiles-relationships) | Add a related table and learn one-to-one Eloquent relationships | `GET`/`PUT /api/users/{id}/profile` work; `hasOne`/`belongsTo` understood |

---

## Day 2 — [SOAP Integration & Failure Notifications](day2.md)

*Consume and expose SOAP services, handle faults, and send an automatic alert email on failure.
Consume before expose; try each call by hand in Postman before coding.*

| # | Topic | Objective | Outcome |
|---|---|---|---|
| 1 | [Enable SOAP, ready tools & start VM download](day2.md#topic-1--enable-soap-ready-your-tools--start-the-vm-download) | Enable PHP's `ext-soap`, start Mailpit, and begin the VirtualBox + Ubuntu downloads for Day 3 | `SOAP OK`; Mailpit inbox open; both installers downloading to disk |
| 2 | [SOAP vs REST (concept)](day2.md#topic-2--soap-vs-rest-concept) | Understand what SOAP is: WSDL, XML envelope, SOAP faults | Can explain a WSDL, an envelope, and how a SOAP error is reported |
| 3 | [Consume a SOAP service → database](day2.md#topic-3--consume-an-external-soap-service-and-map-it-to-the-database) | Call a SOAP service by hand in Postman, then automate it in Laravel and store the result | `POST /api/conversions` returns the number in words and saves a DB row |
| 4 | [Fault handling: faults, timeouts, retry](day2.md#topic-4--fault-handling-soapfault-timeouts--a-simple-retry) | Make the SOAP call robust against faults, timeouts and transient failures | A broken service → graceful **503** + logged warnings, not a 500 crash |
| 5 | [Expose your own SOAP endpoint](day2.md#topic-5--expose-your-own-soap-endpoint-soapserver) | Turn Laravel into a SOAP service and test it in Postman | `getUserByEmail` returns user XML; an unknown email → a `<soap:Fault>` |
| 6 | [Email setup with Mailpit](day2.md#topic-6--email-setup-with-mailpit) | Send email from Laravel into the local Mailpit inbox via a Notification | A test email appears in the Mailpit inbox (`http://localhost:8025`) |
| 7 | [Auto-email on failure (finale)](day2.md#topic-7--auto-email-on-integration-failure-finale) | Send an alert email automatically whenever an integration fails (one line, no queue) | A failing call → **503** + an alert email delivered to Mailpit |

---

## Day 3 — [SFTP, Cron & Scheduled File Transfers](day3.md)

*A self-contained Linux module (no Laravel): build an Ubuntu VM, run an SFTP server, transfer
files by GUI and terminal, and schedule a routine task with cron.*

| # | Topic | Objective | Outcome |
|---|---|---|---|
| 1 | [Concept: SSH & SFTP vs FTP/FTPS](day3.md#topic-1--concept-ssh-and-sftp-vs-ftpftps) | Learn what SSH is and why SFTP beats FTP/FTPS — before any setup | Can explain SFTP vs FTP and how SFTP relates to SSH (runs over it) |
| 2 | [Install VirtualBox & build the Ubuntu VM](day3.md#topic-2--install-virtualbox--build-the-ubuntu-vm) | Install VirtualBox and build/network an Ubuntu 24.04 Server VM | The VM boots and logs in; `ping <VM_IP>` replies from Windows |
| 3 | [Turn on the SFTP server & create a user](day3.md#topic-3--turn-on-the-sftp-server--create-a-dedicated-user) | Verify OpenSSH, open port 22 in the firewall (UFW), and add a dedicated SFTP user with a drop folder | sshd active on port 22; UFW active with SSH allowed; `sftpuser` owns `/home/sftpuser/upload` |
| 4 | [Transfer files with a GUI client (WinSCP & FileZilla)](day3.md#topic-4--transfer-files-with-a-gui-client-winscp--filezilla-in-depth) | Connect from Windows and use each client's real features in depth | Files uploaded via both clients; remote edit + folder compare used |
| 5 | [Terminal transfers with `sftp`](day3.md#topic-5--transfer-files-from-the-terminal) | Use `put`/`get`, wildcards, recursive, `scp` and batch mode | `put`/`get`/`mput` work; `scp` and `sftp -b` run a command list |
| 6 | [Schedule a task with cron](day3.md#topic-6--schedule-a-routine-task-with-cron) | Understand `crontab` and auto-archive the inbox on a timer | cron moves uploaded CSVs to `processed/` every 5 min; `archive.log` grows |

---

*Files in this folder: [`day1.md`](day1.md) · [`day2.md`](day2.md) · [`day3.md`](day3.md) ·
[`Laravel API-SFTP-Scheduler.md`](Laravel%20API-SFTP-Scheduler.md) (original course outline).*
