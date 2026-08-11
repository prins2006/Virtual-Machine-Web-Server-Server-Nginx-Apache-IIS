# Let's Encrypt Certificate — Detailed Guide

## 1. What is Let's Encrypt?

**Let's Encrypt** is a free, automated, and publicly trusted **Certificate Authority (CA)**.

It issues TLS certificates that allow websites and APIs to use HTTPS.

Example:

```text
http://example.com
        |
        | TLS certificate
        v
https://example.com
```

The certificate allows a browser to establish an encrypted HTTPS connection with the server.

---

# 2. Let's Encrypt vs ACME vs Certbot

These three terms are related but are not the same.

```text
Let's Encrypt
      |
      | Certificate Authority
      v
     ACME
      |
      | Protocol
      v
   Certbot
      |
      | ACME Client
      v
Your Server
```

### Let's Encrypt

Let's Encrypt is the **Certificate Authority**.

It:

- Validates domain control
- Signs certificates
- Issues certificates
- Supports ACME automation

### ACME

ACME means:

```text
Automatic Certificate Management Environment
```

ACME is the protocol used between the ACME client and the CA.

### Certbot

Certbot is an ACME client.

It can:

- Request certificates
- Complete challenges
- Install certificates
- Configure Nginx/Apache
- Renew certificates

---

# 3. Complete Architecture

```text
                         Internet
                            |
                            v
                     example.com
                            |
                            v
                     DNS Provider
                            |
                  +---------+---------+
                  |                   |
                  v                   v
             A / AAAA Record      ACME DNS TXT
                  |                   |
                  v                   v
             Nginx Server       _acme-challenge
              :80 / :443              |
                  |                   |
                  +---------+---------+
                            |
                            v
                         Certbot
                            |
                          ACME
                            |
                            v
                   +----------------+
                   | Let's Encrypt  |
                   |      CA        |
                   +-------+--------+
                           |
                     Domain Validation
                           |
                           v
                    Certificate Issued
                           |
                           v
                         Nginx
                           |
                           v
                       HTTPS :443
```

---

# 4. Why Do We Need a Certificate?

Without HTTPS:

```text
Browser
   |
   | HTTP
   v
Server
```

HTTP traffic is not protected against network interception.

With HTTPS:

```text
Browser
   |
   | TLS encrypted connection
   v
Nginx
   |
   v
Backend
```

HTTPS provides:

- Encryption
- Server authentication
- Data integrity

---

# 5. Real Example

Assume:

```text
Domain:
example.com

Operating System:
Ubuntu 24.04

Web Server:
Nginx

ACME Client:
Certbot

Certificate Authority:
Let's Encrypt
```

Goal:

```text
https://example.com
```

The architecture is:

```text
Browser
   |
   | HTTPS :443
   v
Nginx
   |
   | Certificate
   v
Let's Encrypt certificate
```

---

# 6. Step 1 — DNS Configuration

Before requesting a certificate, the domain should point to the server.

Example:

```text
example.com
     |
     v
203.0.113.10
```

DNS record:

```text
Type: A
Name: @
Value: 203.0.113.10
```

For IPv6:

```text
Type: AAAA
Name: @
Value: 2001:db8::10
```

Check DNS:

```bash
dig example.com
```

or:

```bash
nslookup example.com
```

Expected:

```text
example.com -> 203.0.113.10
```

---

# 7. Step 2 — Nginx Installation

Install Nginx:

```bash
sudo apt update
sudo apt install nginx
```

Check:

```bash
sudo systemctl status nginx
```

Check port 80:

```bash
sudo ss -tlnp | grep :80
```

Expected:

```text
LISTEN 0 511 0.0.0.0:80
```

---

# 8. Step 3 — Configure Nginx

Example:

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:3000;
    }
}
```

Test:

```bash
sudo nginx -t
```

Reload:

```bash
sudo systemctl reload nginx
```

Test from a client:

```bash
curl http://example.com
```

---

# 9. Step 4 — Open Firewall Ports

For HTTP-01 validation:

```bash
sudo ufw allow 80/tcp
```

For HTTPS:

```bash
sudo ufw allow 443/tcp
```

Check:

```bash
sudo ufw status
```

Also make sure any cloud/VPS security group or external firewall allows:

```text
TCP 80
TCP 443
```

---

# 10. Step 5 — Install Certbot

On Ubuntu:

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

Check:

```bash
certbot --version
```

---

# 11. Certificate Issuance Methods

Let's Encrypt supports different ACME challenge types.

```text
                ACME Validation
                       |
          +------------+------------+
          |            |            |
          v            v            v
       HTTP-01       DNS-01     TLS-ALPN-01
          |            |            |
          v            v            v
       Web server   DNS TXT       TLS :443
```

---

# 12. HTTP-01 Challenge

HTTP-01 proves domain control using an HTTP request.

The CA requests:

```text
http://example.com/.well-known/acme-challenge/<TOKEN>
```

Architecture:

```text
Let's Encrypt
      |
      | HTTP :80
      v
example.com
      |
      v
Nginx
      |
      v
.well-known/acme-challenge/
      |
      v
Challenge response
      |
      v
Validation
```

HTTP-01 normally requires the CA to reach the server over port 80.

---

# 13. Certbot + Nginx HTTP-01 Example

Run:

```bash
sudo certbot --nginx -d example.com
```

For `www` too:

```bash
sudo certbot --nginx \
  -d example.com \
  -d www.example.com
```

Certbot will:

```text
1. Contact Let's Encrypt
2. Create ACME order
3. Configure/serve challenge
4. Let's Encrypt validates domain
5. Certificate is issued
6. Certbot installs certificate
7. Nginx configuration is updated
8. Nginx is reloaded
```

---

# 14. HTTP-01 Architecture

```text
                 Browser
                    |
                    v
                example.com
                    |
                    v
                 Nginx
                /     \
             :80      :443
              |         |
              |         |
       ACME challenge   |
              |         |
              v         v
        Let's Encrypt  HTTPS
              |
              v
          Validation
              |
              v
       Certificate Issued
```

---

# 15. DNS-01 Challenge

DNS-01 proves domain control using a TXT record.

The record is:

```text
_acme-challenge.example.com
```

Example:

```text
Type: TXT

Name:
_acme-challenge

Value:
CHALLENGE-VALUE
```

Architecture:

```text
Certbot
   |
   | ACME
   v
Let's Encrypt
   |
   | DNS Challenge
   v
_acme-challenge.example.com
   |
   v
DNS Provider
   |
   v
TXT record
   |
   v
Let's Encrypt checks DNS
   |
   v
Validation successful
```

---

# 16. Manual DNS-01 Example

Request a certificate:

```bash
sudo certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d example.com
```

For a wildcard:

```bash
sudo certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d "*.example.com"
```

Certbot will display a challenge similar to:

```text
Create a DNS TXT record:

Name:
_acme-challenge.example.com

Value:
ABC123XYZ456
```

Create this record at your DNS provider.

Then verify:

```bash
dig TXT _acme-challenge.example.com
```

Expected:

```text
_acme-challenge.example.com. 300 IN TXT "ABC123XYZ456"
```

After the CA sees the correct value, validation succeeds.

---

# 17. Wildcard Certificate

A wildcard certificate looks like:

```text
*.example.com
```

It can cover:

```text
api.example.com
admin.example.com
dev.example.com
staging.example.com
```

For example:

```text
                   *.example.com
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
           api         admin         dev
         .example     .example     .example
```

For Let's Encrypt, wildcard certificates require DNS-01 validation.

---

# 18. Base Domain + Wildcard

A common request is:

```text
example.com
*.example.com
```

Command:

```bash
sudo certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d example.com \
  -d "*.example.com"
```

This allows:

```text
example.com
api.example.com
admin.example.com
dev.example.com
```

---

# 19. DNS-01 Manual vs Automated

## Manual DNS

```text
Certbot
   |
   v
Let's Encrypt
   |
   v
DNS Challenge
   |
   v
YOU create TXT
   |
   v
DNS Provider
   |
   v
CA validates
   |
   v
Certificate
```

## Automated DNS API

```text
Certbot
   |
   v
Let's Encrypt
   |
   v
DNS Challenge
   |
   v
DNS API
   |
   v
DNS Provider
   |
   v
TXT automatically created
   |
   v
CA validates
   |
   v
Certificate
```

Automated DNS is better for unattended production renewal when supported.

---

# 20. TLS-ALPN-01

TLS-ALPN-01 validates the domain over TLS on port 443.

Architecture:

```text
Let's Encrypt
      |
      | TLS :443
      | ALPN: acme-tls/1
      v
Your Server
      |
      v
ACME validation response
      |
      v
Validation
```

This method is less commonly used than HTTP-01 and DNS-01 for typical Nginx deployments.

---

# 21. Certificate Files

After Certbot obtains a certificate, typical files are stored under:

```text
/etc/letsencrypt/live/example.com/
```

Common files:

```text
cert.pem
chain.pem
fullchain.pem
privkey.pem
```

### cert.pem

The server certificate.

### chain.pem

The CA intermediate certificate chain.

### fullchain.pem

The server certificate plus intermediate certificates.

### privkey.pem

The private key.

The private key must be protected.

---

# 22. Nginx HTTPS Configuration

Example:

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate
        /etc/letsencrypt/live/example.com/fullchain.pem;

    ssl_certificate_key
        /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
    }
}
```

Test:

```bash
sudo nginx -t
```

Reload:

```bash
sudo systemctl reload nginx
```

---

# 23. HTTP to HTTPS Redirect

A common production configuration is:

```nginx
server {
    listen 80;
    server_name example.com;

    return 301 https://$host$request_uri;
}
```

HTTPS server:

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate
        /etc/letsencrypt/live/example.com/fullchain.pem;

    ssl_certificate_key
        /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
    }
}
```

Flow:

```text
http://example.com
       |
       | 301
       v
https://example.com
       |
       v
      Nginx
       |
       v
    Backend
```

---

# 24. Reverse Proxy + Let's Encrypt

Example production architecture:

```text
                       Internet
                           |
                           v
                    example.com
                           |
                           v
                        Nginx
                     :80 / :443
                           |
                   TLS termination
                           |
                           v
                    Backend API
                    localhost:3000
```

Nginx handles:

- HTTPS
- TLS certificate
- HTTP to HTTPS redirect
- Reverse proxy
- Static files
- Caching
- Security headers

The backend can remain on HTTP internally:

```text
Browser
  |
  | HTTPS
  v
Nginx
  |
  | HTTP
  v
Backend
```

The internal connection can also be HTTPS when required by the security architecture.

---

# 25. Let's Encrypt Certificate Lifecycle

```text
             Request
                |
                v
          ACME Account
                |
                v
           Certificate
              Order
                |
                v
           Challenge
                |
                v
        Domain Validation
                |
                v
          CSR / Finalize
                |
                v
       Certificate Issued
                |
                v
          Deploy Certificate
                |
                v
               Nginx
                |
                v
              HTTPS
                |
                v
          Renewal Process
                |
                v
        New Certificate
```

---

# 26. Certificate Renewal

Let's Encrypt certificates have a limited lifetime.

Do not rely on manually remembering the expiration date.

Check Certbot certificates:

```bash
sudo certbot certificates
```

Test renewal:

```bash
sudo certbot renew --dry-run
```

A successful dry run means the renewal process can complete under the tested conditions.

---

# 27. Automatic Renewal

Modern Certbot installations can use a scheduled systemd timer or another scheduler.

Check systemd timers:

```bash
systemctl list-timers | grep certbot
```

Check the timer:

```bash
systemctl status certbot.timer
```

The general architecture is:

```text
systemd timer
      |
      v
certbot renew
      |
      v
Check certificate
      |
      +---- Not due
      |       |
      |       v
      |      Stop
      |
      +---- Due
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
       Deploy Certificate
              |
              v
        Reload Nginx
```

---

# 28. Important Renewal Point

Renewal is more than simply downloading a new certificate.

The complete process is:

```text
Renewal
   |
   v
Domain Validation
   |
   v
New Certificate
   |
   v
Install/Deploy
   |
   v
Reload Nginx
```

If Nginx continues running with the old certificate, the new certificate file may exist on disk while the live process still uses the previous certificate.

---

# 29. Checking Certificate Details

Use OpenSSL:

```bash
openssl x509 \
  -in /etc/letsencrypt/live/example.com/fullchain.pem \
  -noout \
  -subject \
  -issuer \
  -dates
```

Example output:

```text
subject=CN = example.com
issuer=...
notBefore=...
notAfter=...
```

Check the public HTTPS certificate:

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com
```

---

# 30. Check Nginx Ports

```bash
sudo ss -tlnp | grep -E ':80|:443'
```

Expected:

```text
LISTEN ... :80
LISTEN ... :443
```

---

# 31. Check HTTPS

Using curl:

```bash
curl -I https://example.com
```

Expected:

```text
HTTP/2 200
```

or another valid HTTP status such as:

```text
HTTP/2 301
HTTP/2 302
HTTP/2 404
```

The important point for certificate testing is that TLS negotiation succeeds.

---

# 32. Common Let's Encrypt Problems

## Problem 1 — DNS points to the wrong server

Check:

```bash
dig example.com
```

If the returned IP is not your server:

```text
DNS
 |
 X
Wrong IP
```

ACME validation may fail.

---

## Problem 2 — Port 80 blocked

For HTTP-01:

```bash
sudo ss -tlnp | grep :80
```

Check firewall:

```bash
sudo ufw status
```

Allow:

```bash
sudo ufw allow 80/tcp
```

---

# 33. Problem 3 — Port 443 Blocked

After certificate issuance, HTTPS requires:

```bash
sudo ufw allow 443/tcp
```

Check:

```bash
sudo ss -tlnp | grep :443
```

---

# 34. Problem 4 — Nginx Configuration Error

Test:

```bash
sudo nginx -t
```

If the configuration is invalid, fix it before reloading.

Then:

```bash
sudo systemctl reload nginx
```

---

# 35. Problem 5 — DNS-01 TXT Not Visible

Check:

```bash
dig TXT _acme-challenge.example.com
```

Also check authoritative nameservers:

```bash
dig NS example.com
```

Make sure the TXT record is created at the DNS provider responsible for the authoritative zone.

---

# 36. Problem 6 — Certificate Issued but Nginx Shows Old Certificate

Check the configured certificate:

```bash
sudo nginx -T | grep -E 'ssl_certificate|ssl_certificate_key'
```

Check the certificate on disk:

```bash
sudo openssl x509 \
  -in /etc/letsencrypt/live/example.com/fullchain.pem \
  -noout -dates
```

Reload Nginx:

```bash
sudo systemctl reload nginx
```

Then test:

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com
```

---

# 37. Problem 7 — Certbot Cannot Authenticate Domain

Typical causes:

```text
DNS incorrect
Port 80 blocked
Port 443 blocked
Nginx unavailable
Firewall restriction
Cloud security group restriction
Reverse proxy problem
Incorrect DNS TXT record
DNS propagation delay
```

Troubleshooting flow:

```text
Domain
  |
  v
DNS correct?
  |
  +---- No ---> Fix DNS
  |
 Yes
  |
  v
Port reachable?
  |
  +---- No ---> Fix firewall/network
  |
 Yes
  |
  v
Nginx working?
  |
  +---- No ---> Fix Nginx
  |
 Yes
  |
  v
Challenge accessible?
  |
  +---- No ---> Fix ACME routing
  |
 Yes
  |
  v
Validation
```

---

# 38. HTTP-01 vs DNS-01

| Feature | HTTP-01 | DNS-01 |
|---|---|---|
| Challenge location | Web server | DNS |
| Record/file | HTTP challenge file | TXT record |
| Typical port | 80 | DNS |
| Wildcard certificate | No | Yes |
| Requires public web server | Yes | No |
| Nginx required for validation | Usually | No |
| Manual DNS | No | Possible |
| DNS API automation | Not required | Useful |
| Typical use | Normal websites | Wildcards/private infrastructure |

---

# 39. Manual DNS vs Automated DNS

### Manual DNS

```text
Certbot
   |
   v
Let's Encrypt
   |
   v
Challenge
   |
   v
User
   |
   v
DNS TXT
   |
   v
Validation
```

### Automated DNS

```text
Certbot
   |
   v
Let's Encrypt
   |
   v
Challenge
   |
   v
DNS API
   |
   v
DNS TXT
   |
   v
Validation
```

For production wildcard certificates, automated DNS is generally preferable where the DNS provider supports a suitable API/plugin.

---

# 40. Security Best Practices

## Protect the private key

```text
/etc/letsencrypt/live/example.com/privkey.pem
```

Do not:

- Commit it to Git
- Upload it to public storage
- Put it in a public web directory
- Share it in chat or tickets

## Protect DNS API credentials

If automated DNS is used:

```text
ACME Client
     |
     | DNS API credential
     v
DNS Provider
```

Use least privilege.

## Monitor certificate expiry

A production monitoring system should alert before expiration.

## Test renewal

```bash
sudo certbot renew --dry-run
```

## Reload Nginx after deployment

```bash
sudo systemctl reload nginx
```

---

# 41. Let's Encrypt Production Architecture

```text
                         Internet
                            |
                            v
                         DNS
                            |
                            v
                    Load Balancer
                            |
                            v
                  +-------------------+
                  |       Nginx       |
                  |   TLS Termination |
                  +---------+---------+
                            |
                            v
                     Backend Services
                            |
                            v
                         Database


                  Certificate Management
                            |
                            v
                       ACME Client
                            |
                            | ACME
                            v
                    Let's Encrypt CA
                            |
                            v
                    Domain Validation
                            |
                            v
                  Certificate Issuance
                            |
                            v
                 Certificate Deployment
                            |
                            v
                         Nginx
```

---

# 42. Multi-Server Production Architecture

If multiple Nginx servers need the same certificate:

```text
                       Let's Encrypt
                             |
                             v
                       ACME Client
                             |
                             v
                       Certificate
                             |
                 +-----------+-----------+
                 |                       |
                 v                       v
              Nginx-01                Nginx-02
                 |                       |
                 v                       v
             HTTPS :443               HTTPS :443
```

In a larger environment, certificate deployment should be automated and carefully controlled.

Possible approaches include:

- Central certificate management
- Load balancer TLS termination
- DNS-01 validation
- Secure certificate distribution
- Secret management systems

---

# 43. Let's Encrypt and Certificate Trust

When Let's Encrypt issues a certificate:

```text
Let's Encrypt CA
       |
       | Signs certificate
       v
Certificate
       |
       v
Browser
       |
       v
Trust chain verification
```

Modern browsers and operating systems trust the relevant Let's Encrypt certificate chain through their trusted root/intermediate CA mechanisms.

The certificate must contain the requested hostname in its valid identity information, such as:

```text
example.com
```

or:

```text
*.example.com
```

---

# 44. Certificate Components

A TLS certificate contains information such as:

```text
Subject
Issuer
Validity Period
Public Key
Subject Alternative Names (SANs)
Signature
Certificate Extensions
```

Example conceptual certificate:

```text
Subject:
    example.com

Issuer:
    Let's Encrypt

Validity:
    Not Before: ...
    Not After: ...

SAN:
    example.com
    www.example.com
```

---

# 45. Private Key vs Certificate

These are different things.

```text
Private Key
     |
     | Secret
     v
Server
```

Certificate:

```text
Certificate
     |
     | Public information
     v
Browser
```

Nginx needs both:

```nginx
ssl_certificate /path/to/fullchain.pem;
ssl_certificate_key /path/to/privkey.pem;
```

Never expose the private key.

---

# 46. Complete Practical Lab

## Goal

Set up:

```text
example.com
      |
      v
Nginx
      |
      v
Let's Encrypt
      |
      v
HTTPS
```

### Step 1

Verify DNS:

```bash
dig example.com
```

### Step 2

Verify Nginx:

```bash
sudo systemctl status nginx
```

### Step 3

Verify port 80:

```bash
sudo ss -tlnp | grep :80
```

### Step 4

Test HTTP:

```bash
curl -I http://example.com
```

### Step 5

Install Certbot:

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

### Step 6

Request certificate:

```bash
sudo certbot --nginx -d example.com
```

### Step 7

Test Nginx:

```bash
sudo nginx -t
```

### Step 8

Test HTTPS:

```bash
curl -I https://example.com
```

### Step 9

Check certificate:

```bash
sudo certbot certificates
```

### Step 10

Test renewal:

```bash
sudo certbot renew --dry-run
```

---

# 47. Practical Lab — Manual DNS Wildcard

Goal:

```text
*.example.com
```

### Step 1

Run:

```bash
sudo certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d "*.example.com"
```

### Step 2

Copy the challenge information.

### Step 3

Create:

```text
_acme-challenge.example.com
```

as a TXT record.

### Step 4

Verify:

```bash
dig TXT _acme-challenge.example.com
```

### Step 5

Continue Certbot.

### Step 6

Check:

```bash
sudo certbot certificates
```

### Step 7

Configure Nginx.

### Step 8

Test:

```bash
sudo nginx -t
```

### Step 9

Reload:

```bash
sudo systemctl reload nginx
```

### Step 10

Test:

```bash
curl -I https://api.example.com
```

---

# 48. Important Commands Cheat Sheet

## Install

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

## Nginx certificate

```bash
sudo certbot --nginx -d example.com
```

## Manual DNS

```bash
sudo certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d example.com
```

## Wildcard

```bash
sudo certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d "*.example.com"
```

## List certificates

```bash
sudo certbot certificates
```

## Test renewal

```bash
sudo certbot renew --dry-run
```

## Check DNS

```bash
dig example.com
```

## Check DNS TXT

```bash
dig TXT _acme-challenge.example.com
```

## Check Nginx

```bash
sudo nginx -t
```

## Reload Nginx

```bash
sudo systemctl reload nginx
```

## Check ports

```bash
sudo ss -tlnp | grep -E ':80|:443'
```

## Check certificate

```bash
openssl x509 \
  -in /etc/letsencrypt/live/example.com/fullchain.pem \
  -noout -subject -issuer -dates
```

---

# 49. Key Concepts to Remember

```text
Let's Encrypt
    =
Certificate Authority

ACME
    =
Certificate automation protocol

Certbot
    =
ACME client

HTTP-01
    =
Web-server validation

DNS-01
    =
DNS TXT validation

TLS-ALPN-01
    =
TLS :443 validation

Wildcard
    =
*.example.com

fullchain.pem
    =
Certificate + intermediate chain

privkey.pem
    =
Private key

certbot renew
    =
Certificate renewal

certbot renew --dry-run
    =
Test renewal
```

---

# 50. Final Architecture

```text
                              INTERNET
                                  |
                                  v
                             example.com
                                  |
                                  v
                             DNS Provider
                                  |
                       +----------+----------+
                       |                     |
                       v                     v
                   A / AAAA              TXT record
                       |              _acme-challenge
                       v                     |
                    Nginx                    |
                  :80 / :443                |
                       |                     |
                       v                     |
                    Certbot                  |
                       |                     |
                       | ACME                |
                       +----------+----------+
                                  |
                                  v
                         Let's Encrypt CA
                                  |
                                  v
                         Domain Validation
                                  |
                                  v
                       Certificate Issued
                                  |
                                  v
                      /etc/letsencrypt/
                                  |
                                  v
                                Nginx
                                  |
                                  v
                             HTTPS :443
                                  |
                                  v
                              Backend
```

# 51. Final Takeaway

The complete relationship is:

```text
Let's Encrypt
     |
     | Certificate Authority
     v
    ACME
     |
     | Protocol
     v
   Certbot
     |
     | Certificate request
     v
Domain Validation
     |
     +---- HTTP-01
     |
     +---- DNS-01
     |
     +---- TLS-ALPN-01
     |
     v
Validation Success
     |
     v
Let's Encrypt signs certificate
     |
     v
Certbot installs/deploys certificate
     |
     v
Nginx
     |
     v
HTTPS
     |
     v
Automatic Renewal
```

The most important distinction is:

```text
Let's Encrypt = CA
ACME          = Protocol
Certbot       = Client
DNS-01        = Validation method
TXT record    = DNS proof
Certificate   = TLS identity
Nginx         = Server using the certificate
```

## References

- Let's Encrypt — Challenge Types:
  https://letsencrypt.org/docs/challenge-types/

- Let's Encrypt — Getting Started:
  https://letsencrypt.org/getting-started/

- Certbot Documentation:
  https://eff-certbot.readthedocs.io/

- Certbot Instructions:
  https://certbot.eff.org/
