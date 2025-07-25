
# Tunnel VPN Build  Phase 01: Foundation & Setup

This phase lays the groundwork for your secure tunnel VPN. We'll set up a server, secure access, and link it to a custom domain.

---

## Step 1: Create Your Ubuntu Server (VPS)

### Purpose:
Host your VPN tunnel on a remote Linux server.

### âœ… Minimum Server Specs:
| Spec     | Value            |
|----------|------------------|
| OS       | Ubuntu Server 22.04+ |
| CPU      | 1 Core           |
| RAM      | 1 GB             |
| Storage  | 10â€“20 GB         |
| Ports    | 80 (HTTP), 443 (HTTPS), 20 (FTP - optional) |

### Recommended VPS Providers (Free for Students):
- [Azure for Students](https://azure.microsoft.com/en-us/free/students/)
- [GitHub Student Pack (DigitalOcean, etc.)](https://education.github.com/pack)
- [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/)
- [Google Cloud Free Tier](https://cloud.google.com/free)

### Server Creation (Example: DigitalOcean)
1. Sign up and create a new Droplet
2. Select **Ubuntu 22.04 x64**
3. Choose the Basic Plan: **1vCPU, 1GB RAM**
4. Enable SSH key or password
5. Choose a region near your target users
6. **Allow the following ports** under "Networking" or "Firewall":
   - Port `20`, `80`, `443`
7. Deploy and note down your **serverâ€™s public IP**

---

## Step 2: Access the Server via SSH & Update

### ðŸ“² SSH Clients:
| Platform | Tool        |
|----------|-------------|
| Android  | [Termius](https://termius.com/), JuiceSSH |
| Windows  | [PuTTY](https://www.putty.org/), Termius |
| macOS/Linux | Terminal (built-in `ssh`) |

### SSH Into the Server:
```bash
ssh root@your_server_ip
```

If prompted to trust the host, type `yes`.

### Update & Install Essentials:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget unzip git ufw -y
```

### ðm(Optional) Create a New Sudo User:
```bash
adduser yourname
usermod -aG sudo yourname
su - yourname
```

---

## Step 3: Buy a Domain & Set Up DNS

### ðŸmmBuy a Domain:
Use any of these:
- [Namecheap](https://namecheap.com)
- [Spaceship](https://spaceship.com)
- [Porkbun](https://porkbun.com)
- [Freenom](https://freenom.com) *(Free domains like .tk, .ml)*

> Choose something simple and clean, e.g., `mytunnel.zone` or `safefox.net`

---

### Connect to Cloudflare:
1. Sign up at [Cloudflare](https://cloudflare.com)
2. Add your domain to Cloudflare
3. Cloudflare will give you **two nameservers**
4. Go to your domain registrar, replace nameservers with the ones from Cloudflare

---

### Add a VPN Subdomain:
On Cloudflare â† "Add Record":
- Type: `A`
- Name: `vpn` *(this becomes `vpn.yourdomain.com`)*
- IPv4 Address: **your serverâ€™s IP**
- TTL: Auto
- Proxy status: **DNS Only (turn off orange cloud)**

>Do not proxy the record via Cloudflare â€” it must be "DNS Only" for VPN traffic to work.

---

## Summary of Phase 01:

- Ubuntu 22.04+ server deployed and running
- SSH access working and packages updated
- Domain bought, connected to Cloudflare
- Subdomain pointing to your server's IP (e.g., `vpn.yourdomain.com`)

---

