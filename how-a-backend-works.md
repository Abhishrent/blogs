# What is a backend?

In simple terms, a backend is just a computer accessible via the public internet that serves content. In this blog we shall learn how it's made accessible, how it connects to the frontend and how a general backend is implemented. This blog's contents are supported by an actual working project made available on github at the end of this blog that shows the implementation of this very site in react and node.js.

**Why is it called a server?**
It is called a server because it serves some form of content, like documents and data (html, css, json, xml), media (images, videos, audio), files (pdf, zip, executable) and streams (live video, server-sent events, websocket data).

## Why a backend exists

The frontend (your browser) has real limits. Some are hardware, some are by design:

- Limited computing power  
Computating power on the frontend is capped by the client's hardware which might not be able to handle complex computations. A budget phone and a workstation are not the same thing.
- Sandboxed due to security reasons  
A frontend cannot and should not be able to access the file system for obvious security reasons.
- CORS restrictions that prevent arbitrary cross-origin requests  
We will explore more about CORS in later blogs.
- No direct access to databases  
Exposing database credentials in client-side code would give anyone with DevTools raw access to your data.

Everything a frontend cannot do falls to the backend.

## The General Idea of How Frontends and Backends Communicate over the Internet

Web communication happens via protocols. The frontend and backend communicate using protocols.

The general workflow of a web application is that a frontend (client) makes a request to the backend (server) using protocols such as http, websockets, etc. The backend has handlers (methods/functions) for particular requests. When a certain request is made by the client, the handler for that particular request on the server executes, which performs the appropriate operation on that request and sends back the appropriate response, all using the relevant protocol. It is not necessary that the backend always sends a response but is the general practice.

Different protocols are used based on their use cases. The most used protocol is the http protocol. Http was originally meant to transfer html files and later adopted for css, images, scripts, json, etc. We can safely assume that **the web is just HTTP requests chaining off each other.**

Other protocols and their use cases for reference:

| Use case | Protocol |
|---|---|
| Web pages, REST APIs, file downloads | HTTP / HTTPS |
| Real-time two-way communication (chat, live updates) | WebSocket |
| Live video/audio calls | WebRTC |
| Video streaming (YouTube, Netflix) | HLS / DASH (over HTTP) |
| IoT devices, sensors | MQTT |
| Email sending | SMTP |
| Email fetching | IMAP / POP3 |
| File transfer | FTP / SFTP |
| Low-latency, loss-tolerant data (games, video calls) | UDP |
| DNS resolution | DNS (over UDP) |
| Secure shell / remote server access | SSH |

## How a Web Request Travels Through a Server

When you type a URL and hit enter, the browser (client) doesn't know where to send the request. So it asks DNS (Domain Name System) first.

> **About DNS**
>
> Type `172.217.19.164` in the browser search bar and hit enter. You will land on `google.com`. But in practice we do not actually type the ip address to access `google.com` but rather just type the domain itself. The translation of the domain to the corresponding ip is handled by the DNS.
>
> DNS is the phonebook of the internet. It stores domains and their corresponding IP addresses. DNS looks up the domain in the DNS server and returns an IP address of the server.
>
> A DNS server stores records:
> - `A` records — point to an IP address (e.g. `172.217.19.164`)
> - `CNAME` — point to a domain or subdomain (e.g. `google.com` or `drive.google.com`)

Now the browser knows where to go. It sends an HTTP request to that IP. The request hits the server, but before it reaches your backend application it goes through a firewall. The firewall checks the port.

> **What is a port?**
>
> A port is a number that identifies a specific process or service running on a server. When a request arrives at a server's IP, the port tells the server which application should handle it. For example, HTTP traffic arrives on port 80, HTTPS on port 443, and SSH on port 22. Your Node.js app might run on port 3000. The firewall decides which ports are open to the outside world and drops everything else.

Only the allowed ports pass through, that is why we need to configure the firewall (using iptables or ufw) to accept requests on that port. Everything else gets dropped.

Past the firewall, the request hits nginx (or apache as an alternative). nginx is a reverse proxy server which sits in front of your backend application and decides where to forward the request. It also handles SSL (for encryption), redirects, and routing.

nginx then forwards the request to your application running on localhost. This is the first time your actual code sees it. It processes the request, does whatever it needs to do, and sends a response back.

**The full journey looks like this:**

```
Basic VPS
Browser → DNS → VPS → Firewall (ufw) → nginx → localhost

VPS + Cloudflare
Browser → DNS (Cloudflare) → Cloudflare CDN/DDoS → VPS → Firewall (ufw) → nginx → localhost

AWS
Browser → DNS (Route 53) → CloudFront → Load Balancer → Security Group → EC2 Instance → nginx → localhost
```

**Hosting options compared:**

| | Basic VPS | VPS + Cloudflare | AWS |
|---|---|---|---|
| DNS | Your registrar | Cloudflare | Route 53 |
| Firewall | ufw on the VPS | Cloudflare + ufw | Security groups |
| DDoS protection | None | Cloudflare handles it | Basic, paid for more |
| SSL | Certbot / Let's Encrypt | Cloudflare handles it | ACM |
| CDN | None | Cloudflare handles it | CloudFront |
| Reverse proxy | nginx | nginx | nginx / ALB |
| Your app | PM2 on VPS | PM2 on VPS | EC2 / Lambda / ECS |
| Cost | Cheapest | Cheap + free Cloudflare tier | Expensive |
| Complexity | Low | Low | High |
---
Here you can find the implementation of this site's backend.  
[This site's codebase](https://github.com/Abhishrent/blog-codebase/tree/master/backend)  

To understand the implementation you must go through it yourself. The repository contains a well documented readme that shows you around the codebase. I recommend cloning the repository and navigating around the different files. You can use an ai-agent to understand specific code.
