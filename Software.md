# \# Oracle Cloud Ampere Setup (Modern Method)

# 

# \## Overview

# 

# This sets up a free/low-cost Ubuntu server on Oracle Cloud (Ampere A1) to host the stack.

# 

# This replaces older approaches that relied on:

# 

# \* manual firewall hacks

# \* Azure SQL Edge (deprecated)

# \* cron startup scripts

# \* public IP exposure

# 

# We will:

# 

# \* create a VM

# \* connect via SSH

# \* install Docker

# \* deploy the stack

# \* use Cloudflare Tunnel (no open ports required)

# 

# \---

# 

# \## 1. Create Oracle Cloud Account

# 

# \* Go to: https://www.oracle.com/cloud/free/

# \* Create account

# \* Choose a \*\*US region\*\* (important for availability)

# 

# > Note: Some regions run out of free resources. If instance creation fails, try another region.

# 

# \---

# 

# \## 2. Create Compute Instance

# 

# Navigate:

# 

# ```

# ☰ Menu → Compute → Instances → Create Instance

# ```

# 

# \### Basic Settings

# 

# \* Name: `app-server`

# \* Image: \*\*Canonical Ubuntu 24.04\*\*

# \* Shape:

# 

# &#x20; \* Ampere

# &#x20; \* `VM.Standard.A1.Flex`

# 

# \### Configure Resources

# 

# \* OCPU: `2–4`

# \* Memory: `12–24 GB`

# 

# > Free tier allows up to 4 OCPU / 24GB total

# 

# \---

# 

# \## 3. SSH Key Setup

# 

# \* Click \*\*"Save Private Key"\*\*

# \* Store it securely (you cannot re-download it)

# 

# \---

# 

# \## 4. Networking (IMPORTANT)

# 

# When creating the instance:

# 

# \* Leave default VCN

# \* \*\*Do NOT worry about opening ports\*\*

# \* We will NOT expose ports publicly (Cloudflare handles access)

# 

# \---

# 

# \## 5. Connect via SSH

# 

# From your local machine:

# 

# ```bash

# ssh -i path/to/private\_key ubuntu@YOUR\_PUBLIC\_IP

# ```

# 

# \---

# 

# \## 6. Initial Server Setup

# 

# Run:

# 

# ```bash

# sudo apt update \&\& sudo apt upgrade -y

# ```

# 

# Optional cleanup:

# 

# ```bash

# sudo systemctl disable snapd.service

# sudo apt purge snapd -y

# sudo apt autoremove -y

# ```

# 

# \---

# 

# \## 7. Install Docker

# 

# ```bash

# sudo apt install -y ca-certificates curl gnupg

# 

# sudo install -m 0755 -d /etc/apt/keyrings

# 

# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/keyrings/docker.gpg > /dev/null

# 

# echo \\

# &#x20; "deb \[arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \\

# &#x20; https://download.docker.com/linux/ubuntu $(lsb\_release -cs) stable" \\

# &#x20; | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 

# sudo apt update

# 

# sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 

# sudo usermod -aG docker ubuntu

# newgrp docker

# ```

# 

# Test:

# 

# ```bash

# docker run hello-world

# ```

# 

# \---

# 

# \## 8. Clone Your Project

# 

# ```bash

# git clone https://github.com/YOUR\_USERNAME/YOUR\_REPO.git

# cd YOUR\_REPO

# ```

# 

# \---

# 

# \## 9. Configure Environment

# 

# Create `.env`:

# 

# ```bash

# nano .env

# ```

# 

# Example:

# 

# ```env

# POSTGRES\_USER=appuser

# POSTGRES\_PASSWORD=strongpassword

# POSTGRES\_DB=appdb

# CF\_TUNNEL\_TOKEN=your\_cloudflare\_token

# ```

# 

# \---

# 

# \## 10. Start the Stack

# 

# ```bash

# docker compose up -d --build

# ```

# 

# Verify:

# 

# ```bash

# docker ps

# ```

# 

# \---

# 

# \## 11. Setup Cloudflare Tunnel

# 

# On your local machine OR server:

# 

# ```bash

# cloudflared tunnel login

# cloudflared tunnel create my-tunnel

# cloudflared tunnel route dns my-tunnel app.yourdomain.com

# cloudflared tunnel token my-tunnel

# ```

# 

# Put the token into `.env`.

# 

# Restart stack:

# 

# ```bash

# docker compose up -d

# ```

# 

# \---

# 

# \## 12. Access Your App

# 

# Visit:

# 

# ```

# https://app.yourdomain.com

# ```

# 

# No ports opened. No firewall rules needed.

# 

# \---

# 

# \## 13. Storage (Optional but Recommended)

# 

# If you want persistent disk beyond root volume:

# 

# ```

# ☰ → Storage → Block Volumes → Create

# ```

# 

# Attach to instance, then:

# 

# ```bash

# sudo mkfs.ext4 /dev/sdb

# sudo mkdir /data

# sudo mount /dev/sdb /data

# ```

# 

# Use it for:

# 

# ```text

# /data/postgres

# /data/backups

# ```

# 

# \---

# 

# \## 14. Basic Security

# 

# Run:

# 

# ```bash

# sudo apt install ufw -y

# sudo ufw allow OpenSSH

# sudo ufw enable

# ```

# 

# No need to open:

# 

# \* 80

# \* 443

# \* 5432

# 

# Cloudflare handles ingress.

# 

# \---

# 

# \## 15. Updates / Deployments

# 

# ```bash

# git pull

# docker compose up -d --build

# ```

# 

# \---

# 

# \## 16. Backups

# 

# ```bash

# docker exec postgres pg\_dump -U appuser appdb > backups/backup.sql

# ```

# 

# \---

# 

# \## Notes

# 

# \* This setup is \*\*cloud-agnostic\*\*

# \* Can be moved to local hardware with no changes

# \* Avoids:

# 

# &#x20; \* public port exposure

# &#x20; \* manual iptables rules

# &#x20; \* deprecated SQL Edge setups

# 

# \---

# 

# \## Summary

# 

# You now have:

# 

# \* Oracle Ampere server

# \* Docker-based stack

# \* Secure public access via Cloudflare

# \* Fully portable system (cloud → homelab ready)



