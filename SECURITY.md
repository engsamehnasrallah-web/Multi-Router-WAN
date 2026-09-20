# Security Policy

## Scope

This repository is an educational Cisco Packet Tracer portfolio project simulating a fictional company's network (NovaTech). It contains no production systems, no real infrastructure, and no live services — only configuration files, documentation, and a `.pkt` simulation file.

## Reporting a Vulnerability

Since this project does not run any live service or expose any real network, there is no attack surface to report a vulnerability against in the traditional sense. However, if you notice:

- Credentials, IP ranges, or configuration details that appear to belong to a real organization rather than the fictional NovaTech scenario
- A security-relevant mistake in the documented configuration that misrepresents a genuine Cisco IOS best practice

please open an issue on this repository, or contact the author directly via [GitHub](https://github.com/engsamehnasrallah-web).

## Notes on the Passwords in This Repository

All passwords, secrets, and community strings shown in `configs/*.txt` and throughout the documentation are **lab-only values** used purely for this Packet Tracer simulation. They are not reused on any real device or account, and are intentionally simple since the project's security scope is limited to demonstrating baseline Cisco IOS access-control concepts (console/VTY authentication, `enable secret`, `service password-encryption`) rather than production-grade hardening.
