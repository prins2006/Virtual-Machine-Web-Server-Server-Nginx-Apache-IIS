# ACME Protocol, Certbot, acme.sh, and win-acme

## 1. Overview

**ACME** stands for **Automatic Certificate Management Environment**.

ACME is a protocol used to automate TLS certificate management between an **ACME client** and a **Certificate Authority (CA)**.

Common ACME clients include:

- Certbot
- acme.sh
- win-acme

Common ACME-compatible Certificate Authorities include:

- Let's Encrypt
- ZeroSSL
- Other ACME-compatible CAs

The key idea is:

```text
ACME = Protocol
Certbot = ACME Client
acme.sh = ACME Client
win-acme = ACME Client
CA = Certificate Issuer
```

---

# 2. Why ACME is Needed

Without ACME, certificate management can be manual:

```text
Generate Private Key
        ↓
Generate CSR
        ↓
Submit CSR to CA
        ↓
Prove Domain Ownership
        ↓
Receive Certificate
        ↓
Install Certificate
        ↓
Renew Certificate Before Expiration
```

ACME automates most of this lifecycle:

```text
ACME Client
    ↓
Request Certificate
    ↓
Domain Validation
    ↓
CA Issues Certificate
    ↓
Install Certificate
    ↓
Automatic Renewal
```

---

# 3. ACME Architecture

```text
                         Internet
                            |
                            v
                   +------------------+
                   |     Domain       |
                   |  example.com     |
                   +--------+---------+
                            |
                            v
                   +------------------+
                   | Web Server       |
                   | Nginx / Apache   |
                   | IIS              |
                   +--------+---------+
                            |
                            |
                     ACME Client
             +--------------+--------------+
             |              |              |
             v              v              v
         Certbot         acme.sh       win-acme
             |              |              |
             +--------------+--------------+
                            |
                     ACME Protocol
                            |
                            v
                   +------------------+
                   | ACME Server / CA |
                   | Let's Encrypt    |
                   +--------+---------+
                            |
                            v
                     Domain Challenge
                    /       |        \
                   /        |         \
              HTTP-01     DNS-01    TLS-ALPN-01
                   \        |         /
                    \       |        /
                     +------+-------+
                            |
                            v
                       Validation
                            |
                            v
                     Signed Certificate
                            |
                            v
                  Nginx / Apache / IIS
                            |
                            v
                          HTTPS
```

---

# 4. Main ACME Components

## 4.1 ACME Client

The ACME client runs on your server or management system.

Examples:

```text
Certbot
acme.sh
win-acme
```

Main responsibilities:

- Create/manage ACME account
- Generate or use cryptographic keys
- Create certificate orders
- Complete challenges
- Submit certificate requests
- Download certificates
- Install/deploy certificates
- Renew certificates

---

## 4.2 ACME Server

The ACME server is the CA's ACME API endpoint.

It:

- Receives ACME requests
- Creates certificate orders
- Provides challenges
- Validates domain control
- Issues certificates

---

## 4.3 Certificate Authority

The CA signs and issues the certificate after successful validation.

Example:

```text
Let's Encrypt
```

The CA is responsible for:

```text
Domain Validation
       ↓
Certificate Signing
       ↓
Certificate Issuance
```

---

# 5. ACME Certificate Lifecycle

```text
             ACME Client
                  |
                  v
          Create ACME Account
                  |
                  v
           Create Order
                  |
                  v
          Receive Challenge
                  |
                  v
       Prove Domain Ownership
                  |
                  v
          CA Validates Domain
                  |
                  v
          Submit Certificate CSR
                  |
                  v
           CA Signs Certificate
                  |
                  v
          Download Certificate
                  |
                  v
         Install/Deploy Certificate
                  |
                  v
             HTTPS Active
                  |
                  v
          Automatic Renewal
```

---

# 6. Step 1 - ACME Account

The ACME client first creates or uses an ACME account.

Conceptually:

```text
ACME Client
    |
    | Account Registration
    v
ACME Server
```

The account uses a cryptographic key pair:

```text
ACME Account
     |
     +---- Private Key
     |
     +---- Public Key
```

The private key is used to authenticate ACME requests.

---

# 7. Step 2 - Certificate Order

The client requests a certificate.

Example:

```text
example.com
```

The ACME server creates an order.

```text
ACME Client
     |
     | Certificate Order
     | example.com
     v
ACME Server
     |
     v
Challenges
```

The order contains information about the requested identifiers and validation requirements.

---

# 8. Step 3 - Domain Validation

The CA must verify:

> Does the requester control the domain?

ACME commonly uses:

- HTTP-01
- DNS-01
- TLS-ALPN-01

---

# 9. HTTP-01 Challenge

HTTP-01 is commonly used with Nginx and Apache.

The ACME client receives a challenge token.

Conceptually:

```text
Token:
abc123xyz
```

The client makes the expected challenge response available at:

```text
http://example.com/.well-known/acme-challenge/abc123xyz
```

Architecture:

```text
             Let's Encrypt
                   |
                   | HTTP Request
                   v
          example.com:80
                   |
                   v
                Nginx
                   |
                   v
        /.well-known/acme-challenge/
                   |
                   v
             Challenge File
                   |
                   v
              Verification
```

If the CA receives the expected response, domain control is validated.

### Important

HTTP-01 normally requires the CA to reach the server over:

```text
TCP Port 80
```

Therefore:

```text
DNS
 ↓
Public IP
 ↓
Firewall
 ↓
Port 80
 ↓
Nginx
```

must be correctly configured.

---

# 10. DNS-01 Challenge

DNS-01 proves domain control through a DNS TXT record.

The record is placed under:

```text
_acme-challenge.example.com
```

Example:

```text
_acme-challenge.example.com
        |
        | TXT
        v
"challenge-value"
```

Architecture:

```text
              ACME Client
                   |
                   v
             ACME Server
                   |
                   | Challenge
                   v
              DNS Provider
                   |
                   v
       _acme-challenge.example.com
                   |
                   v
               TXT Record
                   |
                   v
              CA Validation
                   |
                   v
              Certificate
```

## Advantages of DNS-01

DNS-01 is useful when:

- The web server is not publicly reachable
- You need wildcard certificates
- You want centralized DNS-based validation

Example wildcard:

```text
*.example.com
```

A wildcard certificate can cover:

```text
api.example.com
admin.example.com
dev.example.com
```

For Let's Encrypt, wildcard issuance requires DNS-01 validation.

---

# 11. TLS-ALPN-01 Challenge

TLS-ALPN-01 performs validation over TLS.

It uses:

```text
TCP 443
```

and the special ALPN protocol:

```text
acme-tls/1
```

Architecture:

```text
             ACME CA
                |
                | TLS
                | ALPN: acme-tls/1
                v
             Server:443
                |
                v
       ACME Validation Response
                |
                v
             Validation
```

This is mainly useful for specialized TLS infrastructure.

---

# 12. Certbot

Certbot is a popular ACME client, especially on Linux systems running Nginx or Apache.

Typical environment:

```text
Ubuntu
   |
   +---- Nginx
   |
   +---- Certbot
   |
   +---- Let's Encrypt
```

Example:

```bash
sudo certbot --nginx -d example.com
```

Certbot can:

- Request certificates
- Perform domain validation
- Configure Nginx
- Configure Apache
- Install certificates
- Renew certificates

---

# 13. Certbot Architecture

```text
                 Internet
                    |
                    v
              example.com
                    |
                    v
              +-----------+
              |   Nginx   |
              | :80 :443  |
              +-----+-----+
                    |
                    v
              +-----------+
              |  Certbot  |
              +-----+-----+
                    |
                    | ACME
                    v
          +---------------------+
          | Let's Encrypt / CA  |
          +----------+----------+
                     |
                     v
                Validation
                     |
                     v
                Certificate
                     |
                     v
                  Nginx
                     |
                     v
                  HTTPS
```

---

# 14. Certbot Example

Command:

```bash
sudo certbot --nginx -d example.com
```

Simplified process:

```text
Certbot
   |
   | 1. Contact ACME server
   v
CA
   |
   | 2. Send challenge
   v
Certbot / Nginx
   |
   | 3. Make challenge available
   v
CA
   |
   | 4. Validate domain
   v
CA
   |
   | 5. Issue certificate
   v
Certbot
   |
   | 6. Install certificate
   v
Nginx
   |
   | 7. Reload
   v
HTTPS
```

---

# 15. Certbot and Nginx

A common Nginx configuration after certificate installation looks like:

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
    }
}
```

The certificate files commonly used by Nginx are:

```text
fullchain.pem
privkey.pem
```

---

# 16. Certbot Renewal

Certificates have limited lifetimes.

Certbot can automatically check whether certificates need renewal.

Architecture:

```text
Cron / Systemd Timer
        |
        v
     Certbot
        |
        v
Check Certificate
        |
        +------ Still valid ------> Nothing
        |
        +------ Renewal needed
                    |
                    v
              ACME Order
                    |
                    v
                Challenge
                    |
                    v
                Validation
                    |
                    v
             New Certificate
                    |
                    v
             Install Certificate
                    |
                    v
             Reload Nginx
```

Useful command:

```bash
sudo certbot renew --dry-run
```

This tests the renewal process without actually replacing the production certificate.

---

# 17. acme.sh

`acme.sh` is a lightweight ACME client implemented as a shell script.

It is useful on:

- Linux
- Unix-like systems
- Nginx
- Apache
- DNS automation
- Lightweight environments

Architecture:

```text
             Linux Server
                  |
                  v
              acme.sh
                  |
                  | ACME
                  v
             ACME Server
                  |
                  v
                  CA
                  |
                  v
             Validation
                  |
                  v
             Certificate
                  |
                  v
          Nginx / Apache
```

---

# 18. acme.sh Webroot Mode

Example:

```bash
acme.sh --issue \
  -d example.com \
  -w /var/www/html
```

The client uses the webroot:

```text
/var/www/html
```

The challenge becomes available under:

```text
/var/www/html/.well-known/acme-challenge/
```

Architecture:

```text
acme.sh
   |
   v
Create Challenge
   |
   v
/var/www/html/.well-known/acme-challenge/
   |
   v
Nginx
   |
   v
Internet
   |
   v
ACME CA
   |
   v
Validation
```

---

# 19. acme.sh Nginx Mode

Example:

```bash
acme.sh --issue \
  --nginx \
  -d example.com
```

acme.sh can use Nginx during validation.

After obtaining the certificate, it is recommended to explicitly install/deploy it.

Example:

```bash
acme.sh --install-cert -d example.com \
  --key-file /etc/nginx/ssl/example.key \
  --fullchain-file /etc/nginx/ssl/example.pem \
  --reloadcmd "systemctl reload nginx"
```

The reload command ensures that Nginx starts using the renewed certificate.

---

# 20. acme.sh DNS-01

acme.sh can automate DNS validation using supported DNS provider APIs.

Architecture:

```text
                 acme.sh
                    |
                    | ACME
                    v
                  CA
                    |
                    | DNS Challenge
                    v
              DNS Provider API
                    |
                    v
         _acme-challenge.example.com
                    |
                    v
               TXT Record
                    |
                    v
                Validation
                    |
                    v
              Certificate
```

This is especially useful for wildcard certificates.

---

# 21. acme.sh Automatic Renewal

acme.sh can install a scheduled task/cron job to check certificates.

```text
Cron
 |
 v
acme.sh
 |
 v
Check expiration
 |
 +---- No renewal needed
 |
 +---- Renewal needed
          |
          v
       Challenge
          |
          v
       Validation
          |
          v
   New Certificate
          |
          v
     Install Cert
          |
          v
     Reload Nginx
```

---

# 22. win-acme

win-acme is an ACME client designed primarily for Windows environments, especially IIS.

Typical environment:

```text
Windows Server
      |
      +---- IIS
      |
      +---- win-acme
      |
      +---- Let's Encrypt
```

The executable is commonly referred to as:

```text
wacs.exe
```

---

# 23. win-acme Architecture

```text
                    Internet
                       |
                       v
                 example.com
                       |
                       v
                  +---------+
                  |   IIS   |
                  | :80 :443|
                  +----+----+
                       |
                       v
                  +---------+
                  | win-acme|
                  | wacs.exe|
                  +----+----+
                       |
                       | ACME
                       v
                 +-----------+
                 |    CA     |
                 | Let's     |
                 | Encrypt   |
                 +-----+-----+
                       |
                       v
                  Validation
                       |
                       v
                  Certificate
                       |
                       v
             Windows Certificate
                  Store
                       |
                       v
                 IIS Binding
                       |
                       v
                    HTTPS
```

---

# 24. win-acme Plugin Architecture

win-acme has a plugin-oriented design.

Conceptually:

```text
                 win-acme
                    |
          +---------+---------+
          |         |         |
          v         v         v
       Source     Order      CSR
          |
          v
      Validation
          |
          v
        Store
          |
          v
     Installation
```

Typical stages:

```text
IIS Site
   ↓
Find Domains
   ↓
Create Order
   ↓
Create CSR
   ↓
Domain Validation
   ↓
Certificate Issued
   ↓
Windows Certificate Store
   ↓
IIS HTTPS Binding
```

---

# 25. win-acme HTTP Validation

A common Windows/IIS workflow uses HTTP validation.

```text
Let's Encrypt
      |
      | HTTP Request
      v
Windows Server :80
      |
      v
win-acme / IIS validation
      |
      v
Challenge Response
      |
      v
Let's Encrypt
      |
      v
Validation Success
```

After validation:

```text
Certificate
     |
     v
Windows Certificate Store
     |
     v
IIS HTTPS Binding
     |
     v
HTTPS :443
```

---

# 26. win-acme DNS Validation

DNS validation can also be used.

```text
win-acme
    |
    | ACME
    v
CA
    |
    | DNS Challenge
    v
DNS Provider
    |
    v
_acme-challenge.example.com
    |
    | TXT
    v
Validation
    |
    v
Certificate
```

DNS-01 is particularly useful for wildcard certificates.

---

# 27. Certbot vs acme.sh vs win-acme

| Feature | Certbot | acme.sh | win-acme |
|---|---|---|---|
| Primary platform | Linux/Unix | Linux/Unix | Windows |
| Implementation | Python | Shell | .NET |
| Nginx | Excellent | Excellent | Not primary |
| Apache | Excellent | Excellent | Not primary |
| IIS | Not primary | Not primary | Excellent |
| HTTP-01 | Yes | Yes | Yes |
| DNS-01 | Yes | Yes | Yes |
| TLS-ALPN-01 | Yes | Yes | Yes |
| Wildcard | Yes with DNS-01 | Yes with DNS-01 | Yes with DNS-01 |
| Automatic renewal | Yes | Yes | Yes |
| Nginx integration | Strong | Flexible | Not primary |
| Windows integration | Limited | Limited | Strong |
| Best learning choice for Linux/Nginx | **Yes** | Yes | No |

---

# 28. Important Difference

The protocol remains the same:

```text
                 ACME Protocol
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
       Certbot      acme.sh    win-acme
          |           |           |
          v           v           v
       Linux        Linux       Windows
       Nginx        Nginx        IIS
```

The clients differ in:

- Operating system integration
- Web-server integration
- DNS automation
- Installation/deployment
- Configuration
- Renewal mechanism
- User interface

---

# 29. ACME Client vs Certificate Authority

This distinction is important.

## ACME Clients

```text
Certbot
acme.sh
win-acme
```

They:

```text
Request
Validate
Download
Install
Renew
```

## Certificate Authority

```text
Let's Encrypt
ZeroSSL
Other ACME-compatible CAs
```

The CA:

```text
Validate
Sign
Issue
Revoke
```

Architecture:

```text
             ACME Client
                  |
                  | ACME Protocol
                  v
              ACME Server
                  |
                  v
           Certificate Authority
                  |
                  v
           Signed Certificate
```

---

# 30. Certificate Deployment

Obtaining a certificate is only part of the job.

The certificate must be deployed to the web server.

### Linux/Nginx

```text
ACME Client
    |
    v
Certificate Files
    |
    +---- Certificate
    |
    +---- Private Key
    |
    v
Nginx
    |
    v
Reload
    |
    v
HTTPS
```

### Windows/IIS

```text
win-acme
    |
    v
Certificate
    |
    v
Windows Certificate Store
    |
    v
IIS Binding
    |
    v
HTTPS
```

---

# 31. Production Architecture

A production environment may look like:

```text
                         Internet
                            |
                            v
                       DNS Provider
                            |
                            v
                     Load Balancer
                            |
              +-------------+-------------+
              |                           |
              v                           v
        Nginx Server 1              Nginx Server 2
              |                           |
              +-------------+-------------+
                            |
                         Backend
```

Certificate automation can be centralized:

```text
                    ACME CA
                       |
                       | ACME
                       v
               Certificate Manager
                       |
               +-------+-------+
               |               |
               v               v
           Nginx-01        Nginx-02
               |               |
               v               v
          Certificate     Certificate
```

For DNS-01:

```text
                    ACME CA
                       |
                       v
                 ACME Client
                       |
                       | DNS API
                       v
                 DNS Provider
                       |
                       v
             TXT Challenge Record
                       |
                       v
                  Validation
                       |
                       v
                  Certificate
```

---

# 32. Complete Real-World Flow

Suppose you have:

```text
Domain:
prinsdevops.dpdns.org

Server:
Ubuntu

Web Server:
Nginx

ACME Client:
Certbot
```

The complete flow is:

```text
1. DNS
   |
   v
prinsdevops.dpdns.org
   |
   v
Public IP
   |
   v

2. Internet
   |
   v
Port 80
   |
   v

3. Nginx
   |
   v

4. Certbot
   |
   | ACME
   v

5. Let's Encrypt
   |
   v

6. HTTP-01 Challenge
   |
   v

7. Nginx serves challenge
   |
   v

8. Let's Encrypt validates
   |
   v

9. Certificate issued
   |
   v

10. Certbot installs certificate
   |
   v

11. Nginx reload
   |
   v

12. HTTPS :443
```

---

# 33. Renewal Architecture

Certificate automation should continue after initial issuance.

```text
              Scheduler
             /    |    \
            /     |     \
        Cron   Systemd   Windows Task
            \     |     /
             \    |    /
              ACME Client
                   |
                   v
             Check Expiry
                   |
             +-----+------+
             |            |
          Valid       Renewal Needed
             |            |
             v            v
            Stop       ACME Order
                            |
                            v
                         Challenge
                            |
                            v
                         Validate
                            |
                            v
                      New Certificate
                            |
                            v
                         Deploy
                            |
                            v
                    Reload Web Server
                            |
                            v
                         HTTPS
```

---

# 34. Common Problems

## Problem 1: DNS is incorrect

```text
Domain
  |
  X
Wrong IP
```

Result:

```text
ACME validation fails
```

Check:

```bash
dig example.com
```

or:

```bash
nslookup example.com
```

---

## Problem 2: Port 80 blocked

HTTP-01 requires the CA to reach the challenge endpoint.

Check:

```bash
sudo ss -tlnp | grep :80
```

Check firewall:

```bash
sudo ufw status
```

Allow HTTP:

```bash
sudo ufw allow 80/tcp
```

Allow HTTPS:

```bash
sudo ufw allow 443/tcp
```

---

# 35. Common Nginx ACME Problems

### Nginx is not running

```bash
sudo systemctl status nginx
```

### Configuration error

```bash
sudo nginx -t
```

### Reload Nginx

```bash
sudo systemctl reload nginx
```

### Check ports

```bash
sudo ss -tlnp | grep -E ':80|:443'
```

### Check DNS

```bash
dig example.com
```

---

# 36. Security Considerations

Protect:

```text
Private Key
ACME Account Key
DNS API Credentials
```

Never expose private keys publicly.

For DNS automation, use the minimum required DNS API permissions.

Example architecture:

```text
ACME Client
    |
    +---- Private Key
    |
    +---- DNS API Credential
    |
    v
DNS Provider
```

The DNS credential should not have unnecessary permissions.

---

# 37. Best Practices

## Use automatic renewal

Do not depend on manually remembering expiration dates.

## Test renewal

For Certbot:

```bash
sudo certbot renew --dry-run
```

## Monitor expiration

Use monitoring to detect certificates that are approaching expiration.

## Protect private keys

Example:

```text
/etc/letsencrypt/
```

should not be publicly served.

## Use DNS-01 for wildcard certificates

Example:

```text
*.example.com
```

## Keep port 80 available when using HTTP-01

```text
Internet
   ↓
TCP 80
   ↓
Nginx
```

## Reload the web server after renewal

A new certificate on disk does not automatically mean an already-running Nginx process is using it.

---

# 38. Interview Questions

## Q1. What is ACME?

ACME is a protocol that automates TLS certificate issuance and management.

## Q2. What is Certbot?

Certbot is an ACME client commonly used with Linux, Nginx, and Apache.

## Q3. What is acme.sh?

acme.sh is a lightweight shell-based ACME client.

## Q4. What is win-acme?

win-acme is an ACME client designed primarily for Windows and IIS.

## Q5. What is HTTP-01?

A challenge where the CA verifies a token through an HTTP endpoint on the domain.

## Q6. What is DNS-01?

A challenge where the CA verifies a TXT record under `_acme-challenge`.

## Q7. Which challenge is required for wildcard certificates?

DNS-01.

## Q8. Why can HTTP-01 fail?

Common causes include:

- Incorrect DNS
- Port 80 blocked
- Wrong public IP
- Nginx/IIS not reachable
- Reverse proxy misconfiguration
- Incorrect challenge routing

## Q9. Does Certbot issue certificates?

No.

Certbot requests certificates from an ACME-compatible CA. The CA issues the certificate.

## Q10. Are Certbot and ACME the same?

No.

```text
ACME    = Protocol
Certbot = Client
```

---

# 39. Quick Memory Diagram

```text
                ACME
              Protocol
                 |
       +---------+---------+
       |         |         |
       v         v         v
    Certbot   acme.sh   win-acme
       |         |         |
       v         v         v
    Linux      Linux     Windows
    Nginx      Nginx       IIS
       \         |         /
        \        |        /
         +-------+-------+
                 |
                 v
              ACME CA
                 |
                 v
             Challenge
          /      |       \
         /       |        \
    HTTP-01    DNS-01   TLS-ALPN-01
         \       |        /
          \      |       /
             Validation
                 |
                 v
            Certificate
                 |
                 v
        Install / Deploy
                 |
                 v
              HTTPS
                 |
                 v
             Renewal
```

---

# 40. Recommended Learning Path

For Linux + Nginx + DNS + SSL/TLS:

```text
1. ACME fundamentals
       ↓
2. ACME account
       ↓
3. ACME order
       ↓
4. HTTP-01
       ↓
5. DNS-01
       ↓
6. TLS-ALPN-01
       ↓
7. Certbot + Nginx
       ↓
8. Certbot renewal
       ↓
9. acme.sh
       ↓
10. DNS automation
       ↓
11. Wildcard certificates
       ↓
12. win-acme + IIS
       ↓
13. Production certificate automation
       ↓
14. Monitoring and troubleshooting
```

---

# 41. Final Summary

```text
ACME
  |
  | Protocol
  v
ACME Client
  |
  +---- Certbot
  |
  +---- acme.sh
  |
  +---- win-acme
  |
  v
ACME Server / CA
  |
  v
Domain Challenge
  |
  +---- HTTP-01
  |
  +---- DNS-01
  |
  +---- TLS-ALPN-01
  |
  v
Domain Validation
  |
  v
Certificate Issued
  |
  v
Certificate Deployment
  |
  v
Nginx / Apache / IIS
  |
  v
HTTPS
  |
  v
Automatic Renewal
```

**Key concept to remember:**

> **ACME is the protocol, Certbot/acme.sh/win-acme are clients, the CA issues the certificate, challenges prove domain control, and automatic renewal keeps the certificate valid.**
