# Labs

Brucehaus Labs is a small, opinionated collection of exercises for turning
real machines into serious nodes. Each lab is:

- Minimal in scope
- Explicit in configuration
- Paired with clear verification steps

The goal is muscle memory, not theory.

---

## Current labs

### SSH Hardening

**Folder:** [`ssh-hardening`](ssh-hardening/)

Take a stock Ubuntu host and:

- Lock inbound traffic behind a default-deny firewall
- Apply a simple, sane network posture via `sysctl`
- Run OpenSSH with key-only authentication
- Verify behavior with status checks, port inspection, and logs

---

### Golden USB

**Folder:** [`golden-usb`](golden-usb/)  
Placeholder for a future lab focused on building a trusted USB baseline:
known contents, checksums, and repeatable provisioning.

---

### Alpine Drills

**Folder:** [`alpine-drills`](alpine-drills/)  
Placeholder for a future lab centered on fast, repeatable Linux practice
using Alpine and small daily exercises.

---

All labs share the same posture:

> Clear steps, minimal assumptions, and an insistence on “show me” verification.
