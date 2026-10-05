# BreachBlocker Unlocker — TryHackMe CTF Walkthrough

> Hard-difficulty Web Security challenge documenting service enumeration, source disclosure, timing-side-channel credential recovery, SMTP/OTP abuse and banking authorization bypass.

## Attack Chain

`Key Recovery → Nmap → SMTP/HTTPS Enumeration → Source Disclosure → Timing Side Channel → Credential Recovery → Bank Pivot → OTP Delivery Abuse → SMTP Interception → Authorization`

## Skills Demonstrated

- Nmap and service fingerprinting
- Metasploit SMTP enumeration
- Burp Suite request analysis
- ffuf content discovery
- nginx/uWSGI architecture analysis
- Python and JavaScript source review
- SQLite investigation
- Timing-side-channel analysis
- Authentication testing
- SMTP/OTP interception
- Authorization-boundary analysis

## Public Flag Policy

All challenge flags and sensitive answer strings are intentionally redacted from the public portfolio:

`THM{REDACTED_FOR_PUBLICATION}`

## Documentation

- [Full Walkthrough](Documentation/Documentation.md)
- [Word Document](Documentation/Documentation.docx)
- [GitHub Pages](docs/index.md)
- [Technical Notes](Resources/notes.md)

## Responsible Use

This repository documents an authorized TryHackMe laboratory. Do not apply these techniques against systems without explicit permission.

## Author

**Anurag Revankar** — [GitHub](https://github.com/anurag-rvnkr1)
