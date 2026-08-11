# ACME Manual DNS-01 Challenge — Detailed Guide

## 1. Overview

ACME (Automatic Certificate Management Environment) is a protocol used to automate TLS certificate issuance and renewal.

An ACME client such as Certbot communicates with an ACME-compatible Certificate Authority (CA).

Examples of ACME clients:

- Certbot
- acme.sh
- win-acme

This document focuses on:

- ACME DNS-01
- Manual DNS validation
- Certbot manual DNS
- Wildcard certificates
- DNS TXT records
- DNS propagation
- Certificate issuance
- Renewal
- Manual DNS vs automated DNS API
- Nginx architecture
- Troubleshooting
- Production recommendations

> Important: ACME is the protocol. The ACME client performs the ACME operations. The Certificate Authority validates domain control and issues the certificate.

---

# 2. What is Manual DNS-01?

With the **manual DNS-01 challenge**, the ACME client asks you to create a specific DNS TXT record.

You manually add the TXT record to your DNS provider.

The CA then queries DNS and verifies that the expected value exists.

```text
ACME Client
     |
     | Certificate request
     v
ACME Server / CA
     |
     | DNS-01 Challenge
     v
"Create this TXT record"
     |
     v
You manually create TXT record
     |
     v
DNS Provider
     |
     v
Public DNS
     |
     v
CA checks TXT record
     |
     v
Validation successful
     |
     v
Certificate issued
```

Let's Encrypt documents DNS-01 as a challenge where domain control is proven by publishing a specific TXT record at `_acme-challenge.<DOMAIN>`. DNS-01 also supports wildcard certificates. [1]

---

# 3. Why DNS-01 is Used

DNS-01 is useful when:

- You need a wildcard certificate.
- The web server is not publicly reachable.
- HTTP-01 cannot be used.
- You have multiple web servers.
- Certificate validation should be independent of the web server.
- You want DNS-based domain-control verification.

For Let's Encrypt wildcard certificates, DNS-01 is required. [1]

Example:

```text
*.example.com
```

can cover:

```text
api.example.com
admin.example.com
dev.example.com
www.example.com
```

---

# 4. Manual DNS vs Automated DNS

There are two important DNS-01 approaches.

## Manual DNS

```text
ACME Client
     |
     v
CA
     |
     v
DNS Challenge
     |
     v
YOU
     |
     v
DNS Provider
     |
     v
TXT Record
     |
     v
CA validates
```

You manually create the TXT record.

## Automated DNS

```text
ACME Client
     |
     v
CA
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
TXT Record
     |
     v
CA validates
```

The ACME client automatically creates and removes the TXT record through the DNS provider's API.

For production, automated DNS updates are generally preferable when supported because certificate issuance and renewal should be automated. Let's Encrypt specifically recommends DNS-01 automation when the DNS provider has an API. [1]

---

# 5. Complete Architecture

```text
                         Internet
                            |
                            v
                    +---------------+
                    | DNS Provider  |
                    |               |
                    | example.com   |
                    +-------+-------+
                            |
                            |
                 +----------+----------+
                 |                     |
                 v                     v
           A / AAAA Record        TXT Record
           example.com            _acme-challenge
                 |                     |
                 v                     |
              Nginx                    |
              :80/:443                 |
                 |                     |
                 +----------+----------+
                            |
                            |
                      ACME Client
                       (Certbot)
                            |
                            | ACME
                            v
                  +--------------------+
                  | ACME Server / CA   |
                  | Let's Encrypt      |
                  +--------------------+
                            |
                            |
                            | DNS lookup
                            v
                   _acme-challenge
                         TXT
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
                       HTTPS :443
```

---

# 6. Real Example

Assume:

```text
Domain:
example.com

Server:
Ubuntu 24.04

Web Server:
Nginx

ACME Client:
Certbot

Certificate:
*.example.com
```

The goal is:

```text
*.example.com
```

The certificate can then be used for:

```text
api.example.com
admin.example.com
dev.example.com
staging.example.com
```

---

# 7. Step 1 — Install Certbot

On Ubuntu:

```bash
sudo apt update
sudo apt install certbot
```

Check:

```bash
certbot --version
```

Example:

```text
certbot 5.x.x
```

The exact version depends on your package source.

---

# 8. Step 2 — Request a Manual DNS Certificate

Use:

```bash
sudo certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d "*.example.com"
```

If you also want the base domain:

```bash
sudo certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d "*.example.com" \
  -d "example.com"
```

Certbot's manual plugin supports DNS validation and asks the administrator to perform the DNS validation steps. [2]

---

# 9. Step 3 — Certbot Displays a DNS Challenge

Certbot will display instructions similar to:

```text
Please deploy a DNS TXT record under the name:

_acme-challenge.example.com

with the following value:

ABC123XYZ456
```

The exact value is generated for your specific ACME order.

Do not copy this example value into a real DNS zone.

---

# 10. Step 4 — Open DNS Provider

Log in to the DNS provider that is authoritative for:

```text
example.com
```

Create a TXT record.

Example:

```text
Type:
TXT

Name:
_acme-challenge

Value:
ABC123XYZ456

TTL:
300
```

The resulting FQDN is:

```text
_acme-challenge.example.com
```

---

# 11. DNS Zone Example

Your DNS zone might look like:

```text
example.com
|
+-- A
|   +-- 203.0.113.10
|
+-- www
|   +-- A
|       +-- 203.0.113.10
|
+-- api
|   +-- A
|       +-- 203.0.113.10
|
+-- admin
|   +-- A
|       +-- 203.0.113.10
|
+-- _acme-challenge
    +-- TXT
        +-- "ABC123XYZ456"
```

The TXT record is not a web page.

It exists specifically for domain validation.

---

# 12. Step 5 — Verify the TXT Record

Before continuing, check the public DNS.

Using `dig`:

```bash
dig TXT _acme-challenge.example.com
```

Expected result:

```text
_acme-challenge.example.com. 300 IN TXT "ABC123XYZ456"
```

You can also use:

```bash
nslookup -type=TXT _acme-challenge.example.com
```

Expected:

```text
_acme-challenge.example.com
    text = "ABC123XYZ456"
```

If the record is not visible yet, wait for DNS propagation and check again.

---

# 13. Step 6 — CA Performs Validation

The CA performs a DNS lookup.

```text
Let's Encrypt
      |
      | DNS query
      v
_acme-challenge.example.com
      |
      v
Authoritative DNS
      |
      v
TXT = ABC123XYZ456
      |
      v
Expected value?
      |
      +---- YES ----> Validation PASS
```

The CA does not need to access your Nginx server for DNS-01 validation.

This is one of the major differences between DNS-01 and HTTP-01.

---

# 14. Step 7 — Certificate Issued

After successful validation:

```text
DNS Validation
      |
      v
Domain Control Confirmed
      |
      v
ACME Order Finalized
      |
      v
CA Signs Certificate
      |
      v
Certificate Returned
```

Certbot saves the resulting certificate and key in its certificate storage.

Typical Certbot paths include:

```text
/etc/letsencrypt/live/example.com/
```

You may see:

```text
cert.pem
chain.pem
fullchain.pem
privkey.pem
```

---

# 15. Step 8 — Configure Nginx

For example:

```nginx
server {
    listen 443 ssl;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
    }
}
```

Test:

```bash
sudo nginx -t
```

If successful:

```bash
sudo systemctl reload nginx
```

---

# 16. Complete Real Flow

```text
1. User runs Certbot
          |
          v
2. Certbot contacts ACME CA
          |
          v
3. CA creates DNS-01 challenge
          |
          v
4. Certbot displays TXT record
          |
          v
5. User logs in to DNS provider
          |
          v
6. User creates TXT record
          |
          v
7. DNS record becomes publicly visible
          |
          v
8. CA queries DNS
          |
          v
9. TXT value matches
          |
          v
10. Domain validated
          |
          v
11. Certificate issued
          |
          v
12. Certbot stores certificate
          |
          v
13. Nginx uses certificate
          |
          v
14. HTTPS works
```

---

# 17. Wildcard Certificate Example

Request:

```bash
sudo certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d "*.example.com"
```

DNS challenge:

```text
_acme-challenge.example.com
```

TXT:

```text
"CHALLENGE-VALUE"
```

After validation:

```text
Certificate:
*.example.com
```

It can cover:

```text
api.example.com
admin.example.com
dev.example.com
staging.example.com
```

But it does not automatically cover deeper names such as:

```text
server.api.example.com
```

unless the certificate explicitly contains an appropriate name/wildcard.

---

# 18. Base Domain + Wildcard

A common production request is:

```text
example.com
*.example.com
```

This handles:

```text
example.com
api.example.com
admin.example.com
dev.example.com
```

Example command:

```bash
sudo certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d "example.com" \
  -d "*.example.com"
```

Depending on the order, Certbot/CA may present validation requirements for the requested identifiers.

---

# 19. Multiple TXT Records

When multiple DNS-01 authorizations use the same `_acme-challenge` name, you may need multiple TXT values.

Example:

```text
_acme-challenge.example.com
    TXT "VALUE-ONE"
    TXT "VALUE-TWO"
```

Do not replace one required challenge value with another if Certbot asks you to keep both.

Always follow the exact instructions from the current ACME client.

---

# 20. Why DNS Propagation Matters

You create:

```text
TXT
_acme-challenge.example.com
```

But the CA may query DNS from a different network/location.

The record must be visible through the public authoritative DNS infrastructure.

Flow:

```text
Your Computer
     |
     v
DNS Provider
     |
     v
Authoritative DNS
     |
     +---- Public DNS resolvers
                |
                v
              CA
```

If the CA cannot see the TXT record, validation can fail.

---

# 21. Manual DNS and Nginx

One major advantage is that Nginx does not need to serve the ACME challenge.

With HTTP-01:

```text
CA
 |
 | HTTP :80
 v
Nginx
 |
 v
.well-known/acme-challenge/
```

With DNS-01:

```text
CA
 |
 | DNS query
 v
DNS
 |
 v
_acme-challenge TXT
```

Therefore:

```text
DNS-01
   |
   +---- Nginx can be private
   |
   +---- Port 80 does not need to serve the challenge
   |
   +---- Wildcard certificates possible
```

---

# 22. Manual DNS Renewal

This is the major disadvantage.

Suppose the certificate needs renewal.

```text
Certificate
     |
     v
Renewal required
     |
     v
Certbot
     |
     v
New DNS challenge
     |
     v
User manually creates TXT
     |
     v
CA validates
     |
     v
New certificate
```

So manual DNS is not fully hands-off.

Certbot's documentation notes that the manual plugin requires the validation steps to be performed by the user unless automation hooks are supplied. [2]

---

# 23. Automated DNS Renewal

With a DNS API plugin:

```text
Certificate
     |
     v
Renewal required
     |
     v
Certbot
     |
     v
DNS API
     |
     v
Create TXT automatically
     |
     v
CA validates
     |
     v
New certificate
     |
     v
Deploy
     |
     v
Reload Nginx
```

This is preferred for production where possible.

Certbot provides DNS plugins for several DNS providers, allowing DNS challenges to be automated. [3]

---

# 24. Manual DNS vs DNS API

| Feature | Manual DNS | DNS API |
|---|---|---|
| ACME validation | Yes | Yes |
| DNS-01 | Yes | Yes |
| TXT creation | Manual | Automatic |
| Renewal | Manual interaction normally required | Can be automatic |
| Wildcard | Yes | Yes |
| DNS credentials on server | No | Usually yes |
| Operational effort | High | Low |
| Production suitability | Limited | Better |

---

# 25. Security Difference

Manual DNS has an important security advantage:

```text
ACME Server
     |
     v
ACME Client
     |
     X
No DNS API credential
```

The server does not necessarily need permission to modify your DNS.

Automated DNS requires credentials or another authorized mechanism:

```text
ACME Client
     |
     | DNS API credential
     v
DNS Provider
```

Therefore DNS API credentials should have the smallest permissions possible.

---

# 26. Production Architecture

A production DNS-01 architecture could look like:

```text
                     Internet
                        |
                        v
                   DNS Provider
                        |
          +-------------+-------------+
          |                           |
          v                           v
     DNS A/AAAA                    DNS TXT
          |                    _acme-challenge
          v                           |
      Load Balancer                   |
          |                           |
      +---+---+                       |
      |       |                       |
      v       v                       |
   Nginx-1 Nginx-2                    |
      |       |                       |
      +---+---+                       |
          |                           |
          v                           |
       Backend                        |
                                      |
                                      |
                              +-------+-------+
                              |   ACME CA     |
                              | Let's Encrypt |
                              +---------------+
```

The CA validates through DNS rather than connecting directly to Nginx.

---

# 27. ACME Client Architecture

```text
                    ACME Client
                    /          \
                   /            \
                  v              v
            ACME Protocol      DNS
                  |              |
                  v              v
                CA          DNS Provider
                  |              |
                  |              v
                  |       TXT Record
                  |              |
                  +-------> Validation
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

# 28. Manual DNS with acme.sh

The same DNS-01 concept can be implemented with other ACME clients.

For example, acme.sh can issue certificates using DNS validation.

The conceptual process is:

```text
acme.sh
   |
   v
ACME CA
   |
   v
DNS Challenge
   |
   v
Manual TXT Record
   |
   v
DNS Provider
   |
   v
CA Validation
   |
   v
Certificate
```

The protocol is the same; the client command and workflow differ.

---

# 29. Manual DNS with win-acme

On Windows, win-acme can also use DNS-based validation depending on the selected validation plugin/configuration.

Architecture:

```text
Windows Server
      |
      v
win-acme
      |
      v
ACME CA
      |
      v
DNS Challenge
      |
      v
Manual TXT
      |
      v
DNS Provider
      |
      v
Validation
      |
      v
Certificate
      |
      v
Windows Certificate Store
      |
      v
IIS
```

---

# 30. Troubleshooting Manual DNS

## Problem 1 — TXT record not found

Check:

```bash
dig TXT _acme-challenge.example.com
```

If nothing is returned:

```text
DNS record missing
```

Check:

- DNS name
- Record type
- TXT value
- DNS provider
- Authoritative nameservers
- Propagation

---

## Problem 2 — Wrong TXT value

Expected:

```text
ABC123
```

DNS returns:

```text
XYZ789
```

Result:

```text
Validation FAILED
```

Use the exact value currently provided by the ACME client.

---

## Problem 3 — Wrong DNS provider

You may edit DNS at Provider A while the domain's authoritative nameservers are Provider B.

Check:

```bash
dig NS example.com
```

Example:

```text
example.com. NS ns1.provider.example
example.com. NS ns2.provider.example
```

The TXT record must be created in the DNS zone served by the authoritative nameservers.

---

## Problem 4 — DNS propagation delay

Check multiple times:

```bash
dig TXT _acme-challenge.example.com
```

Do not immediately assume the CA is wrong if your local resolver has not updated.

---

# 31. Useful Linux Commands

Check DNS:

```bash
dig example.com
```

Check nameservers:

```bash
dig NS example.com
```

Check TXT:

```bash
dig TXT _acme-challenge.example.com
```

Check with nslookup:

```bash
nslookup -type=TXT _acme-challenge.example.com
```

Check Nginx:

```bash
sudo nginx -t
```

Check Nginx service:

```bash
sudo systemctl status nginx
```

Check HTTPS port:

```bash
sudo ss -tlnp | grep :443
```

Check HTTP port:

```bash
sudo ss -tlnp | grep :80
```

---

# 32. Important Difference: ACME Automation vs DNS Automation

This is the key concept.

You can have:

```text
ACME automated
+
DNS manual
```

Example:

```text
Certbot
  |
  v
ACME automation
  |
  v
Challenge generated
  |
  v
YOU create TXT
  |
  v
CA validates
  |
  v
Certificate
```

Or:

```text
ACME automated
+
DNS automated
```

Example:

```text
Certbot
  |
  v
ACME automation
  |
  v
DNS API
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

The second model is much better for unattended production renewal.

---

# 33. ACMEbot Concept

The term "ACMEbot" can refer to an automation tool/client built around ACME. It is not a different protocol from ACME.

Think:

```text
ACME
 |
 +---- Certbot
 |
 +---- acme.sh
 |
 +---- win-acme
 |
 +---- Other ACME clients/tools
```

The important distinction is:

```text
ACME
    = Protocol

ACME Client / ACMEbot
    = Software implementing the protocol
```

---

# 34. Manual DNS Architecture — Short Version

```text
                 ACME Client
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
             YOU create TXT record
                      |
                      v
                DNS Provider
                      |
                      v
                 Public DNS
                      |
                      v
                CA validates
                      |
                      v
             Certificate issued
                      |
                      v
                  Nginx
                      |
                      v
                 HTTPS :443
```

---

# 35. Manual DNS vs HTTP-01

| Point | HTTP-01 | Manual DNS-01 |
|---|---|---|
| Validation | HTTP file | DNS TXT |
| Port | 80 | DNS |
| Requires public web server | Yes | No |
| Wildcard | No | Yes |
| Nginx involved in validation | Usually | No |
| DNS modification | No | Yes |
| Manual work | Low with normal client | Higher |
| Good for wildcard | No | Yes |

Let's Encrypt documents that HTTP-01 cannot issue wildcard certificates, while DNS-01 can. [1]

---

# 36. Production Recommendation

For learning:

```text
Manual DNS-01
      ↓
Understand TXT
      ↓
Understand DNS
      ↓
Understand ACME validation
```

For production:

```text
DNS-01
  +
DNS API
  +
Automatic renewal
  +
Monitoring
```

Recommended architecture:

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
             TXT challenge
                    |
                    v
                Validation
                    |
                    v
              Certificate
                    |
                    v
              Certificate
                Deployment
                    |
                    v
                 Nginx
                    |
                    v
                HTTPS
                    |
                    v
               Monitoring
```

---

# 37. Best Practices

1. Use DNS-01 for wildcard certificates.
2. Use manual DNS mainly for learning, testing, or special cases.
3. Prefer DNS API automation for production renewals.
4. Protect DNS API credentials.
5. Use least-privilege DNS API permissions.
6. Monitor certificate expiration.
7. Test certificate renewal before production deployment.
8. Verify TXT records with `dig`.
9. Verify authoritative nameservers with `dig NS`.
10. Reload Nginx after certificate renewal.
11. Keep private keys protected.
12. Remove temporary challenge TXT records when appropriate and when the ACME client instructs you to do so.

---

# 38. Interview Questions

## Q1. What is DNS-01?

DNS-01 is an ACME challenge that proves domain control using a DNS TXT record.

## Q2. Where is the TXT record created?

At:

```text
_acme-challenge.example.com
```

## Q3. Can DNS-01 issue wildcard certificates?

Yes.

```text
*.example.com
```

## Q4. Does DNS-01 require port 80?

No. It validates through DNS rather than HTTP.

## Q5. What is the disadvantage of manual DNS?

The administrator must manually create/update DNS challenge records, which makes unattended renewal difficult.

## Q6. What is automated DNS?

The ACME client uses a DNS provider API to create and remove challenge records automatically.

## Q7. Which is better for production?

Generally:

```text
DNS-01 + DNS API + automatic renewal
```

when DNS-01 is required.

## Q8. Is ACMEbot a protocol?

No.

```text
ACME = Protocol
ACMEbot / ACME client = Software
```

---

# 39. Final Memory Diagram

```text
                         ACME
                       Protocol
                          |
                          v
                    ACME Client
                   /            \
                  /              \
                 v                v
            ACME CA           DNS Provider
                 |                 |
                 | Challenge       |
                 v                 |
       _acme-challenge.example.com |
                 |                 |
                 +------TXT--------+
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
                          |
                          v
                      Renewal
```

## Final Takeaway

The most important concept is:

```text
ACME
  |
  | protocol
  v
ACME Client
  |
  +---- Manual DNS-01
  |          |
  |          +---- YOU create TXT
  |
  +---- Automated DNS-01
             |
             +---- DNS API creates TXT
```

**Manual DNS-01** gives you a very good understanding of how ACME domain validation works. **Automated DNS-01** is the production-oriented approach because the DNS challenge can be created and cleaned up automatically during certificate issuance and renewal.

## References

[1] Let's Encrypt, Challenge Types:
https://letsencrypt.org/docs/challenge-types/

[2] Certbot Documentation, Manual Plugin:
https://eff-certbot.readthedocs.io/en/stable/using.html

[3] Certbot Documentation, DNS Plugins:
https://eff-certbot.readthedocs.io/en/stable/using.html
