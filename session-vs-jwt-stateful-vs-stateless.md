# Session vs JWT: Stateful vs Stateless Authentication

## The Core Question

At first glance, both seem the same:

### Session

```text
Client → SessionID
```

### JWT

```text
Client → JWT
```

So why do we call one **stateful** and the other **stateless**?

The difference is **not** whether the client sends something.

The real difference is:

> **Where is the authentication state stored?**

---

# Session Authentication (Stateful)

Suppose Alice logs in.

Server creates:

```text
SessionID = S123
```

Server stores:

```text
Session Store

S123 → Alice
Role → USER
Cart → [Book]
Preferences → Dark Mode
```

Browser stores only:

```text
Cookie: SESSIONID=S123
```

---

## Request Flow

Alice makes a request:

```text
Cookie: SESSIONID=S123
```

Server does:

```text
Lookup S123
   ↓
Find Alice
   ↓
Authenticate
```

---

## What if server deletes the session?

Suppose server removes:

```text
S123 → Alice
```

Now browser still sends:

```text
Cookie: SESSIONID=S123
```

Server responds:

> "I don't know who this is."

Why?

Because the **real authentication state lives on the server**.

Without server memory:

```text
S123
```

is meaningless.

---

# JWT Authentication (Stateless)

Server generates:

```json
{
  "sub": "42",
  "role": "USER",
  "exp": "..."
}
```

Signs it:

```text
JWT = header.payload.signature
```

Client stores the JWT.

Request:

```text
Authorization: Bearer JWT
```

Server verifies:

```text
Signature ✓
Expiry ✓
```

Extracts:

```text
User = Alice
Role = USER
```

---

## What if server restarts?

Did the server lose anything?

**No.**

Why?

Because it never stored:

```text
JWT123 → Alice
```

The token itself contains everything.

---

# The Real Difference

Think of it like this.

---

# Session ID = Locker Key

You go to a gym.

The receptionist stores your bag.

Storage:

```text
Locker #25 → Alice's Bag
```

You receive:

```text
Key #25
```

Later:

```text
You → Key #25
Receptionist → Finds your bag
```

Important:

The key itself means nothing without the receptionist’s storage.

This is:

> **Stateful**

---

# JWT = Signed Certificate

Suppose the government gives you a driver's license.

The license itself says:

```text
Name: Alice
Age: 25
Valid Until: 2030
Official Signature
```

Police officer checks:

```text
Verify signature
   ↓
Read details
   ↓
Done
```

No central lookup needed every time.

The proof travels with you.

This is:

> **Stateless**

---

# Why Session is Stateful

Session cookie only contains:

```text
SESSIONID=S123
```

Server still needs:

```text
S123 → Alice
```

Without that:

```text
S123
```

is useless.

So:

> Server must maintain state between requests.

---

# Why JWT is Stateless

JWT itself contains:

```text
JWT
 ↓
sub = 42
role = USER
exp = ...
signature
```

Server does **not** need:

```text
JWT123 → Alice
```

stored anywhere.

Each request is self-contained.

So:

> Server can process each request independently.

---

# The Subtle Point

Many people think:

> “Stateful means client sends nothing.”

That is incorrect.

Both Session and JWT send something.

Correct definition:

## Stateful

```text
Server must maintain state between requests
```

## Stateless

```text
Server can process requests independently
without stored client context
```

---

# Side-by-Side Comparison

| Feature                  | Session               | JWT        |
| ------------------------ | --------------------- | ---------- |
| Client stores            | Session ID            | Full Token |
| Server stores auth state | Yes                   | No         |
| Requires DB/cache lookup | Yes                   | No         |
| Survives server restart  | No (unless persisted) | Yes        |
| Easy logout              | Yes                   | Harder     |
| Horizontal scaling       | Harder                | Easier     |
| Revocation               | Easy                  | Harder     |

---

# One-Line Interview Answer

> In session-based authentication, the client sends only a session identifier, and the server must look up the associated user state from its session store, making it stateful. In JWT authentication, the client sends the complete signed authentication context in the token itself, allowing the server to authenticate the request without maintaining server-side session state, making it stateless.
