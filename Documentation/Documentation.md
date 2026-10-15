---
title: "BreachBlocker Unlocker — TryHackMe Security Assessment"
description: "A technical security assessment documenting source-code disclosure, timing side-channel credential recovery, OTP recipient validation weaknesses, SMTP interception, and chained authentication bypass."
platform: "TryHackMe"
room: "BreachBlocker Unlocker"
difficulty: "Hard"
author: "Anurag Revankar"
---

# BreachBlocker Unlocker — Security Assessment & CTF Walkthrough

> **Platform:** TryHackMe  
> **Room:** BreachBlocker Unlocker  
> **Difficulty:** Hard  
> **Assessment Type:** Authorized CTF / Web Application Security Assessment  
> **Primary Domains:** Web Security, Authentication, Source-Code Disclosure, Side-Channel Analysis, SMTP, Authorization  
> **Author:** Anurag Revankar

---

## Executive Summary

BreachBlocker Unlocker is a multi-stage cybersecurity challenge that demonstrates how individually exploitable weaknesses across interconnected services can combine into a complete application compromise.

The assessment begins with recovering an access key from a preceding Advent of Cyber artifact and progresses through network enumeration, web application reconnaissance, client-side source inspection, server configuration disclosure, Python source exposure, SQLite credential analysis, timing side-channel exploitation, and multi-factor authentication testing.

The final stages focus on a weakness in OTP recipient validation and a discrepancy between application-level validation and downstream SMTP message delivery. By following the authentication and message-delivery paths, the investigation demonstrates how an OTP can be intercepted despite the use of a randomly generated six-digit value.

The assessment illustrates a critical security principle: **authentication is only as strong as the complete system responsible for generating, validating, delivering, and consuming authentication credentials.**

### Assessment Highlights

- Enumerated exposed network services and mapped the application's attack surface.
- Identified sensitive information exposed through client-side code and server configuration.
- Investigated Python application source and SQLite credential storage.
- Analyzed password-verification behavior for a timing side channel.
- Demonstrated how secret-dependent execution time can enable credential recovery.
- Investigated OTP recipient validation and SMTP delivery behavior.
- Correlated multiple weaknesses to reach a protected banking workflow.
- Developed remediation recommendations addressing the underlying architectural flaws.

### End-to-End Attack Chain

```text
Initial Access-Key Recovery
          |
          v
Network Service Enumeration
          |
          v
Mobile Portal Reconnaissance
          |
          v
Client-Side Source Inspection
          |
          v
Nginx Configuration Disclosure
          |
          v
Python Application Source Exposure
          |
          v
SQLite Credential Analysis
          |
          v
Timing Side-Channel Recovery
          |
          v
Hopflix Authentication
          |
          v
Hopsec Bank Authentication
          |
          v
OTP Recipient Validation Weakness
          |
          v
SMTP Delivery Interception
          |
          v
Bank Authentication Completion
          |
          v
Charity-Fund Authorization
```

> **Public portfolio note:** Challenge flags, recovered credentials, access keys, OTP values, and other challenge-specific secrets are intentionally redacted. This document focuses on the methodology, technical findings, security implications, and defensive recommendations.

---

## Navigation

**Documentation Map**

- [Mission and Assessment Overview](#mission-and-assessment-overview)
- [Scope and Responsible Use](#scope-and-responsible-use)
- [Assessment Objectives](#assessment-objectives)
- [Tools and Techniques](#tools-and-techniques)
- [Initial Access-Key Recovery](#1-initial-access-key-recovery)
- [Network Service Enumeration](#2-network-service-enumeration)
- [SMTP Service Enumeration](#3-smtp-service-enumeration)
- [Mobile Portal Reconnaissance](#4-mobile-portal-reconnaissance)
- [Client-Side Source Inspection](#5-client-side-source-inspection)
- [Bank Login Reconnaissance](#6-bank-login-reconnaissance)
- [Nginx Configuration Disclosure](#7-nginx-configuration-disclosure)
- [Python Source-Code Disclosure](#8-python-source-code-disclosure)
- [SQLite Credential Analysis](#9-sqlite-credential-analysis)
- [Password Verification and Timing Side Channel](#10-password-verification-and-timing-side-channel)
- [Timing-Based Credential Recovery](#11-timing-based-credential-recovery)
- [Hopflix Authentication](#12-hopflix-authentication)
- [Hopsec Bank and OTP Workflow](#13-hopsec-bank-and-otp-workflow)
- [OTP Recipient Validation Weakness](#14-otp-recipient-validation-weakness)
- [SMTP Delivery-Path Interception](#15-smtp-delivery-path-interception)
- [Banking Authentication and Final Impact](#16-banking-authentication-and-final-impact)
- [End-to-End Attack Chain](#end-to-end-attack-chain)
- [Security Findings and Severity](#security-findings-and-severity)
- [Root Cause Analysis](#root-cause-analysis)
- [Remediation and Defensive Recommendations](#remediation-and-defensive-recommendations)
- [Lessons Learned](#lessons-learned)
- [Conclusion](#conclusion)
- [References and Evidence](#references-and-evidence)

---

## Mission and Assessment Overview

The objective of this assessment is to investigate the security posture of the BreachBlocker Unlocker lab environment and determine how weaknesses in application exposure, credential verification, and OTP delivery can be chained to compromise a protected banking workflow.

Rather than treating each endpoint as an isolated target, the investigation follows the relationships between services, application components, authentication mechanisms, and sensitive data.

The assessment emphasizes a structured methodology:

1. Establish the authorized lab context.
2. Enumerate the exposed network services.
3. Identify application entry points and relevant user workflows.
4. Inspect client-side resources and server configuration.
5. Trace authentication logic and credential-storage behavior.
6. Evaluate password-verification timing characteristics.
7. Examine OTP recipient validation and delivery semantics.
8. Correlate findings and assess their combined impact.
9. Document root causes and practical defensive controls.

### Assessment Context

| Attribute | Description |
|---|---|
| Platform | TryHackMe |
| Room | BreachBlocker Unlocker |
| Difficulty | Hard |
| Target type | Intentionally vulnerable lab environment |
| Primary application | Mobile-style web portal |
| Additional components | Python application, SQLite database, SMTP service |
| Authentication mechanisms | Password authentication and OTP-based verification |
| Primary outcome | Access to a protected banking workflow |
| Documentation objective | Technical portfolio and security research record |

---

## Scope and Responsible Use

All testing described in this document is restricted to the intentionally vulnerable TryHackMe environment.

The techniques are documented for educational purposes, authorized security testing, and defensive analysis.

### In Scope

- Network service enumeration within the assigned lab.
- HTTP and HTTPS application reconnaissance.
- Inspection of browser-delivered JavaScript and application resources.
- Analysis of exposed configuration and application source.
- Investigation of credential verification and timing behavior.
- Assessment of OTP recipient validation and SMTP delivery.
- Evaluation of authorization boundaries within the challenge.

### Out of Scope

- Testing unrelated public infrastructure.
- Accessing real banking systems or production accounts.
- Intercepting messages belonging to third parties.
- Reusing recovered lab credentials outside the challenge.
- Publishing live credentials, access keys, OTP values, or challenge flags.

**Responsible disclosure principle:** Findings should be reproduced only in authorized environments. In production assessments, evidence should be collected with the minimum access necessary, sensitive data should be protected, and testing should follow the agreed rules of engagement.

---

## Assessment Objectives

The investigation was designed to answer the following questions:

- Which network services and application entry points are exposed?
- Does the mobile application disclose security-sensitive information?
- Can server configuration or Python source be retrieved through the web layer?
- What does the application reveal about its credential-storage model?
- Does password verification expose information through response timing?
- How is the OTP generated, validated, and delivered?
- Does recipient validation match the interpretation used by the downstream mail service?
- Can the discovered weaknesses be chained to bypass the intended authentication and authorization boundaries?
- Which engineering controls would prevent or substantially reduce the demonstrated risks?

---

## Tools and Techniques

The following tools and techniques were used or referenced during the investigation.

| Tool / Technique | Purpose |
|---|---|
| Nmap | TCP service discovery and service/version enumeration |
| Metasploit Framework | SMTP service enumeration |
| FFUF | Web content and configuration discovery |
| Burp Suite | HTTP request inspection and authentication-flow analysis |
| CyberChef | Base64 decoding, XOR processing, and artifact analysis |
| SQLite tooling | Database structure and credential-record inspection |
| Python | Repeated timing measurements and statistical comparison |
| `aiosmtpd` | Controlled SMTP listener in the authorized lab |
| Browser developer tools | Client-side resource and JavaScript inspection |
| Source-code review | Authentication, validation, and routing analysis |

Tool selection was driven by evidence gathered at each stage. The investigation moved from broad service discovery toward targeted application analysis as new information became available.

---

# Technical Walkthrough

## 1. Initial Access-Key Recovery

### Objective

Recover the access key required to unlock the target environment using the preceding Advent of Cyber challenge artifact.

### Methodology

The initial stage involves analyzing an HTA file associated with the previous challenge material.

The artifact is treated as a file to inspect rather than a program to execute. Its embedded content is normalized, and the encoded data is decoded to isolate the remaining encrypted payload.

An example normalization and decoding command is:

```bash
sed ':a;N;$!ba;s/[&_"]//g;s/[[:space:]]//g' \
  NorthPolePerformanceReview.hta | base64 -d > b64.txt
```

The resulting data requires additional decoding before the remaining payload can be processed in CyberChef.

The relevant transformation is:

```text
Input artifact
     |
     v
Content normalization
     |
     v
Base64 decoding
     |
     v
Additional Base64 decoding
     |
     v
XOR transformation
     |
     v
Image rendering
     |
     v
Access-key recovery
```

The XOR operation uses the following parameters:

| Parameter | Value |
|---|---|
| Operation | XOR |
| Key | `23` |
| Key type | Decimal |
| Scheme | Standard |
| Final operation | Render Image |

### Evidence

![CyberChef key recovery](../docs/assets/01-cyberchef-key-recovery.png)

### Result

The process reveals the SQ4 access key required for the next stage.

The recovered key is intentionally omitted from this public write-up.

### Security Relevance

The initial stage demonstrates the importance of analyzing suspicious or unfamiliar artifacts through controlled inspection. Encoded content should not be assumed to be harmless simply because it is not immediately readable.

---

## 2. Network Service Enumeration

### Objective

Identify the target's exposed TCP services and determine which components warrant further investigation.

### Methodology

After obtaining access to the lab, a targeted Nmap scan is performed against the identified service ports.

```bash
nmap -p22,25,8443,21337 -sCV TARGET -oN fullscan.tcp
```

Replace `TARGET` with the assigned lab address.

The `-sC` option runs the default NSE scripts, while `-sV` attempts service/version detection. The `-oN` option saves the results in normal output format.

### Observed Attack Surface

| Port | Identified Service | Security Relevance |
|---|---|---|
| 22/TCP | SSH | Potential remote administration interface |
| 25/TCP | Postfix SMTP | Mail delivery and recipient-handling surface |
| 8443/TCP | Nginx HTTPS | Mobile application and web entry point |
| 21337/TCP | Werkzeug/Python | Additional Python-backed web application |

### Evidence

![Nmap results](../docs/assets/02-nmap-service-enumeration.png)

### Analysis

The target exposes several services with potentially related roles.

The combination of an HTTPS application, an SMTP service, and a Python-backed application suggests that authentication and message delivery may involve multiple components.

This observation establishes the initial investigation priorities:

- Determine what information is available through the HTTPS portal.
- Inspect the SMTP service and its supported behavior.
- Identify the role of the Python application.
- Trace relationships between the exposed components.

### Result

The service inventory provides the foundation for targeted reconnaissance and subsequent attack-chain development.

---

## 3. SMTP Service Enumeration

### Objective

Identify the mail service implementation and assess whether its exposed behavior provides useful information for the investigation.

### Methodology

Metasploit's SMTP auxiliary scanners are used to inspect the service.

Relevant modules include:

```text
auxiliary/scanner/smtp/smtp_enum
auxiliary/scanner/smtp/smtp_version
```

These modules can help identify SMTP implementation details and evaluate whether the service discloses useful account or configuration information.

### Evidence

![SMTP enumeration](../docs/assets/03-smtp-enumeration.png)

### Analysis

The initial enumeration does not produce an immediate credential breakthrough.

Rather than forcing an unproductive path, the investigation moves to the HTTPS mobile portal while retaining SMTP as a potential component of the authentication workflow.

### Security Relevance

SMTP services should be evaluated not only for their exposed version and configuration but also for how applications construct recipients, validate addresses, and hand messages to the mail server.

A secure mail service cannot compensate for unsafe recipient handling in the application that invokes it.

---

## 4. Mobile Portal Reconnaissance

### Objective

Map the available applications and identify the authentication workflows relevant to the target.

### Methodology

The HTTPS service on port `8443` hosts a mobile-style portal containing multiple application interfaces.

The Hopflix login interface exposes a pre-populated account identifier, providing a starting point for investigating the identity and its associated services.

### Evidence

![Hopflix login](../docs/assets/04-hopflix-login-surface.png)

The associated inbox provides additional context:

![Hopflix inbox](../docs/assets/05-hopflix-inbox.png)

### Analysis

The portal's application interfaces and messages establish a relationship between the target identity and the banking workflow.

The inbox also indicates that protected charity funds are associated with a subsequent authentication process.

These observations help prioritize the Hopflix and banking applications for further analysis.

### Result

The investigation identifies a likely path from the mobile portal to the protected banking application.

---

## 5. Client-Side Source Inspection

### Objective

Determine whether browser-delivered application resources expose security-sensitive values or implementation details.

### Methodology

The application is inspected using browser developer tools and source review.

Because JavaScript and other client-side resources are delivered to the browser, their contents are accessible to users. Any embedded values that are expected to remain secret must therefore be treated as potentially compromised.

### Evidence

![Mobile source review](../docs/assets/06-mobile-source-review.png)

The application JavaScript contains a value used by the authenticator workflow:

![Client-side secret](../docs/assets/07-app-javascript-secret.png)

### Finding

**Client-side security-sensitive value exposure**

The application distributes a security-relevant value to the browser.

If the value is intended to function as a confidential authentication secret, its placement in client-side code undermines that assumption.

### Security Impact

Depending on the role of the exposed value, an attacker may be able to:

- Recover information intended to remain confidential.
- Reconstruct part of the authentication workflow.
- Reduce the effectiveness of an authentication control.
- Use the disclosed information to support further attacks.

### Defensive Recommendation

Confidential authentication secrets should remain on trusted server-side systems or within appropriately protected authenticator devices.

Client-side code should contain only information that is safe to disclose to the user.

---

## 6. Bank Login Reconnaissance

### Objective

Understand the banking application's login request structure and identify relevant authentication parameters.

### Methodology

The banking login workflow is examined using Burp Suite.

Intercepting the request provides visibility into the HTTP method, request path, parameters, headers, and response behavior.

### Evidence

![Burp request](../docs/assets/08-burp-bank-login.png)

### Analysis

Direct account-number enumeration does not immediately provide a useful result.

The investigation therefore shifts toward understanding the application's implementation and the relationship between its authentication components.

This transition illustrates the value of combining black-box testing with source-level evidence when the authorized environment permits both.

### Result

The request capture establishes a baseline for later analysis of the banking authentication and OTP workflow.

---

## 7. Nginx Configuration Disclosure

### Objective

Identify exposed web resources that may reveal the server's routing behavior or internal application structure.

### Methodology

FFUF is used to discover accessible paths on the HTTPS service.

```bash
ffuf -u https://TARGET:8443/FUZZ \
  -w /opt/SecLists/Discovery/Web-Content/quickhits.txt
```

Replace `TARGET` with the assigned lab address.

### Evidence

![FFUF results](../docs/assets/09-fuzz-nginx-config.png)

The discovered configuration reveals a routing pattern involving `try_files` and a Python application served through uWSGI.

![Nginx and uWSGI routing](../docs/assets/10-nginx-uwsgi-routing.png)

### Finding

**Exposure of web-server configuration**

The disclosed configuration provides insight into the relationship between Nginx and the Python application.

The routing information suggests that application files and server-side behavior should be examined more closely.

### Security Impact

Exposed configuration may disclose:

- Internal application paths.
- Routing rules and fallback behavior.
- Backend service relationships.
- Framework and deployment details.
- Potentially sensitive operational information.

The actual impact depends on which files and routes are accessible and whether the disclosed configuration enables further exploitation.

### Defensive Recommendation

Production deployments should prevent direct access to configuration files and internal application artifacts. Routing rules should be designed so that fallback behavior cannot expose source files or other sensitive resources.

---

## 8. Python Source-Code Disclosure

### Objective

Determine whether server-side application source is accessible through the web layer.

### Methodology

Following the configuration discovery, the investigation tests whether common Python application entry points are exposed through the target's web routing behavior.

The primary application source is found to be accessible.

### Evidence

![Application source disclosure](../docs/assets/11-main-py-credential-disclosure.png)

### Observed Sensitive Information

The disclosed source contains references to:

- Database paths.
- Application route definitions.
- Authentication logic.
- Security-sensitive values.
- Challenge-specific flags.

### Finding

**Server-side source-code exposure**

Python application source is accessible through a web-facing interface.

This is a high-impact information-disclosure issue because source code may reveal implementation details that materially reduce the effort required to attack the application.

### Security Impact

Source disclosure can allow an attacker to move beyond assumptions based on visible application behavior and directly inspect:

- Credential verification logic.
- Authentication state transitions.
- Input validation rules.
- Database locations and schema references.
- Sensitive configuration values.
- Internal endpoints and security assumptions.

In this assessment, source disclosure provides the information needed to investigate credential storage and password-verification behavior.

### Defensive Recommendation

- Keep application source outside web-accessible document roots.
- Restrict access to internal files and deployment artifacts.
- Review reverse-proxy routing and fallback rules.
- Disable production debug behavior.
- Store secrets in an appropriate secret-management system.
- Add deployment checks that detect exposed source files and configuration artifacts.

Challenge-specific flags and secrets are intentionally excluded from this document.

---

## 9. SQLite Credential Analysis

### Objective

Understand the target account's credential representation and determine how the application verifies passwords.

### Methodology

The disclosed application source identifies the location of a Hopflix SQLite database.

After recovering the database within the authorized lab, its structure and relevant records are inspected using SQLite tooling.

The target account contains an unusually long password representation.

The source code explains that each character is independently transformed through repeated SHA-1 operations.

### Length Analysis

A SHA-1 digest is 160 bits, normally represented as 40 hexadecimal characters.

If each character is represented by a separate 40-character hexadecimal digest and the account contains 12 characters, the resulting representation would be approximately:

```text
40 hexadecimal characters per digest
× 12 character digests
= 480 hexadecimal characters
```

This calculation provides a structural clue about the representation and its likely relationship to the original password length.

### Finding

**Custom credential representation using repeated SHA-1 operations**

The application uses a custom password-processing design rather than a dedicated password-hashing scheme designed for credential storage.

### Security Analysis

SHA-1 is a general-purpose cryptographic hash function, not a suitable standalone password-storage scheme.

Applying a fast hash repeatedly does not automatically produce a secure password KDF. The security properties depend on the complete construction, including work factor, salt handling, verification logic, and whether the transformation leaks information through other observable behavior.

In this challenge, the important issue is not simply the hash algorithm. The authentication implementation also creates a timing side channel.

### Defensive Recommendation

Use a password-hashing scheme specifically designed for password storage, such as:

- Argon2id.
- scrypt.
- bcrypt.

Use a unique salt for each password and select a work factor appropriate to the deployment.

Password storage should be evaluated together with the verification routine, because a strong KDF does not prevent an application from leaking information through its control flow or response timing.

---

## 10. Password Verification and Timing Side Channel

### Objective

Determine whether the password-verification routine leaks information about candidate credentials through processing time.

### Methodology

Inspection of the authentication code reveals that the server validates the expected password representation and then processes candidate characters sequentially.

The verification behavior depends on how much of the candidate prefix matches the expected value.

### Evidence

![Password checking logic](../docs/assets/12-password-checking-logic.png)

### Observed Behavior

The general pattern is:

```text
Candidate prefix matches
          |
          v
Additional verification work
          |
          v
Longer processing time
          |
          v
Observable response-time difference
```

When a correct prefix causes additional server-side processing, response latency may reveal information about the candidate credential.

### Finding

**Secret-dependent authentication timing**

The application performs measurably different amounts of work depending on the correctness of the candidate password prefix.

### Security Impact

If the timing signal is sufficiently consistent, an attacker may be able to infer the credential incrementally without reversing the entire stored representation.

This turns a password-verification endpoint into a potential information oracle.

### Defensive Recommendation

- Avoid verification logic that reveals how many characters or bytes match.
- Use well-reviewed password-verification implementations.
- Use constant-time comparison where applicable.
- Avoid custom per-character secret-dependent processing.
- Apply rate limiting and monitoring to authentication endpoints.
- Ensure that error responses do not reveal partial credential correctness.
- Test for timing differences under realistic network conditions.

Constant-time comparison is one component of a broader defense; it does not replace a secure password-storage scheme or sound authentication architecture.

---

## 11. Timing-Based Credential Recovery

### Objective

Demonstrate how repeated response-time measurements can be used to infer credential information from a vulnerable verification routine.

### Methodology

The timing attack is statistical rather than a reversal of the underlying hash representation.

The process uses repeated candidate submissions and compares their observed response times.

The general workflow is:

1. Maintain the already identified candidate prefix.
2. Select a candidate character for the next position.
3. Construct a candidate using the known prefix and test character.
4. Submit repeated requests.
5. Record the observed response latency.
6. Compare the measurements across candidates.
7. Select the strongest candidate according to the observed timing signal.
8. Repeat for subsequent positions.

### Representative Pseudocode

The following is a conceptual illustration of the measurement process, not a complete exploit script.

```python
for position in range(secret_length):
    measurements = {}

    for candidate in alphabet:
        guess = prefix + candidate + filler

        samples = [
            probe(guess)
            for _ in range(REPETITIONS)
        ]

        measurements[candidate] = mean(samples)

    prefix += max(
        measurements,
        key=measurements.get
    )
```

### Measurement Considerations

Timing attacks can be sensitive to environmental noise. A single request is generally insufficient to establish a reliable difference.

Important factors include:

- Network latency and jitter.
- Server load and concurrent requests.
- Connection establishment overhead.
- Application caching.
- Request throttling.
- The number of repetitions per candidate.
- The statistical method used to compare measurements.

The investigation uses the AttackBox environment because additional VPN latency can make small timing differences harder to distinguish.

### Finding

**Credential information leakage through response timing**

The application exposes a measurable signal associated with candidate-prefix correctness.

### Defensive Recommendation

Authentication verification should not expose useful differences in processing time based on secret content. Use established verification libraries, reduce unnecessary secret-dependent branching, and validate timing behavior through repeatable testing.

---

## 12. Hopflix Authentication

### Objective

Validate the recovered credential against the Hopflix application and determine whether the authenticated session provides access to the next stage.

### Methodology

After the credential-recovery stage, the resulting credential is used within the authorized lab to authenticate to Hopflix.

### Evidence

![Successful Hopflix login](../docs/assets/13-hopflix-authentication-success.png)

### Result

Authentication succeeds, establishing the transition from credential analysis to the next application workflow.

### Security Relevance

The successful login demonstrates the operational impact of the earlier information-disclosure and timing findings.

The credential-recovery weakness is no longer an isolated implementation issue; it becomes a practical entry point into another application component.

### Defensive Recommendation

Applications should use independent credentials or appropriately designed single sign-on and token-based trust relationships rather than relying on uncontrolled password reuse between services.

---

## 13. Hopsec Bank and OTP Workflow

### Objective

Map the banking application's second authentication stage and identify the trust assumptions governing OTP delivery.

### Methodology

The banking application prompts the authenticated user to select an authorized email address for OTP delivery.

### Evidence: Recipient Selection

![Authorized email selection](../docs/assets/14-bank-authorized-email-selection.png)

The workflow then requests a six-digit verification code.

### Evidence: OTP Prompt

![Two-factor authentication prompt](../docs/assets/15-bank-two-factor-prompt.png)

### Analysis

At this point, the investigation shifts from password authentication to the OTP workflow.

A randomly generated OTP can provide an additional authentication factor, but its effectiveness depends on more than the randomness of the code.

The system must also ensure that:

- The intended recipient is validated correctly.
- The message is delivered only to an authorized destination.
- The recipient used by the application is interpreted consistently by the mail service.
- The code is associated with the correct authentication transaction.
- The code expires and cannot be reused.
- Failed attempts are appropriately restricted.

### Result

The recipient-selection process becomes a key area for further investigation.

---

## 14. OTP Recipient Validation Weakness

### Objective

Assess whether the application validates the OTP recipient consistently and securely before initiating message delivery.

### Methodology

Source inspection identifies the application's OTP-generation logic and recipient-validation condition.

The validation behavior is represented by the following simplified source pattern:

```python
if domain not in allowed_domains and to_addr not in allowed_emails:
    return -1
```

This condition rejects the recipient only when both checks fail.

Consequently, passing either check may be sufficient to avoid rejection, even if the recipient does not satisfy both intended requirements.

The exact security impact depends on how the condition interacts with the subsequent address-handling and mail-delivery logic.

### Finding

**Insufficiently strict OTP recipient validation**

The application does not enforce a single, unambiguous recipient authorization policy.

### Security Impact

A recipient-validation weakness may allow a destination that should have been rejected to reach the downstream mail-delivery component.

If the application and SMTP service interpret the recipient differently, the discrepancy can create an opportunity for OTP interception.

### Why OTP Randomness Is Not Enough

The OTP generator and the OTP delivery path solve different security problems.

- Random generation protects the unpredictability of the code.
- Recipient validation controls who is allowed to receive it.
- Secure delivery protects the confidentiality of the message.
- Transaction binding ensures the code authorizes the intended action.
- Expiration and attempt limits constrain the time and opportunities available to misuse it.

A strong generator cannot compensate for an insecure delivery path.

### Defensive Recommendation

Use an explicit allowlist of authorized recipient identities. Parse and normalize the address before applying authorization rules, and reject ambiguous or malformed values.

Recipient authorization should be based on one canonical representation that is used consistently throughout the application and mail-delivery process.

---

## 15. SMTP Delivery-Path Interception

### Objective

Determine whether the discrepancy between application-level recipient validation and downstream SMTP handling can expose the OTP.

### Methodology

The challenge permits a specially structured recipient that interacts differently with application validation and SMTP message delivery.

The general trust-boundary failure is:

```text
User-supplied recipient
          |
          v
Application-level validation
          |
          v
Recipient interpreted under one set of rules
          |
          v
SMTP processing
          |
          v
Recipient interpreted under different rules
          |
          v
Unexpected message destination
```

This behavior is a validation and parser-consistency problem.

Within the authorized lab, a controlled SMTP listener is used to observe the resulting message.

An example listener command is:

```bash
aiosmtpd -n -l 0.0.0.0:25
```

This command starts an SMTP listener bound to all network interfaces on port 25. It should be used only in an appropriately controlled lab environment with the necessary permissions and network restrictions.

### Evidence

![Captured OTP](../docs/assets/17-smtp-otp-capture.png)

### Finding

**OTP delivery can be redirected through inconsistent recipient handling**

The combination of permissive recipient validation and differing downstream parsing behavior creates a path through which the OTP can be delivered to an unintended destination in the challenge.

### Security Impact

Successful interception undermines the confidentiality of the second authentication factor.

The attack does not need to predict the randomly generated OTP. Instead, it targets the system responsible for delivering the code.

### Defensive Recommendation

- Parse recipient addresses using a trusted email-address parser.
- Reject malformed, ambiguous, or unsupported address forms.
- Enforce an explicit allowlist of recipient identities.
- Canonicalize the address once and reuse the validated representation.
- Avoid passing untrusted recipient strings directly to downstream components.
- Ensure application-level authorization and SMTP envelope recipients cannot diverge.
- Record and monitor OTP delivery destinations.
- Rate-limit OTP requests and prevent repeated destination changes during a single authentication transaction.

---

## 16. Banking Authentication and Final Impact

### Objective

Validate the combined impact of the discovered authentication and OTP delivery weaknesses.

### Methodology

After obtaining the OTP through the lab's message-delivery path, the code is submitted to the banking application to complete the second authentication stage.

### Evidence: Authenticated Banking Dashboard

![Bank dashboard](../docs/assets/18-bank-dashboard.png)

### Result

The authentication workflow completes and exposes the protected charity-fund operation.

### Evidence: Final Operation

![Funds released](../docs/assets/19-charity-funds-released.png)

The final challenge result is shown with its value redacted:

![Final result](../docs/assets/20-final-result-redacted.png)

```text
THM{REDACTED_FOR_PUBLICATION}
```

### Impact Assessment

The combined attack chain demonstrates that weaknesses across different application components can undermine an otherwise multi-stage authentication workflow.

The demonstrated progression is:

1. Recover the initial access key.
2. Enumerate the exposed services.
3. Discover information through the mobile portal.
4. Identify client-side and server-side information disclosure.
5. Analyze the credential representation and verification routine.
6. Recover the credential using timing observations.
7. Authenticate to Hopflix.
8. Investigate the bank's OTP workflow.
9. Exploit recipient-validation and delivery inconsistencies.
10. Complete banking authentication and reach the protected operation.

### Business Impact

In a real application with comparable weaknesses, the potential consequences could include:

- Unauthorized account access.
- Compromise of authentication secrets.
- Bypass of intended second-factor protections.
- Exposure of sensitive application information.
- Unauthorized access to protected financial workflows.
- Loss of confidence in the integrity of authentication and authorization controls.

These are potential consequences of the demonstrated vulnerability classes, not claims about any real banking system.

---

# Security Assessment Results

## End-to-End Attack Chain

The investigation shows how individual observations become actionable when correlated across application layers.

```text
[Initial Artifact]
       |
       v
[Access-Key Recovery]
       |
       v
[Network Enumeration]
       |
       v
[HTTPS / SMTP Reconnaissance]
       |
       v
[Mobile Portal Discovery]
       |
       v
[Client-Side Information Exposure]
       |
       v
[Nginx Configuration Disclosure]
       |
       v
[Python Source-Code Exposure]
       |
       v
[SQLite Credential Analysis]
       |
       v
[Timing Side-Channel Analysis]
       |
       v
[Hopflix Authentication]
       |
       v
[Bank OTP Workflow]
       |
       v
[Recipient Validation Weakness]
       |
       v
[SMTP Delivery Interception]
       |
       v
[Bank Authentication]
       |
       v
[Protected Charity-Fund Operation]
```

The key lesson is that the security impact of a chained attack can exceed the impact of any individual finding.

---

## Security Findings and Severity

The following ratings are qualitative assessment judgments based on the behavior described in the supplied challenge evidence. They are not formal CVSS scores.

| ID | Finding | Qualitative Severity | Primary Impact |
|---|---|---|---|
| F-01 | Web-accessible application source | Critical | Discloses authentication logic, paths, and security-sensitive information |
| F-02 | OTP recipient validation weakness | Critical | May permit OTP delivery to an unintended destination |
| F-03 | Password-verification timing side channel | High | May enable incremental credential recovery |
| F-04 | Exposed Nginx configuration | High | Reveals routing behavior and backend architecture |
| F-05 | Client-side security-sensitive value | High | Weakens confidentiality of the associated authentication control |
| F-06 | Cross-application credential reuse | High | Enables movement from one application to another after credential recovery |

### Severity Interpretation

- **Critical:** A finding that, in the demonstrated chain, directly contributes to bypassing a major authentication or authorization boundary.
- **High:** A finding that exposes sensitive information or provides a meaningful path toward credential compromise or further exploitation.

Final production severity should be assigned only after validating exploitability, affected assets, privilege requirements, exposure, and business impact in the actual deployment.

---

## Root Cause Analysis

### F-01: Web-Accessible Application Source

**Observed weakness:** Python application source is retrievable through the web-facing service.

**Root cause:** Unsafe file-resolution or routing behavior, combined with inadequate protection of internal application files.

**Why it matters:** Source disclosure reveals implementation details that may expose authentication weaknesses, database paths, and security-sensitive values.

**Recommended control:** Separate application code from public document roots, harden reverse-proxy rules, and validate that internal files cannot be retrieved through unexpected routes.

### F-02: OTP Recipient Validation Weakness

**Observed weakness:** The recipient-validation condition does not enforce the intended authorization checks together.

**Root cause:** Incorrect Boolean logic and inconsistent interpretation of recipient input across application and mail-delivery layers.

**Why it matters:** A recipient that should be rejected may reach the mail-delivery component.

**Recommended control:** Use strict recipient allowlisting, canonical parsing, unambiguous authorization logic, and consistent downstream handling.

### F-03: Password Timing Side Channel

**Observed weakness:** Response time varies according to how much of the candidate password prefix is correct.

**Root cause:** Secret-dependent execution and expensive per-character verification operations.

**Why it matters:** An attacker may infer credential information incrementally.

**Recommended control:** Use established password-verification libraries, eliminate useful secret-dependent timing differences, and monitor repeated authentication attempts.

### F-04: Exposed Nginx Configuration

**Observed weakness:** Web discovery reveals server configuration and backend routing details.

**Root cause:** Inadequate protection of configuration files or unsafe routing behavior.

**Why it matters:** Configuration disclosure reduces uncertainty about the application architecture and can guide further testing.

**Recommended control:** Restrict access to configuration artifacts and verify reverse-proxy routing against an explicit resource allowlist.

### F-05: Client-Side Security-Sensitive Value

**Observed weakness:** A value used in an authentication-related workflow is present in browser-delivered code.

**Root cause:** Treating client-side code as a confidential storage location.

**Why it matters:** Users can inspect and extract values delivered to their own browsers.

**Recommended control:** Keep confidential authentication material server-side or within appropriately protected authenticator devices.

### F-06: Cross-Application Credential Reuse

**Observed weakness:** Credential recovery enables authentication to another application component.

**Root cause:** Insufficient identity isolation or uncontrolled credential reuse between applications.

**Why it matters:** Compromise of one authentication boundary can become a pivot into another.

**Recommended control:** Use independent credentials, centrally managed identity where appropriate, strong session isolation, and least-privilege authorization.

---

# Remediation and Defensive Recommendations

## 1. Protect Application Source and Configuration

- Keep Python source, database files, environment files, and server configuration outside public document roots.
- Use explicit routing rules that expose only intended resources.
- Review Nginx `root`, `alias`, `try_files`, and upstream configuration for unsafe fallback behavior.
- Disable unnecessary debug and development endpoints in production.
- Include source-disclosure tests in deployment validation.

## 2. Implement Secure Password Storage

- Replace custom repeated SHA-1 transformations with Argon2id, scrypt, or bcrypt.
- Generate a unique salt for each password.
- Select a work factor appropriate to the application's performance and security requirements.
- Avoid custom credential-storage constructions unless they have been carefully reviewed.
- Ensure that password verification does not expose useful information through response timing.

## 3. Eliminate Authentication Timing Leakage

- Use trusted password-verification implementations.
- Avoid per-character comparisons that stop at the first mismatch.
- Use constant-time comparison for secret material when applicable.
- Normalize response behavior for invalid credentials.
- Monitor repeated authentication attempts and unusual request patterns.
- Test response-time distributions rather than relying on a small number of observations.

## 4. Enforce Strict OTP Recipient Authorization

- Use an explicit list of authorized recipient identities.
- Avoid loose substring or domain-only checks when exact identity authorization is required.
- Parse and canonicalize addresses before validation.
- Apply authorization rules to the parsed representation rather than an untrusted raw string.
- Reject ambiguous or malformed recipient inputs.
- Ensure that downstream SMTP handling uses the same validated destination.

## 5. Harden OTP Lifecycle Management

- Generate OTPs using a cryptographically secure random-number generator.
- Associate each OTP with a specific user, session, and authentication transaction.
- Enforce short expiration windows.
- Invalidate an OTP after successful use.
- Limit verification attempts and OTP issuance frequency.
- Avoid revealing OTP validity through distinguishable error responses.
- Record delivery metadata without unnecessarily logging the OTP itself.

## 6. Secure Email Delivery

- Validate both the application-level recipient and the actual SMTP envelope recipient.
- Avoid constructing mail commands or recipient values through unsafe string manipulation.
- Use established email libraries and well-defined address-parsing behavior.
- Restrict mail delivery to intended destinations.
- Monitor anomalous OTP delivery requests and recipient changes.
- Protect mail services against unauthorized relay and unnecessary exposure.

## 7. Strengthen Identity Separation

- Avoid uncontrolled password reuse across independent applications.
- Use a trusted identity provider where centralized authentication is appropriate.
- Apply least privilege to service accounts and application identities.
- Isolate sessions, tokens, and authorization scopes between applications.
- Ensure that authentication to one application does not automatically grant access to unrelated protected operations.

## 8. Improve Security Testing and Monitoring

- Include source-disclosure checks in application security testing.
- Test authentication endpoints for timing discrepancies.
- Validate recipient-handling logic using malformed and ambiguous input cases.
- Review reverse-proxy and backend routing configurations.
- Monitor repeated failed authentication attempts and unusual OTP delivery behavior.
- Retest remediation changes to confirm that the original attack chain is no longer viable.

---

## Lessons Learned

### 1. Enumerate Before Exploiting

The initial service inventory established the attack surface and helped identify the most relevant application components.

A structured reconnaissance phase reduces wasted effort and creates a reliable basis for later testing.

### 2. Treat Source Code as Security Evidence

Application source can reveal the actual validation rules, data flows, credential-handling logic, and trust assumptions that are difficult to infer from the interface alone.

Where source access is authorized, code review can help explain why a vulnerability exists and how multiple findings interact.

### 3. Follow the Data Across Trust Boundaries

The OTP stage demonstrates why security testing should follow data from user input through validation and into downstream processing.

A value is not secure merely because one component accepts it under a particular interpretation.

### 4. Timing Attacks Are Measurement Problems

Side-channel exploitation depends on identifying a repeatable signal among noisy observations.

Repeated samples, consistent test conditions, and careful interpretation are essential. Apparent timing differences should be validated rather than assumed to represent secret-dependent behavior.

### 5. Strong OTP Generation Does Not Guarantee Secure Authentication

OTP security depends on generation, recipient authorization, delivery, transaction binding, expiration, and verification.

The weakest part of this lifecycle may undermine the protection offered by the other components.

### 6. Parser Differences Create Security Risks

Validation and downstream processing must agree on the meaning of an input.

When the application and a downstream component interpret the same value differently, an attacker may be able to cross a boundary that the application intended to enforce.

### 7. Chained Vulnerabilities Require Architectural Remediation

Fixing a single exposed file or validation condition may prevent one exploitation path, but the broader system should also be reviewed for related weaknesses.

Defense in depth requires secure routing, credential storage, authentication logic, message delivery, and authorization boundaries.

---

## Conclusion

BreachBlocker Unlocker demonstrates a multi-stage compromise built from weaknesses across several connected application components.

The investigation begins with access-key recovery and network enumeration, progresses through source and configuration disclosure, and uses application behavior to identify a credential-recovery side channel. It then follows the authentication flow into an OTP-protected banking workflow, where recipient-validation and SMTP delivery inconsistencies expose a further weakness.

The most significant lesson is architectural: **security controls must remain consistent across every component that handles identity, secrets, validation, and authorization.**

A secure OTP generator cannot compensate for unsafe recipient validation. A secure-looking login interface cannot compensate for exposed application source. A strong credential-storage algorithm cannot compensate for a verification routine that leaks information through timing.

The assessment reinforces a repeatable security methodology:

**Enumerate → Inspect → Correlate → Analyze → Validate → Remediate**

By documenting the attack chain alongside its root causes and defensive controls, this challenge provides practical experience in web application assessment, authentication analysis, side-channel reasoning, and security engineering.

---

## References and Evidence

### Evidence Artifacts

The following screenshots document the key stages of the investigation. They are referenced using the asset paths supplied with the original write-up.

| Evidence | File |
|---|---|
| Initial access-key recovery | `docs/assets/01-cyberchef-key-recovery.png` |
| Network service enumeration | `docs/assets/02-nmap-service-enumeration.png` |
| SMTP enumeration | `docs/assets/03-smtp-enumeration.png` |
| Hopflix login surface | `docs/assets/04-hopflix-login-surface.png` |
| Hopflix inbox | `docs/assets/05-hopflix-inbox.png` |
| Mobile source review | `docs/assets/06-mobile-source-review.png` |
| Client-side security-sensitive value | `docs/assets/07-app-javascript-secret.png` |
| Bank login request | `docs/assets/08-burp-bank-login.png` |
| Web content discovery | `docs/assets/09-fuzz-nginx-config.png` |
| Nginx and uWSGI routing | `docs/assets/10-nginx-uwsgi-routing.png` |
| Python source disclosure | `docs/assets/11-main-py-credential-disclosure.png` |
| Password verification logic | `docs/assets/12-password-checking-logic.png` |
| Successful Hopflix authentication | `docs/assets/13-hopflix-authentication-success.png` |
| Authorized recipient selection | `docs/assets/14-bank-authorized-email-selection.png` |
| OTP verification prompt | `docs/assets/15-bank-two-factor-prompt.png` |
| OTP generation logic | `docs/assets/16-otp-generation-logic.png` |
| SMTP OTP capture | `docs/assets/17-smtp-otp-capture.png` |
| Banking dashboard | `docs/assets/18-bank-dashboard.png` |
| Charity-fund operation | `docs/assets/19-charity-funds-released.png` |
| Redacted final result | `docs/assets/20-final-result-redacted.png` |

> **Repository note:** Confirm that the image files exist at these exact paths in your repository. If the images are stored under a different directory, update the Markdown references accordingly.

### Tools

- [Nmap](https://nmap.org/)
- [Metasploit Framework](https://docs.metasploit.com/)
- [FFUF](https://github.com/ffuf/ffuf)
- [Burp Suite](https://portswigger.net/burp)
- [CyberChef](https://gchq.github.io/CyberChef/)
- [SQLite](https://www.sqlite.org/)
- [Python](https://www.python.org/)
- [aiosmtpd](https://aiosmtpd.readthedocs.io/)

### Platform

- [TryHackMe](https://tryhackme.com/)

---

**Author:** Anurag Revankar  
**Purpose:** Authorized TryHackMe research, technical documentation, and cybersecurity portfolio  
**Disclosure:** Challenge-specific flags, credentials, access keys, and OTP values are intentionally redacted from this public document.
