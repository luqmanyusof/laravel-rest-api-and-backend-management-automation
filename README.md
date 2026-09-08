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
- **[Day 1 — Build a Laravel 12 REST API (Users & Profiles)](day1.md)**
- **[Day 2 — SOAP Integration & Failure Notifications](day2.md)**
- **[Day 3 — SFTP, Cron & Scheduled File Transfers](day3.md)**

**How to read this:** every topic follows the same shape — a short **Objective** (what you set
out to do), the **steps**, and an **Outcome** you can verify (the topic's checkpoint). Topics are
ordered *prerequisites first, easy first* — each one builds on the last.

**Course-level learning outcomes.** By the end, participants can: build a REST API in Laravel with
CRUD endpoints, related tables and seeded data; test APIs in Postman; consume and expose SOAP
services and handle faults; send automatic failure-alert emails; use SFTP by terminal and GUI; and
automate a routine task with cron.

---

## Day 1 — [Build a Laravel 12 REST API (Users & Profiles)](day1.md)

*Start from a blank Laravel 12 project and build up: a Users CRUD API, seeded data, consuming an
external REST API (GET + POST), a related `user_profiles` table (one-to-one), a full Postman test
pass, and a first test email via Mailpit. No validation/auth on the API you build — the focus is
building, seeding and testing.*

| # | Topic | Objective | Outcome |
|---|---|---|---|
| 1 | [Install & verify your tools](day1.md#topic-1--install-and-verify-your-tools) | Install Laragon (PHP 8.3, Composer, MySQL), Postman and VS Code from zero, and verify them | A working local stack: PHP/Composer/MySQL running, Postman and VS Code open |
| 2 | [Create a new Laravel 12 project](day1.md#topic-2--create-a-new-laravel-12-project) | Create a blank project with `composer create-project` and connect it to MySQL | The Laravel welcome page loads at `http://training-app.test` (served by Laragon); the `training` DB has a `users` table |
| 3 | [REST fundamentals (concept)](day1.md#topic-3--rest-fundamentals-concept) | Learn the REST vocabulary: resources, HTTP verbs, status codes, JSON | Can pick the right verb + status for an action (e.g. create → POST → 201) |
| 4 | [Tour a fresh Laravel project](day1.md#topic-4--tour-a-fresh-laravel-project) | See what a new Laravel ships: User model, migration, factory, seeder | Can point to the User model, its migration/factory, and where API routes live |
| 5 | [Build the Users CRUD API](day1.md#topic-5--build-the-users-crud-api) | Scaffold and fill the five CRUD endpoints returning JSON (route model binding) | `POST`/`GET /api/users` work; passwords never appear in responses |
| 6 | [Seed the users table](day1.md#topic-6--seed-the-users-table) | Use the factory + seeder to create ~10 users plus a known admin | `users` has 11 rows incl. `admin@test.com`; `GET /api/users` lists them |
| 7 | [Consume a REST API: GET & POST](day1.md#topic-7--consume-a-rest-api-get-and-post) | Build up from the simplest `Http::get` to a validated call, then POST data to another API | `GET /api/exchange` returns a converted rate; `POST /api/remote-posts` sends data and gets it back with an id |
| 8 | [A related table: user_profiles](day1.md#topic-8--a-related-table-user_profiles-one-to-one) | Add a `user_profiles` migration + model and a one-to-one relationship | `user_profiles` table exists; `hasOne`/`belongsTo` wired between the models |
| 9 | [Seed profiles + profile API](day1.md#topic-9--seed-profiles--add-the-profile-api) | Seed a profile for every user and expose read/update endpoints | `user_profiles` has 11 rows; `GET`/`PUT /api/users/{id}/profile` work |
| 10 | [Test everything in Postman](day1.md#topic-10--test-everything-in-postman) | Build a Postman collection and exercise every endpoint | All CRUD + profile + REST requests return the expected JSON/status |
| 11 | [Send a test email with Mailpit](day1.md#topic-11--send-a-test-email-with-mailpit-a-bridge-to-day-2) | Set up Mailpit, point Laravel at it, and send a simple email | A test email lands in the Mailpit inbox (`http://localhost:8025`) |

---

## Day 2 — [SOAP Integration & Failure Notifications](day2.md)

*Consume and expose SOAP services, and send an automatic alert email on failure. Consume before
expose; try each call by hand in Postman first. (REST was Day 1; today is the SOAP contrast.)*

| # | Topic | Objective | Outcome |
|---|---|---|---|
| 1 | [Enable SOAP, ready tools & start VM download](day2.md#topic-1--enable-soap-ready-your-tools--start-the-vm-download) | Enable PHP's `ext-soap`, start Mailpit, and begin the VirtualBox + Ubuntu downloads for Day 3 | `SOAP OK`; Mailpit inbox open; both installers downloading to disk |
| 2 | [SOAP vs REST (concept)](day2.md#topic-2--soap-vs-rest-concept) | Understand what SOAP is: WSDL, XML envelope, SOAP faults | Can explain a WSDL, an envelope, and how a SOAP error is reported |
| 3 | [Consume a SOAP service → database](day2.md#topic-3--consume-an-external-soap-service-and-map-it-to-the-database) | Call a SOAP service by hand in Postman, then automate it in Laravel, store the result and fail gracefully | `POST /api/conversions` returns the number in words + saves a row; a dead service → clean **503** |
| 4 | [Expose your own SOAP endpoint](day2.md#topic-4--expose-your-own-soap-endpoint-soapserver) | Turn Laravel into a SOAP service and test it in Postman | `getUserByEmail` returns user XML; an unknown email → a `<soap:Fault>` |
| 5 | [Email setup with Mailpit](day2.md#topic-5--email-setup-with-mailpit) | Send email from Laravel into the local Mailpit inbox via a Notification | A test email appears in the Mailpit inbox (`http://localhost:8025`) |
| 6 | [Auto-email on failure (finale)](day2.md#topic-6--auto-email-on-integration-failure-finale) | Send an alert email automatically whenever an integration fails (one line, no queue) | A failing call → **503** + an alert email delivered to Mailpit |

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
