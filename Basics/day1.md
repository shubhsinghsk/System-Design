# 🧠 Day 1 Goal

By the end, you should be able to confidently answer:

👉 *“What happens when you type `google.com` and press Enter?”*

(*This question shows up more often than people admit.*)

---

# 🌐 Part 1: The Journey of a Request (Story Mode)

Think of your request as a traveler trying to reach a destination across a giant digital city.

## Step-by-step flow:

### 1️⃣ DNS Resolution (Finding the address)

- You type: `google.com`
- System asks: “What is its IP?”

Flow:

- Browser cache → OS cache → ISP → DNS server
- Finally gets: `142.x.x.x`

👉 DNS = Phonebook of the internet

---

### 2️⃣ TCP Handshake (Connection setup 🤝)

Before sending data, connection must be established:

- SYN → “Can we talk?”
- SYN-ACK → “Yes, I’m ready”
- ACK → “Let’s go”

👉 Reliable connection established

---

### 3️⃣ HTTPS Handshake 🔐

- SSL/TLS negotiation
- Exchange certificates
- Encryption keys established

👉 Now communication is secure

---

### 4️⃣ HTTP Request Sent 📤

Example:

```
GET / HTTP/1.1
Host: google.com
```

---

### 5️⃣ Load Balancer 🎯

Request hits:

- CDN (maybe)
- Then Load Balancer

👉 Decides which server handles request

---

### 6️⃣ Application Server ⚙️

- Processes request
- Might:
    - Query DB
    - Call other services
    - Use cache

---

### 7️⃣ Database / Cache 🧠

- Fetch data
- Redis (fast) or DB (slower but persistent)

---

### 8️⃣ Response Returned 📥

- HTML/CSS/JS sent back
- Browser renders page

---

# 🧠 Visual Flow (Keep this in your brain)

```
Browser → DNS → TCP → HTTPS → Load Balancer → Server → DB/Cache → Response
```

---

# ⚡ Part 2: Key Concepts (Interview Nuggets)

### 🔹 Latency vs Throughput

- Latency = time for 1 request
- Throughput = number of requests/sec

---

### 🔹 Stateless Servers

- Each request independent
- Helps scaling

---

### 🔹 Why caching matters?

- Avoid hitting DB repeatedly

**Interview Question:**

**When I open google.com, the request goes through multiple steps:**

1. **DNS Resolution**
The browser checks cache → OS → ISP → DNS server to resolve google.com into an IP address.
2. **TCP Connection Establishment**
A 3-way handshake (SYN, SYN-ACK, ACK) is performed to establish a reliable connection.
3. **TLS Handshake (HTTPS)**
The client and server exchange certificates and establish encrypted communication.
4. **HTTP Request Sent**
The browser sends an HTTP request (e.g., GET /) to the server.
5. **CDN / Edge Layer (if applicable)**
Static content may be served directly from a CDN to reduce latency.
6. **Load Balancer**
The request is routed to an appropriate backend server.
7. **Application Server Processing**
The server processes the request, possibly interacting with cache or database.
8. **Database / Cache Access**
Data is fetched (preferably from cache like Redis, else DB).
9. **Response Sent Back**
The server returns the response, and the browser renders the page.

**Where can we optimize this flow for performance?**

We can optimize performance at multiple layers:

1. **DNS caching** → avoid repeated lookups
2. **CDN** → serve static content closer to users
3. **Browser caching** → reduce redundant requests
4. **Load balancing strategies** → better traffic distribution
5. **Application caching (Redis)** → reduce DB load
6. **Database indexing** → faster queries
7. **Connection reuse (keep-alive)** → reduce handshake overhead

**If Google removed caching completely, what would happen?**

If caching is removed, every request would directly hit the database, causing a significant increase in load. This would lead to higher latency, reduced throughput, and potential database bottlenecks. As the database becomes overwhelmed, it can trigger cascading failures, resulting in timeouts and degraded user experience. Overall, the system becomes slower, less scalable, and more expensive to operate.