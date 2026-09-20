# Security Baseline

A basic Cisco IOS security baseline is applied consistently across all four routers. This project is not a dedicated security project — the scope is intentionally limited to fundamental access-control hardening.

## Applied on Every Router (HQ-R, Core-R, Branch01-R, Branch02-R)

| Control | Purpose |
|---|---|
| `hostname <name>` | Clear device identification |
| `enable secret <password>` | MD5-hashed password required to reach Privileged EXEC mode |
| `line console 0` + `password` + `login` | Authentication required for physical console access |
| `line vty 0 4` + `password` + `login` | Authentication required for remote (Telnet) access, across all 5 VTY lines |
| `service password-encryption` | Obfuscates plaintext passwords in the running-config (type 7 encoding) |

## Design Notes

- Console and VTY passwords are intentionally different from each other on each device, and different from the enable secret — separating each access layer so that compromising one credential doesn't automatically expose the others.
- No SSH was configured in this project (Telnet/VTY password auth only) — SSH and more advanced access control (ACLs, AAA) are reserved for a dedicated security-focused project later in the portfolio.
- No `service password-encryption` bypasses `enable secret`, which is already MD5-hashed by default and unaffected by this command; it primarily protects the console/VTY line passwords, which are stored in plaintext (type 0) unless this command is applied.

## Verified Behavior

Console line lockout was tested manually: after repeated incorrect password attempts, IOS enforced its default retry limit and returned an access-denied condition before allowing further attempts — confirming the authentication mechanism is active, not just configured.
