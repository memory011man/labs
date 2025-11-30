# SSH Hardening Lab

Goal: take a stock Ubuntu host and turn it into a quiet, deliberate node with:

- Default-deny firewall using `ufw`
- Basic kernel network hardening (`sysctl`)
- OpenSSH server with **key-only** authentication
- Simple, repeatable verification steps

This lab assumes:

- Ubuntu 22.04+ (server or desktop)
- A non-root user with `sudo` (we’ll call them `user`)
- You have console/physical access if you break SSH

---

## Phase 0 – Orientation

Sanity check who and where you are:

```bash
whoami
hostname
pwd
ip a | sed -n '1,15p'
You should be:

Logged in as your normal user (user)

On the machine you intend to harden (host)

---

Phase 1 – Baseline & tools
1.1 Update and install utilities

sudo apt update && sudo apt upgrade -y

sudo apt install -y \
  ufw \
  net-tools iproute2 traceroute \
  curl wget dnsutils \
  nmap tcpdump tshark

Verify:
ufw --version
ip a
nmap --version

---

Phase 2 – Firewall: default deny inbound
2.1 Configure ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
sudo ufw status verbose
Expected:

Status: active

Default: deny (incoming), allow (outgoing), ...

2.2 See what’s listening (for awareness)
sudo ss -tulpen
# or:
sudo netstat -tulpen
Just note what ports are open and which processes own them. We’ll trim later as needed.

---

Phase 3 – Kernel network hardening (sysctl)

Create a drop-in file:
sudo nano /etc/sysctl.d/99-network-hardening.conf

Paste the contents from config/99-network-hardening.conf in this repo.

Apply it:
sudo sysctl --system

Quick checks:
sysctl net.ipv4.ip_forward
sysctl net.ipv4.tcp_syncookies

Expect:
net.ipv4.ip_forward = 0

net.ipv4.tcp_syncookies = 1

---

Phase 4 – SSH server bring-up & hardening
4.1 Install and start sshd
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
sudo systemctl status ssh

You want Active: active (running).

Allow SSH through ufw:
sudo ufw allow 22/tcp
sudo ufw status numbered

4.2 Confirm it’s listening
sudo ss -tulpen | grep ':22'

You should see LISTEN entries for sshd.

---

4.3 Harden sshd_config

Edit the config:
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
PermitEmptyPasswords no
ChallengeResponseAuthentication no

X11Forwarding no
AllowTcpForwarding yes

ClientAliveInterval 300
ClientAliveCountMax 2
LoginGraceTime 30
MaxAuthTries 3

Note: On modern OpenSSH, Protocol 2 is already the default; we don’t need a Protocol line at all.

Test the config before restarting:

sudo sshd -t
No output = good.

If you see an error, fix the line it mentions and run sshd -t again.

Then reload:
sudo systemctl restart ssh
sudo systemctl status ssh

---

Phase 5 – Keys & authorized_keys
5.1 Generate an ed25519 keypair

If the client is the same machine:
ssh-keygen -t ed25519 -C "user@host"

Accept default path (/home/user/.ssh/id_ed25519)

Optional passphrase (recommended in real life)

Check:
ls -l ~/.ssh

You should see at least:

id_ed25519

id_ed25519.pub

5.2 Authorize the key

Append your public key to authorized_keys:
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys

chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

Verify:
cat ~/.ssh/authorized_keys

You should see a line starting with ssh-ed25519.

---

Phase 6 – Verification
6.1 Connect via SSH

From the same host:
ssh user@localhost
# or, explicitly:
ssh -i ~/.ssh/id_ed25519 user@localhost

Expected:

If you set a key passphrase → it prompts for that.

If you didn’t → you drop straight into a shell.

Inside the SSH session:
whoami
hostname

Make sure you’re user on host, then exit.

---

6.2 Confirm password auth is off

Try to force password auth from another machine (or temporarily move your private key aside) and connect. It should fail with Permission denied (publickey).

---

6.3 Logs (optional but good habit)

In one terminal:
sudo journalctl -fu ssh

In another:
ssh user@localhost
Watch the logs as you connect and disconnect; this is how you’ll debug access in a real environment.

---

Notes & extensions

This lab is intentionally small and opinionated. Ideas to extend it:

Add fail2ban and watch how it reacts to bad auth attempts.

Restrict SSH by AllowUsers or AllowGroups in sshd_config.

Move SSH to a different port (only if you understand the tradeoffs).

Most importantly: rerun these steps on a fresh VM or host until it feels boring. That’s when you’ve really learned it.
