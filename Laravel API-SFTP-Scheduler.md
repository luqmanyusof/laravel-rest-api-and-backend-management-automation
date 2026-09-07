# **COURSE OUTLINE** 

|**PROGRAMME TITLE**|Laravel API Integraton: REST, SOAP, SFTP & Task Scheduling|
|---|---|
|**OVERVIEW**|This practcal 3-day training teaches partcipants to build and integrate web services<br>using Laravel. A pre-built Laravel app with a CRUD UI is provided, so the focus is purely<br>on developing the REST API and SOAP layers. Day 1 covers REST APIs with validaton and<br>error handling, tested in Postman. Day 2 covers SOAP — consuming and exposing<br>services — plus email alerts on integraton failure. Day 3 covers SFTP fle transfer and<br>task scheduling with cron and the Laravel scheduler.<br>The Laravel, API and SOAP work runs locally on XAMPP or Laragon. Only the Day 3 Linux<br>topics (SFTP and cron) use a provided VirtualBox VM. Partcipants code the exercises<br>themselves and fnish with a working end-to-end integraton.|
|**LEARNING**<br>**OUTCOME**|By the end of the session, the partcipants will be able to:<br>• Build REST API endpoints with validaton and consistent error handling<br>• Secure APIs with token authentcaton (Laravel Sanctum)<br>• Test APIs using Postman<br>• Consume and expose SOAP services, and handle SOAP faults safely<br>• Send email alerts automatcally on integraton failure<br>• Use SFTP via terminal and GUI clients (WinSCP, FileZilla)<br>• Automate tasks with cron and the Laravel scheduler<br>• Build an end-to-end integraton workfow|
|**DURATION (days)**|3 Days|
|**TARGET**<br>**PARTICIPANTS**|• IT staf, developers and system integrators who build or maintain integratons<br>• Backend developers who want to add SOAP/REST integraton skills<br>• IT / operatons staf responsible for fle transfers and scheduled jobs<br>• Basic PHP and command-line familiarity is helpful but not mandatory|
|**COURSE**<br>**MATERIALS**<br>**PROVIDED**|Sofware:<br>• XAMPP or Laragon (PHP 8.x, Composer, Laravel 11, MySQL) — for the Laravel, API<br>and SOAP work<br>• Postman (REST API testng) & SoapUI (SOAP testng)<br>• Mailtrap or a test SMTP account (for email notfcaton testng)<br>• Oracle VirtualBox with a provided Linux VM — for the Day 3 SFTP and cron topics<br>only<br>• WinSCP and FileZilla (SFTP GUI clients)<br>• Visual Studio Code<br>Practce Files:<br>• Pre-built Laravel applicaton with a working CRUD UI (partcipants add the API &<br>SOAP layers)<br>• Sample WSDL and a mock SOAP service for exercises<br>• Postman collecton and sample datasets<br>• Step-by-step lab guides for each module<br>• Final integraton project brief|
|**COURSE CONTENT**|**DAY 1: LARAVEL REST API — VALIDATION & ERROR HANDLING**<br>**_Morning Session:_**<br>• Environment setup: run the provided Laravel applicaton locally on XAMPP or<br>Laragon<br>• Tour of the provided Laravel applicaton: existng models, migratons and CRUD UI —<br>where the API layer fts<br>• REST fundamentals: resources, HTTP methods, status codes and JSON responses|



- Add REST API endpoints on top of the existing models: API routes, resource controllers, route model binding 

### **_Afternoon Session:_** 

- Request validation: Form Request classes, validation rules and custom messages 

- API Resources: shaping consistent, predictable JSON output 

- Error handling: the exception handler, standardized error responses and HTTP exceptions 

- API authentication with Laravel Sanctum (token-based access) 

- Testing with Postman: building requests, collections and environments, and saving auth tokens 

- Hands-on: expose a validated CRUD REST API and test the full flow in Postman 

## **DAY 2: SOAP INTEGRATION & FAILURE NOTIFICATIONS** 

### **_Morning Session — Consuming SOAP:_** 

- SOAP vs REST: WSDL, XML envelopes and when SOAP is used 

- PHP SoapClient: connecting to an external WSDL and calling operations 

- Mapping SOAP responses into the Laravel application and database 

- Fault handling: SoapFault, timeouts and a simple retry strategy 

### **_Afternoon Session — Exposing SOAP & Email Alerts:_** 

- Building a SOAP endpoint in Laravel (SoapServer and WSDL definition) 

- Testing SOAP endpoints with SoapUI and Postman (raw XML requests) 

- Email setup: SMTP configuration (Mailtrap / test account), Mailable classes and Notifications 

- Trigger email alerts automatically on integration failure (SOAP fault or API error) using queued notifications 

- Hands-on: consume a SOAP service, expose one, and email on failure 

## **DAY 3: SFTP, CRONJOB & TASK SCHEDULER** 

### **_Morning Session — SFTP:_** 

- Import and start the provided Oracle VirtualBox Linux VM for the SFTP and cron exercises 

- What SFTP is versus FTP/FTPS, and SSH basics 

- Install and enable an SFTP server (OpenSSH) on the Linux VM; create users and set permissions 

- Connect via terminal: the sftp command, put/get, and key-based authentication 

- GUI clients: configure and transfer files with WinSCP and FileZilla 

- Automate SFTP transfers from Laravel using the Flysystem SFTP adapter 

### **_Afternoon Session — Cronjob & Scheduler:_** 

- Linux cron basics: crontab syntax and scheduling jobs 

- Laravel Task Scheduler: schedule(), defining scheduled tasks and the single cron entry 

- Writing custom Artisan commands for scheduled integration tasks 

- Practical: schedule an SFTP file pull, a SOAP/REST sync, and a failure email 

- Logging, monitoring and verifying that scheduled jobs run correctly 

- Wrap-up: assemble the modules into one end-to-end integration workflow 

**LEARNING** The training will be delivered through a combination of: **METHODOLOGIES** 

- Short, focused demonstrations (show, then do) 

- Hands-on coding exercises with sample data and lab guides 

- Live demos and real-world integration scenarios 

- A final integration project with presentation 

- • Individual coaching and support throughout 

