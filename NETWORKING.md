## Remote Access & Game Server Networking

To manage the server securely from outside the local network while hosting public game servers 24/7 without exposing the home IP address, this homelab uses a dual-layer networking architecture:

1. **WireGuard VPN (Bare-Metal):** Self-hosted inbound VPN for encrypted remote administration (SSH & AMP Panel).
2. **playit.gg Linux Agent:** Outbound reverse-proxy tunnel (`systemd` service) for 24/7 Cobblemon Minecraft server access with zero router port forwarding for game traffic.

---

### 1. Self-Hosted WireGuard VPN (Remote Administration)

Configured directly on Ubuntu Server (`26.04 LTS`) using `wg-quick` and `iptables` NAT masquerading over the primary Ethernet interface (`eno1`).

#### Kernel IP Forwarding
Enabled persistent IPv4 packet forwarding via drop-in configuration at `/etc/sysctl.d/99-wireguard.conf`:
```ini
net.ipv4.ip_forward=1
```
Applied with `sudo sysctl --system`.

#### Server Interface Configuration (`/etc/wireguard/wg0.conf`)
* **VPN Subnet:** `<VPN_SUBNET>/24`
* **Listen Port:** `51820/UDP` (forwarded on eero router to static LAN reservation `<SERVER_LOCAL_IP>`)
* **Outbound NAT Interface:** `eno1`

```ini
[Interface]
Address = <VPN_SERVER_IP>/24
ListenPort = 51820
PrivateKey = <SERVER_PRIVATE_KEY>
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eno1 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eno1 -j MASQUERADE

[Peer]
# Mobile Client
PublicKey = <CLIENT_PUBLIC_KEY>
AllowedIPs = <VPN_CLIENT_IP>/32
```

#### Service Management
```bash
sudo systemctl enable --now wg-quick@wg0
sudo wg show
```

---

### 2. 24/7 Game Server Tunnel (`playit.gg`)

Migrated the `playit.gg` tunnel agent from a Windows 11 desktop client to a native Linux background service on the Ubuntu server so the Cobblemon Minecraft server stays accessible 24/7 independently of client machines and without opening port `25565` on the home router.

#### Installation & Service Setup
```bash
# Add official playit.gg repository and install agent
curl -fsSL https://playit.gg | bash

# Link agent to playit.gg account
playit setup

# Enable persistent background daemon
sudo systemctl enable --now playit
sudo systemctl status playit
```

#### Tunnel Routing
* **Service:** `playit.service` (`systemd`)
* **Tunnel Type:** Minecraft Java (TCP)
* **Local Target:** `<SERVER_LOCAL_IP>:25565` (AMP Cobblemon Instance)
* **Relay Region:** US Miami, FL
