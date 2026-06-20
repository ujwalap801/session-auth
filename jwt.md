# JWT (JSON Web Token) - Complete Guide

## Table of Contents

1. Introduction
2. The Problem JWT Solves
3. Session-Based Authentication
4. Problems with Sessions
5. Enter JWT
6. What is JWT?
7. Structure of JWT
   - Header
   - Payload
   - Signature
8. Types of JWT
9. JWT Authentication Flow
10. Access Token vs Refresh Token
11. Security Mechanisms
12. JWT vs Session Authentication
13. Real World Usage
14. Key Takeaways

---

# 1. Introduction

JWT stands for:

```text
JSON Web Token
```

JWT is a compact and secure way of transferring information between parties.

It is primarily used for:

- Authentication
- Authorization
- Secure information exchange

---

# 2. The Problem JWT Solves

Before JWT, most applications used Session-Based Authentication.

Example:

```text
Client ── Login ──► Server
```

After successful login:

```text
Server creates session
Server stores session
Server sends Session ID
```

Every request:

```text
Client ── Session ID ──► Server
```

Server must look up session data.

---

# 3. Session-Based Authentication

## Flow

### Step 1

User logs in.

```text
Username + Password
```

### Step 2

Server validates credentials.

### Step 3

Server creates a session.

```text
SessionID = S123
```

### Step 4

Server stores:

```text
S123 → Alice
```

### Step 5

Browser stores:

```text
SESSIONID=S123
```

### Step 6

Future requests:

```text
Cookie: SESSIONID=S123
```

Server checks session store.

---

## Diagram

```text
Client
   |
   | Login
   v
Server
   |
   | Create Session
   v
Session Store

S123 → Alice
```

---

# 4. Problems with Sessions

## Problem 1: Scaling

Single server:

```text
Server
 └── Session Store
```

Works fine.

Multiple servers:

```text
Load Balancer
      |
 ┌────┴────┐
Server1  Server2
```

Now sessions must be shared.

Possible solutions:

- Redis
- Database

Extra complexity.

---

## Problem 2: Performance Overhead

Every request requires:

```text
Request
   ↓
DB / Redis Lookup
   ↓
Authentication
```

Extra network calls.

---

## Problem 3: Microservices

Suppose:

```text
Auth Service
Order Service
Payment Service
Profile Service
```

Every service needs session access.

Managing sessions becomes difficult.

---

## Problem 4: Mobile & APIs

Cookies are not always ideal.

Mobile apps often prefer token-based authentication.

---

# 5. Enter JWT

JWT solves these problems by being:

```text
Stateless
```

The server no longer needs to store session information.

Instead:

```text
Client carries identity information.
```

---

# 6. What is JWT?

JWT is a compact token format.

Structure:

```text
HEADER.PAYLOAD.SIGNATURE
```

Example:

```text
eyJhbGciOiJIUzI1NiIs...
```

Characteristics:

- Self-contained
- Stateless
- Signed
- Tamper-resistant

---

# 7. Structure of JWT

A JWT consists of three parts.

```text
HEADER.PAYLOAD.SIGNATURE
```

---

## Part 1: Header

Example:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Meaning:

### alg

Algorithm used for signing.

Examples:

```text
HS256
RS256
ES256
```

### typ

Token type.

```text
JWT
```

---

### Header Encoded

```text
Base64Url(Header)
```

---

## Part 2: Payload

Payload contains claims.

Example:

```json
{
  "userId": 123,
  "name": "Alice",
  "role": "admin",
  "exp": 1762389012
}
```

---

### Common Claims

#### sub

Subject

```json
{
  "sub": "123"
}
```

#### name

```json
{
  "name": "Alice"
}
```

#### role

```json
{
  "role": "admin"
}
```

#### exp

Expiration time

```json
{
  "exp": 1762389012
}
```

---

### Important

Payload is NOT encrypted.

Anyone can decode it.

Never store:

- Passwords
- Credit cards
- Secrets

Inside payload.

---

## Part 3: Signature

Generated using:

```text
Header
+
Payload
+
Secret Key
```

Example:

```text
HMACSHA256(
  Base64Url(Header)
  +
  "."
  +
  Base64Url(Payload),
  Secret
)
```

---

### Why Signature?

It ensures:

- Token not modified
- Token created by trusted server

---

# 8. Types of JWT

## 1. JWS (JSON Web Signature)

Most common.

```text
Signed
Not Encrypted
```

Anyone can read payload.

Cannot modify payload.

---

## 2. JWE (JSON Web Encryption)

```text
Encrypted
```

Used when payload contains sensitive information.

---

# 9. JWT Authentication Flow

## Step 1

User logs in.

```text
Username
Password
```

---

## Step 2

Server validates credentials.

---

## Step 3

Server generates JWT.

Example payload:

```json
{
  "userId": 123,
  "role": "admin"
}
```

---

## Step 4

Server sends token.

```json
{
  "token": "eyJhbG..."
}
```

---

## Step 5

Client stores token.

Usually:

- HttpOnly Cookie
- Local Storage
- Session Storage

Preferred:

```text
HttpOnly Cookie
```

---

## Step 6

Client sends token.

```http
Authorization: Bearer eyJhbG...
```

---

## Step 7

Server verifies signature.

If valid:

```text
Authenticated
```

No database lookup needed.

---

## Diagram

```text
Client
   |
   | Login
   v
Server
   |
   | Create JWT
   v
Client

Future Requests

Client
   |
   | JWT
   v
Server

Verify Signature
Authenticate User
```

---

# 10. Access Token vs Refresh Token

Modern systems usually use both.

---

## Access Token

Short-lived.

Example:

```text
15 minutes
```

Used for API access.

---

## Refresh Token

Long-lived.

Example:

```text
7 days
30 days
```

Used to obtain a new access token.

---

## Flow

```text
Login
  |
  ├── Access Token
  └── Refresh Token
```

Access token expires.

```text
Client
   |
   | Refresh Token
   v
Server

New Access Token
```

---

# 11. Security Mechanisms

## 1. HTTPS

Always use:

```text
HTTPS
```

Without HTTPS:

```text
Token can be stolen.
```

---

## 2. Short Expiration

Good:

```text
15 minutes
```

Bad:

```text
30 days
```

---

## 3. Strong Secret Key

Example:

```text
JWT_SECRET=
s8D7#kL9@vQ!x2...
```

---

## 4. HttpOnly Cookies

Prevents JavaScript access.

Helps reduce XSS attacks.

---

## 5. Refresh Token Rotation

Issue a new refresh token every time.

Improves security.

---

# 12. JWT vs Session Authentication

| Feature | Session | JWT |
|----------|----------|------|
| Server Stores State | Yes | No |
| Stateful | Yes | No |
| DB Lookup | Usually Yes | No |
| Scalability | Lower | Higher |
| Logout | Easy | Harder |
| Microservices | Harder | Easier |
| Mobile Apps | Less Common | Excellent |
| APIs | Less Suitable | Excellent |

---

# 13. Real World Usage

## Use JWT For

- REST APIs
- React Applications
- Angular Applications
- Vue Applications
- Mobile Apps
- Microservices

---

## Use Sessions For

- Traditional websites
- Banking systems
- Internal admin portals

---

# 14. Key Takeaways

JWT stands for:

```text
JSON Web Token
```

JWT structure:

```text
HEADER.PAYLOAD.SIGNATURE
```

Payload:

```text
Readable
Not Encrypted
```

Signature:

```text
Protects against tampering
```

JWT is:

```text
Stateless
```

Benefits:

- No session storage
- Better scalability
- Great for APIs
- Great for microservices

Memory Trick:

```text
Session = Server remembers you.

JWT = You carry proof of who you are.
```