# IIS Server — Complete Guide

## 1. What is IIS?

**IIS (Internet Information Services)** is Microsoft's web server for Windows Server and Windows.

IIS is used to host:

* HTML websites
* ASP.NET applications
* ASP.NET Core applications
* APIs
* Static files
* Web applications
* HTTPS websites
* Reverse proxy applications

Basic architecture:

```text
Client
   |
   | HTTP / HTTPS
   v
 IIS Server
   |
   +---- Static Website
   |
   +---- ASP.NET Application
   |
   +---- ASP.NET Core Application
   |
   +---- API
```

---

# 2. IIS vs Nginx

| Feature            | IIS                    | Nginx                           |
| ------------------ | ---------------------- | ------------------------------- |
| Platform           | Windows                | Linux/Windows                   |
| Configuration      | GUI + XML              | Configuration files             |
| Main config        | applicationHost.config | nginx.conf                      |
| Default HTTP Port  | 80                     | 80                              |
| Default HTTPS Port | 443                    | 443                             |
| Web applications   | Excellent .NET support | Excellent reverse proxy support |
| Management         | IIS Manager            | CLI/config files                |
| Authentication     | Windows Authentication | Config/modules                  |
| Logs               | IIS logs               | Nginx logs                      |
| Application pools  | Yes                    | No equivalent                   |
| Reverse proxy      | URL Rewrite + ARR      | Native proxy_pass               |

---

# 3. IIS Components

Important IIS components:

```text
IIS Server
│
├── Sites
│
├── Applications
│
├── Virtual Directories
│
├── Application Pools
│
├── Bindings
│
├── Authentication
│
├── Authorization
│
├── SSL/TLS
│
├── Logging
│
└── Modules
```

---

# 4. IIS Architecture

```text
                 Internet
                    |
                    v
                Windows Server
                    |
                    v
                   IIS
                    |
       +------------+------------+
       |            |            |
       v            v            v
    Website       API        Admin App
       |            |            |
       v            v            v
 Application    Application   Application
    Pool           Pool          Pool
```

---

# 5. IIS Installation

## Method 1 — Server Manager

Open:

```text
Server Manager
```

Select:

```text
Manage
   ↓
Add Roles and Features
```

Select:

```text
Role-based or feature-based installation
```

Select the server.

Choose:

```text
Web Server (IIS)
```

Install the required IIS role services.

---

# 6. Install IIS Using PowerShell

Open PowerShell as Administrator:

```powershell
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
```

Check installation:

```powershell
Get-WindowsFeature Web-Server
```

---

# 7. Verify IIS

Open browser:

```text
http://localhost
```

You should see the IIS welcome page.

You can also test:

```powershell
Invoke-WebRequest http://localhost
```

---

# 8. IIS Manager

Open:

```text
inetmgr
```

or:

```text
Start
→ IIS Manager
```

Main sections:

```text
Server
│
├── Application Pools
│
├── Sites
│
└── Management
```

---

# 9. Default IIS Website

The default IIS website is usually:

```text
Default Web Site
```

Default physical directory:

```text
C:\inetpub\wwwroot
```

Example:

```text
C:\inetpub\wwwroot
├── index.html
├── css
├── js
└── images
```

---

# 10. Create a Static Website

Create:

```text
C:\Websites\MyWebsite
```

Create:

```text
C:\Websites\MyWebsite\index.html
```

Example:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My IIS Website</title>
</head>
<body>
    <h1>Welcome to IIS Server</h1>
</body>
</html>
```

---

# 11. Add Website in IIS

Open:

```text
IIS Manager
→ Sites
→ Add Website
```

Configure:

```text
Site name:
MyWebsite

Physical path:
C:\Websites\MyWebsite

Type:
http

IP Address:
All Unassigned

Port:
80

Host name:
mywebsite.example.com
```

Click:

```text
OK
```

---

# 12. IIS Website Architecture

```text
Client
  |
  v
DNS
  |
  v
Windows Server
  |
  v
IIS
  |
  v
MyWebsite
  |
  v
C:\Websites\MyWebsite
```

---

# 13. IIS Bindings

Bindings tell IIS which requests should be handled by a website.

A binding contains:

```text
Protocol
IP Address
Port
Host Name
```

Example:

```text
HTTP
IP: All Unassigned
Port: 80
Host: example.com
```

---

# 14. Multiple Websites on One IIS Server

IIS can host multiple websites using different hostnames.

Example:

```text
example.com
api.example.com
admin.example.com
```

Architecture:

```text
                 IIS
                  |
       +----------+----------+
       |          |          |
       v          v          v
 example.com  api.example  admin.example
    :80          :80          :80
```

---

# 15. Host Header Routing

Example:

```text
example.com
     ↓
Frontend

api.example.com
     ↓
Backend API

admin.example.com
     ↓
Admin Application
```

IIS uses the host name in the binding to determine which website should receive the request.

---

# 16. Ports

Common IIS ports:

```text
HTTP  → 80
HTTPS → 443
```

Example custom port:

```text
http://example.com:8080
```

Check listening ports:

```powershell
Get-NetTCPConnection -State Listen
```

Or:

```powershell
netstat -ano
```

---

# 17. Application Pools

An **Application Pool** provides an isolated execution environment for IIS applications.

Example:

```text
IIS
 |
 +-- AppPool-Frontend
 |
 +-- AppPool-API
 |
 +-- AppPool-Admin
```

Benefits:

* Application isolation
* Process isolation
* Independent recycling
* Better reliability
* Different runtime configurations

---

# 18. Application Pool Process

Typical architecture:

```text
IIS
 |
 +---- Application Pool
           |
           v
       w3wp.exe
           |
           v
      Web Application
```

`w3wp.exe` is the IIS worker process.

---

# 19. Create Application Pool

Open:

```text
IIS Manager
→ Application Pools
→ Add Application Pool
```

Example:

```text
Name:
MyAppPool
```

Choose the appropriate managed runtime/configuration for your application.

---

# 20. Application Pool Identity

Common identities include:

```text
ApplicationPoolIdentity
LocalSystem
LocalService
NetworkService
Specific User
```

Recommended:

```text
ApplicationPoolIdentity
```

Use the minimum permissions required by the application.

---

# 21. Application Pool Recycling

IIS can recycle worker processes.

Reasons:

```text
Memory limits
Scheduled recycling
Configuration changes
Request limits
Time-based recycling
```

Purpose:

```text
Improve application stability
```

---

# 22. Virtual Directory

A virtual directory maps a URL path to a physical directory.

Example:

```text
URL:

example.com/files/

Physical directory:

D:\SharedFiles
```

IIS configuration:

```text
/files/
    ↓
D:\SharedFiles
```

---

# 23. IIS Application

An IIS application provides an application boundary under a website.

Example:

```text
example.com
│
├── /
│
├── /api
│
└── /admin
```

Each can potentially have different application settings and application pools.

---

# 24. IIS Routing

IIS can route requests based on:

```text
Hostname
Port
URL path
Rewrite rules
Application configuration
```

Example:

```text
example.com/
      ↓
Frontend

example.com/api/
      ↓
API

example.com/admin/
      ↓
Admin
```

---

# 25. URL Rewrite

IIS URL Rewrite is used to:

* Redirect URLs
* Rewrite URLs
* Create routing rules
* Force HTTPS
* Redirect domains
* Implement SPA routing

Example:

```text
/old-page
    ↓
/new-page
```

---

# 26. HTTPS Redirect

A common production requirement:

```text
HTTP
 ↓
301 Redirect
 ↓
HTTPS
```

Example:

```text
http://example.com
        ↓
https://example.com
```

IIS can implement this using URL Rewrite.

---

# 27. SSL/TLS in IIS

HTTPS requires:

```text
SSL/TLS Certificate
```

Basic architecture:

```text
Client
  |
  | HTTPS :443
  v
IIS
  |
  | TLS Certificate
  v
Application
```

---

# 28. IIS HTTPS Binding

In IIS:

```text
Site
→ Bindings
→ Add
```

Configure:

```text
Type:
https

Port:
443

Host name:
example.com

SSL certificate:
Your Certificate
```

---

# 29. SSL Certificate Types

Common options:

```text
Let's Encrypt
Commercial CA
Internal CA
Self-signed certificate
```

For production internet-facing websites, use a trusted CA certificate.

---

# 30. Windows Certificate Store

IIS certificates are commonly managed through:

```text
Windows Certificate Store
```

Open:

```text
certlm.msc
```

Common location:

```text
Local Computer
└── Personal
    └── Certificates
```

---

# 31. HTTP to HTTPS

Production architecture:

```text
                 Client
                    |
                    v
              HTTP :80
                    |
                    v
              IIS Redirect
                    |
                    v
              HTTPS :443
                    |
                    v
             Web Application
```

---

# 32. IIS Authentication

IIS supports several authentication methods.

Important methods:

```text
Anonymous Authentication
Windows Authentication
Basic Authentication
Digest Authentication
```

---

# 33. Anonymous Authentication

Users access the website without providing credentials.

Common for:

```text
Public websites
Public APIs
Static websites
```

---

# 34. Windows Authentication

Uses Windows/Active Directory credentials.

Common in:

```text
Corporate intranet
Internal applications
Domain environments
```

Architecture:

```text
User
 |
 v
IIS
 |
 v
Active Directory
 |
 v
Authentication
```

---

# 35. Authorization

Authentication answers:

```text
Who are you?
```

Authorization answers:

```text
What are you allowed to access?
```

Example:

```text
/user
    ↓
Normal users

/admin
    ↓
Administrators
```

---

# 36. IIS Access Control

IIS can restrict access based on:

```text
Users
Groups
IP addresses
Authentication method
Authorization rules
```

---

# 37. IIS Directory Permissions

IIS applications require appropriate Windows filesystem permissions.

Example:

```text
C:\Websites\MyWebsite
```

The IIS application pool identity may need:

```text
Read
```

For uploads:

```text
Read
Write
```

Grant only the permissions actually required.

---

# 38. IIS Logging

IIS provides access logs.

Common location:

```text
C:\inetpub\logs\LogFiles
```

Logs can contain:

```text
Timestamp
Client IP
HTTP Method
URL
Status Code
Response Time
User Agent
```

---

# 39. IIS Log Example

Example:

```text
2026-08-11 10:15:20
192.168.1.10
GET
/index.html
200
```

Meaning:

```text
Client:
192.168.1.10

Request:
GET /index.html

Status:
200 OK
```

---

# 40. Important HTTP Status Codes

```text
200 → Success
301 → Permanent Redirect
302 → Temporary Redirect
400 → Bad Request
401 → Unauthorized
403 → Forbidden
404 → Not Found
500 → Internal Server Error
502 → Bad Gateway
503 → Service Unavailable
```

---

# 41. IIS Error 403

Common causes:

```text
Insufficient permissions
Directory browsing disabled
Authorization rules
IP restrictions
```

Check:

```text
IIS Manager
→ Authentication
→ Authorization Rules
```

---

# 42. IIS Error 404

Possible causes:

```text
File does not exist
Wrong URL
Wrong physical path
Incorrect application
Incorrect routing
Rewrite problem
```

---

# 43. IIS Error 500

Usually application/server configuration related.

Check:

```text
IIS logs
Windows Event Viewer
Application logs
web.config
Application Pool
```

---

# 44. IIS Error 503

Common causes:

```text
Application Pool stopped
Worker process unavailable
Application startup failure
Resource exhaustion
```

Check:

```text
IIS Manager
→ Application Pools
```

---

# 45. Windows Event Viewer

For IIS troubleshooting:

```text
Event Viewer
```

Check:

```text
Windows Logs
    ├── Application
    └── System
```

Also check relevant IIS/application logs.

---

# 46. `web.config`

ASP.NET/IIS applications commonly use:

```text
web.config
```

Example:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>

    <system.webServer>

        <defaultDocument>
            <files>
                <add value="index.html" />
            </files>
        </defaultDocument>

    </system.webServer>

</configuration>
```

---

# 47. IIS Default Document

Common default documents:

```text
index.html
default.htm
default.aspx
```

Configure through:

```text
IIS Manager
→ Default Document
```

---

# 48. Directory Browsing

IIS can display directory contents.

However, it is generally better to keep directory browsing disabled unless there is a specific requirement.

Configuration:

```text
IIS Manager
→ Directory Browsing
```

---

# 49. MIME Types

IIS uses MIME types to identify file types.

Examples:

```text
.html → text/html
.css  → text/css
.js   → text/javascript
.json → application/json
.png  → image/png
```

If IIS does not serve a file type correctly, check:

```text
IIS Manager
→ MIME Types
```

---

# 50. Static Content

For static websites, IIS serves:

```text
HTML
CSS
JavaScript
Images
Fonts
JSON
```

Make sure the **Static Content** IIS feature is installed.

---

# 51. Reverse Proxy with IIS

IIS can work as a reverse proxy using:

```text
URL Rewrite
+
Application Request Routing (ARR)
```

Architecture:

```text
Client
  |
  v
IIS
  |
  v
Backend Application
  |
  v
localhost:5000
```

---

# 52. IIS Reverse Proxy Example

Client requests:

```text
https://example.com/api/users
```

IIS forwards it to:

```text
http://localhost:5000/api/users
```

Architecture:

```text
Browser
   |
   v
IIS :443
   |
   | Reverse Proxy
   v
Backend :5000
```

---

# 53. IIS Routing with Multiple Applications

Example:

```text
example.com/
        ↓
Frontend

example.com/api/
        ↓
Backend :5000

example.com/admin/
        ↓
Admin :4000
```

Conceptually:

```text
                    IIS
                     |
          +----------+----------+
          |          |          |
          v          v          v
       Frontend      API       Admin
        :3000       :5000       :4000
```

URL Rewrite + ARR can implement reverse-proxy routing.

---

# 54. IIS Load Balancing

IIS can participate in load-balancing architectures using ARR and other Microsoft/networking components.

Example:

```text
                  IIS
                   |
        +----------+----------+
        |          |          |
        v          v          v
    Server 1    Server 2    Server 3
```

Traffic is distributed across backend servers.

---

# 55. IIS + ASP.NET

Typical architecture:

```text
Browser
   |
   v
IIS
   |
   v
ASP.NET Application
   |
   v
Database
```

IIS provides:

```text
HTTPS
Authentication
Process management
Logging
Request handling
Application isolation
```

---

# 56. IIS + ASP.NET Core

ASP.NET Core applications commonly run behind IIS.

Architecture:

```text
Client
  |
  v
IIS
  |
  v
ASP.NET Core Application
  |
  v
Database
```

IIS can act as the public-facing web server/reverse proxy while the ASP.NET Core application runs through its hosting model.

---

# 57. IIS + Database Architecture

```text
                  Client
                    |
                    v
                  IIS
                    |
                    v
              Web Application
                    |
                    v
                API/Service
                    |
                    v
                Database
```

Do not expose the database directly to the public internet unless there is a very specific and secured requirement.

---

# 58. IIS Firewall Configuration

Windows Firewall may need to allow:

```text
TCP 80
TCP 443
```

Check firewall rules:

```powershell
Get-NetFirewallRule | Where-Object {$_.Enabled -eq "True"}
```

Create an HTTP rule:

```powershell
New-NetFirewallRule `
    -DisplayName "Allow HTTP 80" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 80 `
    -Action Allow
```

HTTPS:

```powershell
New-NetFirewallRule `
    -DisplayName "Allow HTTPS 443" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 443 `
    -Action Allow
```

---

# 59. Check IIS Port

PowerShell:

```powershell
Get-NetTCPConnection -LocalPort 80
```

HTTPS:

```powershell
Get-NetTCPConnection -LocalPort 443
```

Or:

```powershell
netstat -ano | findstr :80
```

---

# 60. IIS Service

The main IIS service is:

```text
W3SVC
```

Check:

```powershell
Get-Service W3SVC
```

Start:

```powershell
Start-Service W3SVC
```

Stop:

```powershell
Stop-Service W3SVC
```

Restart:

```powershell
Restart-Service W3SVC
```

---

# 61. IIS Installation Verification

Run:

```powershell
Get-WindowsFeature Web-Server
```

Check service:

```powershell
Get-Service W3SVC
```

Check port:

```powershell
netstat -ano | findstr :80
```

Test:

```powershell
Invoke-WebRequest http://localhost
```

---

# 62. IIS Troubleshooting Flow

```text
Client
  |
  v
DNS
  |
  v
Firewall
  |
  v
Port 80/443
  |
  v
IIS
  |
  v
Website
  |
  v
Binding
  |
  v
Application Pool
  |
  v
Application
  |
  v
Database
```

Check each layer one by one.

---

# 63. IIS Troubleshooting Commands

Check IIS service:

```powershell
Get-Service W3SVC
```

Check ports:

```powershell
netstat -ano | findstr ":80"
netstat -ano | findstr ":443"
```

Check HTTP:

```powershell
curl http://localhost
```

Check HTTPS:

```powershell
curl https://localhost
```

Check DNS:

```powershell
nslookup example.com
```

Check connectivity:

```powershell
Test-NetConnection example.com -Port 443
```

---

# 64. IIS Configuration Backup

IIS configuration should be backed up before major changes.

Example:

```powershell
%windir%\system32\inetsrv\appcmd add backup MyIISBackup
```

List backups:

```powershell
%windir%\system32\inetsrv\appcmd list backup
```

---

# 65. AppCmd

`AppCmd.exe` is an IIS command-line administration tool.

Location:

```text
C:\Windows\System32\inetsrv\AppCmd.exe
```

List sites:

```cmd
appcmd list site
```

List application pools:

```cmd
appcmd list apppool
```

List applications:

```cmd
appcmd list app
```

---

# 66. PowerShell IIS Management

List websites:

```powershell
Get-Website
```

List application pools:

```powershell
Get-IISAppPool
```

List IIS applications:

```powershell
Get-IISApp
```

IIS PowerShell cmdlets may require the IIS administration module to be installed/imported.

---

# 67. Production IIS Architecture

```text
                         Internet
                            |
                            v
                           DNS
                            |
                            v
                       Firewall/WAF
                            |
                            v
                       Load Balancer
                            |
                            v
                       IIS Server
                            |
          +-----------------+-----------------+
          |                 |                 |
          v                 v                 v
      Frontend             API              Admin
          |                 |                 |
          v                 v                 v
      App Pool           App Pool          App Pool
          |                 |                 |
          +-----------------+-----------------+
                            |
                            v
                         Database
```

---

# 68. IIS Security Best Practices

1. Use HTTPS.
2. Keep Windows Server updated.
3. Keep IIS updated.
4. Disable unnecessary IIS modules.
5. Use least-privilege application pool identities.
6. Disable directory browsing unless required.
7. Restrict administrative applications.
8. Use Windows Firewall.
9. Monitor IIS logs.
10. Monitor Windows Event Logs.
11. Protect sensitive configuration files.
12. Use trusted TLS certificates.
13. Disable obsolete TLS protocols where appropriate.
14. Keep backend services off public interfaces when possible.
15. Use authentication and authorization correctly.
16. Regularly back up IIS configuration.
17. Avoid exposing server version details unnecessarily.
18. Apply security headers where appropriate.
19. Use rate limiting/WAF protections for internet-facing applications.
20. Monitor application pool failures.

---

# 69. IIS Learning Roadmap

## Beginner

```text
1. What is IIS?
2. IIS installation
3. IIS Manager
4. Default Web Site
5. wwwroot
6. Create website
7. Site bindings
8. Ports
9. Static content
10. Default documents
11. MIME types
```

## Intermediate

```text
12. Application Pools
13. Worker Process
14. Virtual Directories
15. IIS Applications
16. Authentication
17. Authorization
18. Windows Authentication
19. IIS permissions
20. web.config
21. IIS logging
22. URL Rewrite
23. HTTPS
24. SSL certificates
```

## Advanced

```text
25. Reverse Proxy
26. ARR
27. Load Balancing
28. ASP.NET hosting
29. ASP.NET Core hosting
30. API routing
31. WebSocket
32. Security hardening
33. Performance tuning
34. Application Pool recycling
35. IIS monitoring
36. Production architecture
37. Backup and recovery
38. Troubleshooting
```

---

# 70. Practical IIS DevOps Task

### Task 1 — Install IIS

```powershell
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
```

### Task 2 — Verify IIS

```powershell
Get-Service W3SVC
```

### Task 3 — Create Website

```text
C:\Websites\DevOpsApp
```

### Task 4 — Create HTML

```text
C:\Websites\DevOpsApp\index.html
```

### Task 5 — Create IIS Website

Configure:

```text
Site:
DevOpsApp

Path:
C:\Websites\DevOpsApp

Port:
80

Host:
devops.example.com
```

### Task 6 — Test

```powershell
curl http://devops.example.com
```

### Task 7 — Configure HTTPS

```text
Port:
443

Certificate:
Trusted TLS certificate
```

### Task 8 — Redirect HTTP → HTTPS

Implement a permanent redirect.

### Task 9 — Configure Application Pool

Create:

```text
DevOpsAppPool
```

Assign it to the website.

### Task 10 — Enable Logging

Verify:

```text
C:\inetpub\logs\LogFiles
```

### Task 11 — Test Failure Scenarios

Practice troubleshooting:

```text
404
403
500
503
SSL errors
Wrong binding
Stopped application pool
Wrong physical path
Firewall blocking
DNS failure
```

---

# 71. IIS vs Nginx Practical Architecture

### Nginx

```text
Internet
   |
   v
Nginx
   |
   +---- Frontend
   |
   +---- API
   |
   +---- Backend
```

### IIS

```text
Internet
   |
   v
IIS
   |
   +---- Website
   |
   +---- ASP.NET
   |
   +---- ASP.NET Core
   |
   +---- Reverse Proxy
```

---

# 72. Final IIS Summary

The most important IIS concepts for DevOps are:

```text
IIS Installation
IIS Manager
Sites
Bindings
Ports
Application Pools
Worker Process
Virtual Directories
Applications
Static Content
Default Documents
MIME Types
Authentication
Authorization
Windows Authentication
NTFS Permissions
web.config
URL Rewrite
SSL/TLS
HTTPS
IIS Logging
Reverse Proxy
ARR
Load Balancing
ASP.NET
ASP.NET Core
Firewall
AppCmd
PowerShell
Monitoring
Backup
Security
Troubleshooting
Production Architecture
```

Core architecture to remember:

```text
                    Client
                      |
                      v
                     DNS
                      |
                      v
               Windows Firewall
                      |
                      v
                     IIS
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
       Website       API        Admin
          |           |           |
          v           v           v
     App Pool     App Pool    App Pool
          |           |           |
          +-----------+-----------+
                      |
                      v
                   Database
```
