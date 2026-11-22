# Ubuntu Home Server

Documentation, configuration files, and scripts for my self-hosted Ubuntu home server.

## Hardware
* **Machine:** HP EliteDesk 800 G6 Mini
* **CPU:** Intel Core i5-10500T (6 Cores / 12 Threads, 2.3 GHz Base, 35W TDP)
* **GPU:** Integrated Intel UHD Graphics 630 (with Intel Quick Sync Video for hardware transcoding)
* **RAM:** 16 GB DDR4 @2667 MHz
* **Storage:** 512 GB Western Digital PC SN810 NVMe M.2 SSD (`SDCPNRY-512G-1006`)
* **Networking:** Integrated Gigabit Ethernet (RJ45)
* **OS:** Ubuntu Server 26.04 LTS

## Hosted Services
**Game Server Management:** CubeCoders AMP (Application Management Panel)
* **Minecraft:** Cobblemon Fabric Server 1.8.4
* **Minecraft:** Java Edition Server 26.3
* **Media:** Jellyfin

## Setup & Maintenance
* Remote management via SSH and Webmin
* **External Tunneling:** [playit.gg](https://playit.gg) agent for public game server access without router port forwarding or exposing home IP
* **Remote VPN Access:** Self-hosted bare-metal WireGuard VPN for encrypted remote administration (SSH & AMP Panel)
* **Network Documentation:** See [NETWORKING.md](./NETWORKING.md) for full WireGuard and `playit.gg` configuration details
