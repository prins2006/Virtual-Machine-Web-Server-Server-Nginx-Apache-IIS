# Apache Server Architecture – Deep Dive Guide

## What is Apache?

Apache HTTP Server (Apache2) is an open-source web server that receives HTTP/HTTPS requests from clients and returns web pages, APIs, files, or application responses. It is one of the oldest and most widely used web servers in production environments.

Apache is especially popular for PHP applications such as WordPress, Drupal, Joomla, and many enterprise web applications.

---

# Real-Life Analogy

Imagine a large restaurant.

* **Reception Manager** → Apache Parent Process
* **Waiters** → Apache Child Processes
* **Service Staff** → Threads inside child processes
* **Customers** → Client Requests
* **Kitchen** → PHP / Application Logic
* **Warehouse** → Database

When customers arrive, the reception manager assigns them to available waiters. The waiters communicate with the kitchen and return the food to customers.

This is similar to how Apache handles web requests.

---

# High-Level Apache Architecture

```text
                 Internet / Clients
                         │
                         ▼
                 Apache Parent Process
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
   Child Process     Child Process     Child Process
        │                │                │
   ┌────┴────┐      ┌────┴────┐      ┌────┴────┐
   ▼    ▼    ▼      ▼    ▼    ▼      ▼    ▼    ▼
 Thread Thread Thread Thread Thread Thread Thread Thread Thread
        │                │                │
        └──────────────┬────────────────┘
                       ▼
                PHP / Application
                       ▼
                   MySQL Database
```

---

# Core Components

## 1. Parent Process

The parent process starts when Apache service starts.

### Responsibilities

* Read configuration files
* Load modules
* Open listening ports (80, 443)
* Spawn child processes
* Monitor child processes
* Restart failed children
* Graceful reload of configuration
* Shutdown management

### Important Point

The parent process usually **does not serve client requests directly**.

Check it:

```bash
ps -ef | grep apache2
```

You will see one root-owned parent process and several worker processes.

---

## 2. Child Processes

Child processes perform the actual work.

Responsibilities:

* Accept client connections
* Read HTTP requests
* Process requests
* Execute modules
* Run PHP (depending on setup)
* Send responses

---

## 3. Threads

In some Apache MPMs, each child process contains multiple threads.

Example:

```text
Child Process 1
 ├── Thread 1
 ├── Thread 2
 ├── Thread 3
 └── Thread 4
```

Each thread can handle one request at a time.

---

# Apache Multi-Processing Modules (MPMs)

Apache supports different internal architectures called **MPMs**.

Check active MPM:

```bash
apachectl -V | grep MPM
```

---

# Prefork MPM

## Architecture

```text
Parent
  │
  ├── Process 1 → Request 1
  ├── Process 2 → Request 2
  ├── Process 3 → Request 3
  └── Process 4 → Request 4
```

### Characteristics

* One process per request
* No threads
* High stability
* Compatible with old PHP modules (`mod_php`)

### Problem

1000 concurrent users may create 1000 processes.

Memory usage becomes very high.

### When Used

* Legacy PHP applications
* Older hosting environments

---

# Worker MPM

## Architecture

```text
Parent
  │
  ├── Child Process 1
  │      ├── Thread 1
  │      ├── Thread 2
  │      └── Thread 3
  └── Child Process 2
         ├── Thread 1
         ├── Thread 2
         └── Thread 3
```

### Characteristics

* Multiple threads per process
* Lower memory usage
* Better concurrency than Prefork

### Example

2 processes × 25 threads = 50 simultaneous requests.

---

# Event MPM (Modern Apache)

## Architecture

```text
Parent
  │
  ├── Listener Thread
  ├── Worker Threads
  └── KeepAlive Manager
```

### Why It Exists

In Worker MPM, idle KeepAlive connections still occupy threads. Event MPM separates idle connections from active workers.

### Benefits

* Better scalability
* Lower memory usage
* Better performance for many idle connections

This is the recommended MPM for modern Apache deployments.

---

# Request Lifecycle (Step-by-Step)

Suppose a user visits:

```
https://shop.example.com/products.php
```

## Step 1 – DNS Resolution

Browser finds server IP.

## Step 2 – TCP Connection

Connection to port 443 is established.

## Step 3 – SSL Handshake

Apache negotiates HTTPS encryption.

## Step 4 – Parent Accepts Connection

Connection is handed to a child process/thread.

## Step 5 – HTTP Parsing

Apache reads headers, method, URL, cookies, etc.

## Step 6 – VirtualHost Selection

Apache selects the correct website configuration.

## Step 7 – Module Processing

Modules such as `mod_rewrite`, `mod_auth`, `mod_headers` run.

## Step 8 – PHP Execution

Request is sent to PHP engine.

## Step 9 – Database Query

PHP communicates with MySQL.

## Step 10 – Response Generation

HTML is generated.

## Step 11 – Response Sent

Apache returns the response to the browser.

---

# Virtual Host Architecture

Apache can host multiple websites on one server.

```text
                 Apache
                    │
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼
 site1.com      site2.com      site3.com
```

Example configuration:

```apache
<VirtualHost *:80>
    ServerName site1.com
    DocumentRoot /var/www/site1
</VirtualHost>

<VirtualHost *:80>
    ServerName site2.com
    DocumentRoot /var/www/site2
</VirtualHost>
```

Apache chooses the site based on the `Host` header.

---

# Module Architecture

Apache is modular. Features are loaded as modules.

Important modules:

| Module         | Purpose                  |
| -------------- | ------------------------ |
| mod_ssl        | HTTPS                    |
| mod_rewrite    | URL rewriting            |
| mod_proxy      | Reverse proxy            |
| mod_proxy_http | HTTP proxy               |
| mod_headers    | Header manipulation      |
| mod_security   | Web Application Firewall |
| mod_deflate    | Compression              |
| mod_http2      | HTTP/2 support           |
| mod_cache      | Caching                  |

Enable module:

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

# .htaccess Architecture

Apache uniquely supports per-directory configuration.

Example:

```apache
RewriteEngine On
RewriteRule ^product/([0-9]+)$ product.php?id=$1 [L,QSA]
```

Apache checks `.htaccess` while traversing directories, which adds flexibility but also some performance overhead.

---

# KeepAlive Architecture

Without KeepAlive:

```text
Request 1 → New TCP connection
Request 2 → New TCP connection
Request 3 → New TCP connection
```

With KeepAlive:

```text
One TCP connection
 ├── Request 1
 ├── Request 2
 └── Request 3
```

Fewer TCP handshakes improve performance. Event MPM handles KeepAlive more efficiently than Prefork.

---

# Memory Usage Example

Assume:

* Prefork process = 25 MB
* Worker thread = 2 MB equivalent

### Prefork

500 users × 25 MB ≈ 12.5 GB

### Worker

20 processes × 25 threads ≈ much lower memory usage.

This is why Worker/Event scale better.

---

# Logging Architecture

Apache writes logs independently of request processing.

## Access Log

```
/var/log/apache2/access.log
```

Contains client IP, method, URL, status code, bytes, etc.

## Error Log

```
/var/log/apache2/error.log
```

Contains PHP errors, module errors, startup issues, SSL errors, etc.

Monitor logs:

```bash
tail -f /var/log/apache2/access.log
tail -f /var/log/apache2/error.log
```

---

# Reverse Proxy Architecture

Apache can proxy requests to backend servers.

```text
Client
   │
   ▼
Apache
   │
   ▼
Node.js App
```

Example:

```apache
ProxyPass /api http://127.0.0.1:3000/
ProxyPassReverse /api http://127.0.0.1:3000/
```

---

# Load Balancing Architecture

```text
Clients
   │
   ▼
Apache Load Balancer
   ├── App Server 1
   ├── App Server 2
   └── App Server 3
```

Apache distributes requests among backend servers.

---

# Production Example – WordPress Server

## Architecture

```text
Internet
   │
   ▼
Apache (Event MPM)
   │
   ├── mod_ssl
   ├── mod_rewrite
   ├── PHP-FPM
   └── Static Files
           │
           ▼
        MySQL Database
```

### Request Flow

1. Browser requests `/blog/post-1`.
2. `mod_rewrite` converts friendly URL to `index.php`.
3. PHP-FPM executes WordPress.
4. WordPress queries MySQL.
5. HTML response is generated.
6. Apache sends response.

---

# Monitoring Apache Architecture

## Service Status

```bash
sudo systemctl status apache2
```

## Open Connections

```bash
sudo ss -tulpn | grep apache2
```

## Process Count

```bash
ps -ef | grep apache2 | wc -l
```

## Apache Status Page

Enable `mod_status`:

```bash
sudo a2enmod status
```

Visit:

```
http://localhost/server-status
```

---

# Common Bottlenecks

| Problem                 | Cause                     | Solution               |
| ----------------------- | ------------------------- | ---------------------- |
| High RAM                | Prefork MPM               | Switch to Event MPM    |
| Too many idle processes | KeepAlive                 | Tune KeepAliveTimeout  |
| Slow PHP                | mod_php                   | Use PHP-FPM            |
| Slow static files       | Apache serving everything | Use Nginx in front     |
| High CPU                | Heavy modules             | Disable unused modules |

---

# Apache vs Nginx Architecture (Quick View)

| Feature           | Apache         | Nginx        |
| ----------------- | -------------- | ------------ |
| Core Model        | Process/Thread | Event-driven |
| Concurrency       | Medium         | Very High    |
| Memory Usage      | Higher         | Lower        |
| .htaccess         | Yes            | No           |
| PHP Compatibility | Excellent      | Via PHP-FPM  |
| Static File Speed | Good           | Excellent    |
| Reverse Proxy     | Good           | Excellent    |

---

# Interview Questions

### What is an MPM?

An MPM (Multi-Processing Module) defines how Apache creates processes and threads to handle requests.

### Which MPM is recommended today?

**Event MPM** is generally recommended for modern production deployments.

### What is the role of the parent process?

Configuration management, process management, and lifecycle control.

### Why is Prefork memory intensive?

Because each request requires a separate process.

### Why use PHP-FPM with Apache?

Better performance, lower memory usage, and separation of PHP execution from the web server.

---

# Key Takeaways

* Apache uses a **parent + child process architecture**.
* Request handling depends on the selected **MPM**.
* **Prefork** is simple but memory heavy.
* **Worker** uses threads and scales better.
* **Event** is the modern, scalable choice.
* Apache's biggest strengths are **modules** and **.htaccess** flexibility.
* For high-traffic environments, Apache is often combined with **Nginx** or uses **Event MPM + PHP-FPM** for better scalability.
* Understanding Apache architecture helps with performance tuning, troubleshooting, and production deployment design.
