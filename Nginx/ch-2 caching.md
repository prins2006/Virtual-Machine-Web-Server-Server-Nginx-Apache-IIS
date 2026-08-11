# Caching — Complete Guide

## 1. What is Caching?

Caching is the process of temporarily storing frequently requested data in a faster storage location so that future requests can be served more quickly.

### Simple Example

Without caching:

```text
User
  ↓
Nginx
  ↓
Backend Application
  ↓
Database
  ↓
Response
```

With caching:

```text
User
  ↓
Nginx
  ↓
Cache
  ├── HIT  → Return cached response
  │
  └── MISS → Backend → Database
```

### Real-Life Example

Suppose a website has:

```text
/logo.png
/style.css
/app.js
```

Thousands of users request these files.

Instead of loading them repeatedly from the server, the browser, CDN, or Nginx can cache them.

```text
First Request:
Browser → Server → logo.png
                 ↓
              Cache it

Next Request:
Browser → Cached logo.png
```

---

# 2. Why Do We Need Caching?

Caching improves:

* Performance
* Response time
* Scalability
* Server capacity
* Database performance
* Bandwidth usage
* User experience
* Infrastructure cost

### Example

Without caching:

```text
1000 users
    ↓
1000 backend requests
    ↓
1000 database queries
```

With caching:

```text
1000 users
    ↓
Cache
    ├── 900 HIT
    └── 100 MISS
             ↓
          Backend
```

The backend receives significantly fewer requests.

---

# 3. Main Types of Caching

Common caching layers:

```text
                    User
                      │
                      ▼
               Browser Cache
                      │
                      ▼
                CDN / Edge
                      │
                      ▼
              Nginx Proxy Cache
                      │
                      ▼
              Application Cache
                      │
                      ▼
               Database Cache
                      │
                      ▼
                  Database
```

Main types:

1. Browser caching
2. CDN caching
3. Reverse proxy caching
4. Application caching
5. Database caching

---

# 4. Browser Caching

The browser stores frequently requested resources locally.

Examples:

```text
HTML
CSS
JavaScript
Images
Fonts
Videos
```

Example:

```http
Cache-Control: public, max-age=604800
```

`604800` seconds = 7 days.

The browser can reuse the resource for 7 days.

---

# 5. HTTP Cache-Control

`Cache-Control` is one of the most important HTTP caching headers.

Example:

```http
Cache-Control: public, max-age=604800
```

Meaning:

```text
public
    ↓
Can be cached by shared caches.

max-age=604800
    ↓
Fresh for 7 days.
```

## Common Cache-Control Directives

### public

```http
Cache-Control: public
```

The response can be stored by shared caches.

### private

```http
Cache-Control: private
```

The response should only be stored in a private cache such as the user's browser.

### no-cache

```http
Cache-Control: no-cache
```

The response may be stored, but it must be revalidated before reuse.

> `no-cache` does not mean "do not store".

### no-store

```http
Cache-Control: no-store
```

The response should not be stored.

Useful for sensitive information.

### max-age

```http
Cache-Control: max-age=3600
```

The response is considered fresh for 3600 seconds.

### must-revalidate

```http
Cache-Control: must-revalidate
```

Once stale, the cache must validate the response before reuse.

---

# 6. Cache-Control Examples

## Static Image

```http
Cache-Control: public, max-age=2592000
```

30 days.

## CSS and JavaScript

```http
Cache-Control: public, max-age=604800
```

7 days.

## Dynamic API

```http
Cache-Control: no-store
```

Useful when the response contains sensitive or highly dynamic data.

## Private User Information

```http
Cache-Control: private, max-age=300
```

Private cache for 5 minutes.

---

# 7. Expires Header

`Expires` is an older HTTP caching mechanism.

Example:

```http
Expires: Wed, 19 Aug 2026 12:00:00 GMT
```

Modern applications generally prefer:

```http
Cache-Control
```

because it provides more flexible caching behavior.

---

# 8. ETag

An `ETag` identifies a particular version of a resource.

Example:

```http
ETag: "abc123"
```

The browser later sends:

```http
If-None-Match: "abc123"
```

If the resource has not changed:

```http
HTTP/1.1 304 Not Modified
```

The server does not need to send the entire file again.

### Example

```text
Browser
   ↓
If-None-Match: "abc123"
   ↓
Server
   ↓
Resource unchanged
   ↓
304 Not Modified
```

---

# 9. Last-Modified

Another cache validation mechanism is:

```http
Last-Modified: Tue, 11 Aug 2026 08:00:00 GMT
```

The browser sends:

```http
If-Modified-Since: Tue, 11 Aug 2026 08:00:00 GMT
```

If nothing changed:

```http
304 Not Modified
```

---

# 10. ETag vs Last-Modified

| Feature                     | ETag             | Last-Modified              |
| --------------------------- | ---------------- | -------------------------- |
| Identifies resource version | Yes              | Based on modification time |
| Precision                   | Generally better | Time-based                 |
| Request header              | If-None-Match    | If-Modified-Since          |
| Response                    | 304              | 304                        |

---

# 11. Static Asset Caching

Static assets include:

```text
.css
.js
.png
.jpg
.jpeg
.gif
.svg
.webp
.woff
.woff2
```

## Nginx CSS/JS Cache

```nginx
location ~* \.(css|js)$ {
    expires 7d;
    add_header Cache-Control "public";
}
```

## Nginx Image Cache

```nginx
location ~* \.(jpg|jpeg|png|gif|webp|svg)$ {
    expires 30d;
    add_header Cache-Control "public";
}
```

## Nginx Font Cache

```nginx
location ~* \.(woff|woff2|ttf|otf)$ {
    expires 30d;
    add_header Cache-Control "public";
}
```

---

# 12. HTML Caching

HTML pages are often dynamic.

For frequently changing HTML:

```nginx
location ~* \.html$ {
    add_header Cache-Control "no-cache";
}
```

For highly sensitive or dynamic pages:

```http
Cache-Control: no-store
```

Examples:

```text
/login
/account
/dashboard
/payment
```

---

# 13. API Caching

APIs require careful caching.

Example:

```text
GET /api/products
```

If products change infrequently:

```http
Cache-Control: public, max-age=60
```

The response can be cached for 60 seconds.

Avoid blindly caching state-changing requests:

```text
POST
PUT
PATCH
DELETE
```

---

# 14. Reverse Proxy Caching

Reverse proxy caching is one of the most important Nginx caching concepts.

Architecture:

```text
Client
   ↓
Nginx
   ↓
Cache
   ├── HIT → Response
   │
   └── MISS
         ↓
      Backend
         ↓
      Database
```

---

# 15. Nginx Reverse Proxy Cache Configuration

```nginx
proxy_cache_path /var/cache/nginx
                 levels=1:2
                 keys_zone=my_cache:10m
                 max_size=1g
                 inactive=60m
                 use_temp_path=off;

server {
    listen 80;

    location /api/ {
        proxy_cache my_cache;

        proxy_pass http://localhost:3000;

        proxy_cache_valid 200 60s;
    }
}
```

---

# 16. Understanding proxy_cache_path

```nginx
proxy_cache_path /var/cache/nginx
```

Defines the cache storage directory.

```nginx
levels=1:2
```

Defines the directory structure used for cache files.

```nginx
keys_zone=my_cache:10m
```

Creates a shared memory zone for cache metadata.

```nginx
max_size=1g
```

Maximum cache size is approximately 1 GB.

```nginx
inactive=60m
```

Cache entries that have not been accessed for 60 minutes can be removed.

```nginx
use_temp_path=off
```

Stores temporary files directly in the cache directory.

---

# 17. Cache HIT and MISS

A cache **HIT** means the cached response was found and served.

A cache **MISS** means the response was not found in cache and the request went to the backend.

Add:

```nginx
add_header X-Cache-Status $upstream_cache_status;
```

Response:

```http
X-Cache-Status: HIT
```

or:

```http
X-Cache-Status: MISS
```

Possible statuses include:

```text
HIT
MISS
BYPASS
EXPIRED
STALE
UPDATING
REVALIDATED
```

---

# 18. Cache TTL

TTL means **Time To Live**.

Example:

```nginx
proxy_cache_valid 200 60s;
```

The cached response is valid for 60 seconds.

Example:

```text
10:00 → Backend request
10:00 → Cache created

10:30 → Cache HIT

10:59 → Cache HIT

11:01 → Cache expired
       ↓
     Backend
```

---

# 19. Different TTLs for Different Status Codes

```nginx
proxy_cache_valid 200 10m;
proxy_cache_valid 301 1h;
proxy_cache_valid 404 30s;
```

Meaning:

```text
200 → 10 minutes
301 → 1 hour
404 → 30 seconds
```

Avoid caching errors for long periods.

---

# 20. Cache Key

The cache key determines which cached response belongs to a request.

Example:

```nginx
proxy_cache_key "$scheme$request_method$host$request_uri";
```

Example cache key:

```text
httpGETexample.com/api/products?page=1
```

Different URLs can therefore have different cache entries.

---

# 21. Query String Caching

Consider:

```text
/api/products?page=1
/api/products?page=2
```

These should normally be separate cache entries.

```text
Cache Entry 1:
page=1

Cache Entry 2:
page=2
```

Incorrect cache-key design can return the wrong data.

---

# 22. Cache Bypass

Some requests should bypass cache.

Examples:

```text
/api/login
/api/logout
/api/profile
/api/account
```

Example:

```nginx
location /api/ {
    proxy_cache my_cache;

    proxy_cache_bypass $http_authorization;
}
```

If an Authorization header exists, the cache can be bypassed.

---

# 23. Cache No-Cache Rules

Nginx can prevent responses from being stored:

```nginx
proxy_no_cache $http_authorization;
```

Example:

```nginx
location /api/ {
    proxy_pass http://localhost:3000;

    proxy_cache my_cache;

    proxy_cache_bypass $http_authorization;
    proxy_no_cache $http_authorization;
}
```

This helps prevent authenticated responses from being shared incorrectly.

---

# 24. Cache GET and HEAD Requests

Normally, cache only safe/read operations.

```nginx
proxy_cache_methods GET HEAD;
```

Example:

```text
GET    → Cache
HEAD   → Cache
POST   → Do not cache
PUT    → Do not cache
PATCH  → Do not cache
DELETE → Do not cache
```

---

# 25. Cache Purging

When content changes, an old cached response may still exist.

Example:

```text
Old:
logo.png

New:
logo.png
```

A simple development/test environment can clear the cache:

```bash
sudo rm -rf /var/cache/nginx/*
```

Then reload Nginx:

```bash
sudo systemctl reload nginx
```

> In production, use controlled and targeted cache invalidation rather than blindly deleting the entire cache.

---

# 26. Microcaching

Microcaching means caching dynamic content for a very short period.

Example:

```nginx
proxy_cache_valid 200 1s;
```

Even a 1-second cache can reduce backend load during traffic spikes.

Example:

```text
1000 requests/second
        ↓
1-second microcache
        ↓
Much fewer backend requests
```

Useful for:

* High-traffic APIs
* Dynamic pages
* News websites
* Traffic bursts

---

# 27. Stale Cache

A stale cache is an expired cache entry.

Normally:

```text
Cache expires
     ↓
Backend request
     ↓
New response
```

Nginx can sometimes serve stale content when the backend is unavailable.

Example:

```nginx
proxy_cache_use_stale error timeout updating;
```

This can improve application resilience.

---

# 28. Cache Revalidation

Revalidation checks whether cached content is still valid.

Common mechanisms:

```text
ETag
Last-Modified
```

Example:

```text
Browser
   ↓
If-None-Match: "abc123"
   ↓
Server
   ↓
No changes
   ↓
304 Not Modified
```

This saves bandwidth.

---

# 29. CDN Caching

A CDN caches content at edge locations close to users.

Architecture:

```text
User
  ↓
CDN Edge
  ├── HIT → Cached Response
  │
  └── MISS
        ↓
      Origin
```

Benefits:

* Lower latency
* Lower origin load
* Better global performance
* Better scalability
* Reduced bandwidth to origin

---

# 30. Application-Level Caching

Applications can cache data using technologies such as:

```text
Redis
Memcached
```

Architecture:

```text
Application
     ↓
   Redis
   ├── HIT → Return data
   │
   └── MISS
        ↓
     Database
        ↓
       Redis
```

Example Redis key:

```text
user:1001
```

Value:

```json
{
  "name": "Prins",
  "role": "DevOps"
}
```

---

# 31. Cache-Aside Pattern

Process:

```text
1. Application checks cache
2. Cache HIT → Return data
3. Cache MISS → Query database
4. Store result in cache
5. Return result
```

Architecture:

```text
Application
     ↓
   Redis
    │
    ├── HIT → Return
    │
    └── MISS
         ↓
      Database
         ↓
       Redis
```

---

# 32. Write-Through Cache

Data is written to cache and database together.

```text
Application
     ↓
   Cache
     ↓
 Database
```

### Benefits

* Better cache consistency
* Simple read behavior

### Disadvantage

Writes may be slower.

---

# 33. Write-Back Cache

Data is first written to cache and later persisted to the database.

```text
Application
     ↓
   Cache
     ↓
   Later
     ↓
 Database
```

### Benefit

Very fast writes.

### Risk

Data may be lost if the cache fails before persistence.

---

# 34. Write-Around Cache

Data is written directly to the database.

```text
Application
     ↓
 Database
```

The cache is populated when the data is later read.

Useful when data is written frequently but not immediately read.

---

# 35. Cache Stampede

Cache stampede happens when a popular cache entry expires and many requests simultaneously hit the backend.

Example:

```text
Cache expires
     ↓
10,000 requests
     ↓
10,000 backend requests
     ↓
Backend overloaded
```

Also called:

```text
Thundering Herd
```

### Solutions

* Cache locking
* Request coalescing
* Stale-while-revalidate
* Randomized TTL
* Cache warming

---

# 36. Cache Penetration

Cache penetration happens when users repeatedly request data that does not exist.

Example:

```text
GET /user/999999999
```

Flow:

```text
Cache MISS
    ↓
Database
    ↓
Not Found
```

Repeated requests can overload the database.

### Solutions

* Cache negative results
* Validate input
* Bloom filters
* Rate limiting

Example:

```text
user:999999
    ↓
NOT_FOUND
    ↓
TTL = 30 seconds
```

---

# 37. Cache Avalanche

Cache avalanche occurs when many cache entries expire at the same time.

Example:

```text
10,000 cache entries
       ↓
All expire at 12:00
       ↓
Huge backend traffic
```

### Solutions

* Randomized TTL
* Staggered expiration
* Cache warming
* Multiple cache layers

---

# 38. Cache Poisoning

Cache poisoning occurs when an attacker causes an incorrect or malicious response to be cached and served to other users.

Possible causes:

* Incorrect cache key
* Untrusted headers
* Improper Host header handling
* Incorrect query parameter handling

### Prevention

* Carefully design cache keys
* Validate request headers
* Never trust unvalidated cache-related input
* Avoid caching sensitive responses

---

# 39. Personalized Content and Caching

Be extremely careful with:

```text
/login
/profile
/dashboard
/account
/payment
```

Example:

```text
User A → /profile → A's data
User B → /profile → B's data
```

Incorrect shared caching could cause:

```text
User A's cached profile
        ↓
User B receives A's data
```

This is a serious security vulnerability.

---

# 40. Authentication and Caching

Requests containing:

```http
Authorization: Bearer TOKEN
```

require special handling.

Example:

```nginx
proxy_cache_bypass $http_authorization;
proxy_no_cache $http_authorization;
```

This reduces the risk of accidentally sharing authenticated responses.

---

# 41. Cache Compression

Caching can work together with compression.

Architecture:

```text
Origin Response
      ↓
Compression
      ↓
Cache
      ↓
Client
```

Nginx example:

```nginx
gzip on;

gzip_types
    text/plain
    text/css
    application/javascript
    application/json;
```

Compression and caching should be configured consistently for different content encodings.

---

# 42. Cache Security

Do not blindly cache:

```text
Passwords
Payment information
Private user data
Authentication responses
Sensitive personal information
```

Always consider:

```text
Authentication
Authorization
Cookies
Authorization headers
Cache keys
Cache poisoning
HTTPS
Private responses
```

---

# 43. Cache Warming

Cache warming means populating the cache before users request the data.

Example:

```text
Deploy application
       ↓
Warm popular URLs
       ↓
Cache populated
       ↓
Users receive fast responses
```

Useful after:

* Deployment
* Cache flush
* Server restart
* Large content update

---

# 44. Cache Eviction

When the cache becomes full, entries must be removed.

Common strategies:

```text
LRU
LFU
TTL expiration
Size-based eviction
```

## LRU

Least Recently Used.

Removes data that has not been accessed recently.

## LFU

Least Frequently Used.

Removes data that is accessed least frequently.

---

# 45. Cache Monitoring

Important metrics:

```text
Cache Hit Ratio
Cache Miss Ratio
Cache Size
Evictions
TTL
Backend Response Time
Origin Traffic
Error Rate
```

Example:

```text
Total Requests = 1000
Cache Hits     = 900
Cache Misses   = 100
```

Hit ratio:

```text
900 / 1000 × 100 = 90%
```

---

# 46. Nginx Cache Monitoring

Add:

```nginx
add_header X-Cache-Status $upstream_cache_status;
```

Test:

```bash
curl -I http://example.com/api/products
```

First request may show:

```http
X-Cache-Status: MISS
```

Second request may show:

```http
X-Cache-Status: HIT
```

This verifies that the response is being served from Nginx cache.

---

# 47. Complete Nginx Caching Example

```nginx
proxy_cache_path /var/cache/nginx/api
                 levels=1:2
                 keys_zone=api_cache:20m
                 max_size=1g
                 inactive=60m
                 use_temp_path=off;

server {
    listen 80;
    server_name example.com;

    # CSS and JavaScript
    location ~* \.(css|js)$ {
        expires 7d;
        add_header Cache-Control "public";
    }

    # Images
    location ~* \.(jpg|jpeg|png|gif|webp|svg)$ {
        expires 30d;
        add_header Cache-Control "public";
    }

    # API
    location /api/ {
        proxy_pass http://localhost:3000;

        proxy_cache api_cache;
        proxy_cache_methods GET HEAD;

        proxy_cache_valid 200 60s;
        proxy_cache_valid 404 10s;

        proxy_cache_bypass $http_authorization;
        proxy_no_cache $http_authorization;

        add_header X-Cache-Status $upstream_cache_status;
    }
}
```

Test configuration:

```bash
sudo nginx -t
```

Reload:

```bash
sudo systemctl reload nginx
```

Test:

```bash
curl -I http://example.com/api/products
```

Expected first response:

```text
X-Cache-Status: MISS
```

Second request:

```text
X-Cache-Status: HIT
```

---

# 48. Complete Production Caching Architecture

```text
                         Users
                           │
                           ▼
                         DNS
                           │
                           ▼
                       CDN / WAF
                           │
                           ▼
                    Load Balancer
                           │
                           ▼
                      Nginx Layer
                    ┌──────┴──────┐
                    │             │
              Static Cache     API Cache
                    │             │
                    │             ▼
                    │          Backend
                    │             │
                    │           Redis
                    │             │
                    │             ▼
                    │          Database
                    │
                    ▼
              Static Storage
```

Different layers solve different problems:

```text
Browser → Local user-side caching
CDN     → Global edge caching
Nginx   → Reverse proxy caching
Redis   → Application data caching
DB      → Database/query-level caching
```

---

# 49. Recommended Cache Strategy

| Content        | Recommended Strategy      |
| -------------- | ------------------------- |
| Images         | 30 days or longer         |
| CSS            | 7 days or longer          |
| JavaScript     | 7 days or longer          |
| Fonts          | 30 days or longer         |
| HTML           | Short TTL or revalidation |
| Public API     | Short TTL if safe         |
| Private API    | Private/no shared cache   |
| Login          | No shared cache           |
| Payment        | No shared cache           |
| Session data   | No shared cache           |
| User dashboard | Private/no shared cache   |

For versioned assets:

```text
app.a82f31.js
style.91ab22.css
```

long cache lifetimes are safer because changing the content normally produces a new filename.

---

# 50. Cache Invalidation

Cache invalidation is the process of removing or updating stale cached data.

Common approaches:

```text
TTL expiration
Explicit purge
Versioned URLs
Event-driven invalidation
Cache-aside deletion
Write-through updates
```

Example:

```text
Database
   ↓
Data updated
   ↓
Old cache invalidated
   ↓
Next request
   ↓
Fresh data
```

---

# 51. Important Caching Problems

Remember these:

```text
1. Cache Stampede
2. Cache Penetration
3. Cache Avalanche
4. Cache Poisoning
5. Stale Data
6. Incorrect Cache Key
7. Personalized Data Leakage
8. Cache Invalidation
9. Memory/Disk Usage
10. Incorrect TTL
```

---

# 52. Production Caching Best Practices

1. Cache only safe content.
2. Use appropriate TTL values.
3. Use long TTLs for immutable static assets.
4. Use short TTLs for frequently changing data.
5. Avoid shared caching of personalized responses.
6. Carefully design cache keys.
7. Monitor cache HIT/MISS ratios.
8. Implement cache invalidation.
9. Protect against cache stampede.
10. Protect against cache poisoning.
11. Test cache behavior before production.
12. Monitor cache storage usage.
13. Use HTTPS for secure traffic.
14. Use versioned static assets.
15. Do not blindly cache authenticated responses.
16. Use microcaching where appropriate.
17. Use CDN caching for globally distributed static content.
18. Use Redis/application caching for frequently accessed data.

---

# 53. Caching Learning Roadmap

## Beginner

```text
1. What is caching?
2. Why caching is required
3. Browser caching
4. Cache-Control
5. Expires
6. ETag
7. Last-Modified
8. Static asset caching
9. HTML caching
10. API caching
```

## Intermediate

```text
11. Cache TTL
12. Cache HIT/MISS
13. Nginx proxy caching
14. Cache key
15. Cache bypass
16. Cache no-cache rules
17. Cache purge
18. Microcaching
19. Stale caching
20. Cache revalidation
```

## Advanced

```text
21. CDN caching
22. Redis caching
23. Cache-aside
24. Write-through
25. Write-back
26. Write-around
27. Cache stampede
28. Cache penetration
29. Cache avalanche
30. Cache poisoning
31. Cache invalidation
32. Cache warming
33. Cache eviction
34. Cache monitoring
35. Production caching architecture
36. Caching security best practices
```

---

# 54. Practical Nginx Caching Lab

Follow this order for hands-on DevOps practice:

```text
Static Asset Caching
        ↓
Cache-Control
        ↓
ETag / Last-Modified
        ↓
API Caching
        ↓
Nginx proxy_cache
        ↓
HIT / MISS Verification
        ↓
Cache Bypass
        ↓
Cache Purge
        ↓
Microcaching
        ↓
Stale Cache
        ↓
Cache Revalidation
        ↓
Production Caching Best Practices
```

## Useful Commands

Test Nginx:

```bash
sudo nginx -t
```

Reload:

```bash
sudo systemctl reload nginx
```

Check Nginx status:

```bash
sudo systemctl status nginx
```

Check cache directory:

```bash
sudo ls -lah /var/cache/nginx/
```

Test headers:

```bash
curl -I http://example.com/
```

Test API:

```bash
curl -I http://example.com/api/products
```

Monitor access logs:

```bash
sudo tail -f /var/log/nginx/access.log
```

Monitor error logs:

```bash
sudo tail -f /var/log/nginx/error.log
```

---

# 55. Final Summary

Caching stores frequently accessed data closer to the user or application so it can be returned faster.

The complete caching architecture can be understood as:

```text
User
 ↓
Browser Cache
 ↓
CDN Cache
 ↓
Nginx Reverse Proxy Cache
 ↓
Application Cache
 ↓
Redis
 ↓
Database
```

The most important concepts to understand for DevOps are:

```text
Cache-Control
ETag
Last-Modified
TTL
Cache HIT
Cache MISS
Cache Key
Cache Bypass
Cache Purge
Microcaching
Stale Cache
Cache Revalidation
CDN
Redis
Cache Invalidation
Cache Stampede
Cache Penetration
Cache Avalanche
Cache Poisoning
Cache Monitoring
Production Best Practices
```
