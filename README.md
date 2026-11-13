# Elevvo Task 1 — Secure Linux Bastion Host & Process Management

## Overview

This project provisions and hardens an Ubuntu Linux environment, configures SSH for public-key authentication, and deploys a simple web server managed by systemd.

## Environment

- **OS:** Ubuntu 24.04.5 LTS
- **User:** `laborant` (sudo access)
- **Init/service manager:** systemd
- **Web server:** Python 3 HTTP server
- **Web port:** TCP 8080

## 1. Linux navigation and inspection

The environment was inspected using standard Linux commands, including:

```bash
whoami
pwd
id
cat /etc/os-release
ps -p 1 -o comm=
ls -la
```

## 2. SSH key authentication and hardening

An Ed25519 key pair was generated for this task:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/task1_ed25519 -C "elevvo-task1"
```

The private key is stored locally at `~/.ssh/task1_ed25519`; the public key is at `~/.ssh/task1_ed25519.pub`. **Private keys must never be committed to this repository or shared.**

SSH hardening was applied through `/etc/ssh/sshd_config.d/99-hardening.conf`:

```sshconfig
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
```

The configuration was syntax-checked and the effective settings verified:

```bash
sudo sshd -t
sudo sshd -T | grep -E 'passwordauthentication|permitrootlogin|pubkeyauthentication'
sudo systemctl reload ssh
```

Expected effective settings:

```text
permitrootlogin no
pubkeyauthentication yes
passwordauthentication no
```

## 3. Web service

The web root is `/opt/task1-web`, containing an `index.html` page for the demonstration.

The systemd unit is `/etc/systemd/system/task1-web.service`:

```ini
[Unit]
Description=Elevvo Task 1 Web Server
After=network.target

[Service]
User=laborant
WorkingDirectory=/opt/task1-web
ExecStart=/usr/bin/python3 -m http.server 8080 --bind 0.0.0.0
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

The service was registered with systemd, started, and enabled to launch at boot:

```bash
sudo systemctl daemon-reload
sudo systemctl restart task1-web
sudo systemctl enable task1-web
```

## 4. Lifecycle and verification

The following checks were performed:

```bash
sudo systemctl status task1-web --no-pager
sudo systemctl is-enabled task1-web
sudo systemctl stop task1-web
sudo systemctl is-active task1-web
sudo systemctl start task1-web
curl http://localhost:8080
```

Observed results:
- systemd reported the web service as **active (running)**.
- The service was **enabled** for startup at boot.
- After stopping it, `systemctl is-active` reported `inactive`.
- After starting it again, `curl http://localhost:8080` returned the demonstration HTML page.

## Security and scope notes

- SSH password authentication and direct root login are disabled in the effective SSH configuration.
- Public-key authentication is enabled.
- The task-specific Ed25519 key pair was generated in the VM. Do not publish its private half.
- The web server is a basic Python HTTP server for a lab demonstration, not a production web server. It binds to all network interfaces on port 8080; restrict network access appropriately outside an isolated lab.
- A separate external SSH client login test was not documented as part of this run.

## Skills demonstrated

Linux filesystem navigation, SSH key generation, SSH daemon hardening, configuration validation, systemd unit creation, service lifecycle management, boot-time enablement, and HTTP verification.
