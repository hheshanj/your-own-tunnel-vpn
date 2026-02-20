# 3x-ui Ultimate ISP Bypass Guide

This is the complete working A to Z setup. This uses the MHSanaei 3x-ui fork, which is significantly more stable and feature rich than the original x-ui.

---

## Phase 01: Infrastructure & DNS Setup

First we prep the base and make sure Cloudflare and your server can talk correctly.

* Provision your VPS: A $5/mo basic instance with 1 vCPU and 1GB RAM is more than enough. Use **Ubuntu 22.04 or 24.04 LTS** for maximum compatibility.
* Firewall Config: Open the following inbound ports on your VPS firewall:
  * `80` and `443` for HTTP/HTTPS and SSL certificate issuance
  * A custom high port (e.g. `54321`) for accessing the 3x-ui admin panel

### Cloudflare Setup
1.  Point your domain nameservers to Cloudflare
2.  Create an A record: use a subdomain like `vpn.yourdomain.com` pointing to your VPS public IP

3.  > [!WARNING]
    > Critical: Set the Proxy Status to **DNS Only (Grey Cloud)** during setup. You can turn proxy on *after* SSL has been successfully issued (optional, not recommended)

---

## Phase 02: Server Prepping & SSL

Connect to your server over SSH to run these steps.

1.  First run standard system updates:
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

2.  Install 3x-ui using the official MHSanaei script:
    ```bash
    bash <(curl -Ls https://raw.githubusercontent.com/MHSanaei/3x-ui/master/install.sh)
    ```

3.  SSL Generation (this is the part everyone messes up):
    1.  Type `x-ui` in your terminal to open the 3x-ui management menu
    2.  Select **Option 19: Cloudflare SSL Certificate**
    3.  Follow the prompts to issue a certificate for your subdomain. This makes your traffic look identical to normal HTTPS web traffic.

---

## Phase 03: Panel Configuration

1.  Access the admin panel by visiting `http://<your-vps-ip>:<your-custom-port>` in your browser
2.  > [!WARNING]
    > First thing you do: go to Settings and change the default username, password, and set a custom panel URL root. Do not leave the defaults in place.
3.  For standard unblocking use VLESS + Reality or VMess + TLS. These are the current gold standard for defeating deep packet inspection.

---

## Phase 04: The Bypass Configuration (ISP Spoofing)

This is the part that turns restricted Work & Learn / Zoom / zero rated data into unlimited unrestricted data.

1.  Create a new inbound with the following settings:
    | Setting | Value |
    |---|---|
    | Protocol | `VLESS` (recommended for speed) / `VMess` |
    | Port | `443` **this is mandatory** |
    | Transmission | `ws` (Websocket, most stable for bypass) / `grpc` |

2.  Security & TLS Settings:
    * Set TLS to `Enabled`
    * **SNI**: Enter the host whitelisted by your ISP zero rate package:
      * Dialog Work & Learn / MS Office packs: `aka.ms` or `teams.microsoft.com`
      * Zoom packs: `zoom.us`
    * Select the SSL certificate you generated in Phase 02

3.  Client side setup (netmod / Shadowrocket / v2rayN):
    * Import the config via QR code from the 3x-ui panel
    * Open the config settings on your device
    * Confirm the **SNI** field is set exactly to the whitelisted host you used above
    * Set Header Type to `none` and confirm the Host field matches your SNI.((if only prompted to)

---

### How this works

When your ISP runs deep packet inspection on your connection, it only sees a TLS connection going to port 443 with an SNI of `aka.ms`. It marks this as allowed zero rated traffic and counts it against your Work & Learn quota instead of your paid anytime data.

> [!TIP]
> Watch your anytime data balance the first 10 minutes you use this. If your anytime data goes down, your SNI is wrong, or your ISP uses IP based whitelisting instead of SNI.

---
