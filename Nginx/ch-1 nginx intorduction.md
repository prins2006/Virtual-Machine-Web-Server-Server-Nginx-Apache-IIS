# Nginx Architecture – Complete Deep Dive (Production Level)

> **Goal:** Understand how Nginx works internally, why it is fast, and how it is used in real production environments.

---

# Table of Contents

1. What is Nginx?
2. Why Was Nginx Created?
3. Nginx Architecture Overview
4. Master Process
5. Worker Processes
6. Event-Driven Architecture
7. Non-Blocking I/O
8. Request Lifecycle
9. Nginx Modules
10. Reverse Proxy
11. Load Balancing
12. SSL/TLS Termination
13. Static File Serving
14. Caching
15. Compression
16. Logging Architecture
17. Security Features
18. Production Architecture
19. Real-World Examples
20. Performance Tuning
21. Memory Usage
22. Advantages & Limitations
23. Interview Questions
24. Summary

---

# 1. What is Nginx?

Nginx (pronounced **Engine-X**) is a high-performance web server, reverse proxy, load balancer, API gateway, and HTTP cache.

Unlike traditional web servers, Nginx uses an **event-driven, asynchronous, non-blocking architecture**, allowing a small number of worker processes to handle thousands of simultaneous client connections.

---

# 2. Why Was Nginx Created?

Around the early 2000s, traditional web servers struggled to handle very large numbers of simultaneous users.

This challenge became known as the **C10K Problem**, meaning:

> How can one server efficiently handle **10,000 concurrent client connections**?

Traditional servers created one process or thread per request, which increased memory and CPU usage.

Nginx solved this problem using an event-driven architecture.

---

# 3. Nginx Architecture Overview

```text
                     Internet
                         │
                         ▼
                  Master Process
                         │
      ┌──────────────────┼──────────────────┐
      │                  │                  │
      ▼                  ▼                  ▼
 Worker Process 1   Worker Process 2   Worker Process 3
      │                  │                  │
   Event Loop         Event Loop         Event Loop
      │                  │                  │
 Thousands of       Thousands of       Thousands of
 Client Requests    Client Requests    Client Requests
```

The **Master Process** manages Nginx.

The **Worker Processes** handle client requests.

---

# 4. Master Process

The Master Process does **not** serve websites directly.

Its responsibilities include:

* Reading configuration files
* Starting worker processes
* Reloading configuration
* Stopping workers
* Restarting workers
* Monitoring worker health

Example:

```bash
sudo systemctl reload nginx
```

What happens?

```text
Administrator
       │
       ▼
Master Process
       │
Reads new configuration
       │
Starts new workers
       │
Stops old workers gracefully
```

This allows configuration changes without interrupting active users.

---

# 5. Worker Processes

Worker processes perform the actual work.

Responsibilities:

* Accept client connections
* Read requests
* Serve static files
* Forward requests to backend servers
* Return responses

Example:

A server with four CPU cores:

```text
CPU Core 1 → Worker 1

CPU Core 2 → Worker 2

CPU Core 3 → Worker 3

CPU Core 4 → Worker 4
```

Configuration:

```nginx
worker_processes auto;
```

Nginx automatically creates one worker process per CPU core.

---

# 6. Event-Driven Architecture

Traditional servers often dedicate one thread to each client.

Nginx is different.

One worker process can manage thousands of clients by continuously checking which connections are ready to perform work.

Example:

```text
Worker Process

Client 1
Client 2
Client 3
Client 4
...
Client 10000
```

The worker only spends CPU time on connections that are ready.

---

# 7. Non-Blocking I/O

Suppose a client requests a large image.

Traditional approach:

```text
Read File

↓

Wait

↓

Send Response
```

CPU remains idle while waiting.

Nginx:

```text
Read File

↓

Serve Other Clients

↓

File Ready

↓

Send Response
```

Instead of waiting, Nginx continues serving other clients.

---

# 8. Request Lifecycle

Suppose a user opens:

```text
https://shop.example.com/products
```

The request passes through these steps:

```text
Browser
      │
      ▼
DNS Resolution
      │
      ▼
TCP Connection
      │
      ▼
SSL/TLS Handshake (HTTPS)
      │
      ▼
Master Process
      │
      ▼
Worker Process
      │
      ▼
HTTP Request Parsing
      │
      ▼
Location Matching
      │
      ├────────► Static File?
      │             │
      │             ▼
      │        Return File
      │
      ▼
Reverse Proxy
      │
      ▼
Application Server
      │
      ▼
Database
      │
      ▼
Application Response
      │
      ▼
Nginx
      │
      ▼
Browser
```

---

# 9. Nginx Modules

Nginx is modular.

Common modules include:

## Core Module

Controls:

* Worker processes
* Connections
* Events

Example:

```nginx
worker_processes auto;
```

---

## HTTP Module

Handles:

* HTTP requests
* Server blocks
* Location blocks

Example:

```nginx
server {
    listen 80;
}
```

---

## Stream Module

Handles:

* TCP
* UDP

Example:

* MySQL Proxy
* Redis Proxy

---

## Mail Module

Supports:

* SMTP
* POP3
* IMAP

---

# 10. Reverse Proxy

Nginx often sits in front of application servers.

```text
Browser

↓

Nginx

↓

Node.js
```

Configuration example:

```nginx
location / {
    proxy_pass http://localhost:3000;
}
```

Benefits:

* Security
* HTTPS
* Caching
* Load balancing

---

# 11. Load Balancing

One application server may not be enough.

```text
Internet
     │
     ▼
    Nginx
     │
 ┌───┼───┐
 ▼   ▼   ▼
App1 App2 App3
```

Example:

```nginx
upstream backend {
    server 192.168.1.10;
    server 192.168.1.11;
    server 192.168.1.12;
}
```

Nginx distributes requests among backend servers.

---

# 12. SSL/TLS Termination

Instead of every backend server handling encryption:

```text
Users

↓

HTTPS

↓

Nginx

↓

HTTP

↓

Application Servers
```

Advantages:

* Lower CPU usage on backend servers
* Centralized certificate management

---

# 13. Static File Serving

Nginx is highly optimized for static files.

Examples:

* Images
* CSS
* JavaScript
* Fonts
* Videos

Flow:

```text
Browser

↓

Nginx

↓

Image Returned
```

The application server is not involved.

---

# 14. Caching

Nginx can cache backend responses.

First request:

```text
Client

↓

Backend

↓

Nginx Cache
```

Second request:

```text
Client

↓

Nginx Cache

↓

Response
```

The backend is skipped, reducing response time.

---

# 15. Compression

Nginx supports Gzip compression.

Without compression:

```text
CSS File = 500 KB
```

With Gzip:

```text
CSS File = 120 KB
```

Benefits:

* Faster downloads
* Reduced bandwidth

---

# 16. Logging Architecture

Two primary log files:

## Access Log

Records:

* Client IP
* Request
* Status code
* Response size

Example:

```text
192.168.1.20 - - [28/Jul/2026] "GET /index.html HTTP/1.1" 200
```

---

## Error Log

Records:

* Configuration errors
* Permission issues
* Backend failures

Example:

```text
connect() failed (111: Connection refused)
```

---

# 17. Security Features

Nginx supports:

* HTTPS
* Rate limiting
* IP allow/deny
* Header filtering
* Reverse proxy isolation
* Request size limits
* Hiding backend servers

---

# 18. Production Architecture

Example:

```text
                 Internet
                     │
                     ▼
                 Firewall
                     │
                     ▼
                 Load Balancer
                     │
                     ▼
                  Nginx
        ┌────────────┴────────────┐
        │                         │
   Static Files              Reverse Proxy
                                   │
                ┌──────────────────┼──────────────────┐
                ▼                  ▼                  ▼
           App Server 1       App Server 2      App Server 3
                │                  │                  │
                └──────────────┬───┴──────────────────┘
                               ▼
                           Database
```

---

# 19. Real-World Example

Imagine an online shopping platform.

Traffic:

* 80,000 image requests
* 10,000 CSS requests
* 5,000 API requests
* 5,000 checkout requests

Nginx serves:

* Images
* CSS
* JavaScript

Nginx forwards:

* API requests
* Checkout requests

This greatly reduces backend workload.

---

# 20. Performance Tuning

Important settings:

```nginx
worker_processes auto;

worker_connections 4096;

keepalive_timeout 65;

sendfile on;

gzip on;
```

These improve performance by:

* Matching workers to CPU cores
* Allowing more simultaneous connections
* Reducing file-copy overhead
* Compressing responses

---

# 21. Memory Usage

Traditional thread-based servers allocate memory for each thread.

Nginx primarily stores connection state.

Example:

```text
10,000 Connections

Traditional Server
≈ High memory usage

Nginx
≈ Much lower memory usage
```

This efficiency is one reason Nginx is popular for high-traffic environments.

---

# 22. Advantages

* Very high performance
* Low memory usage
* Excellent reverse proxy
* Built-in load balancing
* SSL termination
* Fast static file serving
* Scales well for modern web applications

## Limitations

* No `.htaccess` support
* Per-directory configuration is not available
* Dynamic modules are less flexible than Apache's module ecosystem
* Some legacy PHP applications are easier to run on Apache

---

# 23. Interview Questions

### Q1. Why is Nginx faster than traditional web servers?

**Answer:** Because it uses an event-driven, asynchronous, non-blocking architecture where a small number of worker processes handle many concurrent connections efficiently.

---

### Q2. What is the purpose of the Master Process?

**Answer:** It manages configuration, starts and monitors worker processes, and performs graceful reloads and shutdowns.

---

### Q3. What do Worker Processes do?

**Answer:** They accept client connections, process requests, serve files, proxy requests, and return responses.

---

### Q4. Why is Nginx commonly used as a reverse proxy?

**Answer:** It provides HTTPS termination, load balancing, caching, request routing, and protects backend application servers.

---

### Q5. Why is Nginx popular in cloud and Kubernetes environments?

**Answer:** It handles high traffic efficiently, integrates well as an Ingress Controller or reverse proxy, and scales effectively with containerized applications.

---

# 24. Summary

| Component          | Purpose                                                      |
| ------------------ | ------------------------------------------------------------ |
| Master Process     | Reads configuration, starts and manages workers              |
| Worker Process     | Handles client requests                                      |
| Event Loop         | Allows one worker to manage many connections                 |
| Reverse Proxy      | Routes requests to backend servers                           |
| Load Balancer      | Distributes traffic across multiple servers                  |
| Static File Server | Serves images, CSS, JavaScript, and other static assets      |
| SSL/TLS            | Encrypts client communication                                |
| Cache              | Stores responses to reduce backend load                      |
| Logs               | Records access and errors for monitoring and troubleshooting |

## Final Key Takeaway

Think of **Nginx as the traffic controller of a busy airport**:

* It quickly directs every incoming request to the correct destination.
* It serves static content itself whenever possible.
* It forwards only the necessary dynamic requests to backend applications.
* It balances traffic across multiple servers and protects them from overload.

This architecture is why Nginx is widely used in modern production systems for high-performance websites, APIs, microservices, and cloud-native applications.
