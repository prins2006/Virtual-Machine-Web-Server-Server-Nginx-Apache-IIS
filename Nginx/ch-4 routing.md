# Nginx Routing — Complete Guide

## 1. What is Routing?

Routing is the process of deciding **where an incoming request should go**.

In a web server such as Nginx, routing determines:

* Which website should handle the request
* Which backend application should receive the request
* Which static files should be served
* Which API service should receive traffic
* Whether the request should be redirected
* Which backend server should receive the request

Example:

```text
Client
   |
   v
Nginx
   |
   +---- /              ---> Frontend
   |
   +---- /api/          ---> Backend API
   |
   +---- /admin/        ---> Admin Application
   |
   +---- /static/       ---> Static Files
```

---

# 2. Why Routing is Required

Suppose one server hosts:

```text
Frontend     → Port 3000
Backend API  → Port 5000
Admin Panel  → Port 4000
```

Users should not need to remember different ports.

Instead:

```text
https://example.com/
       ↓
Frontend

https://example.com/api/
       ↓
Backend

https://example.com/admin/
       ↓
Admin Panel
```

Nginx acts as the routing layer.

---

# 3. Basic Routing Architecture

```text
                    Internet
                       |
                       v
                    Nginx
                       |
          +------------+------------+
          |            |            |
          v            v            v
      Frontend       API          Admin
      :3000          :5000         :4000
```

Nginx receives the request and forwards it according to the routing rules.

---

# 4. Types of Nginx Routing

Important routing types include:

```text
1. Path-based routing
2. Domain-based routing
3. Subdomain routing
4. Reverse proxy routing
5. Static file routing
6. API routing
7. Admin routing
8. Redirect routing
9. Load balancing
10. WebSocket routing
11. Regex-based routing
12. Default server routing
```

---

# 5. Nginx Routing Directives

Important Nginx directives:

```nginx
server
location
proxy_pass
root
try_files
return
rewrite
upstream
```

---

# 6. Server Block

A `server` block defines how Nginx handles a particular virtual host.

Example:

```nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/html;
}
```

Meaning:

```text
Port:
80

Domain:
example.com

Website:
 /var/www/html
```

---

# 7. Location Block

The `location` block defines routing rules based on the request URI.

Example:

```nginx
location / {
    root /var/www/html;
}
```

For:

```text
http://example.com/
```

Nginx uses:

```text
location /
```

---

# 8. Path-Based Routing

Path-based routing routes traffic based on the URL path.

Example:

```text
/
    ↓
Frontend

/api/
    ↓
Backend

/admin/
    ↓
Admin
```

Nginx:

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:3000;
    }

    location /api/ {
        proxy_pass http://localhost:5000;
    }

    location /admin/ {
        proxy_pass http://localhost:4000;
    }
}
```

---

# 9. Frontend Routing

Suppose frontend application runs on:

```text
localhost:3000
```

Nginx:

```nginx
location / {
    proxy_pass http://localhost:3000;
}
```

Request:

```text
http://example.com/
```

Flow:

```text
Browser
   |
   v
Nginx :80
   |
   v
localhost:3000
   |
   v
Frontend
```

---

# 10. API Routing

Suppose backend API runs on:

```text
localhost:5000
```

Configuration:

```nginx
location /api/ {
    proxy_pass http://localhost:5000;
}
```

Request:

```text
GET /api/users
```

Flow:

```text
Client
   |
   v
Nginx
   |
   | /api/
   v
Backend :5000
   |
   v
/ users
```

---

# 11. Admin Routing

Suppose the admin application runs on:

```text
localhost:4000
```

Nginx:

```nginx
location /admin/ {
    proxy_pass http://localhost:4000;
}
```

Request:

```text
https://example.com/admin/
```

Flow:

```text
Client
   |
   v
Nginx
   |
   | /admin/
   v
Admin Application :4000
```

---

# 12. Domain-Based Routing

Instead of paths, Nginx can route based on domains.

Example:

```text
example.com
    ↓
Frontend

api.example.com
    ↓
Backend

admin.example.com
    ↓
Admin
```

Nginx:

```nginx
server {
    listen 80;
    server_name example.com;

    proxy_pass http://localhost:3000;
}

server {
    listen 80;
    server_name api.example.com;

    proxy_pass http://localhost:5000;
}

server {
    listen 80;
    server_name admin.example.com;

    proxy_pass http://localhost:4000;
}
```

Note:

`proxy_pass` normally belongs inside a `location` block, so the production form should be:

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:3000;
    }
}

server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:5000;
    }
}

server {
    listen 80;
    server_name admin.example.com;

    location / {
        proxy_pass http://localhost:4000;
    }
}
```

---

# 13. Path Routing vs Domain Routing

## Path-Based

```text
example.com/
example.com/api/
example.com/admin/
```

Advantages:

* One domain
* Simple DNS
* Good for unified applications

## Domain-Based

```text
example.com
api.example.com
admin.example.com
```

Advantages:

* Separate services
* Independent configuration
* Easier service separation
* Clear service boundaries

---

# 14. Subdomain Routing

Example:

```text
www.example.com
api.example.com
admin.example.com
```

Nginx:

```nginx
server {
    listen 80;
    server_name www.example.com;

    location / {
        proxy_pass http://localhost:3000;
    }
}

server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:5000;
    }
}
```

---

# 15. Static File Routing

Nginx can serve files directly without a backend.

Example:

```nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Directory:

```text
/var/www/html/
├── index.html
├── css/
├── js/
└── images/
```

Request:

```text
/images/logo.png
```

Nginx looks for:

```text
/var/www/html/images/logo.png
```

---

# 16. `root` Directive

Example:

```nginx
root /var/www/html;
```

Request:

```text
/css/style.css
```

Nginx maps it to:

```text
/var/www/html/css/style.css
```

---

# 17. `try_files`

`try_files` checks whether files/directories exist.

Example:

```nginx
location / {
    try_files $uri $uri/ =404;
}
```

Meaning:

```text
1. Check requested file
2. Check requested directory
3. If not found → 404
```

---

# 18. SPA Routing

Single Page Applications such as React, Angular, and Vue often need fallback routing.

Example:

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

Suppose the user requests:

```text
/dashboard
```

If `/dashboard` is not a physical file, Nginx returns:

```text
/index.html
```

The frontend router then handles `/dashboard`.

---

# 19. Reverse Proxy Routing

A reverse proxy receives client requests and forwards them to internal servers.

```text
Client
   |
   v
Nginx
   |
   +----> Backend 1
   |
   +----> Backend 2
   |
   +----> Backend 3
```

Example:

```nginx
location /api/ {
    proxy_pass http://localhost:5000;
}
```

---

# 20. Proxy Headers

When using Nginx as a reverse proxy, forward useful request information:

```nginx
location /api/ {
    proxy_pass http://localhost:5000;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

These headers allow the backend to understand the original request.

---

# 21. `X-Real-IP`

```nginx
proxy_set_header X-Real-IP $remote_addr;
```

Passes the client IP address to the backend.

---

# 22. `X-Forwarded-For`

```nginx
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

Maintains the chain of client/proxy IP addresses.

---

# 23. `X-Forwarded-Proto`

```nginx
proxy_set_header X-Forwarded-Proto $scheme;
```

Tells the backend whether the original request used:

```text
http
```

or:

```text
https
```

---

# 24. `proxy_pass` Trailing Slash

One of the most important Nginx routing concepts is the difference between:

```nginx
proxy_pass http://localhost:5000;
```

and:

```nginx
proxy_pass http://localhost:5000/;
```

The trailing slash can change how Nginx constructs the upstream URI.

---

# 25. Example Without Trailing Slash

```nginx
location /api/ {
    proxy_pass http://localhost:5000;
}
```

Request:

```text
/api/users
```

The upstream generally receives:

```text
/api/users
```

---

# 26. Example With Trailing Slash

```nginx
location /api/ {
    proxy_pass http://localhost:5000/;
}
```

Request:

```text
/api/users
```

The matching `/api/` prefix is replaced by `/`.

The upstream generally receives:

```text
/users
```

This behavior is extremely important when routing APIs.

---

# 27. API Prefix Removal

Suppose backend expects:

```text
/users
```

but users access:

```text
/api/users
```

Use:

```nginx
location /api/ {
    proxy_pass http://localhost:5000/;
}
```

Flow:

```text
Client:
 /api/users

Nginx:
 remove /api/

Backend:
 /users
```

---

# 28. API Prefix Preservation

Suppose backend expects:

```text
/api/users
```

Use:

```nginx
location /api/ {
    proxy_pass http://localhost:5000;
}
```

Flow:

```text
Client:
 /api/users

Backend:
 /api/users
```

---

# 29. HTTP to HTTPS Routing

Redirect HTTP to HTTPS:

```nginx
server {
    listen 80;
    server_name example.com;

    return 301 https://example.com$request_uri;
}
```

Flow:

```text
HTTP
  |
  v
301 Redirect
  |
  v
HTTPS
```

---

# 30. WWW to Non-WWW Redirect

Example:

```nginx
server {
    listen 80;
    server_name www.example.com;

    return 301 https://example.com$request_uri;
}
```

Flow:

```text
www.example.com
       |
       v
example.com
```

---

# 31. Non-WWW to WWW Redirect

```nginx
server {
    listen 80;
    server_name example.com;

    return 301 https://www.example.com$request_uri;
}
```

Choose one canonical domain and redirect the other to it.

---

# 32. URL Redirect

Example:

```nginx
location /old-page {
    return 301 /new-page;
}
```

Request:

```text
/old-page
```

Redirects to:

```text
/new-page
```

---

# 33. Temporary Redirect

HTTP status:

```text
302 Found
```

Nginx:

```nginx
return 302 /temporary-page;
```

Use temporary redirects when the redirect is not intended to be permanent.

---

# 34. Permanent Redirect

HTTP status:

```text
301 Moved Permanently
```

Nginx:

```nginx
return 301 /new-page;
```

Use for permanent URL changes.

---

# 35. Rewrite

Example:

```nginx
rewrite ^/old/(.*)$ /new/$1 permanent;
```

Request:

```text
/old/test
```

Becomes:

```text
/new/test
```

---

# 36. Exact Match Location

Use `=` for an exact URI.

```nginx
location = /health {
    return 200 "OK\n";
}
```

Request:

```text
/health
```

returns:

```text
OK
```

---

# 37. Prefix Match

Example:

```nginx
location /api/ {
    proxy_pass http://localhost:5000;
}
```

Matches:

```text
/api/
/api/users
/api/products
/api/orders
```

---

# 38. Regex Location

Example:

```nginx
location ~* \.(jpg|jpeg|png|gif|webp)$ {
    expires 30d;
}
```

Matches image extensions.

`~*` means case-insensitive regular expression matching.

---

# 39. Location Matching Priority

Nginx location matching can be summarized as:

```text
Exact match
    ↓
Longest prefix
    ↓
Regex locations
    ↓
Prefix fallback
```

Example:

```nginx
location = /login {
    ...
}

location / {
    ...
}

location ~* \.css$ {
    ...
}
```

The exact `/login` location is more specific than `/`.

---

# 40. Routing Example — Frontend + API + Admin

Complete example:

```nginx
server {
    listen 80;
    server_name example.com;

    # Frontend
    location / {
        proxy_pass http://localhost:3000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Backend API
    location /api/ {
        proxy_pass http://localhost:5000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Admin
    location /admin/ {
        proxy_pass http://localhost:4000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

# 41. Routing Architecture

```text
                         Client
                            |
                            v
                         Nginx
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
            /              /api/          /admin/
             |              |              |
             v              v              v
        Frontend         Backend          Admin
         :3000            :5000            :4000
```

---

# 42. Domain-Based Production Architecture

```text
                         Internet
                            |
                            v
                          Nginx
                            |
          +-----------------+-----------------+
          |                 |                 |
          v                 v                 v
   example.com       api.example.com   admin.example.com
          |                 |                 |
          v                 v                 v
      Frontend           Backend            Admin
       :3000              :5000              :4000
```

Configuration:

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:3000;
    }
}

server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:5000;
    }
}

server {
    listen 80;
    server_name admin.example.com;

    location / {
        proxy_pass http://localhost:4000;
    }
}
```

---

# 43. Load Balancing Routing

Nginx can route requests across multiple backend servers.

```nginx
upstream backend_servers {
    server 127.0.0.1:5000;
    server 127.0.0.1:5001;
    server 127.0.0.1:5002;
}

server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://backend_servers;
    }
}
```

Architecture:

```text
                 Nginx
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
    Backend 1   Backend 2   Backend 3
      :5000       :5001       :5002
```

---

# 44. Load Balancing Methods

Common methods:

```text
Round Robin
Least Connections
IP Hash
Hash
```

## Round Robin

Default behavior:

```text
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
```

## Least Connections

```nginx
upstream backend_servers {
    least_conn;

    server 127.0.0.1:5000;
    server 127.0.0.1:5001;
}
```

Sends traffic toward the server with fewer active connections.

---

# 45. IP Hash

```nginx
upstream backend_servers {
    ip_hash;

    server 127.0.0.1:5000;
    server 127.0.0.1:5001;
}
```

A client is generally mapped consistently to the same backend while the upstream set remains stable.

Useful for some stateful applications, though modern applications should preferably externalize session state.

---

# 46. Health Checks

Routing should avoid unhealthy backends.

Basic Nginx open-source upstream behavior can detect certain connection failures and temporarily avoid failed servers.

Example:

```nginx
upstream backend_servers {
    server 127.0.0.1:5000 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:5001 max_fails=3 fail_timeout=30s;
}
```

For advanced active health checks, additional Nginx products/modules or external load balancers may be used.

---

# 47. WebSocket Routing

WebSocket requires connection upgrade headers.

Example:

```nginx
location /socket/ {
    proxy_pass http://localhost:5000;

    proxy_http_version 1.1;

    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    proxy_set_header Host $host;
}
```

Architecture:

```text
Client
  |
  | WebSocket
  v
Nginx
  |
  | Upgrade
  v
WebSocket Backend
```

---

# 48. gRPC Routing

Nginx can proxy gRPC traffic.

Example:

```nginx
location / {
    grpc_pass grpc://localhost:50051;
}
```

For TLS-enabled upstream:

```nginx
location / {
    grpc_pass grpcs://localhost:50051;
}
```

---

# 49. Health Endpoint Routing

Create a simple health endpoint:

```nginx
location = /health {
    access_log off;
    return 200 "OK\n";
    add_header Content-Type text/plain;
}
```

Test:

```bash
curl http://example.com/health
```

Expected:

```text
OK
```

---

# 50. Block Specific Routes

Example:

```nginx
location /private/ {
    deny all;
}
```

Request:

```text
/private/file
```

returns:

```text
403 Forbidden
```

---

# 51. Allow Specific IPs

Example:

```nginx
location /admin/ {
    allow 192.168.1.100;
    deny all;
}
```

Only the allowed IP can access the route.

In production, carefully consider proxies/load balancers and trusted client-IP configuration before relying on source IP.

---

# 52. Route Static Files and API Separately

Example:

```nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/frontend;

    location /assets/ {
        try_files $uri =404;
    }

    location /api/ {
        proxy_pass http://localhost:5000;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

This is a common SPA + API architecture.

---

# 53. Nginx Routing with HTTPS

Production architecture:

```text
                  Internet
                     |
                     v
                HTTPS :443
                     |
                     v
                  Nginx
                TLS Termination
                     |
        +------------+------------+
        |            |            |
        v            v            v
    Frontend        API          Admin
     :3000         :5000         :4000
```

Example:

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    location / {
        proxy_pass http://localhost:3000;
    }

    location /api/ {
        proxy_pass http://localhost:5000;
    }

    location /admin/ {
        proxy_pass http://localhost:4000;
    }
}
```

---

# 54. Routing with Caching

Routing and caching can work together.

```nginx
proxy_cache_path /var/cache/nginx/api
                 keys_zone=api_cache:10m
                 max_size=1g
                 inactive=60m;

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:3000;
    }

    location /api/ {
        proxy_cache api_cache;
        proxy_cache_valid 200 60s;

        proxy_pass http://localhost:5000;

        add_header X-Cache-Status $upstream_cache_status;
    }
}
```

Architecture:

```text
Client
  |
  v
Nginx
  |
  +---- /       → Frontend
  |
  +---- /api/   → Cache → Backend
```

---

# 55. Routing with SSL + Caching

A production-style architecture can combine:

```text
DNS
 ↓
HTTPS
 ↓
Nginx
 ├── TLS
 ├── Routing
 ├── Caching
 ├── Rate Limiting
 └── Reverse Proxy
       ├── Frontend
       ├── API
       └── Admin
```

---

# 56. Common Routing Errors

## 502 Bad Gateway

Usually means Nginx cannot successfully communicate with the upstream.

Check:

```bash
sudo systemctl status backend-service
```

Check backend port:

```bash
sudo ss -tlnp | grep :5000
```

Test backend directly:

```bash
curl http://127.0.0.1:5000
```

Check Nginx logs:

```bash
sudo tail -f /var/log/nginx/error.log
```

---

# 57. 404 Not Found

Possible causes:

* Wrong `location`
* Wrong `root`
* Backend route does not exist
* Incorrect `proxy_pass`
* SPA fallback missing
* Incorrect trailing slash

Check:

```bash
sudo nginx -t
```

---

# 58. Connection Refused

Example:

```text
connect() failed (111: Connection refused)
```

Usually means the upstream application is not listening on the configured port.

Check:

```bash
sudo ss -tlnp
```

Then:

```bash
curl http://127.0.0.1:5000
```

---

# 59. Connection Timeout

Possible causes:

```text
Firewall
Network problem
Wrong IP
Backend unavailable
Security group
Routing problem
```

Test:

```bash
curl -v http://127.0.0.1:5000
```

---

# 60. Common Nginx Routing Mistakes

### Mistake 1 — Incorrect `proxy_pass`

Wrong:

```nginx
proxy_pass http;//localhost:5000;
```

Correct:

```nginx
proxy_pass http://localhost:5000;
```

---

### Mistake 2 — Wrong Backend Port

```nginx
proxy_pass http://localhost:5000;
```

But application is actually running on:

```text
localhost:3000
```

Result:

```text
502 Bad Gateway
```

---

### Mistake 3 — Incorrect Path Handling

```nginx
location /api/ {
    proxy_pass http://localhost:5000/;
}
```

vs:

```nginx
location /api/ {
    proxy_pass http://localhost:5000;
}
```

The trailing slash changes the URI sent upstream.

---

### Mistake 4 — Missing Headers

For reverse proxies:

```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

---

# 61. Nginx Routing Troubleshooting Commands

Check configuration:

```bash
sudo nginx -t
```

Check Nginx status:

```bash
sudo systemctl status nginx
```

Reload:

```bash
sudo systemctl reload nginx
```

Restart:

```bash
sudo systemctl restart nginx
```

Check listening ports:

```bash
sudo ss -tlnp
```

Check port:

```bash
sudo ss -tlnp | grep :80
```

Check HTTPS:

```bash
sudo ss -tlnp | grep :443
```

Check access log:

```bash
sudo tail -f /var/log/nginx/access.log
```

Check error log:

```bash
sudo tail -f /var/log/nginx/error.log
```

Test frontend:

```bash
curl -I http://localhost:3000
```

Test backend:

```bash
curl -I http://localhost:5000
```

Test Nginx:

```bash
curl -I http://example.com
```

---

# 62. Routing Debugging Flow

When routing does not work:

```text
1. Check Nginx configuration
        ↓
   nginx -t
        ↓
2. Check Nginx service
        ↓
   systemctl status nginx
        ↓
3. Check listening ports
        ↓
   ss -tlnp
        ↓
4. Test backend directly
        ↓
   curl localhost:PORT
        ↓
5. Check location block
        ↓
6. Check proxy_pass
        ↓
7. Check trailing slash
        ↓
8. Check DNS
        ↓
9. Check firewall
        ↓
10. Check Nginx logs
```

---

# 63. DNS + Routing Architecture

Example:

```text
example.com
     |
     v
DNS A Record
     |
     v
Server IP
     |
     v
Nginx
     |
     +---- /
     |      ↓
     |   Frontend
     |
     +---- /api/
     |      ↓
     |   Backend
     |
     +---- /admin/
            ↓
         Admin
```

DNS determines **which server IP** receives traffic.

Nginx determines **which application** receives the request.

---

# 64. Routing vs DNS

| DNS                              | Nginx Routing                  |
| -------------------------------- | ------------------------------ |
| Resolves hostname                | Routes HTTP requests           |
| Maps domain to IP                | Maps URI/domain to application |
| Layer outside HTTP               | HTTP reverse-proxy layer       |
| Example: A record                | Example: `/api/`               |
| Does not select backend URL path | Selects backend service        |

---

# 65. Routing vs Load Balancing

Routing:

```text
/api/
   ↓
Backend
```

Load balancing:

```text
/api/
   ↓
Nginx
 ┌─┴─┐
 ↓   ↓
API1 API2
```

Routing decides **where traffic should go**.

Load balancing decides **which backend instance should receive it**.

---

# 66. Production Routing Architecture

```text
                         Internet
                            |
                            v
                         DNS/CDN
                            |
                            v
                          WAF
                            |
                            v
                     Load Balancer
                            |
                            v
                         Nginx
                 ┌──────────┼──────────┐
                 │          │          │
                 v          v          v
             Frontend      API       Admin
                 │          │          │
                 │          v          │
                 │       Backend       │
                 │       Cluster       │
                 │          │          │
                 │          v          │
                 │       Redis         │
                 │          │          │
                 └──────────┼──────────┘
                            v
                         Database
```

---

# 67. Routing Best Practices

1. Keep routing rules simple.
2. Use meaningful URL structures.
3. Separate frontend, API, and admin routes.
4. Use HTTPS in production.
5. Use correct `proxy_pass` syntax.
6. Understand trailing slash behavior.
7. Forward required proxy headers.
8. Protect administrative routes.
9. Use health endpoints.
10. Monitor Nginx logs.
11. Test upstream services independently.
12. Use load balancing for multiple backend instances.
13. Avoid unnecessary rewrites.
14. Keep configuration modular.
15. Validate configuration with `nginx -t`.
16. Use graceful reloads.
17. Use caching where appropriate.
18. Use rate limiting for sensitive endpoints.
19. Do not expose internal backend ports unnecessarily.
20. Document the routing architecture.

---

# 68. Example Production Configuration

```nginx
# HTTP → HTTPS
server {
    listen 80;
    server_name example.com;

    return 301 https://example.com$request_uri;
}

# HTTPS
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    ssl_protocols TLSv1.2 TLSv1.3;

    # Frontend
    location / {
        proxy_pass http://127.0.0.1:3000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # API
    location /api/ {
        proxy_pass http://127.0.0.1:5000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Admin
    location /admin/ {
        proxy_pass http://127.0.0.1:4000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Health Check
    location = /health {
        access_log off;
        return 200 "OK\n";
        add_header Content-Type text/plain;
    }
}
```

---

# 69. Practical Routing Lab

## Step 1 — Create Applications

Example:

```text
Frontend → localhost:3000
Backend  → localhost:5000
Admin    → localhost:4000
```

---

## Step 2 — Verify Applications

Frontend:

```bash
curl http://localhost:3000
```

Backend:

```bash
curl http://localhost:5000
```

Admin:

```bash
curl http://localhost:4000
```

All applications should respond before configuring Nginx.

---

## Step 3 — Configure Nginx

Edit:

```bash
sudo vim /etc/nginx/sites-available/example.com
```

Add routing:

```nginx
location / {
    proxy_pass http://localhost:3000;
}

location /api/ {
    proxy_pass http://localhost:5000;
}

location /admin/ {
    proxy_pass http://localhost:4000;
}
```

---

## Step 4 — Enable Configuration

```bash
sudo ln -s \
    /etc/nginx/sites-available/example.com \
    /etc/nginx/sites-enabled/example.com
```

---

## Step 5 — Test Configuration

```bash
sudo nginx -t
```

Expected:

```text
syntax is ok
test is successful
```

---

## Step 6 — Reload Nginx

```bash
sudo systemctl reload nginx
```

---

## Step 7 — Test Routes

Frontend:

```bash
curl http://example.com/
```

API:

```bash
curl http://example.com/api/
```

Admin:

```bash
curl http://example.com/admin/
```

Health:

```bash
curl http://example.com/health
```

---

# 70. Routing Learning Roadmap

## Beginner

```text
1. What is routing?
2. Nginx server blocks
3. Location blocks
4. Root directive
5. try_files
6. Static file routing
7. Exact match
8. Prefix match
9. Regex match
```

## Intermediate

```text
10. Path-based routing
11. Domain-based routing
12. Subdomain routing
13. Reverse proxy
14. proxy_pass
15. Proxy headers
16. API routing
17. Admin routing
18. Frontend routing
19. SPA routing
20. HTTP → HTTPS redirect
21. WWW redirects
22. URL rewrites
```

## Advanced

```text
23. Load balancing
24. Upstream groups
25. Round Robin
26. Least Connections
27. IP Hash
28. Health checks
29. WebSocket routing
30. gRPC routing
31. TLS termination
32. Routing + caching
33. Routing + authentication
34. Routing security
35. Production architecture
36. Troubleshooting
```

---

# 71. Final Routing Summary

The main concept is:

```text
                    Client
                      |
                      v
                    Nginx
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
         /           /api/      /admin/
          |           |           |
          v           v           v
     Frontend      Backend       Admin
      :3000         :5000        :4000
```

Important Nginx routing concepts:

```text
server
location
root
try_files
proxy_pass
proxy_set_header
upstream
rewrite
return
proxy_cache
TLS
redirects
load balancing
WebSocket
gRPC
health checks
```

The most important practical routing topics for DevOps are:

```text
Path-Based Routing
Domain-Based Routing
Subdomain Routing
Reverse Proxy
Frontend Routing
API Routing
Admin Routing
proxy_pass
Trailing Slash
HTTP → HTTPS
Redirects
Load Balancing
WebSocket
Health Checks
Routing Security
Routing Troubleshooting
Production Architecture
```
