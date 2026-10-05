# BreachBlocker Unlocker — Technical Notes

## Attack Path

```text
AOC key recovery
→ Nmap
→ SMTP/HTTPS enumeration
→ source disclosure
→ SQLite analysis
→ timing side channel
→ credential recovery
→ Hopsec Bank
→ OTP recipient validation weakness
→ SMTP interception
→ charity-fund authorization
```

## Core Weaknesses

- exposed application source/configuration
- client-side authentication secret
- timing side channel in credential verification
- custom repeated SHA-1 password construction
- permissive OTP recipient validation
- SMTP/application parser discrepancy
- cross-application credential reuse

## Public Flag

`THM{REDACTED_FOR_PUBLICATION}`
