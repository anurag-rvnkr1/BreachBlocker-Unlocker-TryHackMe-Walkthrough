---
title: "BreachBlocker Unlocker — TryHackMe | Security Assessment"
description: "Portfolio-grade assessment documenting staged key recovery, service enumeration, source disclosure, timing side-channel credential recovery, SMTP interception and banking authorization bypass."
---

# BreachBlocker Unlocker — Security Assessment Walkthrough

> **Platform:** TryHackMe  
> **Room:** BreachBlocker Unlocker  
> **Difficulty:** Hard  
> **Focus:** Web Security · Authentication · Source Disclosure · Timing Side Channel · SMTP

## Executive Summary

BreachBlocker Unlocker is a chained security assessment in which access to a high-value banking workflow depends on information recovered from an earlier challenge.

The investigation crosses multiple trust boundaries:

```text
AOC Key Recovery
      ↓
Service Enumeration
      ↓
Mobile Portal Recon
      ↓
Source / Configuration Disclosure
      ↓
Credential Database Analysis
      ↓
Timing Side-Channel Recovery
      ↓
Hopflix Authentication
      ↓
Hopsec Bank Authentication
      ↓
OTP Recipient Validation Weakness
      ↓
SMTP Interception
      ↓
Charity-Fund Authorization
```

The most important lesson is architectural: secure components can still fail when information disclosure, authentication logic, and message-delivery trust assumptions interact.

> **Public release:** final challenge flags and challenge-only secrets are intentionally redacted.

---

## 1. Scope & Safety

All activity described here is limited to the intentionally vulnerable TryHackMe environment. Credentials, machine addresses, and challenge secrets are treated as lab-only material.

---

## 2. Recovering the Initial Access Key

The first stage requires recovering an access key from the preceding Advent of Cyber material.

The HTA file is treated as a code artifact and inspected without executing it. The embedded payload is normalized and Base64-decoded until the remaining encrypted data is isolated.

Example normalization:

```bash
sed ':a;N;$!ba;s/[&_"]//g;s/[[:space:]]//g' NorthPolePerformanceReview.hta   | base64 -d > b64.txt
```

After the second Base64 layer, the resulting bytes are processed in CyberChef:

```text
XOR
Key: 23
Type: Decimal
Scheme: Standard

→ Render Image
```

![CyberChef key recovery](../docs/assets/01-cyberchef-key-recovery.png)

This yields the challenge's SQ4 access key.

**The actual key is intentionally omitted from the public portfolio.**

---

## 3. Network Service Enumeration

After unlocking the target, begin with a focused Nmap scan:

```bash
nmap -p22,25,8443,21337 -sCV TARGET -oN fullscan.tcp
```

The identified surface is:

| Port | Service | Relevance |
|---|---|---|
| 22 | SSH | Remote administrative access |
| 25 | Postfix SMTP | Potential account and delivery surface |
| 8443 | nginx HTTPS | Mobile application |
| 21337 | Werkzeug/Python | Additional web application |

![Nmap results](../docs/assets/02-nmap-service-enumeration.png)

The coexistence of HTTPS, SMTP and a Python application suggests that the challenge may span several application layers rather than one endpoint.

---

## 4. SMTP Enumeration

Metasploit can quickly inspect the mail service:

```text
auxiliary/scanner/smtp/smtp_enum
auxiliary/scanner/smtp/smtp_version
```

![SMTP enumeration](../docs/assets/03-smtp-enumeration.png)

No immediate credential breakthrough is obtained, so the investigation moves to the mobile portal.

---

## 5. Mobile Portal Reconnaissance

The HTTPS service hosts a mobile-style portal with several applications.

The Hopflix login interface exposes a pre-populated account identifier:

![Hopflix login](../docs/assets/04-hopflix-login-surface.png)

Its inbox provides additional context:

![Hopflix inbox](../docs/assets/05-hopflix-inbox.png)

The messages establish that the target identity is associated with the banking workflow and that protected charity funds are waiting behind further authentication.

---

## 6. Client-Side Source Inspection

A browser-delivered application must be considered inspectable by the user.

Reviewing the mobile source reveals additional security-relevant information:

![Mobile source review](../docs/assets/06-mobile-source-review.png)

The JavaScript contains a hard-coded value used by the authenticator flow:

![Client-side secret](../docs/assets/07-app-javascript-secret.png)

### Security significance

A browser-accessible secret is not a secret. Any security control that can be recovered from client-side code should be assumed compromised.

---

## 7. Bank Login Reconnaissance

The banking application exposes a login endpoint.

Capturing the request in Burp Suite provides the exact request structure:

![Burp request](../docs/assets/08-burp-bank-login.png)

Direct account-number enumeration does not immediately produce a result, so the application architecture is investigated further.

---

## 8. Discovering Hidden Server Configuration

Content discovery against the HTTPS application reveals an exposed nginx configuration:

```bash
ffuf -u https://TARGET:8443/FUZZ      -w /opt/SecLists/Discovery/Web-Content/quickhits.txt
```

![ffuf results](../docs/assets/09-fuzz-nginx-config.png)

The configuration exposes a routing pattern in which `try_files` falls back into the Python application through uWSGI:

![nginx / uWSGI routing](../docs/assets/10-nginx-uwsgi-routing.png)

This is a major pivot because it suggests that server-side application files may be reachable through the web layer.

---

## 9. Python Source Disclosure

Testing common Python entry points reveals the primary application source.

The disclosed code contains:

- database paths;
- route definitions;
- authentication logic;
- security-sensitive values;
- challenge flags.

![Application source disclosure](../docs/assets/11-main-py-credential-disclosure.png)

The presence of secrets and database references inside web-reachable source represents a critical information-disclosure issue.

> Challenge flags are redacted from this portfolio.

---

## 10. SQLite Credential Analysis

The disclosed application reveals the location of a Hopflix SQLite database.

After recovering the database, inspect its structure with SQLite tooling.

The target account contains a very long password representation. The code explains why: every character is independently transformed using repeated SHA-1 operations.

The structure is a strong clue:

```text
SHA-1 output = 40 hexadecimal characters
```

Therefore:

```text
40 × 12 characters
≈ 480 hexadecimal characters
```

This reveals the expected password length without reversing the entire value.

---

## 11. Password Verification Logic

The authentication routine first validates the total password length and then processes each character sequentially.

![Password checking logic](../docs/assets/12-password-checking-logic.png)

Because the server performs expensive processing for each correct character, the response time becomes correlated with how much of the candidate prefix is correct.

Conceptually:

```text
correct prefix
      ↓
more server-side work
      ↓
measurably slower response
```

This turns authentication into a timing side channel.

---

## 12. Timing-Based Credential Recovery

The practical attack is statistical rather than cryptographic reversal.

For each position:

1. keep the already-recovered prefix;
2. test each candidate character;
3. submit several repeated requests;
4. calculate average latency;
5. retain the strongest timing candidate;
6. move to the next position.

Representative pseudocode:

```python
for position in range(secret_length):
    measurements = {}

    for candidate in alphabet:
        guess = prefix + candidate + filler
        samples = [probe(guess) for _ in range(REPETITIONS)]
        measurements[candidate] = mean(samples)

    prefix += max(measurements, key=measurements.get)
```

Repeated measurements are important because network jitter can otherwise create false positives.

The AttackBox is preferable for this phase because VPN latency can make the timing signal significantly noisier.

---

## 13. Hopflix Authentication

After recovering the credential, the login succeeds:

![Successful Hopflix login](../docs/assets/13-hopflix-authentication-success.png)

This becomes the pivot into the banking application.

---

## 14. Hopsec Bank — OTP Workflow

The bank asks which authorized email address should receive the OTP:

![Authorized email selection](../docs/assets/14-bank-authorized-email-selection.png)

The next stage requests a six-digit code:

![2FA prompt](../docs/assets/15-bank-two-factor-prompt.png)

At first glance, the OTP appears secure because it is randomly generated.

That shifts the investigation to the **delivery path**, not the generation algorithm.

---

## 15. OTP Generation Analysis

Source inspection confirms that the application generates a six-digit random value:

![OTP generation code](../docs/assets/16-otp-generation-logic.png)

The random generator therefore does not need to be defeated.

The actual weakness is the recipient-validation logic.

A simplified form is:

```python
if domain not in allowed_domains and to_addr not in allowed_emails:
    return -1
```

This condition does not require both checks to succeed.

The application also interprets the recipient address differently from the downstream SMTP delivery behavior.

---

## 16. OTP Delivery-Path Abuse

The challenge permits a specially structured recipient that appears to satisfy the application's domain test while causing SMTP delivery toward an attacker-controlled destination.

The general security pattern is:

```text
Application validation
        ↓
one interpretation of recipient
        ↓
SMTP parser
        ↓
different delivery interpretation
```

This is a **parser / validation discrepancy**.

A controlled SMTP listener is used to receive the resulting message in the lab.

```bash
aiosmtpd -n -l 0.0.0.0:25
```

The resulting message contains the requested OTP:

![Captured OTP](../docs/assets/17-smtp-otp-capture.png)

The important lesson is that perfect OTP randomness does not protect an insecure delivery channel.

---

## 17. Banking Authentication

With the captured OTP, the second authentication stage is completed.

The resulting dashboard exposes the protected charity-fund operation:

![Bank dashboard](../docs/assets/18-bank-dashboard.png)

At this point the assessment has crossed the intended authorization boundary.

---

## 18. Final Impact

The charity funds can now be released:

![Funds released](../docs/assets/19-charity-funds-released.png)

The final challenge result is shown below with the answer intentionally redacted:

![Final result](../docs/assets/20-final-result-redacted.png)

```text
THM{REDACTED_FOR_PUBLICATION}
```

---

# 19. End-to-End Attack Chain

```text
Previous AOC Artifact
        ↓
Recover SQ4 Key
        ↓
Nmap Service Enumeration
        ↓
SMTP + HTTPS Recon
        ↓
Hopflix Account Discovery
        ↓
Source / JS Inspection
        ↓
nginx Configuration Disclosure
        ↓
Python Source Exposure
        ↓
SQLite Credential Extraction
        ↓
Timing Side-Channel Analysis
        ↓
Hopflix Authentication
        ↓
Hopsec Bank Login
        ↓
OTP Recipient Validation Weakness
        ↓
SMTP Delivery Manipulation
        ↓
OTP Capture
        ↓
Bank Authentication
        ↓
Charity-Fund Authorization
```

---

# 20. Findings & Severity

| Finding | Severity | Impact |
|---|---:|---|
| Web-accessible application source | Critical | Exposes security logic, paths and secrets |
| OTP recipient validation weakness | Critical | Enables OTP interception |
| Timing side channel | High | Enables credential recovery |
| Exposed nginx configuration | High | Reveals architecture and application routing |
| Client-side authentication secret | High | Weakens authenticator protection |
| Cross-service credential reuse | High | Enables lateral application pivot |

---

# 21. Root Cause Analysis

## Source Exposure

Production source and configuration are reachable through the web layer.

**Root cause:** unsafe file-resolution/routing behavior and inadequate source protection.

## Timing Side Channel

The credential comparison leaks information through processing time.

**Root cause:** secret-dependent execution and per-character expensive hashing.

## OTP Recipient Validation

Recipient validation does not establish a single canonical interpretation before handing the address to SMTP.

**Root cause:** weak Boolean logic combined with parser inconsistency.

## Client-Side Secret

An authentication-control value is distributed to the browser.

**Root cause:** treating client-delivered code as a secure secret container.

## Cross-Application Trust

The same credentials can be reused across distinct application components.

**Root cause:** insufficient identity segmentation.

---

# 22. Remediation

### Protect application source

Never expose Python, nginx, database files or configuration through a production web route.

### Remove security secrets from client code

Anything sent to the browser must be considered recoverable.

### Use password KDFs

Replace custom repeated SHA-1 processing with a dedicated password-hashing KDF such as Argon2id, scrypt or bcrypt.

### Use constant-time verification

Avoid timing differences that reveal secret prefixes.

### Canonicalize email addresses

Parse, normalize and validate the recipient once before sending it. The exact normalized value should be used by the downstream SMTP component.

### Strictly allowlist OTP recipients

Use explicit recipient identities instead of loose domain matching.

### Separate application credentials

Do not rely on reusable passwords across independent applications.

### Protect configuration and disable debug behavior

Sensitive server details should not be disclosed through production endpoints or error messages.

---

# 23. Lessons Learned

**Enumerate the full surface.** The breakthrough required combining HTTPS, SMTP, source, database and timing observations.

**Treat source as evidence.** The application code revealed the real security model more accurately than the UI.

**Follow the data, not the feature label.** The OTP generator was not the weakness; the delivery path was.

**Timing attacks are measurement problems.** Repetition, statistical ranking and low-noise execution environments are essential.

**Parser differences are security boundaries.** A validation rule is not safe when the next component interprets the same input differently.

---

# 24. Conclusion

BreachBlocker Unlocker is a strong example of chained web exploitation where no single flaw explains the full compromise.

The practical methodology is:

**recover → enumerate → inspect → correlate → measure → exploit → validate**

The challenge combines web enumeration, source disclosure, database investigation, timing analysis, authentication testing and SMTP behavior into one cohesive assessment.

The broader security lesson is architectural: **authentication is only as strong as every component involved in generating, validating, transporting and consuming its security decisions.**

---

**Author:** Anurag Revankar  
**Purpose:** Authorized TryHackMe research and cybersecurity portfolio
