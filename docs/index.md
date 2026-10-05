---
layout: default
title: "BreachBlocker Unlocker — TryHackMe"
permalink: /
---

# 🛡️ BreachBlocker Unlocker

## TryHackMe — Hard Difficulty

A premium portfolio walkthrough documenting a chained web-security assessment involving source disclosure, timing-side-channel credential recovery, SMTP/OTP delivery abuse and banking authorization.

| Property | Value |
|---|---|
| Platform | TryHackMe |
| Room | BreachBlocker Unlocker |
| Difficulty | Hard |
| Focus | Web Security / Authentication |
| Core Techniques | Source Disclosure · Timing Attack · SMTP |
| Outcome | Charity-fund authorization bypass |

## Attack Chain

```text
Key Recovery
 ↓
Nmap
 ↓
HTTPS / SMTP Recon
 ↓
Source Disclosure
 ↓
Timing Side Channel
 ↓
Credential Recovery
 ↓
Bank Pivot
 ↓
OTP Recipient Validation Weakness
 ↓
SMTP Interception
 ↓
Authorization
```

## Evidence

### 01 — Key Recovery

![CyberChef](assets/01-cyberchef-key-recovery.png)

### 02 — Service Enumeration

![Nmap](assets/02-nmap-service-enumeration.png)

### 03 — SMTP

![SMTP enumeration](assets/03-smtp-enumeration.png)

### 04–05 — Hopflix Intelligence

![Hopflix](assets/04-hopflix-login-surface.png)

![Inbox](assets/05-hopflix-inbox.png)

### 06–07 — Source & Client Secret

![Source](assets/06-mobile-source-review.png)

![Secret](assets/07-app-javascript-secret.png)

### 08–10 — Bank Recon & Routing

![Burp](assets/08-burp-bank-login.png)

![ffuf](assets/09-fuzz-nginx-config.png)

![nginx](assets/10-nginx-uwsgi-routing.png)

### 11–12 — Application & Credential Logic

![Source disclosure](assets/11-main-py-credential-disclosure.png)

![Timing surface](assets/12-password-checking-logic.png)

### 13–16 — Authentication & OTP Analysis

![Hopflix authenticated](assets/13-hopflix-authentication-success.png)

![Email selection](assets/14-bank-authorized-email-selection.png)

![2FA](assets/15-bank-two-factor-prompt.png)

![OTP generation](assets/16-otp-generation-logic.png)

### 17–20 — SMTP Interception & Impact

![Captured OTP](assets/17-smtp-otp-capture.png)

![Bank dashboard](assets/18-bank-dashboard.png)

![Funds released](assets/19-charity-funds-released.png)

![Final result](assets/20-final-result-redacted.png)

## Key Findings

| Finding | Severity |
|---|---:|
| Source-code disclosure | Critical |
| OTP recipient validation weakness | Critical |
| Timing side channel | High |
| Exposed nginx configuration | High |
| Client-side authentication secret | High |

## Public Flag Policy

The challenge flag is intentionally redacted:

```text
THM{REDACTED_FOR_PUBLICATION}
```

## Full Documentation

[Read the complete walkthrough](../Documentation/Documentation.md)

## Repository

[GitHub Repository](https://github.com/anurag-rvnkr1/BreachBlocker-Unlocker-TryHackMe-Walkthrough)

---

**Author:** Anurag Revankar
