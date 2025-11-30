# SSH Hardening Lab

Turn a stock Ubuntu host into a small, serious node:

- Default-deny firewall with a single explicit SSH exception  
- Basic kernel network hardening (`sysctl`)  
- OpenSSH server with **key-only** authentication  
- Simple verification habits (status, ports, logs)

This lab is meant to be small, repeatable, and practical — something you can rerun on fresh machines until it’s muscle memory.

---

## Goals

By the end of the lab you will:

- Lock inbound traffic with `ufw` and only allow `22/tcp` on purpose  
- Apply a minimal network-hardening profile with `/etc/sysctl.d/99-network-hardening.conf`  
- Install and enable `openssh-server`  
- Configure SSH for:
  - Protocol 2 (modern default)
  - Pubkey authentication
  - No password logins
  - No root logins
- Generate an `ed25519` keypair and populate `~/.ssh/authorized_keys`
- Prove the setup with `ssh user@localhost` and basic log inspection

This is not a full security benchmark — it’s a clean, opinionated baseline.

---

## Prerequisites

- Ubuntu 22.04+ (server or desktop)
- A non-root user with `sudo`
- Physical/console access or an alternate way in if you lock yourself out
- `openssh-server` available from the distro repos

Optional but useful:

- Basic familiarity with the shell (`cd`, `ls`, `sudo`, editing with `nano`)

---

## Files in this lab

Inside `ssh-hardening/`:

- `ssh-hardening-lab.md`  
  Full step-by-step walkthrough of the lab, including commands and verification.

- `config/99-network-hardening.conf`  
  Example `sysctl` drop-in for baseline IPv4/IPv6 network posture.

- `config/sshd_config.example`  
  Example SSH server configuration with key-only authentication.

- `assets/posters/`  
  Optional artwork / poster files for this lab (for printing or reference).

---

## High-level steps

1. **Baseline & tools**
   - Update the system with `apt`
   - Install `ufw`, `net-tools`, `iproute2`, `curl`, `dnsutils`, etc.

2. **Firewall**
   - `ufw default deny incoming`
   - `ufw default allow outgoing`
   - `ufw allow 22/tcp`
   - `ufw enable` and confirm status

3. **Kernel network hardening**
   - Drop `99-network-hardening.conf` into `/etc/sysctl.d/`
   - Apply with `sudo sysctl --system`

4. **SSH server**
   - Install `openssh-server`
   - Enable and start `ssh` with `systemctl`
   - Use `ss`/`netstat` to confirm port 22 is listening

5. **Harden `sshd_config`**
   - Enable pubkey auth
   - Disable password and root logins
   - Keep Protocol 2 and basic keepalive settings
   - Validate with `sudo sshd -t` and restart the service

6. **Keys**
   - Generate an `ed25519` keypair with `ssh-keygen`
   - Append the public key to `~/.ssh/authorized_keys`
   - Fix permissions on `~/.ssh` and `authorized_keys`

7. **Verification**
   - Connect with `ssh user@localhost`
   - Confirm that password auth is rejected
   - Optionally watch log entries with `journalctl -fu ssh`

---

## Safety and scope

- Only run this lab on systems you control.
- If SSH is your **only** access to a remote host, make sure key-based login works *before* disabling password auth.
- Always adapt this baseline to match your own environment and any organizational policies.

This lab lives under the `labs` repo so it can grow alongside other drills (Golden USB, Alpine drills, etc.), and be reused whenever you’re bringing a new machine into your orbit.
