# SSL/TLS — Complete Guide

## 1. What is SSL/TLS?

**SSL (Secure Sockets Layer)** and **TLS (Transport Layer Security)** are cryptographic protocols used to secure communication between a client and server.

They provide:

* Encryption
* Authentication
* Data integrity
* Protection against eavesdropping
* Protection against tampering
* Secure HTTPS communication

Modern systems use **TLS**. SSL is the older protocol and should no longer be used.

```text
HTTP
  ↓
HTTPS
  ↓
HTTP + TLS Encryption
```

---

# 2. HTTP vs HTTPS

## HTTP

```text
Client
  ↓
HTTP
  ↓
Server
```

HTTP sends data without TLS encryption.

Example:

```text
Username: prins
Password: example123
```

An attacker monitoring the connection may be able to read the traffic.

## HTTPS

```text
Client
  ↓
TLS Encryption
  ↓
Server
```

The data is encrypted while traveling across the network.

```text
Client
   │
   │ Encrypted HTTPS
   ▼
Server
```

---

# 3. Why Do We Need TLS?

TLS provides three major security properties.

## 3.1 Confidentiality

Prevents unauthorized users from reading the communication.

```text
Client → Encrypted Data → Server
```

## 3.2 Integrity

Ensures data is not modified during transmission.

```text
Original Data
     ↓
TLS Protection
     ↓
Modified data detected
```

## 3.3 Authentication

Helps the client verify that it is communicating with the intended server.

Example:

```text
https://example.com
```

The server presents a certificate proving control of the domain.

---

# 4. SSL vs TLS

SSL versions:

```text
SSL 2.0
SSL 3.0
```

These are obsolete and insecure.

TLS versions:

```text
TLS 1.0
TLS 1.1
TLS 1.2
TLS 1.3
```

For modern production systems, **TLS 1.2 and TLS 1.3** are the relevant versions.

---

# 5. TLS Architecture

Basic architecture:

```text
                 HTTPS
                   │
       ┌───────────┴───────────┐
       │                       │
     Client                  Server
       │                       │
       └────── TLS ────────────┘
```

More detailed:

```text
Browser
   │
   │ ClientHello
   ▼
Web Server
   │
   │ ServerHello
   │ Certificate
   │ Key Exchange
   ▼
Encrypted Session
   │
   ▼
HTTPS Application Data
```

---

# 6. TLS Handshake

The TLS handshake establishes a secure connection.

Simplified flow:

```text
Client                          Server
  │                               │
  │──── ClientHello ─────────────>│
  │                               │
  │<──── ServerHello ─────────────│
  │<──── Certificate ─────────────│
  │<──── Key Exchange ────────────│
  │                               │
  │──── Key Exchange ────────────>│
  │                               │
  │════ Encrypted Communication ══│
```

---

# 7. TLS 1.3 Handshake

TLS 1.3 simplified flow:

```text
Client                          Server
  │                               │
  │──── ClientHello ─────────────>│
  │                               │
  │<──── ServerHello ─────────────│
  │<──── Certificate ─────────────│
  │<──── CertificateVerify ───────│
  │<──── Finished ────────────────│
  │                               │
  │════ Encrypted Application ════│
```

TLS 1.3 reduces handshake latency compared with older TLS designs.

---

# 8. ClientHello

The client starts the TLS handshake with a `ClientHello`.

It can contain information such as:

```text
TLS versions
Supported cipher suites
Supported groups
Key share
Random value
Extensions
Server Name Indication (SNI)
ALPN
```

Example:

```text
Client
  ↓
ClientHello
  ├── TLS 1.3
  ├── Supported cipher suites
  ├── Key share
  └── SNI: example.com
```

---

# 9. ServerHello

The server responds with:

```text
ServerHello
```

It selects compatible cryptographic parameters.

For example:

```text
TLS Version: TLS 1.3
Cipher Suite: TLS_AES_256_GCM_SHA384
```

---

# 10. Digital Certificate

A TLS certificate allows a client to authenticate a server's identity.

Example:

```text
Certificate
 ├── Domain Name
 ├── Public Key
 ├── Issuer
 ├── Valid From
 ├── Valid To
 ├── Serial Number
 └── Digital Signature
```

For example:

```text
Subject:
example.com

Issuer:
Certificate Authority

Public Key:
Server Public Key
```

---

# 11. Certificate Authority (CA)

A **Certificate Authority** is a trusted organization that issues certificates.

Examples include:

* Let's Encrypt
* DigiCert
* GlobalSign
* Sectigo

Trust architecture:

```text
Root CA
   │
   ▼
Intermediate CA
   │
   ▼
Server Certificate
   │
   ▼
example.com
```

---

# 12. Root CA

A Root CA is a trusted certificate authority whose certificate is trusted by operating systems and browsers.

Example:

```text
Root CA
   │
   ├── Intermediate CA
   │       │
   │       └── Website Certificate
   │
   └── Trust Store
```

Root certificates are generally stored in system/browser trust stores.

---

# 13. Intermediate CA

An intermediate CA sits between the root CA and the website certificate.

```text
Root CA
   ↓
Intermediate CA
   ↓
Website Certificate
```

This creates a certificate chain.

---

# 14. Certificate Chain

Example:

```text
Root CA
   ↓
Intermediate CA
   ↓
example.com Certificate
```

The browser verifies:

```text
Is the certificate valid?
        ↓
Is it signed by the intermediate CA?
        ↓
Is the intermediate CA trusted?
        ↓
Does the chain lead to a trusted root?
        ↓
YES
        ↓
Certificate trusted
```

---

# 15. Public Key and Private Key

TLS uses asymmetric cryptography for authentication and key establishment.

## Public Key

The public key can be shared.

```text
Public Key
   ↓
Certificate
   ↓
Client can receive it
```

## Private Key

The private key must remain secret.

```text
Private Key
   ↓
Server only
```

Never expose:

```text
server.key
private.key
```

---

# 16. Public Key vs Private Key

| Key         | Purpose                                                  | Should be secret? |
| ----------- | -------------------------------------------------------- | ----------------- |
| Public Key  | Shared with clients                                      | No                |
| Private Key | Proves server identity / performs private-key operations | Yes               |

---

# 17. Symmetric Encryption

After the TLS handshake, application traffic is protected using symmetric encryption.

Example:

```text
Client
   │
   │ Shared Session Key
   │
   ▼
Server
```

Both sides use the established symmetric keys to protect application data.

Symmetric encryption is efficient for large amounts of data.

---

# 18. Asymmetric vs Symmetric Encryption

| Feature      | Asymmetric                         | Symmetric                 |
| ------------ | ---------------------------------- | ------------------------- |
| Keys         | Public + Private                   | Shared secret keys        |
| Main TLS use | Authentication/key establishment   | Application data          |
| Performance  | Slower                             | Faster                    |
| Example      | RSA/ECDSA/ECDHE-related mechanisms | AES-GCM/ChaCha20-Poly1305 |

---

# 19. Session Keys

TLS establishes symmetric keys for the connection.

Example:

```text
Client
   │
   │ Key Exchange
   ▼
Server
   │
   ▼
Session Keys
   │
   ▼
Encrypted Communication
```

The actual HTTPS data is then protected using symmetric cryptography.

---

# 20. Key Exchange

Key exchange allows the client and server to establish shared secret material without directly sending the final session key across the network.

Modern TLS commonly uses ephemeral Diffie-Hellman mechanisms such as:

```text
ECDHE
```

The important property is that the parties derive shared secret material from their exchanged key shares.

---

# 21. Perfect Forward Secrecy

**Perfect Forward Secrecy (PFS)** means that compromise of a server's long-term private key should not allow an attacker to decrypt previously captured TLS sessions, assuming ephemeral key exchange was used correctly.

Modern TLS configurations commonly achieve this through ephemeral key exchange.

```text
Session 1 → Temporary Key A
Session 2 → Temporary Key B
Session 3 → Temporary Key C
```

Compromise of the long-term certificate key does not automatically reveal old session keys.

---

# 22. Cipher Suite

A cipher suite defines cryptographic algorithms used by TLS.

Example TLS 1.3 cipher suite:

```text
TLS_AES_256_GCM_SHA384
```

It indicates:

```text
AES-256-GCM
+
SHA-384
```

TLS 1.3 has a smaller and more modern cipher-suite design than TLS 1.2.

---

# 23. Common Cryptographic Algorithms

### AES

Symmetric encryption algorithm.

```text
AES-128
AES-256
```

### ChaCha20-Poly1305

Modern authenticated encryption option.

### SHA-256 / SHA-384

Hash functions used in TLS-related cryptographic operations.

### RSA

Asymmetric cryptographic algorithm.

### ECDSA

Elliptic Curve Digital Signature Algorithm.

### ECDHE

Elliptic Curve Diffie-Hellman Ephemeral key exchange.

---

# 24. TLS 1.2 vs TLS 1.3

| Feature           | TLS 1.2                  | TLS 1.3                                       |
| ----------------- | ------------------------ | --------------------------------------------- |
| Modern            | Yes                      | Yes                                           |
| Handshake         | More messages            | Reduced latency                               |
| Cipher suites     | Larger selection         | Smaller modern set                            |
| Forward secrecy   | Depends on configuration | Designed around modern ephemeral key exchange |
| 0-RTT             | No                       | Supported in appropriate scenarios            |
| Legacy algorithms | More possibilities       | Removed/restricted                            |

---

# 25. 0-RTT

TLS 1.3 supports **0-RTT** for certain resumed connections.

It can reduce latency.

```text
Normal resumed connection:

Client → Server
       handshake
Client → Server
       application data
```

With 0-RTT:

```text
Client → Server
       early application data
```

### Security consideration

0-RTT data can be vulnerable to replay.

Therefore, avoid using 0-RTT blindly for operations that must not be replayed.

---

# 26. SNI — Server Name Indication

SNI allows a client to indicate the hostname it wants during the TLS handshake.

Example:

```text
Client
   ↓
SNI: example.com
   ↓
Nginx
   ↓
Select certificate for example.com
```

This allows multiple HTTPS websites to share the same IP address.

Example:

```text
IP: 192.0.2.10

example.com
api.example.com
admin.example.com
```

Nginx can select the appropriate server block/certificate based on the hostname.

---

# 27. SAN — Subject Alternative Name

A certificate can contain multiple DNS names using the SAN extension.

Example:

```text
DNS: example.com
DNS: www.example.com
DNS: api.example.com
```

The certificate can therefore cover multiple hostnames.

---

# 28. Wildcard Certificate

A wildcard certificate can cover multiple subdomains.

Example:

```text
*.example.com
```

It can cover:

```text
api.example.com
admin.example.com
www.example.com
```

It does not generally cover:

```text
example.com
```

unless the base domain is also included separately in the certificate SANs.

---

# 29. Single-Domain Certificate

Example:

```text
example.com
```

This certificate is intended for that hostname.

---

# 30. Multi-Domain Certificate

A certificate can contain multiple independent domain names.

Example:

```text
example.com
example.net
example.org
```

This is often called a SAN or multi-domain certificate.

---

# 31. Self-Signed Certificate

A self-signed certificate is signed by itself rather than by a publicly trusted CA.

Useful for:

* Development
* Testing
* Internal environments
* Labs

Example:

```text
Server
   ↓
Self-Signed Certificate
   ↓
Browser
   ↓
Certificate warning
```

For public production websites, use a certificate trusted by the relevant clients.

---

# 32. Generate a Private Key with OpenSSL

Example:

```bash
openssl genrsa -out server.key 2048
```

This generates an RSA private key.

Check the key:

```bash
openssl rsa -in server.key -check
```

View key information:

```bash
openssl rsa -in server.key -text -noout
```

---

# 33. Generate an RSA Private Key

Modern OpenSSL can also use:

```bash
openssl genpkey -algorithm RSA \
    -out server.key \
    -pkeyopt rsa_keygen_bits:2048
```

---

# 34. Generate an ECDSA Private Key

Example:

```bash
openssl ecparam -name prime256v1 \
    -genkey \
    -noout \
    -out server.key
```

---

# 35. Generate a CSR

CSR means:

**Certificate Signing Request**

Create one:

```bash
openssl req -new \
    -key server.key \
    -out server.csr
```

The CSR contains information such as:

```text
Subject
Public Key
Requested certificate information
Signature
```

---

# 36. CSR Architecture

```text
Private Key
     │
     ▼
    CSR
     │
     ▼
Certificate Authority
     │
     ▼
Signed Certificate
```

The private key remains on the server.

---

# 37. Generate a Self-Signed Certificate

For lab purposes:

```bash
openssl req -x509 \
    -nodes \
    -days 365 \
    -newkey rsa:2048 \
    -keyout server.key \
    -out server.crt
```

This creates:

```text
server.key
server.crt
```

---

# 38. Understand the OpenSSL Command

```bash
openssl req
```

Create/process certificate requests.

```bash
-x509
```

Create a self-signed X.509 certificate.

```bash
-nodes
```

Do not encrypt the generated private key with a passphrase.

```bash
-days 365
```

Certificate validity period.

```bash
-newkey rsa:2048
```

Generate a new 2048-bit RSA key.

```bash
-keyout server.key
```

Private key output.

```bash
-out server.crt
```

Certificate output.

---

# 39. Nginx SSL/TLS Configuration

Example:

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    root /var/www/html;
    index index.html;
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

# 40. HTTPS Port

HTTPS normally uses:

```text
443/tcp
```

HTTP normally uses:

```text
80/tcp
```

Architecture:

```text
HTTP
Port 80
   ↓
Redirect
   ↓
HTTPS
Port 443
```

---

# 41. HTTP to HTTPS Redirect

Nginx:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    return 301 https://example.com$request_uri;
}
```

HTTPS server:

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
}
```

Test:

```bash
curl -I http://example.com
```

Expected:

```text
HTTP/1.1 301 Moved Permanently
Location: https://example.com/...
```

---

# 42. Let's Encrypt

Let's Encrypt provides publicly trusted TLS certificates through automated issuance.

A common Linux tool is:

```text
Certbot
```

Example:

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

Then:

```bash
sudo certbot --nginx -d example.com
```

For multiple names:

```bash
sudo certbot --nginx \
    -d example.com \
    -d www.example.com
```

---

# 43. Certbot Workflow

```text
Domain
   ↓
DNS
   ↓
Server
   ↓
Certbot
   ↓
ACME Challenge
   ↓
Let's Encrypt
   ↓
Certificate
   ↓
Nginx
   ↓
HTTPS
```

---

# 44. ACME Challenge

Certbot uses the ACME protocol to prove domain control.

Common challenge types include:

```text
HTTP-01
DNS-01
TLS-ALPN-01
```

## HTTP-01

The CA checks a token over HTTP.

```text
http://example.com/.well-known/acme-challenge/...
```

Port 80 generally needs to be reachable.

## DNS-01

A DNS TXT record proves domain control.

Useful for:

* Wildcard certificates
* Servers that cannot expose HTTP

---

# 45. DNS and TLS

DNS resolves:

```text
example.com
      ↓
IP address
```

TLS then establishes secure communication with that hostname.

Architecture:

```text
Browser
   ↓
DNS
   ↓
IP Address
   ↓
Nginx
   ↓
TLS
   ↓
HTTPS
```

DNS itself does not provide HTTPS encryption.

---

# 46. Certificate Validation

A browser checks several things.

### 1. Certificate Chain

Does it lead to a trusted CA?

### 2. Domain Name

Does the certificate cover the requested hostname?

### 3. Validity Period

Is the certificate currently valid?

### 4. Signature

Is the certificate cryptographically valid?

### 5. Key Usage / Extensions

Are the certificate extensions appropriate?

---

# 47. Certificate Expiration

Certificates have:

```text
Not Before
Not After
```

Example:

```text
Valid From:
2026-01-01

Valid Until:
2026-04-01
```

Expired certificates cause browser warnings/errors.

---

# 48. Check Certificate with OpenSSL

```bash
openssl x509 \
    -in server.crt \
    -text \
    -noout
```

Check expiration:

```bash
openssl x509 \
    -in server.crt \
    -noout \
    -dates
```

Example:

```text
notBefore=...
notAfter=...
```

---

# 49. Check Certificate Subject

```bash
openssl x509 \
    -in server.crt \
    -noout \
    -subject
```

Check issuer:

```bash
openssl x509 \
    -in server.crt \
    -noout \
    -issuer
```

---

# 50. Check Certificate SAN

```bash
openssl x509 \
    -in server.crt \
    -noout \
    -ext subjectAltName
```

Example:

```text
DNS:example.com
DNS:www.example.com
DNS:api.example.com
```

---

# 51. Test Remote HTTPS

```bash
openssl s_client -connect example.com:443
```

Specify hostname/SNI:

```bash
openssl s_client \
    -connect example.com:443 \
    -servername example.com
```

Show certificate:

```bash
openssl s_client \
    -connect example.com:443 \
    -servername example.com \
    -showcerts
```

---

# 52. Test HTTPS with curl

```bash
curl -I https://example.com
```

Verbose TLS information:

```bash
curl -v https://example.com
```

Ignore certificate verification for testing only:

```bash
curl -k https://example.com
```

> Do not use `-k` as a production solution.

---

# 53. Verify Nginx TLS Port

```bash
sudo ss -tlnp | grep :443
```

Expected:

```text
LISTEN 0 511 0.0.0.0:443
```

Check port 80:

```bash
sudo ss -tlnp | grep :80
```

---

# 54. TLS Configuration in Nginx

Example:

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    ssl_protocols TLSv1.2 TLSv1.3;

    root /var/www/html;
    index index.html;
}
```

---

# 55. Disable Old TLS Versions

Avoid obsolete protocols.

Example:

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
```

Do not enable obsolete SSL/TLS versions just for compatibility without a strong reason.

---

# 56. TLS Session Resumption

TLS session resumption allows a client that has previously connected to reconnect more efficiently.

TLS 1.3 uses mechanisms including session tickets.

Simplified:

```text
First connection
      ↓
Full TLS handshake
      ↓
Session information
      ↓
Later connection
      ↓
Faster resumed handshake
```

---

# 57. HSTS

HSTS means:

**HTTP Strict Transport Security**

It tells browsers to use HTTPS for the site.

Example:

```nginx
add_header Strict-Transport-Security \
    "max-age=31536000; includeSubDomains" always;
```

Meaning:

```text
max-age=31536000
```

Remember HTTPS for one year.

### Important

Do not enable `includeSubDomains` blindly if every subdomain is not HTTPS-ready.

---

# 58. HSTS Preload

Some sites can be included in browser-maintained HSTS preload lists.

A typical header for a site intentionally prepared for preload may include:

```text
max-age=31536000; includeSubDomains; preload
```

Do not add `preload` casually. It can make HTTP-only subdomains difficult to recover.

---

# 59. OCSP

OCSP means:

**Online Certificate Status Protocol**

It can be used to check whether a certificate has been revoked.

However, modern browser/CA ecosystems use multiple mechanisms for certificate revocation/status, including OCSP stapling.

---

# 60. OCSP Stapling

With OCSP stapling:

```text
Server
   ↓
Obtains certificate status information
   ↓
Staples it to TLS communication
   ↓
Client
```

This can reduce the need for the client to contact the CA directly.

Nginx configuration:

```nginx
ssl_stapling on;
ssl_stapling_verify on;
```

Proper DNS resolver/trust-chain configuration is also required.

---

# 61. TLS Certificate File Types

Common formats:

```text
.crt
.cer
.pem
.key
.csr
.p12
.pfx
```

### `.key`

Private key.

### `.crt` / `.cer`

Certificate.

### `.pem`

PEM-encoded certificate/key/container depending on contents.

### `.csr`

Certificate Signing Request.

### `.p12` / `.pfx`

PKCS#12 archive, often containing certificate + private key + chain.

---

# 62. PEM Format

A PEM certificate looks like:

```text
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
```

Private key:

```text
-----BEGIN PRIVATE KEY-----
...
-----END PRIVATE KEY-----
```

Older RSA PEM keys may look like:

```text
-----BEGIN RSA PRIVATE KEY-----
...
-----END RSA PRIVATE KEY-----
```

---

# 63. Certificate vs Private Key

Certificate:

```text
server.crt
```

Contains public information.

Private key:

```text
server.key
```

Must remain secret.

Never commit:

```text
*.key
```

to a public Git repository.

Add sensitive files to `.gitignore`.

Example:

```gitignore
*.key
*.pem
*.p12
*.pfx
```

---

# 64. Nginx SSL Directory

Example:

```text
/etc/nginx/ssl/
├── server.crt
├── server.key
└── chain.crt
```

Permissions should protect private keys.

Example:

```bash
sudo chmod 600 /etc/nginx/ssl/server.key
```

Ownership and permissions should follow the minimum required access model.

---

# 65. Full Nginx HTTPS Example

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    return 301 https://example.com$request_uri;
}

server {
    listen 443 ssl;
    server_name example.com www.example.com;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    ssl_protocols TLSv1.2 TLSv1.3;

    add_header Strict-Transport-Security \
        "max-age=31536000" always;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
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

Test:

```bash
curl -I http://example.com
```

Then:

```bash
curl -I https://example.com
```

---

# 66. HTTPS Reverse Proxy Architecture

A common production architecture:

```text
                    Internet
                       │
                       ▼
                  HTTPS :443
                       │
                       ▼
                  ┌─────────┐
                  │  Nginx  │
                  │ TLS     │
                  │ Term.   │
                  └────┬────┘
                       │
                HTTP/HTTPS internal
                       │
                       ▼
                   Backend
                       │
                       ▼
                    Database
```

Nginx can terminate TLS and proxy requests to an internal backend.

Example:

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    location / {
        proxy_pass http://127.0.0.1:3000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

# 67. TLS Termination

TLS termination means the TLS connection is decrypted at a proxy/load balancer.

```text
Client
  │
  │ HTTPS
  ▼
Nginx
  │
  │ HTTP/internal HTTPS
  ▼
Backend
```

Benefits:

* Centralized certificate management
* Easier application configuration
* TLS handling at the edge

---

# 68. TLS Passthrough

With TLS passthrough, the proxy forwards the encrypted TLS connection to the backend.

```text
Client
   │
   │ HTTPS
   ▼
Load Balancer
   │
   │ Encrypted TLS
   ▼
Backend
```

The backend performs TLS termination.

---

# 69. TLS Termination vs Passthrough

| Feature                | TLS Termination          | TLS Passthrough         |
| ---------------------- | ------------------------ | ----------------------- |
| TLS decrypted at proxy | Yes                      | No                      |
| Certificate at proxy   | Yes                      | Usually no              |
| Backend handles TLS    | Optional                 | Yes                     |
| Central management     | Easier                   | More distributed        |
| End-to-end encryption  | Depends on internal link | Preserved through proxy |

---

# 70. Mutual TLS (mTLS)

Normal TLS:

```text
Client → Server
Server authenticates itself
```

mTLS:

```text
Client Certificate
        ↕
     Server
        ↕
Server Certificate
```

Both sides authenticate each other.

Useful for:

* Microservices
* Internal APIs
* Service-to-service authentication
* Zero-trust environments

---

# 71. mTLS Architecture

```text
Service A
   │
   │ Client Certificate
   ▼
TLS Connection
   ▲
   │ Server Certificate
   │
Service B
```

Both services must trust the appropriate CA.

---

# 72. TLS Troubleshooting

## Problem 1 — Certificate Expired

Check:

```bash
openssl x509 -in server.crt -noout -dates
```

---

## Problem 2 — Wrong Domain

Check SAN:

```bash
openssl x509 \
    -in server.crt \
    -noout \
    -ext subjectAltName
```

Make sure the requested hostname exists.

---

## Problem 3 — Nginx Configuration Error

Run:

```bash
sudo nginx -t
```

Then inspect:

```bash
sudo systemctl status nginx
```

---

## Problem 4 — Port 443 Not Listening

Run:

```bash
sudo ss -tlnp | grep :443
```

If nothing appears, inspect Nginx configuration and service status.

---

## Problem 5 — Firewall

Check:

```bash
sudo ufw status
```

If UFW is being used, allow HTTPS:

```bash
sudo ufw allow 443/tcp
```

HTTP for ACME HTTP-01:

```bash
sudo ufw allow 80/tcp
```

---

# 73. Certbot Troubleshooting

Check certificates:

```bash
sudo certbot certificates
```

Test renewal:

```bash
sudo certbot renew --dry-run
```

Check Certbot version:

```bash
certbot --version
```

Check systemd timers:

```bash
systemctl list-timers | grep certbot
```

---

# 74. DNS Troubleshooting

Check DNS:

```bash
dig example.com
```

or:

```bash
nslookup example.com
```

Check specific record:

```bash
dig A example.com
```

Check IPv6:

```bash
dig AAAA example.com
```

A common ACME problem is that the domain resolves to an unreachable or incorrect server.

---

# 75. Test TLS Versions

Using OpenSSL:

```bash
openssl s_client \
    -connect example.com:443 \
    -tls1_2
```

TLS 1.3:

```bash
openssl s_client \
    -connect example.com:443 \
    -tls1_3
```

This helps verify which protocol versions the server accepts.

---

# 76. Check TLS Certificate Chain

```bash
openssl s_client \
    -connect example.com:443 \
    -servername example.com \
    -showcerts
```

Look for:

```text
Certificate chain
```

and:

```text
Verify return code
```

A successful verification should not show a certificate validation error.

---

# 77. Common TLS Errors

### `certificate has expired`

Certificate validity period ended.

### `certificate verify failed`

The client could not validate the certificate chain.

### `hostname mismatch`

Certificate does not cover the requested hostname.

### `handshake failure`

Client and server could not agree on compatible TLS parameters.

### `wrong version number`

Often caused by connecting to an HTTP service as if it were HTTPS, or by an incorrect proxy/listener configuration.

### `connection refused`

Nothing is listening on the requested port or a firewall/network policy is rejecting the connection.

### `connection timed out`

Traffic may be blocked or the destination may be unreachable.

---

# 78. TLS Security Best Practices

## Protocols

Use:

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
```

Avoid obsolete SSL/TLS versions.

## Private Keys

Protect private keys:

```bash
sudo chmod 600 /etc/nginx/ssl/server.key
```

## Certificates

Monitor certificate expiration.

## HTTPS

Redirect HTTP to HTTPS.

## HSTS

Use HSTS when the entire site is HTTPS-ready.

## Strong Cryptography

Use modern TLS versions and approved cipher suites.

## Secure Cookies

For authenticated web applications:

```http
Set-Cookie: session=...; Secure; HttpOnly; SameSite=Lax
```

## Certificate Renewal

Automate renewal and test it regularly.

---

# 79. TLS and Secure Cookies

`Secure` means the cookie should only be sent over HTTPS.

```http
Set-Cookie: session=abc; Secure
```

`HttpOnly` prevents normal JavaScript access to the cookie.

```http
Set-Cookie: session=abc; HttpOnly
```

`SameSite` helps control cross-site cookie behavior.

```http
Set-Cookie: session=abc; Secure; HttpOnly; SameSite=Lax
```

---

# 80. TLS Monitoring

Monitor:

```text
Certificate expiration
TLS versions
Handshake failures
Certificate chain
HTTPS availability
Port 443
Renewal status
Cipher configuration
OCSP/stapling where used
```

Useful commands:

```bash
openssl s_client -connect example.com:443
```

```bash
curl -Iv https://example.com
```

```bash
sudo ss -tlnp | grep :443
```

---

# 81. Production TLS Architecture

```text
                         Internet
                             │
                             ▼
                       DNS / CDN / WAF
                             │
                             ▼
                      Load Balancer
                             │
                             ▼
                         Nginx
                     TLS Termination
                             │
                    ┌────────┴────────┐
                    │                 │
                 Frontend           Backend
                                      │
                                      ▼
                                   Database
```

Certificates can be managed at the CDN/load balancer/Nginx layer depending on the architecture.

---

# 82. Complete SSL/TLS Request Flow

```text
1. User enters:
   https://example.com

2. Browser performs DNS lookup.

3. Browser connects to:
   example.com:443

4. TLS handshake starts.

5. Client sends ClientHello.

6. Server sends ServerHello.

7. Server sends certificate.

8. Client validates certificate.

9. Key exchange occurs.

10. Both sides derive session keys.

11. TLS handshake completes.

12. HTTP request is sent inside TLS.

13. Nginx receives the HTTPS request.

14. Nginx serves the static file or proxies to backend.

15. Encrypted response returns to browser.
```

---

# 83. SSL/TLS Practical Lab

## Step 1 — Install OpenSSL

```bash
sudo apt update
sudo apt install openssl
```

Check:

```bash
openssl version
```

---

## Step 2 — Create SSL Directory

```bash
sudo mkdir -p /etc/nginx/ssl
```

---

## Step 3 — Generate Private Key

```bash
sudo openssl genrsa \
    -out /etc/nginx/ssl/server.key \
    2048
```

---

## Step 4 — Generate Self-Signed Certificate

```bash
sudo openssl req \
    -x509 \
    -nodes \
    -days 365 \
    -new \
    -key /etc/nginx/ssl/server.key \
    -out /etc/nginx/ssl/server.crt
```

---

## Step 5 — Configure Nginx

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    ssl_protocols TLSv1.2 TLSv1.3;

    root /var/www/html;
}
```

---

## Step 6 — Test Nginx

```bash
sudo nginx -t
```

---

## Step 7 — Reload Nginx

```bash
sudo systemctl reload nginx
```

---

## Step 8 — Check Port 443

```bash
sudo ss -tlnp | grep :443
```

---

## Step 9 — Test HTTPS

```bash
curl -k https://example.com
```

`-k` is acceptable for testing a self-signed certificate, but should not be used to bypass certificate verification in production.

---

# 84. Production Certificate Lab with Certbot

Install:

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

Request certificate:

```bash
sudo certbot --nginx -d example.com
```

Test renewal:

```bash
sudo certbot renew --dry-run
```

List certificates:

```bash
sudo certbot certificates
```

---

# 85. SSL/TLS File Structure

Example:

```text
/etc/nginx/
├── nginx.conf
├── sites-available/
│   └── example.com
├── sites-enabled/
│   └── example.com
└── ssl/
    ├── server.crt
    ├── server.key
    └── chain.crt
```

---

# 86. SSL/TLS vs HTTPS

These terms are related but not identical.

```text
TLS
 ↓
Security protocol

HTTP
 ↓
Application protocol

HTTPS
 ↓
HTTP over TLS
```

So:

```text
HTTPS = HTTP + TLS
```

---

# 87. Important SSL/TLS Terms

| Term          | Meaning                                   |
| ------------- | ----------------------------------------- |
| SSL           | Old security protocol                     |
| TLS           | Modern security protocol                  |
| HTTPS         | HTTP over TLS                             |
| CA            | Certificate Authority                     |
| CSR           | Certificate Signing Request               |
| Certificate   | Identity/public-key document              |
| Public Key    | Shareable cryptographic key               |
| Private Key   | Secret cryptographic key                  |
| SAN           | Subject Alternative Name                  |
| SNI           | Server Name Indication                    |
| Cipher Suite  | Cryptographic algorithm selection         |
| TLS Handshake | Process establishing secure connection    |
| Session Key   | Symmetric key material for connection     |
| PFS           | Perfect Forward Secrecy                   |
| mTLS          | Mutual TLS                                |
| HSTS          | HTTP Strict Transport Security            |
| OCSP          | Certificate status protocol               |
| ACME          | Automated certificate management protocol |

---

# 88. SSL/TLS Learning Roadmap

## Beginner

```text
1. HTTP vs HTTPS
2. SSL vs TLS
3. Why TLS is required
4. Encryption
5. Authentication
6. Integrity
7. Public Key
8. Private Key
9. Certificates
10. Certificate Authority
```

## Intermediate

```text
11. Certificate Chain
12. Root CA
13. Intermediate CA
14. CSR
15. OpenSSL
16. TLS Handshake
17. ClientHello
18. ServerHello
19. Cipher Suites
20. Symmetric Encryption
21. Asymmetric Encryption
22. Key Exchange
23. SNI
24. SAN
25. Wildcard Certificates
```

## Advanced

```text
26. TLS 1.2
27. TLS 1.3
28. Perfect Forward Secrecy
29. Session Resumption
30. 0-RTT
31. HSTS
32. OCSP
33. OCSP Stapling
34. mTLS
35. TLS Termination
36. TLS Passthrough
37. CDN TLS
38. Load Balancer TLS
39. Certificate Automation
40. Production TLS Security
```

---

# 89. Practical Nginx SSL/TLS Learning Sequence

For DevOps practice, follow this order:

```text
SSL/TLS Concepts
      ↓
HTTP vs HTTPS
      ↓
Certificate & CA
      ↓
Public/Private Keys
      ↓
OpenSSL
      ↓
Generate CSR
      ↓
Self-Signed Certificate
      ↓
Nginx HTTPS
      ↓
HTTP → HTTPS Redirect
      ↓
TLS 1.2 / TLS 1.3
      ↓
SNI / SAN
      ↓
Certbot
      ↓
Let's Encrypt
      ↓
Certificate Renewal
      ↓
HSTS
      ↓
OCSP Stapling
      ↓
TLS Termination
      ↓
mTLS
      ↓
TLS Troubleshooting
      ↓
Production Best Practices
```

---

# 90. Final Summary

The complete HTTPS security flow is:

```text
                       User
                         │
                         ▼
                       DNS
                         │
                         ▼
                    Server :443
                         │
                         ▼
                  TLS Handshake
                         │
          ┌──────────────┴──────────────┐
          │                             │
    Certificate                    Key Exchange
    Validation                          │
          │                             │
          └──────────────┬──────────────┘
                         ▼
                    Session Keys
                         │
                         ▼
                 Encrypted HTTPS
                         │
                         ▼
                      Nginx
                         │
                         ▼
                  Backend/API
                         │
                         ▼
                     Database
```

The most important topics for DevOps are:

```text
TLS Handshake
Certificates
CA and Certificate Chain
Public/Private Keys
Symmetric Encryption
Asymmetric Encryption
Key Exchange
TLS 1.2 / TLS 1.3
Cipher Suites
SNI
SAN
OpenSSL
CSR
Certbot
Let's Encrypt
HTTPS
HTTP → HTTPS Redirect
HSTS
OCSP Stapling
TLS Termination
mTLS
Certificate Renewal
TLS Troubleshooting
Production TLS Security
```
