**Session:** Phase 1, Week 1, Session 2

**Date:** February 8, 2026

**Duration:** ~50 minutes

**Status:** Complete ✅

---

## 🎯 What is Networking?

Networking is how computers communicate over the internet. When you visit a website, send an email, or stream a video, networking protocols make it possible.

**This session covers:**

- HTTP/HTTPS - Web communication
- REST - API design pattern
- TCP vs UDP - Transport protocols
- gRPC - Modern alternative to REST

---

## 🌐 HTTP/HTTPS Fundamentals

### What Happens When You Visit a Website?

**Example:** You type [`www.google.com`](http://www.google.com) and hit Enter

**Step 1: DNS Lookup**

```
Browser asks DNS: "What's the IP address of google.com?"
DNS responds: "142.250.185.46"
```

DNS = Domain Name System (phonebook of the internet)

- Translates human-readable names to IP addresses
- [google.com](http://google.com) → 142.250.185.46
- Note: 8.8.8.8 is Google's DNS server, not [google.com](http://google.com) itself

---

**Step 2: HTTP Request**

```
GET / HTTP/1.1
Host: www.google.com
```

Browser sends this request to the IP address.

---

**Step 3: Server Response**

```
HTTP/1.1 200 OK
Content-Type: text/html

<html>
  <body>Google's homepage HTML</body>
</html>
```

---

**Step 4: Browser Renders**

Browser receives HTML and displays the page.

---

### HTTP Request Structure

Every HTTP request has 4 parts:

**1. Method** (What you want to do)

```
GET     - Retrieve/read data
POST    - Send/create data
PUT     - Update data
DELETE  - Remove data
```

**2. URL** (Where to go)

```
https://api.example.com/users/123
```

**3. Headers** (Metadata)

```
Content-Type: application/json
Authorization: Bearer token123
```

**4. Body** (Data you're sending, for POST/PUT)

```json
{
  "name": "John",
  "age": 30
}
```

---

### HTTP Response Structure

**1. Status Code**

```
200 - Success
201 - Created
400 - Bad Request
401 - Unauthorized
404 - Not Found
500 - Server Error
```

**2. Headers**

```
Content-Type: application/json
Set-Cookie: sessionId=abc123
```

**3. Body**

```json
{
  "id": 123,
  "name": "John"
}
```

---

### GET vs POST

**GET = Retrieving/Reading Data**

```
GET /search?q=java HTTP/1.1
```

- Data in URL
- Visible in browser history
- Safe to bookmark
- Use for: Loading pages, searching, reading

**Example:**

```
example.com/search?q=java&limit=10
```

---

**POST = Sending/Creating Data**

```
POST /login HTTP/1.1
Content-Type: application/json

{
  "username": "john",
  "password": "secret123"
}
```

- Data in request body (hidden)
- Not visible in URL
- More secure
- Use for: Login, signup, form submission

---

**Why login uses POST, not GET:**

If we used GET:

```
example.com/login?username=john&password=secret123
```

- Password visible in URL! ❌
- Saved in browser history ❌
- Could be logged by servers ❌

With POST:

- Password in request body ✅
- Not in URL ✅
- More secure ✅

---

### HTTP vs HTTPS

**HTTP (No 'S' - Insecure)**

```
Browser → [username: john, password: 123] → Server
         ↑ Plain text - anyone can read! ↑
```

Data travels unencrypted - WiFi providers, ISPs, hackers can see everything.

---

**HTTPS (With 'S' - Secure)**

```
Browser → [encrypted: x8#f@2k$9m...] → Server
         ↑ Encrypted - can't read! ↑
```

Data is encrypted - only browser and server can read it.

---

### How HTTPS Works

**Step 1: Handshake**

- Browser: "Can we talk securely?"
- Server sends certificate (proves identity)
- Browser verifies certificate

**Step 2: Key Agreement**

- Browser and server create a **unique session key**
- This key is used ONLY for this session
- When you close browser, key is deleted
- Next session gets a completely different key

**Step 3: Encrypted Communication**

- All data encrypted with session key
- Even if intercepted, unreadable

---

**Why session keys change:**

If same key was used forever:

- Hacker steals key once → decrypt ALL traffic ❌

With new key per session:

- Hacker steals today's key → only decrypt today's traffic ✅
- Tomorrow's traffic uses different key → still safe ✅

This is called **forward secrecy**.

---

**Analogy:**

- HTTP = Postcard (anyone can read)
- HTTPS = Sealed envelope (only recipient can open)

---

## 🚀 REST APIs

**REST** = Representational State Transfer

A pattern for designing web APIs using HTTP.

---

### Core Concept: Resources + Actions

**Everything is a resource (noun):**

- Users
- Tweets
- Products
- Orders

**Perform actions (verbs) using HTTP methods:**

- GET - Read
- POST - Create
- PUT - Update
- DELETE - Remove

---

### REST API Design Pattern

**Example: Twitter Clone**

```
GET    /tweets          → Get all tweets
GET    /tweets/123      → Get tweet with ID 123
POST   /tweets          → Create new tweet
PUT    /tweets/123      → Update tweet 123
DELETE /tweets/123      → Delete tweet 123
```

**Pattern:**

- URL = Resource (`/tweets`)
- HTTP Method = Action (GET, POST, etc.)
- Clean and predictable

---

### Creating a Tweet (Example)

**Request:**

```
POST /tweets HTTP/1.1
Content-Type: application/json

{
  "text": "Hello World!",
  "userId": 42
}
```

**Response:**

```
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 999,
  "text": "Hello World!",
  "userId": 42,
  "createdAt": "2026-02-08T10:30:00Z"
}
```

---

### RESTful URL Patterns

**Pattern 1: Resource Hierarchy**

```
GET /users/42/tweets          → User 42's tweets
GET /users/42/tweets/999      → User 42's tweet 999
GET /tweets/999/comments      → Comments on tweet 999
```

Reads naturally: "Get user 42's tweets"

---

**Pattern 2: Query Parameters (Filtering)**

```
GET /tweets?userId=42          → Filter tweets by user
GET /tweets?userId=42&limit=10 → First 10 tweets by user 42
GET /tweets?limit=10           → Pagination
GET /tweets?sort=date          → Sorting
```

---

**When to use which?**

**Use hierarchy:**

- When resource belongs to a parent
- Clear relationship
- Example: `/users/42/tweets`

**Use query params:**

- Filtering/searching
- Optional parameters
- Example: `/tweets?userId=42&limit=5`

**Both are valid REST!**

---

### REST Best Practices

**1. Use nouns, not verbs**

```
✅ GET /tweets
❌ GET /getTweets
```

**2. Use HTTP methods for actions**

```
✅ DELETE /tweets/123
❌ POST /tweets/123/delete
```

**3. Return appropriate status codes**

```
200 - Success
201 - Created
204 - No Content (successful deletion)
400 - Bad Request
404 - Not Found
500 - Server Error
```

**4. Be consistent**

```
✅ /users, /tweets, /products (plural)
❌ /user, /tweet, /product (singular)
```

---

## 📦 TCP vs UDP

HTTP runs on top of lower-level **transport protocols**.

The two main ones:

- **TCP** - Transmission Control Protocol
- **UDP** - User Datagram Protocol

---

### Analogy: Sending a Package

**TCP = Registered Mail**

- Guaranteed delivery
- Confirms receipt
- Arrives in order
- Slower (waits for confirmation)

**UDP = Throwing letters over a fence**

- No guarantee of delivery
- No confirmation
- Might arrive out of order
- Faster (no waiting)

---

### TCP (Reliable)

**How it works:**

```
Client: "Sending packet 1"
Server: "Got packet 1, send next"
Client: "Sending packet 2"
Server: "Got packet 2, send next"
Client: "Sending packet 3"
Server: "Got packet 3, done"
```

**Characteristics:**

- ✅ Guaranteed delivery (resends if lost)
- ✅ Packets arrive in order
- ✅ Error checking
- ✅ Connection-oriented (handshake)
- ❌ Slower (overhead from confirmations)

**Used for:**

- Web browsing (HTTP/HTTPS)
- Email (SMTP, IMAP)
- File transfers (FTP)
- SSH
- Anything where data **must** be correct

---

### UDP (Fast but Unreliable)

**How it works:**

```
Client: "Sending packets 1, 2, 3, 4, 5..."
Server: (receives 1, 3, 5... packets 2 and 4 got lost)
Client: (doesn't care, keeps sending)
```

**Characteristics:**

- ✅ Very fast (no confirmations)
- ✅ Low latency
- ✅ Connectionless (no handshake)
- ❌ No guarantee of delivery
- ❌ Packets might arrive out of order
- ❌ No error checking

**Used for:**

- Video streaming (Netflix, YouTube)
- Online gaming
- Video calls (Zoom, Teams)
- DNS lookups
- Live broadcasts
- Anything where **speed > accuracy**

---

### Why UDP for Video Calls?

**Imagine you're on Zoom:**

With TCP:

```
Frame 100 lost → Wait... resending... now everything is delayed
→ Lag and stuttering
```

With UDP:

```
Frame 100 lost → Whatever, show frame 101, keep going
→ Smooth real-time experience
```

**Key insight:** Losing one video frame is okay. Waiting to resend it creates lag.

---

### When to Use What?

| Use Case | Protocol | Why |
| --- | --- | --- |
| Text messages | TCP | Accuracy matters |
| File download | TCP | Must be complete |
| Video call | UDP | Speed > accuracy |
| Online game | UDP | Real-time movement |
| Bank transfer | TCP | Cannot lose data |
| Live sports | UDP | Real-time is critical |

---

## ⚡ REST vs gRPC

**gRPC** = Modern alternative to REST for high-performance internal communication

---

### REST (Traditional)

```
POST /tweets HTTP/1.1
Content-Type: application/json

{
  "text": "Hello World",
  "userId": 42
}
```

**Characteristics:**

- Uses HTTP/1.1
- Data in JSON (text format)
- Human-readable
- Works in browsers
- Easy to debug with curl/Postman
- Widely understood

---

### gRPC (Modern)

**Define API in .proto file:**

```protobuf
service TweetService {
  rpc CreateTweet(TweetRequest) returns (TweetResponse);
}

message TweetRequest {
  string text = 1;
  int32 userId = 2;
}

message TweetResponse {
  int32 id = 1;
  string text = 2;
  int32 userId = 3;
}
```

**Characteristics:**

- Uses HTTP/2 (faster)
- Data in Protocol Buffers (binary)
- Much smaller and faster
- Not human-readable
- Requires special tools
- Auto-generates client code
- Strong type safety

---

### Comparison Table

| Feature | REST | gRPC |
| --- | --- | --- |
| Protocol | HTTP/1.1 | HTTP/2 |
| Format | JSON (text) | Protobuf (binary) |
| Speed | Slower | **Faster** |
| Size | Larger | **Smaller** |
| Human-readable | ✅ Yes | ❌ No |
| Browser support | ✅ Yes | ❌ Limited |
| Type safety | ❌ No | ✅ Yes |
| Streaming | ❌ Limited | ✅ Excellent |
| Learning curve | Easy | Steeper |

---

### When to Use What?

**Use REST when:**

- Building public APIs (Twitter API, GitHub API)
- Need browser support
- Team is familiar with REST
- Human-readable debugging is important
- Third-party integration

**Use gRPC when:**

- Microservices talking to each other (internal)
- Need high performance
- Strong typing is important
- Bi-directional streaming needed
- Language-agnostic client generation

**Real-world example:**

Google, Netflix, Uber use gRPC internally for microservices.

---

### Typical Architecture

```
Mobile/Web App
      ↓
    REST API
      ↓
  API Gateway
      ↓
     gRPC
      ↓
Microservices (talk to each other via gRPC)
```

- **External:** REST (easy for clients)
- **Internal:** gRPC (fast, efficient)

---

## 📝 Quick Reference

### HTTP Methods

```
GET    - Read/retrieve
POST   - Create
PUT    - Update (full replacement)
PATCH  - Update (partial)
DELETE - Remove
```

### HTTP Status Codes

```
2xx - Success
  200 OK
  201 Created
  204 No Content

4xx - Client Error
  400 Bad Request
  401 Unauthorized
  404 Not Found

5xx - Server Error
  500 Internal Server Error
  503 Service Unavailable
```

### REST URL Patterns

```
GET    /resources         - List all
GET    /resources/123     - Get one
POST   /resources         - Create
PUT    /resources/123     - Update
DELETE /resources/123     - Delete
```

---

## 🎯 Key Takeaways

✅ **HTTP/HTTPS** - Request/response model, HTTPS encrypts with session keys

✅ **GET vs POST** - GET for reading (data in URL), POST for writing (data in body)

✅ **REST** - Resources + HTTP methods, clean URLs, JSON format

✅ **TCP** - Reliable, ordered, slower (web, email, files)

✅ **UDP** - Fast, no guarantee (video, gaming, calls)

✅ **gRPC** - Modern, fast, binary (internal microservices)

---

## 🔗 Additional Resources

**REST API Design:**

- "RESTful API Design" by Microsoft Azure
- "REST API Tutorial" ([restfulapi.net](http://restfulapi.net))

**Networking:**

- "Computer Networking: A Top-Down Approach" book
- "How Does the Internet Work?" (Khan Academy)

**gRPC:**

- Official gRPC documentation ([grpc.io](http://grpc.io))
- "gRPC Crash Course" by Hussein Nasser (YouTube)

---