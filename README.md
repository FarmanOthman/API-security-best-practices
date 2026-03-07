# API-security-best-practices

This prioritized to-do list converts API security best practices into actionable tasks for an engineering team, ranked from highest-risk vulnerabilities to long-term operational security.

---

# 1. Authentication & Authorization

### Endpoint Authentication

* **Audit Endpoint Protection**
  Verify that all endpoints require authentication unless explicitly intended to be public.

* **Use Standard Authentication Protocols**
  Deprecate **Basic Authentication**. Use:

  * OAuth 2.0
  * OpenID Connect
  * JWT-based authentication

---

### Identity Security

* **Enforce Password Entropy**

  * Minimum 12–14 characters
  * Require uppercase, lowercase, numbers, and symbols
  * Prevent password reuse
  * Use adaptive hashing (`bcrypt`, `argon2`)

* **Implement Brute-Force Protections**

  * Max retry limits
  * Temporary account lockouts
  * CAPTCHA after repeated failures
  * IP-based rate limiting

---

### JWT Hardening

* Use **strong JWT secrets (≥256-bit entropy)**.
* Do **not trust the `alg` header** — validate it on the server.
* Minimize token lifetime:

  * Access token TTL: **5–15 minutes**
  * Refresh token rotation
* Keep payloads minimal and **exclude sensitive data**.
* Use **`jti` claims** for token tracking and replay detection.

---

### Replay Attack Protection

* Implement **nonce or timestamp validation** for sensitive operations.
* Use **`jti` claim tracking** to detect reused tokens.
* Reject tokens outside acceptable clock skew.

---

### OAuth Security

* Validate `redirect_uri` server-side.
* Use `response_type=code` instead of `token`.
* Enforce `state` parameter for CSRF protection.
* Prefer **PKCE** for public clients.

---

# 2. Access Control & Network Security

### Transport Security

* Enforce **HTTPS only**.
* Disable weak TLS versions (TLS 1.0 / 1.1).
* Use strong ciphers.
* Enable **HSTS** header.

---

### Cross-Origin Security

* **Lock down CORS configuration**

  * Explicit `Access-Control-Allow-Origin`
  * Never use `*` for authenticated APIs
  * Restrict allowed headers and methods

---

### API Exposure Control

* Restrict **private APIs to allowlisted IP ranges**.
* Use **mutual TLS (mTLS)** for internal services.

---

### Gateway-Level Security

* Deploy an **API Gateway or WAF** to centralize:

  * Authentication
  * Rate limiting
  * Request filtering
  * IP allowlists
  * Bot protection

Examples:

* Kong
* AWS API Gateway
* NGINX
* Cloudflare

---

### Rate Limiting

Apply request throttling to prevent:

* brute force
* scraping
* DDoS

Typical limits:

```
100 requests/min per user
1000 requests/min per IP
```

---

### Disable Directory Listing

Ensure directory browsing is disabled on all servers.

---

# 3. Processing & Input Validation

### Resource Identifier Security

* Avoid sequential IDs (`/users/242`)
* Use **UUIDs or ULIDs**
* Always validate **object ownership** to prevent **BOLA attacks**.

---

### Input Validation

Validate:

* request schema
* allowed characters
* expected formats

Reject unexpected fields.

---

### Content-Type Enforcement

Only accept approved payload formats:

```
Content-Type: application/json
```

Reject:

```
text/plain
application/xml
multipart/*
```

Unless explicitly required.

---

### Request Size Limits

Prevent memory exhaustion attacks.

Example limits:

```
JSON body: 1MB
File upload: 10MB
Headers: 8KB
```

Reject requests exceeding limits.

---

### Injection & XXE Protection

* Disable XML entity expansion.
* Disable YAML unsafe parsing.
* Use parameterized queries.

---

### URL Data Sanitization

Never place sensitive data in:

```
Query parameters
URLs
GET requests
```

Sensitive examples:

```
?token=
?apikey=
?password=
```

These leak via:

* browser history
* reverse proxy logs
* referer headers

---

### Production Environment Lockdown

* Disable debug mode
* Disable stack traces in responses
* Use non-executable stacks where possible

---

### Large Data Handling

* Offload uploads to **CDN / object storage**
* Use async processing
* Avoid blocking requests

---

# 4. Output Security & Data Protection

### Secure Response Headers

Set the following headers:

```
X-Content-Type-Options: nosniff
X-Frame-Options: deny
Content-Security-Policy: default-src 'none'
Referrer-Policy: no-referrer
Strict-Transport-Security
```

---

### Remove Fingerprinting Headers

Strip technology identifiers:

```
x-powered-by
server
framework headers
```

---

### Payload Cleansing

Ensure responses never expose:

* access tokens
* refresh tokens
* API keys
* passwords
* stack traces

---

### Content-Type Control

Force response types:

```
Content-Type: application/json
```

---

### Correct HTTP Status Codes

Examples:

```
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
429 Too Many Requests
500 Internal Server Error
```

---

# 5. API Lifecycle Management

### API Versioning Strategy

Implement versioning to maintain backward compatibility.

Options:

```
/v1/users
/v2/users
```

or

```
Accept: application/vnd.api.v2+json
```

---

### Deprecation Policy

Define a **Sunset lifecycle**:

Example:

```
v1 deprecated → support for 6 months
v1 removed → after 12 months
```

Include deprecation headers:

```
Sunset: <date>
Deprecation: true
```

---

### Legacy Endpoint Retirement

Remove:

* unused endpoints
* insecure legacy versions
* deprecated authentication flows

Old APIs are a major **BOLA attack vector**.

---

# 6. Cryptographic & Secret Management

### Centralized Secret Storage

Never store secrets in:

```
.env files
source code
configuration files
```

Use secret management systems:

* HashiCorp Vault
* AWS Secrets Manager
* Azure Key Vault
* GCP Secret Manager

---

### Secret Rotation Policy

Rotate regularly:

| Secret Type          | Rotation   |
| -------------------- | ---------- |
| JWT secrets          | 30–90 days |
| API keys             | 60–90 days |
| DB credentials       | 90 days    |
| Encryption root keys | yearly     |

---

### Secure Key Distribution

Use **short-lived credentials** whenever possible.

Avoid long-lived tokens.

---

# 7. CI/CD Security & Maintenance

### Automated Security Scanning

Integrate into CI pipeline:

* dependency scanning
* SAST
* container scanning
* secret detection

Examples:

* Snyk
* Dependabot
* Trivy
* Semgrep

---

### Secure Code Review Policy

* Mandatory peer review
* No self-approval merges
* Security review for auth logic

---

### Security Testing

Include automated tests for:

* authentication failures
* access control bypass
* input validation
* rate limits

---

### Deployment Rollback

Every deployment must support **immediate rollback**.

---

# 8. Monitoring & Incident Detection

### Centralized Logging

Aggregate logs using:

* ELK stack
* Datadog
* Splunk

Ensure logs **do not contain sensitive data**.

---

### Real-Time API Monitoring

Track:

* request rate
* error rates
* latency
* unusual activity patterns

---

### Intrusion Detection

Deploy IDS / IPS systems.

Generate alerts to:

* Slack
* SMS
* PagerDuty
* SIEM platforms

---

# 9. Defense in Depth Strategy

Validate that security controls **overlap** so that failure of one layer does not expose the API.

Example layered model:

```
WAF
↓
API Gateway
↓
Authentication
↓
Authorization
↓
Input Validation
↓
Business Logic Validation
```

---

# 10. Security Documentation

### Security Assumptions

Document the guarantees for each endpoint.

Example:

```
Endpoint: POST /payments

Assumptions:
- Request passed WAF validation
- JWT verified by gateway
- User authenticated
```

---

### Threat Model Documentation

Maintain documentation for:

* attack surfaces
* authentication flow
* sensitive endpoints
* trust boundaries

---

# Result

Your checklist now covers **three layers of API security**:

| Layer                   | Covered |
| ----------------------- | ------- |
| Code security           | ✔       |
| Infrastructure security | ✔       |
| Lifecycle security      | ✔       |
