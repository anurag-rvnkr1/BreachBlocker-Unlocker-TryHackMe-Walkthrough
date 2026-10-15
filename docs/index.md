<div class="ctf-hero">

  <h1>BreachBlocker Unlocker</h1>

  <p>
    A technical security assessment investigating how application source-code exposure, credential-verification timing differences, weak OTP recipient validation, and inconsistent SMTP delivery behavior can combine to compromise a protected banking workflow.
  </p>

  <div class="ctf-badges">
    <span class="ctf-badge">TryHackMe</span>
    <span class="ctf-badge">Hard</span>
    <span class="ctf-badge">Web Security</span>
    <span class="ctf-badge">Authentication</span>
    <span class="ctf-badge">Side-Channel Analysis</span>
    <span class="ctf-badge">SMTP</span>
  </div>

</div>

## Mission and Executive Summary

BreachBlocker Unlocker is a multi-stage cybersecurity challenge demonstrating how weaknesses across interconnected services can combine into a broader application compromise.

The investigation begins with access-key recovery from a preceding Advent of Cyber artifact and progresses through network enumeration, mobile portal reconnaissance, client-side source inspection, web-server configuration disclosure, Python source exposure, SQLite credential analysis, and password-verification timing analysis.

The later stages examine an OTP-protected banking workflow. Analysis of recipient-validation logic and SMTP message delivery demonstrates how inconsistent input handling can undermine an authentication control even when its verification code is randomly generated.

The central security lesson is that authentication depends on the integrity of the entire workflow: credential storage, verification, recipient authorization, message delivery, session management, and authorization of the protected operation.

<div class="key-finding">

  <div class="key-finding-title">Assessment Outcome</div>

  The documented challenge sequence reaches the protected banking workflow by correlating information disclosure, a password-verification timing side channel, cross-application authentication, and an OTP delivery weakness. Challenge-specific secrets and the final flag remain redacted.

</div>

### Assessment at a Glance

<div class="ctf-card-grid">

  <div class="ctf-card">
    <div class="ctf-card-title">Platform</div>
    <div class="ctf-card-value">TryHackMe</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Difficulty</div>
    <div class="ctf-card-value">Hard</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Assessment Type</div>
    <div class="ctf-card-value">Authorized CTF / Web Security Assessment</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Core Components</div>
    <div class="ctf-card-value">Nginx, Python, SQLite, SMTP</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Authentication Focus</div>
    <div class="ctf-card-value">Password Verification and OTP</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Documented Outcome</div>
    <div class="ctf-card-value">Protected Banking Workflow Reached</div>
  </div>

</div>

### Key Technical Findings

- Sensitive application information was discoverable through client-side resources and server-side file exposure.
- Nginx configuration disclosure provided insight into application routing and the Python backend.
- SQLite credential analysis exposed a custom password representation based on repeated SHA-1 operations.
- Password-verification behavior exhibited a timing side channel associated with candidate-prefix correctness.
- Recipient-validation logic did not unambiguously enforce the intended OTP recipient policy.
- The OTP delivery path demonstrated inconsistent handling between application validation and downstream SMTP processing.
- Correlation of these weaknesses enabled the documented challenge progression to the protected banking workflow.

### End-to-End Attack Chain

<div class="attack-chain">
  <div class="attack-step">Artifact Analysis</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Network Enumeration</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Portal Reconnaissance</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Source Disclosure</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Credential Analysis</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Timing Side Channel</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Hopflix Authentication</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">OTP Validation Weakness</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">SMTP Interception</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Banking Workflow</div>
</div>

> **Public portfolio disclosure:** Challenge flags, recovered credentials, access keys, OTP values, and other challenge-specific secrets are intentionally redacted. This document focuses on the methodology, evidence references, technical findings, security impact, and remediation.

---

## Navigation

<div class="ctf-toc">
<div class="ctf-toc-title">Documentation Map</div>

- <a href="#mission-and-executive-summary">Mission and Executive Summary</a>
- <a href="#scope-and-responsible-use">Scope and Responsible Use</a>
- <a href="#assessment-objectives">Assessment Objectives</a>
- <a href="#tools-and-techniques">Tools and Techniques</a>
- <a href="#target-information-and-attack-surface">Target Information and Attack Surface</a>
- <a href="#technical-walkthrough">Technical Walkthrough</a>
- <a href="#01-initial-access-key-recovery">01. Initial Access-Key Recovery</a>
- <a href="#02-network-service-enumeration">02. Network Service Enumeration</a>
- <a href="#03-smtp-service-enumeration">03. SMTP Service Enumeration</a>
- <a href="#04-mobile-portal-reconnaissance">04. Mobile Portal Reconnaissance</a>
- <a href="#05-client-side-source-inspection">05. Client-Side Source Inspection</a>
- <a href="#06-bank-login-reconnaissance">06. Bank Login Reconnaissance</a>
- <a href="#07-nginx-configuration-disclosure">07. Nginx Configuration Disclosure</a>
- <a href="#08-python-source-code-disclosure">08. Python Source-Code Disclosure</a>
- <a href="#09-sqlite-credential-analysis">09. SQLite Credential Analysis</a>
- <a href="#10-password-verification-and-timing-side-channel">10. Password Verification and Timing Side Channel</a>
- <a href="#11-timing-based-credential-recovery">11. Timing-Based Credential Recovery</a>
- <a href="#12-hopflix-authentication">12. Hopflix Authentication</a>
- <a href="#13-hopsec-bank-and-otp-workflow">13. Hopsec Bank and OTP Workflow</a>
- <a href="#14-otp-recipient-validation-weakness">14. OTP Recipient Validation Weakness</a>
- <a href="#15-smtp-delivery-path-interception">15. SMTP Delivery-Path Interception</a>
- <a href="#16-banking-authentication-and-final-impact">16. Banking Authentication and Final Impact</a>
- <a href="#end-to-end-attack-chain">End-to-End Attack Chain</a>
- <a href="#security-findings-and-severity">Security Findings and Severity</a>
- <a href="#root-cause-analysis">Root Cause Analysis</a>
- <a href="#remediation-and-defensive-recommendations">Remediation and Defensive Recommendations</a>
- <a href="#lessons-learned">Lessons Learned</a>
- <a href="#conclusion">Conclusion</a>
- <a href="#references-and-evidence">References and Evidence</a>

</div>

---

## Scope and Responsible Use

All testing documented here is restricted to the intentionally vulnerable TryHackMe challenge environment. The techniques are presented for cybersecurity education, authorized assessment, and defensive analysis.

### In Scope

- Network service enumeration within the assigned lab.
- HTTP and HTTPS reconnaissance.
- Inspection of browser-delivered JavaScript and application resources.
- Analysis of exposed web-server configuration and application source.
- Investigation of credential storage and password-verification behavior.
- Evaluation of timing differences during authentication.
- Analysis of OTP recipient validation and SMTP delivery.
- Assessment of the resulting authentication and authorization impact.

### Out of Scope

- Testing unrelated public infrastructure.
- Accessing real banking systems or production accounts.
- Intercepting messages belonging to third parties.
- Reusing challenge credentials outside the lab.
- Publishing real credentials, access keys, OTP values, or challenge flags.

> **Responsible use:** The techniques in this report should only be used against systems for which explicit permission has been granted. Real-world assessments should follow the agreed rules of engagement, minimize access to sensitive information, and protect all collected evidence.

---

## Assessment Objectives

The investigation was designed to answer the following questions:

1. Which network services and application entry points are exposed?
2. Does the mobile application disclose security-sensitive information?
3. Can server configuration or Python source be retrieved through the web layer?
4. What does the application reveal about its credential-storage design?
5. Does password verification expose candidate information through response timing?
6. How is the OTP generated, validated, and delivered?
7. Does the application's recipient-validation policy match the interpretation applied by the mail-delivery layer?
8. Can the documented weaknesses be correlated into a path across authentication and authorization boundaries?
9. Which engineering controls would prevent or substantially reduce the identified risks?

### Assessment Methodology

<div class="ctf-details">

  <div class="ctf-detail">
    <span class="ctf-detail-label">Phase 01</span>
    <div class="ctf-detail-value">Reconnaissance and Service Discovery</div>
  </div>

  <div class="ctf-detail">
    <span class="ctf-detail-label">Phase 02</span>
    <div class="ctf-detail-value">Application and Source Inspection</div>
  </div>

  <div class="ctf-detail">
    <span class="ctf-detail-label">Phase 03</span>
    <div class="ctf-detail-value">Credential and Timing Analysis</div>
  </div>

  <div class="ctf-detail">
    <span class="ctf-detail-label">Phase 04</span>
    <div class="ctf-detail-value">OTP and SMTP Workflow Testing</div>
  </div>

  <div class="ctf-detail">
    <span class="ctf-detail-label">Phase 05</span>
    <div class="ctf-detail-value">Impact Assessment and Remediation</div>
  </div>

</div>

---

## Tools and Techniques

The following tools and techniques are documented in the supplied challenge notes.

| Tool or Technique | Assessment Purpose |
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
| Source-code review | Analysis of authentication, validation, and routing logic |

Tool selection followed the evidence gathered during each phase. Broad network discovery narrowed the investigation to relevant application services, while source inspection and measurement techniques helped explain the observed security behavior.

---

## Target Information and Attack Surface

### Lab Context

| Attribute | Documented Value |
|---|---|
| Platform | TryHackMe |
| Room | BreachBlocker Unlocker |
| Difficulty | Hard |
| Target type | Intentionally vulnerable lab environment |
| Primary application | Mobile-style web portal |
| Additional components | Python application, SQLite database, SMTP service |
| Authentication mechanisms | Password authentication and OTP verification |
| Primary objective | Reach a protected banking workflow |

The original notes identify the target by a placeholder rather than a literal IP address. Accordingly, the actual address is not inferred or fabricated.

### Observed Services

The following ports and service identifications are recorded in the supplied enumeration results.

| Port | Identified Service | Assessment Relevance |
|---:|---|---|
| 22/TCP | SSH | Potential remote administration interface |
| 25/TCP | Postfix SMTP | Mail-delivery and recipient-handling surface |
| 8443/TCP | Nginx HTTPS | Mobile portal and web application entry point |
| 21337/TCP | Werkzeug/Python | Additional Python-backed web application |

The discovered services establish several investigation paths. The HTTPS portal is relevant to application reconnaissance, the Python service warrants examination of backend behavior, and the SMTP service is relevant to OTP delivery.

Service names and roles are retained from the supplied assessment notes; no additional version numbers are asserted.

---

# Technical Walkthrough

## 01. Initial Access-Key Recovery

### Objective

Recover the access key required to unlock the target environment using an artifact associated with the preceding Advent of Cyber challenge.

### Methodology

The initial stage examines an HTA file associated with the previous challenge material. The artifact is treated as data to inspect rather than a program to execute.

The documented command normalizes the file's content, removes selected delimiter characters and whitespace, and attempts Base64 decoding.

```bash
sed ':a;N;$!ba;s/[&_"]//g;s/[[:space:]]//g' \
  NorthPolePerformanceReview.hta | base64 -d > b64.txt
```

The resulting data requires additional decoding before the remaining payload can be processed in CyberChef.

### Decoding Workflow

```text
HTA Artifact
     |
     v
Content Normalization
     |
     v
Base64 Decoding
     |
     v
Additional Base64 Decoding
     |
     v
XOR Transformation
     |
     v
Image Rendering
     |
     v
Access-Key Recovery
```

### XOR Parameters

| Parameter | Value |
|---|---|
| Operation | XOR |
| Key | `23` |
| Key type | Decimal |
| Scheme | Standard |
| Final operation | Render Image |

### Evidence

![CyberChef key recovery](../docs/assets/01-cyberchef-key-recovery.png)

*Figure 01 — CyberChef artifact analysis associated with access-key recovery.*

### Result

The decoding sequence reveals the SQ4 access key required for the next stage. The recovered value is intentionally omitted from this public write-up.

### Security Relevance

The initial stage demonstrates why unfamiliar artifacts should be inspected in a controlled environment. Encoded content is not necessarily harmless, and layered transformations can conceal information that is not immediately visible in a file's original representation.

---

## 02. Network Service Enumeration

### Objective

Identify exposed TCP services and establish an initial map of the target's attack surface.

### Command

```bash
nmap -p22,25,8443,21337 -sCV TARGET -oN fullscan.tcp
```

Replace `TARGET` with the assigned lab address.

### Command Breakdown

| Option | Purpose |
|---|---|
| `-p22,25,8443,21337` | Restricts the scan to the listed TCP ports |
| `-sC` | Runs Nmap's default NSE scripts |
| `-sV` | Attempts service/version detection |
| `-oN fullscan.tcp` | Saves results in normal output format |

The command combines targeted port discovery with service identification and selected default script checks. Saving the results supports subsequent review and correlation.

### Observed Attack Surface

| Port | Service | Investigation Priority |
|---:|---|---|
| 22/TCP | SSH | Identify the role of the remote administration service |
| 25/TCP | Postfix SMTP | Examine exposed mail-service behavior |
| 8443/TCP | Nginx HTTPS | Inspect the mobile portal and web routes |
| 21337/TCP | Werkzeug/Python | Investigate the Python-backed application |

### Evidence

![Nmap results](../docs/assets/02-nmap-service-enumeration.png)

*Figure 02 — Network service enumeration results.*

### Analysis

The HTTPS portal, SMTP server, and Python-backed application may participate in separate stages of the same authentication workflow. This is an investigation hypothesis based on the observed service inventory, not an assertion that the services are necessarily vulnerable.

The next steps focus on identifying application entry points, exposed resources, and relationships between authentication and message delivery.

### Result

The service inventory established the reconnaissance priorities for the remainder of the assessment.

---

## 03. SMTP Service Enumeration

### Objective

Identify the mail service implementation and determine whether its exposed behavior provides useful information for later analysis.

### Methodology

The documented SMTP investigation uses Metasploit's auxiliary scanner modules:

```text
auxiliary/scanner/smtp/smtp_enum
auxiliary/scanner/smtp/smtp_version
```

SMTP version enumeration can reveal service implementation details, while enumeration behavior can provide information about the service's handling of candidate account names.

### Evidence

![SMTP enumeration](../docs/assets/03-smtp-enumeration.png)

*Figure 03 — SMTP service enumeration.*

### Analysis

The initial SMTP enumeration does not produce an immediate credential breakthrough. The investigation therefore shifts toward the HTTPS mobile portal while retaining SMTP as a potential component of the authentication and message-delivery workflow.

### Security Relevance

Mail infrastructure should be assessed in the context of the applications that use it. Even a correctly configured SMTP service cannot compensate for unsafe recipient construction or inadequate authorization in the application submitting a message.

---

## 04. Mobile Portal Reconnaissance

### Objective

Map the exposed application interfaces and identify the user workflows relevant to the protected banking operation.

### Methodology

The HTTPS service on port `8443` hosts a mobile-style portal containing multiple application interfaces.

The Hopflix login interface presents a pre-populated account identifier. This provides a starting point for examining the identity and the services associated with it.

### Evidence: Hopflix Login

![Hopflix login](../docs/assets/04-hopflix-login-surface.png)

*Figure 04 — Hopflix login interface.*

### Evidence: Inbox

![Hopflix inbox](../docs/assets/05-hopflix-inbox.png)

*Figure 05 — Hopflix inbox interface.*

### Analysis

The application interfaces and inbox provide context linking the target identity to a subsequent banking workflow. The inbox also indicates that protected charity funds are associated with an additional authentication process.

These observations help prioritize Hopflix authentication, banking login behavior, and the associated OTP flow for further testing.

### Result

The investigation identifies a potential progression from the mobile portal to the protected banking application.

---

## 05. Client-Side Source Inspection

### Objective

Determine whether browser-delivered code exposes security-sensitive values or implementation details.

### Methodology

Browser developer tools are used to inspect the portal's client-side resources and JavaScript.

JavaScript delivered to a user's browser is visible to that user. A value embedded in client-side code therefore cannot be assumed to remain confidential.

### Evidence: Mobile Source Review

![Mobile source review](../docs/assets/06-mobile-source-review.png)

*Figure 06 — Inspection of the mobile application's source resources.*

### Evidence: Client-Side Value

![Client-side secret](../docs/assets/07-app-javascript-secret.png)

*Figure 07 — Security-sensitive value identified in client-side JavaScript.*

### Finding: Client-Side Security-Sensitive Value Exposure

The supplied notes record an authentication-related value in JavaScript delivered to the browser.

The value's exact intended security role is not fully established by the supplied documentation. If it is intended to function as a confidential authentication secret, exposing it to the client undermines that security assumption.

### Potential Impact

Depending on the value's actual purpose, client-side exposure may allow an attacker to recover information intended to remain confidential, reconstruct part of an authentication workflow, or reduce the effectiveness of an associated security control.

### Defensive Recommendation

Confidential authentication secrets should remain on trusted server-side systems or within appropriately protected authenticator devices. Client-side code should contain only information that is safe to disclose to the user.

---

## 06. Bank Login Reconnaissance

### Objective

Understand the banking application's login request structure and establish a baseline for later authentication-flow analysis.

### Methodology

Burp Suite is used to inspect the banking login request and its HTTP behavior.

Request interception provides visibility into the HTTP method, path, parameters, headers, and response. This makes it possible to document the interaction without relying only on what the browser interface displays.

### Evidence

![Burp request](../docs/assets/08-burp-bank-login.png)

*Figure 08 — Banking login request inspected using Burp Suite.*

### Analysis

The documented account-number enumeration attempt does not immediately produce a useful result.

The investigation consequently shifts toward source and implementation analysis. Where source access is available within the authorized lab, it can explain validation and authentication behavior that may not be obvious from black-box requests alone.

### Result

The captured request establishes a baseline for examining the bank's authentication and OTP workflow.

---

## 07. Nginx Configuration Disclosure

### Objective

Identify accessible web resources that may reveal server-side routing or internal application structure.

### Methodology

FFUF is used to discover content on the HTTPS service.

```bash
ffuf -u https://TARGET:8443/FUZZ \
  -w /opt/SecLists/Discovery/Web-Content/quickhits.txt
```

Replace `TARGET` with the assigned lab address.

The `FUZZ` keyword represents the path component tested with entries from the selected wordlist.

### Evidence: Web Content Discovery

![FFUF results](../docs/assets/09-fuzz-nginx-config.png)

*Figure 09 — FFUF results from web content discovery.*

### Evidence: Backend Routing

![Nginx and uWSGI routing](../docs/assets/10-nginx-uwsgi-routing.png)

*Figure 10 — Disclosed routing information associated with Nginx and uWSGI.*

### Finding: Web-Server Configuration Disclosure

The discovered configuration provides insight into how Nginx routes requests and interacts with the Python application through uWSGI.

The configuration also suggests areas where file-resolution and fallback behavior should be examined.

### Potential Impact

Depending on which configuration artifacts are accessible, disclosure may reveal internal application paths, routing rules, backend relationships, framework details, and operational information that reduces uncertainty during subsequent testing.

The significance of the finding depends on the exact content exposed and whether the disclosure enables further exploitation.

### Defensive Recommendations

- Prevent direct retrieval of server configuration and internal application artifacts.
- Review `root`, `alias`, `try_files`, and upstream configuration for unsafe fallback behavior.
- Expose only explicitly intended public resources.
- Validate routing behavior during deployment testing.
- Ensure that application source and operational configuration do not reside within publicly retrievable paths.

---

## 08. Python Source-Code Disclosure

### Objective

Determine whether server-side Python application code can be retrieved through the web-facing service.

### Methodology

Following configuration discovery, the investigation tests whether the routing behavior exposes common Python application entry points. The supplied notes record that the primary application source is accessible.

### Evidence

![Application source disclosure](../docs/assets/11-main-py-credential-disclosure.png)

*Figure 11 — Python source exposure and credential-related application logic.*

### Observed Sensitive Information

The disclosed source contains references to:

- Database paths.
- Application route definitions.
- Authentication logic.
- Security-sensitive values.
- Challenge-specific flags.

### Finding: Server-Side Source-Code Exposure

Python application source is retrievable through a web-facing interface.

This is a significant information-disclosure issue because source code may reveal implementation details that materially reduce the effort required to investigate the application.

### Security Impact

Source disclosure can make it possible to inspect:

- Credential verification logic.
- Authentication state transitions.
- Input-validation rules.
- Database locations and schema references.
- Sensitive configuration values.
- Internal endpoints and security assumptions.

In this assessment, the source disclosure provides the basis for investigating credential storage and password-verification behavior.

### Defensive Recommendations

- Keep application source outside web-accessible document roots.
- Restrict access to internal files and deployment artifacts.
- Review reverse-proxy routing and fallback rules.
- Disable production debug behavior.
- Store secrets in an appropriate secret-management system.
- Add deployment checks for exposed source files and configuration artifacts.

Challenge-specific flags and secrets remain excluded from this portfolio document.

---

## 09. SQLite Credential Analysis

### Objective

Understand the target account's credential representation and identify how the application processes candidate passwords during verification.

### Methodology

The disclosed source identifies the location of a Hopflix SQLite database. After recovering the database within the authorized lab, its structure and relevant records are inspected using SQLite tooling.

The supplied notes describe an unusually long password representation. The source code indicates that each character is independently processed through repeated SHA-1 operations.

### Length Analysis

A SHA-1 digest contains 160 bits and is conventionally represented by 40 hexadecimal characters.

If each character is represented by a separate 40-character hexadecimal digest, a 12-character input would produce a representation of approximately:

```text
40 hexadecimal characters per digest
× 12 character digests
= 480 hexadecimal characters
```

This arithmetic illustrates how a long stored representation may correspond to a sequence of per-character digests. It is a structural observation, not proof by itself of a particular user's original password length.

### Finding: Custom Credential Representation

The application uses a custom password-processing construction involving repeated SHA-1 operations rather than a purpose-built password-storage scheme.

### Security Analysis

SHA-1 is a general-purpose cryptographic hash function, not a suitable standalone password-storage scheme.

Repeatedly applying a fast hash does not automatically provide the security properties of a dedicated password KDF. The overall construction matters, including work factor, salt handling, verification logic, and the presence of observable side channels.

In this challenge, the key concern is not solely the selected hash algorithm. The verification routine also creates a timing side channel.

### Defensive Recommendations

- Replace custom repeated SHA-1 transformations with a dedicated password-hashing scheme such as Argon2id, scrypt, or bcrypt.
- Use a unique salt for each password.
- Choose a work factor appropriate to the deployment.
- Use well-reviewed password-verification libraries.
- Assess verification control flow and response timing alongside the stored hash format.

A strong password KDF does not automatically prevent application-level information leakage.

---

## 10. Password Verification and Timing Side Channel

### Objective

Determine whether password verification reveals information about candidate credentials through observable processing time.

### Methodology

Inspection of the authentication logic reveals sequential processing associated with candidate characters and the expected password representation.

The documented behavior depends on how much of a candidate prefix matches the expected value.

### Evidence

![Password checking logic](../docs/assets/12-password-checking-logic.png)

*Figure 12 — Password verification logic examined during source review.*

### Observed Behavior

The reported behavior is summarized below.

```text
Candidate Prefix Matches
          |
          v
Additional Verification Work
          |
          v
Longer Processing Time
          |
          v
Observable Response-Time Difference
```

When a matching prefix causes additional work, the resulting latency may reveal information about the candidate credential.

### Finding: Secret-Dependent Authentication Timing

The authentication implementation performs measurably different amounts of work depending on the correctness of a candidate password prefix.

### Security Impact

If the timing signal is sufficiently consistent, an attacker may infer credential information incrementally rather than reversing the entire stored representation. The password-verification endpoint consequently risks becoming an information oracle.

### Defensive Recommendations

- Avoid verification logic that reveals how many characters or bytes match.
- Use established password-verification implementations.
- Use constant-time comparison where applicable.
- Avoid custom per-character processing of secret values.
- Apply rate limiting and monitoring to authentication endpoints.
- Avoid error responses that reveal partial correctness.
- Validate timing behavior under realistic network conditions.

Constant-time comparison is one component of a broader defense. It does not replace secure password storage or sound authentication architecture.

---

## 11. Timing-Based Credential Recovery

### Objective

Demonstrate how repeated response-time measurements can reveal information about candidate credentials when a vulnerable verification routine produces a distinguishable timing signal.

### Methodology

The documented approach treats timing analysis as a measurement problem rather than as reversal of the stored hash representation.

The conceptual sequence is:

1. Maintain the identified candidate prefix.
2. Select a candidate character for the next position.
3. Construct a candidate from the known prefix and the test character.
4. Submit repeated requests in the authorized lab.
5. Record observed response latency.
6. Compare measurements across candidates.
7. Evaluate the strongest candidate according to the measured signal.
8. Repeat for subsequent positions.

### Representative Pseudocode

The following is a conceptual illustration of the measurement loop, not the original exploit script.

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

Timing attacks are sensitive to environmental noise. A single request is generally insufficient to establish a reliable difference.

Relevant factors include:

- Network latency and jitter.
- Server load and concurrent requests.
- Connection-establishment overhead.
- Application caching.
- Request throttling.
- Repetition count.
- The statistical method used to compare measurements.

The supplied notes indicate that the AttackBox environment was used because additional VPN latency can make small timing differences harder to distinguish.

### Finding: Credential Information Leakage Through Timing

The password-verification behavior provides a measurable signal associated with candidate-prefix correctness. The documented assessment uses that signal to support incremental credential recovery.

### Defensive Recommendations

- Remove useful secret-dependent differences in verification behavior.
- Use established password-verification libraries.
- Avoid unnecessary branching based on secret content.
- Apply reasonable authentication rate limits.
- Measure timing distributions during security testing rather than relying on isolated requests.

The underlying defensive goal is to ensure that observed response timing does not provide a practical means of learning secret information.

---

## 12. Hopflix Authentication

### Objective

Validate the recovered challenge credential against Hopflix and establish whether the resulting session advances the investigation to the next workflow.

### Methodology

Following the credential-analysis stage, the resulting credential is used within the authorized lab to authenticate to Hopflix.

### Evidence

![Successful Hopflix login](../docs/assets/13-hopflix-authentication-success.png)

*Figure 13 — Successful Hopflix authentication.*

### Result

Authentication succeeds, establishing the transition from credential analysis to the subsequent application workflow.

### Security Relevance

The successful login demonstrates how information disclosure and a timing side channel can become a practical application-entry path.

The weakness is therefore not limited to a theoretical difference in response time; in the documented challenge chain, the credential-recovery stage supports access to another application component.

### Defensive Recommendations

- Avoid uncontrolled password reuse between independent applications.
- Use centralized identity management where appropriate.
- Apply strong session isolation and least-privilege authorization.
- Review cross-application trust relationships.
- Ensure that compromise of one account does not automatically grant access to unrelated protected workflows.

---

## 13. Hopsec Bank and OTP Workflow

### Objective

Map the banking application's second authentication stage and identify the trust assumptions governing OTP delivery.

### Methodology

The banking application prompts the authenticated user to select an authorized email destination for OTP delivery.

### Evidence: Recipient Selection

![Authorized email selection](../docs/assets/14-bank-authorized-email-selection.png)

*Figure 14 — Banking application's authorized email-selection interface.*

### Evidence: OTP Verification Prompt

![Two-factor authentication prompt](../docs/assets/15-bank-two-factor-prompt.png)

*Figure 15 — Banking application's second-factor verification prompt.*

### Analysis

The investigation now focuses on the OTP workflow.

A randomly generated OTP can add protection to authentication, but the strength of the workflow depends on more than the unpredictability of the code.

The system must also ensure that:

- The recipient is authorized.
- The message is delivered only to an intended destination.
- The application and mail service interpret the recipient consistently.
- The code is bound to the appropriate authentication transaction.
- The code expires and cannot be reused.
- Failed verification attempts are appropriately restricted.

### Result

The recipient-selection process becomes the focus of the subsequent application-logic analysis.

---

## 14. OTP Recipient Validation Weakness

### Objective

Determine whether the OTP recipient is validated consistently and securely before the application initiates message delivery.

### Methodology

Source inspection identifies the OTP-generation workflow and the application's recipient-validation condition.

The supplied source pattern is:

```python
if domain not in allowed_domains and to_addr not in allowed_emails:
    return -1
```

### Logic Analysis

The condition rejects the recipient only if **both** checks fail.

In Boolean terms, the rejection behavior can be expressed as:

| Domain Check | Exact Email Check | Rejected by the Condition |
|---|---|---|
| Fails | Fails | Yes |
| Passes | Fails | No |
| Fails | Passes | No |
| Passes | Passes | No |

Consequently, passing either check may be enough to avoid rejection, even if the recipient does not meet both intended authorization requirements.

The precise impact depends on the downstream address-handling and mail-delivery behavior.

### Finding: Insufficiently Strict Recipient Validation

The application does not implement an unambiguous recipient-authorization policy through the documented condition.

### Security Impact

A destination that should have been rejected may reach the downstream mail-delivery component. If the application and SMTP service interpret the destination differently, the discrepancy can create an opportunity for OTP interception.

### Why OTP Randomness Is Not Enough

Different parts of the OTP lifecycle address different security requirements:

| Control | Security Objective |
|---|---|
| Cryptographically secure generation | Makes the code difficult to predict |
| Recipient authorization | Restricts who may receive the code |
| Secure message delivery | Protects the confidentiality of the message |
| Transaction binding | Ensures the code authorizes the intended action |
| Expiration and single use | Limits the opportunity for misuse |
| Attempt restrictions | Constrains repeated guessing and verification attempts |

A strong OTP generator cannot compensate for an insecure delivery path.

### Defensive Recommendations

- Enforce an explicit allowlist of authorized recipient identities.
- Parse and normalize email addresses before applying authorization rules.
- Avoid relying on loosely defined domain-substring checks.
- Reject malformed or ambiguous recipient values.
- Apply authorization to one canonical representation.
- Reuse the validated recipient representation consistently throughout message delivery.

---

## 15. SMTP Delivery-Path Interception

### Objective

Determine whether differences between application-level recipient validation and downstream SMTP processing can expose the OTP to an unintended destination.

### Methodology

The supplied challenge notes describe a specially structured recipient that interacts differently with application validation and downstream mail delivery.

The trust-boundary problem is summarized below.

<div class="attack-chain">
  <div class="attack-step">User-Supplied Recipient</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Application Validation</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">SMTP Processing</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Inconsistent Interpretation</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Unexpected Destination</div>
</div>

The core issue is a validation and parser-consistency failure: separate components do not enforce the same recipient interpretation.

### Controlled SMTP Listener

Within the authorized lab, the supplied notes document use of `aiosmtpd` to observe resulting message delivery.

```bash
aiosmtpd -n -l 0.0.0.0:25
```

This command starts an SMTP listener on port 25, bound to all network interfaces. It requires the necessary permissions and should only be used in an appropriately controlled environment with network access restricted to the authorized test.

### Evidence

![Captured OTP](../docs/assets/17-smtp-otp-capture.png)

*Figure 17 — SMTP message capture associated with the OTP workflow.*

### Finding: OTP Delivery Through Inconsistent Recipient Handling

The combination of permissive recipient validation and differing downstream parsing behavior creates a path through which the OTP can be delivered to an unintended destination in the challenge.

### Security Impact

Successful interception undermines the confidentiality of the second authentication factor.

The attack does not require predicting the randomly generated OTP. Instead, it targets the system responsible for authorizing and delivering the code.

### Defensive Recommendations

- Parse recipient addresses using a trusted email-address parser.
- Reject malformed, ambiguous, or unsupported address forms.
- Enforce an explicit allowlist of authorized recipient identities.
- Canonicalize the recipient once and reuse the validated representation.
- Do not pass untrusted recipient strings directly into downstream components.
- Ensure the SMTP envelope recipient matches the application's authorized destination.
- Record and monitor OTP delivery destinations.
- Rate-limit OTP requests and prevent unauthorized destination changes during an authentication transaction.

---

## 16. Banking Authentication and Final Impact

### Objective

Validate the combined impact of the discovered authentication and OTP delivery weaknesses within the authorized challenge.

### Methodology

After obtaining the OTP through the documented lab message-delivery path, the code is submitted to the banking application to complete the second authentication stage.

### Evidence: Authenticated Banking Dashboard

![Bank dashboard](../docs/assets/18-bank-dashboard.png)

*Figure 18 — Banking dashboard reached after authentication.*

### Result

The documented workflow completes authentication and exposes the protected charity-fund operation.

### Evidence: Protected Operation

![Funds released](../docs/assets/19-charity-funds-released.png)

*Figure 19 — Charity-fund operation reached in the challenge.*

### Evidence: Redacted Final Result

![Final result](../docs/assets/20-final-result-redacted.png)

*Figure 20 — Final challenge result, with the secret intentionally redacted.*

```text
THM{REDACTED_FOR_PUBLICATION}
```

### Combined Impact

The documented progression is:

1. Recover the initial access key.
2. Enumerate the exposed services.
3. Discover application interfaces through the mobile portal.
4. Identify client-side and server-side information exposure.
5. Analyze the credential representation and verification routine.
6. Use timing observations to support credential recovery.
7. Authenticate to Hopflix.
8. Investigate the bank's OTP workflow.
9. Identify recipient-validation and delivery inconsistencies.
10. Complete the banking authentication flow and reach the protected operation.

### Potential Business Impact

In a real application with comparable weaknesses, possible consequences could include:

- Unauthorized account access.
- Compromise of authentication-related secrets.
- Circumvention of intended second-factor protections.
- Exposure of sensitive application implementation details.
- Unauthorized access to protected financial workflows.
- Reduced confidence in authentication and authorization controls.

These are potential consequences of the demonstrated vulnerability classes, not claims about an actual banking system.

---

## End-to-End Attack Chain

The assessment correlates findings across the application's network, source, credential, authentication, and mail-delivery layers.

<div class="attack-chain">
  <div class="attack-step">Initial Artifact</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Access-Key Recovery</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Network Enumeration</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Portal Discovery</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Source Disclosure</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Credential Analysis</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Timing Analysis</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Hopflix Authentication</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">OTP Validation Weakness</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">SMTP Interception</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Protected Banking Workflow</div>
</div>

### Attack-Chain Interpretation

| Phase | Evidence or Observation | Security Significance |
|---|---|---|
| Artifact analysis | Encoded HTA artifact | Provides the initial challenge access key |
| Network enumeration | Four identified service ports | Establishes the reachable service surface |
| Application reconnaissance | Mobile portal and inbox | Identifies relevant user workflows |
| Source inspection | Client-side and Python source exposure | Reveals implementation and trust assumptions |
| Credential analysis | Custom repeated-SHA-1 representation | Establishes how stored credentials are represented |
| Timing analysis | Prefix-dependent verification time | Creates a potential information oracle |
| Cross-application authentication | Successful Hopflix login | Demonstrates practical use of recovered credentials |
| OTP analysis | Permissive recipient-validation logic | Exposes a weakness in destination authorization |
| SMTP analysis | OTP observed through controlled message capture | Demonstrates the delivery-path impact in the lab |
| Final workflow | Banking dashboard and protected operation | Establishes the combined impact of the attack chain |

The security impact of a chained attack can exceed that of any individual finding. Remediation should therefore address both the individual defects and the trust relationships that allow the findings to combine.

---

## Security Findings and Severity

The following qualitative ratings are preserved from the supplied assessment. They are assessment judgments based on the described challenge behavior, not independently calculated CVSS scores.

| ID | Finding | Qualitative Severity | Primary Impact |
|---|---|---|---|
| F-01 | Web-accessible application source | Critical | Discloses authentication logic, paths, and security-sensitive information |
| F-02 | OTP recipient-validation weakness | Critical | May permit OTP delivery to an unintended destination |
| F-03 | Password-verification timing side channel | High | May enable incremental credential recovery |
| F-04 | Exposed Nginx configuration | High | Reveals routing behavior and backend architecture |
| F-05 | Client-side security-sensitive value | High | May weaken the associated authentication control |
| F-06 | Cross-application credential reuse or trust | High | Enables movement between applications after credential recovery |

### Severity Interpretation

- **Critical:** A finding that, in the demonstrated chain, directly contributes to bypassing a significant authentication or authorization boundary.
- **High:** A finding that exposes sensitive information or provides a meaningful path toward credential compromise or further exploitation.

A production assessment should determine severity based on validated exploitability, affected assets, required privileges, exposure, and business impact. The qualitative labels above should not be interpreted as formal CVSS ratings.

### Evidence and Confidence Boundaries

The source write-up documents the principal observations and associated screenshots. However, not every underlying implementation detail is reproduced verbatim, and the supplied material does not contain formal CVSS vectors or independently verified production context.

The following distinctions therefore apply:

- **Documented observations:** Service ports, source exposure, credential representation, the validation expression, successful authentication, and the supplied screenshot references.
- **Assessment interpretations:** The security implications and qualitative severity assigned to those observations.
- **Potential production impact:** Consequences that could arise in systems with comparable weaknesses; these are not claims about real production environments.

No additional CVEs, exploit identifiers, or MITRE ATT&CK mappings are asserted.

---

## Root Cause Analysis

### F-01 — Web-Accessible Application Source

**Observed weakness:** Python application source can be retrieved through a web-facing service.

**Root-cause hypothesis:** Unsafe file-resolution or routing behavior combined with inadequate protection of internal application files.

**Why it matters:** Source disclosure reveals implementation details that may expose authentication weaknesses, database paths, and security-sensitive values.

**Recommended control:** Separate application code from public document roots, harden reverse-proxy rules, and verify that internal files cannot be retrieved through unexpected routes.

### F-02 — OTP Recipient-Validation Weakness

**Observed weakness:** The supplied recipient-validation condition does not require both the allowed-domain and allowed-email checks to pass.

**Root cause:** Boolean logic that rejects input only when both checks fail, combined with inconsistent interpretation of recipient values across the application and mail-delivery layers.

**Why it matters:** A recipient that should have been rejected may reach the mail-delivery component.

**Recommended control:** Enforce strict recipient allowlisting, canonical parsing, unambiguous authorization logic, and consistent downstream handling.

### F-03 — Password Timing Side Channel

**Observed weakness:** Response time varies with candidate-prefix correctness.

**Root-cause hypothesis:** Secret-dependent execution and verification work that varies according to the candidate's content.

**Why it matters:** An attacker may infer credential information incrementally.

**Recommended control:** Use established password-verification libraries, eliminate useful secret-dependent timing differences, and monitor repeated authentication attempts.

### F-04 — Exposed Nginx Configuration

**Observed weakness:** Web discovery reveals routing and backend configuration details.

**Root-cause hypothesis:** Inadequate protection of configuration artifacts or unsafe routing behavior.

**Why it matters:** Configuration disclosure reduces uncertainty about the application's architecture and can guide further testing.

**Recommended control:** Restrict access to configuration artifacts and review reverse-proxy routing against an explicit resource allowlist.

### F-05 — Client-Side Security-Sensitive Value

**Observed weakness:** A value associated with an authentication workflow is present in browser-delivered code.

**Root-cause hypothesis:** Treating client-side code as a confidential storage location.

**Why it matters:** Users can inspect and extract data delivered to their own browsers.

**Recommended control:** Keep confidential authentication material server-side or within appropriately protected authenticator devices.

### F-06 — Cross-Application Credential Reuse or Trust

**Observed weakness:** Credential recovery supports authentication to another application component.

**Root-cause hypothesis:** Insufficient identity isolation or uncontrolled credential reuse between applications.

**Why it matters:** Compromise of one authentication boundary can enable access to another.

**Recommended control:** Use independent credentials or carefully designed centralized identity, strong session isolation, and least-privilege authorization.

---

## Remediation and Defensive Recommendations

The following controls are tied to the documented findings and the attack chain.

### 1. Protect Application Source and Configuration

**Finding:** Web-accessible Python source and exposed server configuration.

**Risk:** Disclosure of authentication logic, internal routes, database paths, and operational details.

**Recommended controls:**

- Keep Python source, database files, environment files, and server configuration outside public document roots.
- Use explicit routing rules that expose only intended resources.
- Review Nginx `root`, `alias`, `try_files`, and upstream configuration for unsafe fallback behavior.
- Disable unnecessary debug and development endpoints in production.
- Include source-disclosure checks in deployment validation.

### 2. Implement Secure Password Storage

**Finding:** Custom credential processing uses repeated SHA-1 operations.

**Risk:** The stored representation and verification design do not provide the assurance expected from a purpose-built password KDF.

**Recommended controls:**

- Replace custom repeated-SHA-1 transformations with Argon2id, scrypt, or bcrypt.
- Generate a unique salt for each password.
- Choose a work factor appropriate to the application's performance and security requirements.
- Avoid custom credential-storage constructions without careful security review.
- Evaluate the verification routine for information leakage as well as the stored hash format.

### 3. Eliminate Authentication Timing Leakage

**Finding:** Verification time varies with candidate-prefix correctness.

**Risk:** Observable latency can reveal information about candidate credentials.

**Recommended controls:**

- Use trusted password-verification implementations.
- Avoid per-character comparisons that stop at the first mismatch.
- Use constant-time comparison for secret material where applicable.
- Normalize invalid-credential responses without introducing unnecessary secret-dependent behavior.
- Monitor unusual authentication volume and repeated failure patterns.
- Test timing distributions under realistic network conditions.

### 4. Enforce Strict OTP Recipient Authorization

**Finding:** The recipient-validation condition does not unambiguously enforce the intended policy.

**Risk:** A destination that should be rejected may reach the mail-delivery component.

**Recommended controls:**

- Use an explicit allowlist of authorized recipient identities.
- Avoid loose substring or domain-only checks where exact identity authorization is required.
- Parse and canonicalize addresses before applying authorization rules.
- Apply policy to the parsed representation rather than untrusted raw input.
- Reject ambiguous or malformed recipient values.
- Ensure downstream SMTP handling uses the same validated destination.

### 5. Harden OTP Lifecycle Management

**Finding:** OTP confidentiality depends on several linked validation and delivery stages.

**Risk:** A weak delivery or verification control may undermine the intended second authentication factor.

**Recommended controls:**

- Generate OTPs using a cryptographically secure random-number generator.
- Associate each OTP with a specific user, session, and authentication transaction.
- Enforce short expiration windows.
- Invalidate an OTP after successful use.
- Limit verification attempts and OTP issuance frequency.
- Avoid revealing OTP validity through distinguishable responses.
- Record delivery metadata without unnecessarily logging the OTP itself.

### 6. Secure Email Delivery

**Finding:** The challenge demonstrates inconsistent recipient handling across application and SMTP processing.

**Risk:** OTP messages may be redirected to destinations that the application intended to reject.

**Recommended controls:**

- Validate the application recipient and actual SMTP envelope recipient.
- Avoid constructing mail commands or recipient values through unsafe string manipulation.
- Use established email libraries and well-defined address-parsing behavior.
- Restrict mail delivery to intended destinations.
- Monitor anomalous OTP delivery requests and recipient changes.
- Confirm that application-level authorization and SMTP delivery cannot diverge.

### 7. Strengthen Identity Separation

**Finding:** Credential recovery supports a transition from Hopflix to the banking workflow.

**Risk:** Compromise of one credential or application may affect another service.

**Recommended controls:**

- Avoid uncontrolled password reuse across independent applications.
- Use a trusted identity provider where centralized authentication is appropriate.
- Apply least privilege to service accounts and application identities.
- Isolate sessions, tokens, and authorization scopes between applications.
- Ensure that authentication to one application does not automatically authorize unrelated protected operations.

### 8. Improve Security Testing and Monitoring

**Finding:** The combined chain crosses web routing, credential verification, application authorization, and message delivery.

**Risk:** Testing each component independently may miss weaknesses that emerge at integration boundaries.

**Recommended controls:**

- Include source-disclosure checks in application security testing.
- Test authentication endpoints for timing discrepancies.
- Validate recipient-handling logic using malformed and ambiguous input cases.
- Review reverse-proxy and backend routing configurations.
- Monitor repeated failed authentication and unusual OTP delivery activity.
- Retest remediation changes to confirm that the original attack chain is no longer viable.

---

## Lessons Learned

### 1. Enumerate Before Exploiting

The initial service inventory established the attack surface and helped identify the most relevant application components.

A structured reconnaissance phase reduces wasted effort and creates a reliable basis for subsequent investigation.

### 2. Treat Source Code as Security Evidence

Application source can reveal actual validation rules, data flows, credential-handling logic, and trust assumptions that are difficult to infer from the user interface alone.

Where source access is authorized, code review can help explain why a vulnerability exists and how multiple findings interact.

### 3. Follow Data Across Trust Boundaries

The OTP stage demonstrates why security testing should follow user-controlled data from input, through validation, to downstream processing.

A value is not secure simply because one component accepts it under a particular interpretation.

### 4. Timing Attacks Are Measurement Problems

Side-channel analysis requires identifying a repeatable signal among noisy observations.

Repeated samples, consistent conditions, and careful statistical interpretation are important. Apparent timing differences should be validated rather than assumed to represent secret-dependent behavior.

### 5. Strong OTP Generation Does Not Guarantee Secure Authentication

OTP security depends on generation, recipient authorization, delivery, transaction binding, expiration, and verification.

A weakness in any critical part of this lifecycle may undermine the protection offered by the other components.

### 6. Parser Differences Create Security Risks

Application validation and downstream processing must agree on the meaning of an input.

When two components interpret the same value differently, an attacker may be able to cross a boundary that the application intended to enforce.

### 7. Chained Vulnerabilities Require Architectural Remediation

Fixing one exposed file or validation condition may close an individual exploitation path, but the system should also be reviewed for related weaknesses.

Defense in depth requires secure routing, credential storage, authentication logic, message delivery, and authorization boundaries.

---

## Conclusion

BreachBlocker Unlocker demonstrates a multi-stage compromise built from weaknesses across connected application components.

The investigation begins with access-key recovery and network enumeration, progresses through source and configuration disclosure, and uses application behavior to identify a credential-recovery timing side channel. It then follows the authentication flow into an OTP-protected banking workflow, where recipient-validation and SMTP delivery inconsistencies expose a further weakness.

The most important lesson is architectural: **security controls must remain consistent across every component that handles identity, secrets, validation, and authorization.**

A secure OTP generator cannot compensate for unsafe recipient validation. A login interface cannot compensate for exposed application source. A password-storage scheme cannot compensate for a verification routine that leaks information through timing.

The assessment reinforces a repeatable security methodology:

<div class="attack-chain">
  <div class="attack-step">Enumerate</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Inspect</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Correlate</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Analyze</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Validate</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Remediate</div>
</div>

By documenting the attack chain alongside its root causes and defensive controls, this challenge provides practical experience in web application assessment, source-code review, authentication analysis, side-channel reasoning, and security engineering.

---

## References and Evidence

### Evidence Artifacts

The screenshot paths below are preserved from the supplied source. They are referenced relative to `docs/index.md` exactly as originally provided.

| Evidence | Image Path |
|---|---|
| Initial access-key recovery | `../docs/assets/01-cyberchef-key-recovery.png` |
| Network service enumeration | `../docs/assets/02-nmap-service-enumeration.png` |
| SMTP enumeration | `../docs/assets/03-smtp-enumeration.png` |
| Hopflix login surface | `../docs/assets/04-hopflix-login-surface.png` |
| Hopflix inbox | `../docs/assets/05-hopflix-inbox.png` |
| Mobile source review | `../docs/assets/06-mobile-source-review.png` |
| Client-side security-sensitive value | `../docs/assets/07-app-javascript-secret.png` |
| Bank login request | `../docs/assets/08-burp-bank-login.png` |
| Web content discovery | `../docs/assets/09-fuzz-nginx-config.png` |
| Nginx and uWSGI routing | `../docs/assets/10-nginx-uwsgi-routing.png` |
| Python source disclosure | `../docs/assets/11-main-py-credential-disclosure.png` |
| Password verification logic | `../docs/assets/12-password-checking-logic.png` |
| Successful Hopflix authentication | `../docs/assets/13-hopflix-authentication-success.png` |
| Authorized recipient selection | `../docs/assets/14-bank-authorized-email-selection.png` |
| OTP verification prompt | `../docs/assets/15-bank-two-factor-prompt.png` |
| OTP generation logic | `../docs/assets/16-otp-generation-logic.png` |
| SMTP OTP capture | `../docs/assets/17-smtp-otp-capture.png` |
| Banking dashboard | `../docs/assets/18-bank-dashboard.png` |
| Charity-fund operation | `../docs/assets/19-charity-funds-released.png` |
| Redacted final result | `../docs/assets/20-final-result-redacted.png` |

> **Repository validation note:** The original document uses paths beginning with `../docs/assets/`. Those references are deliberately preserved rather than silently rewritten. Confirm that they resolve correctly in the repository's actual directory structure before publishing.

### Tools and Documentation

- [Nmap](https://nmap.org/)
- [Metasploit Framework](https://docs.metasploit.com/)
- [FFUF](https://github.com/ffuf/ffuf)
- [Burp Suite](https://portswigger.net/burp)
- [CyberChef](https://gchq.github.io/CyberChef/)
- [SQLite](https://www.sqlite.org/)
- [Python](https://www.python.org/)
- [aiosmtpd](https://aiosmtpd.readthedocs.io/)
- [TryHackMe](https://tryhackme.com/)

The links above are the tool and platform URLs provided in the original documentation.

---

## Responsible Use

This documentation was created for authorized cybersecurity training and CTF environments. Techniques described here should only be used against systems for which explicit permission to test has been granted.

Evidence from real assessments should be collected proportionately, stored securely, and handled according to the applicable rules of engagement and disclosure requirements.
