# JWT Verification in Distributed Systems

## System Architecture

Suppose we have:

```text
            Client
               |
               ▼
         Load Balancer
       /      |      \
      ▼       ▼       ▼
  Server1  Server2  Server3
```

A user logs in through **Server1**.

The next request may go to any server.

Question:

**How do all servers verify the JWT?**

---

# Case 1: Symmetric Key (HS256)

In HS256:

All servers share the same secret key.

```text
SECRET_KEY = "mySuperSecret"
```

Used for:

- Signing JWT
- Verifying JWT

---

## Login Flow

### Step 1

Client logs in.

```text
Client
  |
username/password
  |
  ▼
Server1
```

---

### Step 2

Server1 generates JWT.

Payload:

```json
{
  "sub": "42",
  "role": "USER",
  "exp": 1710000900
}
```

Signature:

```text
HMACSHA256(
    header.payload,
    SECRET_KEY
)
```

JWT:

```text
header.payload.signature
```

Server sends JWT.

---

## API Request

Suppose next request goes to **Server3**.

```text
Client
   |
Bearer JWT
   |
   ▼
Server3
```

Server3 also has:

```text
SECRET_KEY = "mySuperSecret"
```

It computes:

```text
HMACSHA256(
    header.payload,
    SECRET_KEY
)
```

Then compares signatures.

If equal:

```text
Authenticated
```

---

## HS256 Architecture

```text
                 SECRET_KEY
                     |
        -----------------------------
        |             |             |
        ▼             ▼             ▼
    Server1       Server2       Server3
      Sign          Verify        Verify
```

---

## Advantages

### Fast

Only HMAC calculation.

No DB calls.

No external service.

---

### Simple

Easy to implement.

---

## Problems

### Key Distribution

Every server needs:

```text
SECRET_KEY
```

If you have:

```text
100 microservices
```

Then:

```text
100 copies of SECRET_KEY
```

exist.

---

### Security Risk

If one service is compromised:

Attacker steals:

```text
SECRET_KEY
```

Now attacker can generate valid JWTs.

Example:

```json
{
  "sub": "1",
  "role": "ADMIN"
}
```

Sign it.

All servers trust it.

Very dangerous.

---

# Case 2: Asymmetric Keys (RS256)

This is what large systems prefer.

Uses:

- Private Key
- Public Key

Example:

```text
Private Key → Auth Service only
Public Key  → Everyone else
```

---

## Architecture

```text
                 Auth Server
                 (Private Key)
                      |
                Signs JWT
                      |
                      ▼

          -----------------------
          |         |          |
          ▼         ▼          ▼
      Server1   Server2    Server3
      Public    Public     Public
       Key       Key        Key
```

---

## Login Flow

Client sends:

```text
username/password
```

↓

Auth Server verifies.

↓

Auth Server signs JWT using:

```text
PRIVATE KEY
```

Example:

```text
RSASHA256(
    header.payload,
    PRIVATE_KEY
)
```

JWT issued.

---

## API Verification

Suppose request reaches **Server2**.

```text
Bearer JWT
```

Server2 verifies using:

```text
PUBLIC KEY
```

If valid:

```text
Authenticated
```

---

# Important Difference

## Symmetric

```text
Sign   = SECRET
Verify = SECRET
```

---

## Asymmetric

```text
Sign   = PRIVATE KEY
Verify = PUBLIC KEY
```

---

# Why is RS256 better?

Suppose Payment Service gets hacked.

---

## HS256

Payment Service has:

```text
SECRET_KEY
```

Attacker can:

✅ Create JWTs

Entire system compromised.

---

## RS256

Payment Service only has:

```text
PUBLIC KEY
```

Attacker can:

✅ Verify JWTs

but CANNOT:

❌ Generate JWTs

because they don't have:

```text
PRIVATE KEY
```

Much safer.

---

# Real Example (University Analogy)

## Symmetric

Everyone has the principal’s stamp.

- Teacher A → can issue certificates
- Teacher B → can issue certificates
- Teacher C → can issue certificates

Anyone can forge degrees.

---

## Asymmetric

Only principal has the official stamp.

Teachers only have sample copies.

- Principal → Issues degree
- Teachers → Verify authenticity

Teachers cannot create degrees.

---

# Public Key Distribution

How do services get the public key?

Auth server exposes:

```text
/.well-known/jwks.json
```

JWKS = **JSON Web Key Set**

Flow:

```text
Auth Server
      |
      ▼
JWKS Endpoint
      |
      ▼
Microservices download public keys
```

They cache them.

---

# Key Rotation

Suppose private key leaks.

Generate:

```text
Old Key Pair
↓
New Key Pair
```

JWT header contains:

```json
{
  "alg": "RS256",
  "kid": "key-2026"
}
```

`kid` means:

Which public key should be used.

Services fetch matching key from JWKS.

This enables seamless key rotation.

---

# System Design Comparison

| Feature                  | HS256 (Symmetric)         | RS256 (Asymmetric)                  |
| ------------------------ | ------------------------- | ----------------------------------- |
| Sign                     | Secret Key                | Private Key                         |
| Verify                   | Secret Key                | Public Key                          |
| Services can issue JWTs  | Yes                       | No                                  |
| Services can verify JWTs | Yes                       | Yes                                 |
| Key compromise impact    | Entire system compromised | Only Auth Server critical           |
| Performance              | Faster                    | Slightly slower                     |
| Best for                 | Small internal systems    | Large microservice architectures    |
| Used by                  | Small apps                | OAuth providers, enterprise systems |

---

# What Companies Use?

## Small Monolith Apps

HS256 is common.

---

## Microservices

RS256 / ES256 is preferred.

Used by:

- Auth0
- Okta
- Google
- Microsoft

Reason:

Thousands of services can verify tokens without ever having the private signing key.

---

# Interview Answer (30 seconds)

> With HS256, all services share the same secret key, so any service can both sign and verify JWTs. It’s simple and fast but risky because compromising one service compromises the whole system. With RS256, only the authentication service holds the private key and signs tokens, while all other services verify using the public key. This is more secure and is the preferred approach for distributed microservice architectures.
