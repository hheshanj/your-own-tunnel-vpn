
# 3x-ui Ultimate ISP Bypass Guide

> **The complete A to Z working setup for turning restricted educational data into unrestricted tunnel traffic.**

This guide uses the **[MHSanaei 3x-ui fork](https://github.com/MHSanaei/3x-ui)** — significantly more stable, actively maintained, and feature-rich than the original x-ui or other forks.

---

## 📋 Prerequisites

Before starting, make sure you have:
- A domain name (Namecheap, Porkbun, or any registrar works)
- Cloudflare account (free tier is sufficient)
- VPS with root access (any provider: Racknerd, PQ.Hosting, Vultr, DigitalOcean)
- SSH client (Termius, PuTTY, or terminal)

---

## Phase 01: Infrastructure & DNS Setup

### 1.1 VPS Provisioning

A **$5/month** instance is more than enough:
- **Minimum specs:** 1 vCPU, 1GB RAM, 20GB SSD
- **Recommended OS:** Ubuntu 22.04 LTS or 24.04 LTS
- **Region:** Choose one geographically close to your users

### 1.2 Firewall Configuration

Open these inbound ports in your VPS firewall (UFW or provider dashboard):

| Port | Purpose |
|------|---------|
| `80` | HTTP (Let's Encrypt / ACME challenge) |
| `443` | HTTPS (main tunnel traffic) |
| `54321` (example) | 3x-ui admin panel access |

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 54321/tcp
sudo ufw enable
```

### 1.3 Cloudflare DNS Setup

1. Point your domain's nameservers to Cloudflare (done at your registrar)
2. In Cloudflare Dashboard → DNS → Records, add:
   - **Type:** `A`
   - **Name:** `vpn` (creates `vpn.yourdomain.com`)
   - **IPv4 address:** Your VPS public IP
   - **Proxy status:** ☁️ **DNS only (grey cloud)**

> ⚠️ **Critical:** Keep it grey until SSL is issued. Enabling proxy (orange cloud) will break the certificate issuance.

---

## Phase 02: Server Setup & SSL

SSH into your server and run:

### 2.1 System Preparation

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget socat
```

### 2.2 Install 3x-ui

```bash
bash <(curl -Ls https://raw.githubusercontent.com/MHSanaei/3x-ui/master/install.sh)
```

During installation, note down:
- Panel port (default: `2053` or your custom choice)
- Default username and password (displayed at end)

### 2.3 Generate SSL Certificate

This is where most people fail. Follow carefully:

```bash
x-ui
```

Select **Option 19: Cloudflare SSL Certificate** and follow prompts:
1. Enter your Cloudflare **Global API Key** (from Cloudflare Dashboard → My Profile → API Tokens)
2. Enter your **email** associated with Cloudflare
3. Enter your **full subdomain** (`vpn.yourdomain.com`)

✅ **Success indicator:** Certificate files created in `/root/cert/`

> 💡 **Pro Tip:** If Option 19 fails, use **Option 18 (Let's Encrypt)** instead. Both work, but Cloudflare's method is faster and uses ECC certificates (smaller, faster).

---

## Phase 03: Panel Configuration

### 3.1 Access the Panel

Visit: `http://<VPS-IP>:<PANEL-PORT>`

Example: `http://123.45.67.89:54321`

### 3.2 Security Hardening (Do This FIRST)

Navigate to **Settings** and change:

| Setting | Recommendation |
|---------|----------------|
| Username | Anything except `admin` |
| Password | 16+ characters, random |
| Panel path | `/your-secret-path/` (not `/xui/`) |
| Panel port | Keep high port or change to something else |

> 🔒 **Why this matters:** Automated bots scan for `/xui`, `/x-ui`, `/panel` paths. Custom paths prevent 99% of brute force attempts.

### 3.3 Create Base Inbound (Standard Bypass)

Go to **Inbounds** → **Add Inbound**:

**For maximum stealth (recommended):**
- **Protocol:** `vless`
- **Transport:** `reality`
- **Port:** `443`
- **TLS:** `reality`
- **Show advanced:** Enable `uTLS` with `chrome` fingerprint

**For compatibility (if reality fails):**
- **Protocol:** `vmess` or `vless`
- **Transport:** `ws`
- **Port:** `443`
- **TLS:** `tls`
- **Path:** `/your-ws-path`

---

## Phase 04: ISP Bypass Configuration (Zero-Rating Spoof)

> **This is the money shot.** This configuration tricks your ISP into billing your traffic against whitelisted services instead of regular data.

### 4.1 Create Bypass Inbound

**Protocol Settings:**
| Field | Value |
|-------|-------|
| Protocol | `vless` |
| Port | `443` ⚠️ **Must be 443** |
| Transport | `ws` (most reliable) or `grpc` |
| TLS | `tls` |

**TLS/SNI Settings:**
- **SNI (Server Name Indication):** The whitelisted hostname
- **Certificates:** Select the SSL cert from Phase 2.3

### 4.2 SNI Mapping by ISP Package

| ISP Package | Recommended SNI | Alternative SNI |
|-------------|-----------------|-----------------|
| Dialog Work & Learn | `aka.ms` | `teams.microsoft.com`, `office.com` |
| Dialog Zoom | `zoom.us` | `zoom.com` |
| Hutch Work From Home | `aka.ms` | `microsoft.com` |
| Airtel Edu Pack | `google.com` | `classroom.google.com` |
| SLT Fiber Edu | `zoom.us` | `microsoft.com` |

> 🔍 **How to find your SNI:** Check your ISP's official website for their zero-rated package. The SNI is usually the primary domain they advertise (e.g., "Unlimited Zoom" → `zoom.us`).

### 4.3 Client Configuration

**Android (v2rayNG):**
1. Tap QR code from 3x-ui panel → Scan
2. Long press config → **Edit**
3. Scroll to **TLS Settings** → Set **SNI** = your chosen hostname
4. **Host** field must match SNI exactly
5. **Header Type:** `none`

**iOS (Shadowrocket):**
1. Import config via QR code
2. Tap config → **Edit Server**
3. **TLS SNI:** Enter your hostname
4. **TLS Host:** Same as SNI
5. Save and connect

**Windows (v2rayN):**
1. Import config (VMess) from panel
2. Right-click config → **Edit**
3. **Stream Settings** → **TLS** → **Server Name:** your SNI
4. **Host:** same value
5. Apply and connect

---

## 🧠 How It Works (The Bro-Science)

```
Your Device → [Encrypted Tunnel] → Your VPS → Internet
                ↑
         SNI: aka.ms
         Port: 443
```

When your ISP's **Deep Packet Inspection (DPI)** examines your traffic:
1. Sees TLS handshake on port **443** (standard HTTPS port)
2. Reads the **SNI field** = `aka.ms` (Microsoft domain)
3. Thinks: "Student doing homework on Microsoft Teams"
4. Bills against **Work & Learn quota**, not **Anytime data**

**What the ISP CANNOT see:**
- Your actual destination (YouTube, Netflix, etc.)
- That you're tunneling through a VPS
- Anything inside the encrypted TLS tunnel

---

## ⚠️ Troubleshooting

### Connection Slow / High Latency
- **Cause:** Wrong server region or ISP throttling
- **Fix:** Try a VPS closer to your location; switch transport from `ws` to `grpc`

### Anytime Data Still Decreasing
- **Cause:** Wrong SNI or ISP uses **IP-based whitelisting**
- **Fix:** Try alternative SNI from table above; check if your ISP whitelists by IP (rare but possible)

### "Certificate Invalid" on Client
- **Cause:** System time mismatch or wrong SNI
- **Fix:** Sync device time; ensure client SNI matches server SNI exactly

### Panel Won't Load After Reboot
- **Cause:** 3x-ui service not starting
- **Fix:**
  ```bash
  x-ui restart
  ```
  Check logs:
  ```bash
  x-ui log
  ```

### Cloudflare SSL Option Missing
- **Cause:** Outdated 3x-ui version
- **Fix:** Update:
  ```bash
  x-ui update
  ```

---


## 📱 Quick Reference Card

```yaml
Protocol: vless
Port: 443
Transport: ws
TLS: enabled
SNI: aka.ms (or your ISP's whitelisted domain)
Path: /random-string
```

---

## 📚 Resources

- [3x-ui GitHub](https://github.com/MHSanaei/3x-ui)
- [XTLS/Xray Documentation](https://xtls.github.io/)
- [v2rayNG (Android)](https://github.com/2dust/v2rayNG)
- [Shadowrocket (iOS)](https://apps.apple.com/app/shadowrocket/id932747118)
- [v2rayN (Windows)](https://github.com/2dust/v2rayN)

---

> **Disclaimer:** This guide is for educational purposes. Use responsibly and in compliance with your ISP's terms of service. The authors are not responsible for any misuse.


