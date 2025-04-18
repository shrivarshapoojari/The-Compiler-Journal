---
title: "What Really Happens When You Type a URL in Your Browser?"
date: 2025-04-18
description: "A detailed, step-by-step breakdown of everything that happens from the moment you hit Enter after typing a URL in your browser."
tags: ["networking", "browsers", "web", "dns", "tcp/ip", "http", "tls", "rendering"]
---

> Ever stopped to think about what really happens behind the scenes when you type a URL like `www.google.com` and hit Enter? The process involves everything from DNS lookups and network protocols to security handshakes and page rendering. Let’s break it all down clearly and completely.

---

## 1. You Hit Enter

As soon as you hit Enter, your browser begins the process of resolving and requesting the webpage. It has to find the server that hosts the website, establish a connection with it, and ask for the right resource. This starts a chain of events involving many systems working together across the internet.

---

## 2. URL Parsing

The browser first parses the URL to break it into components:
- Protocol (e.g., https)
- Domain (e.g., www.google.com)
- Port (e.g., 443 for HTTPS)
- Path (e.g., /search)
- Query string (e.g., q=chatgpt)

Each part tells the browser something essential: how to connect, where to connect, and what exactly to request.

---

## 3. Checking the Cache

Before going to the network, the browser checks if it already has what it needs. It looks at:
- DNS cache (to resolve the domain name)
- HTTP cache (to avoid re-downloading content)
- Service workers or local storage

If the requested data is cached and still valid, the browser may serve it directly, skipping network steps.

---

## 4. DNS Resolution

If the domain isn’t cached, the browser resolves it to an IP address using DNS. It may query:
- The OS DNS cache
- The local hosts file
- External DNS servers like 8.8.8.8

The DNS process can involve multiple lookups: starting from root, to TLD (like `.com`), then to the authoritative server that knows about the domain.

---

## 5. TCP Connection Setup

With the IP address resolved, the browser establishes a TCP connection to the server. TCP ensures reliable, ordered data transmission. The connection is made using the TCP three-way handshake:
1. SYN
2. SYN-ACK
3. ACK

Once the handshake completes, both client and server can start sending data.

---

## 6. TLS Handshake (If HTTPS)

If the URL uses HTTPS, the browser performs a TLS handshake to establish a secure connection. This process includes:
- Exchanging supported cipher suites and versions
- Verifying the server’s SSL certificate
- Agreeing on a shared secret key

The result is a secure, encrypted connection to protect the communication.

---

## 7. Sending the HTTP Request

Now the browser is ready to send the HTTP request. This includes:
- The method (e.g., GET or POST)
- Headers (like User-Agent and Accept)
- Cookies or authentication tokens (if any)

This request travels to the server over the secured TCP/TLS connection.

---

## 8. Server Processes the Request

The server receives the request and decides how to handle it. This may involve:
- Routing the request to the correct handler
- Interacting with databases or APIs
- Assembling a response (HTML, JSON, etc.)

It then sends this response back to the browser with a status code like 200 OK or 404 Not Found.

---

## 9. Receiving the Response

The browser receives the response and checks its headers and body. It examines:
- Status codes to see if the request was successful
- Content-Type to decide how to handle it
- Cache-related headers for future optimization

If the response is HTML, the rendering process begins.

---

## 10. Rendering the Page

Rendering turns the HTML response into a visible webpage. The browser:
- Parses the HTML into a DOM tree
- Fetches and parses CSS into a CSSOM
- Executes JavaScript, possibly modifying the DOM
- Calculates layout and paint instructions

This whole process happens in stages and may continue dynamically as more data arrives.

---

## 11. Loading Subresources

The initial HTML may reference other resources like:
- Images
- CSS files
- JavaScript files

Each of these triggers separate network requests. The browser may prioritize or delay some of them based on their role in rendering.

---

## 12. JavaScript Execution

Once fetched, JavaScript is parsed and executed. It can:
- Modify the DOM (add/remove content)
- Make AJAX/fetch calls for more data
- Attach event listeners for interactivity

JavaScript can block rendering, so browsers try to load it asynchronously if possible.

---

## 13. Reflows and Repaints

When JavaScript changes styles or content, the browser may trigger:
- A reflow (layout recalculation)
- A repaint (pixel update)

Too many of these can cause jank or slow performance, so browsers optimize aggressively to avoid unnecessary work.

---

## 14. Page Fully Interactive

Once all scripts have loaded, layouts are done, and event listeners are active, the page becomes fully interactive. This point is measured by metrics like:
- First Contentful Paint (FCP)
- Time to Interactive (TTI)
- Largest Contentful Paint (LCP)

These help developers understand how quickly a user can start using the site.

---

## 🧠 Conclusion

Typing a URL might seem like a simple act, but it sets off a complex and elegant chain of events behind the scenes. From resolving domain names to establishing secure connections, processing HTML, loading assets, and finally rendering pixels on your screen — your browser works hard in milliseconds to bring the internet to life.

Understanding this process not only deepens your appreciation for web technology but also helps you become a better developer, designer, or curious mind. The next time you press Enter, you’ll know just how much magic is happening beneath the surface.

---
## 🔗 Further Reading

- [How browsers work: Behind the scenes of modern web browsers](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)
- [MDN Web Docs - HTTP Overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
- [MDN Web Docs - DNS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/DNS)
- [Google Web Fundamentals - Critical Rendering Path](https://developers.google.com/web/fundamentals/performance/critical-rendering-path)
- [TLS Handshake Explained](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/)

---

💬 *Feel free to connect with me to discuss any project ideas or for collaboration [Connect](https://www.shrivarshapoojary.in/contact)*

---