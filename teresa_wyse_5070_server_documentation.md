# Teresa Fares - Wyse 5070 Server Documentation

## 1. Overview
**Device:** Dell Wyse 5070 Thin Client  
**Operating System:** Ubuntu 24.04 LTS (64-bit)  
**Purpose:** Home media and automation server for comics, metadata, and remote access  
**Primary User:** `teresa`  
**Services:** Komga, Mylar, qBittorrent-nox, Prowlarr, Cloudflare Tunnel  
**Network:** Static IP with Cloudflare DNS tunnel for external access

---

## 2. System Setup
### OS & User Setup
- Installed Ubuntu 24.04 LTS via USB image.
- Created primary user `teresa` with sudo privileges.
- Automatic updates enabled via `unattended-upgrades`.
- Configured hostname: `wyse5070`.

### SSH Access
```bash
sudo apt install openssh-server -y
sudo systemctl enable ssh --now
```
Access via LAN IP or Cloudflare tunnel.

### Network Configuration
- Static IP assigned via router DHCP reservation.
- DNS managed through Cloudflare.
- UFW enabled with specific port rules (see below).

### Firewall Rules
```bash
sudo ufw allow 22/tcp      # SSH
sudo ufw allow 8080/tcp    # Komga
sudo ufw allow 8090/tcp    # Mylar
sudo ufw allow 8081/tcp    # Prowlarr
sudo ufw allow 6881/udp    # qBittorrent (DHT)
sudo ufw allow 8999/tcp    # qBittorrent WebUI
sudo ufw reload
sudo ufw enable
```

---

## 3. Directory Structure
```bash
/srv
├── komga
├── mylar
├── qbittorrent
├── downloads
└── configs
```
Each service stores its configuration files inside `/srv/configs/<service-name>`.

Ownership and permissions:
```bash
sudo chown -R teresa:teresa /srv
sudo chmod -R 775 /srv
```

---

## 4. Core Services

### Komga (Comics Library Server)
**Purpose:** Organizes and serves comic book collections over LAN or Cloudflare tunnel.  
**Port:** 8080  
**Service File:** `/etc/systemd/system/komga.service`

**Commands:**
```bash
sudo systemctl start komga
sudo systemctl stop komga
sudo systemctl enable komga
sudo systemctl status komga
```

**Logs:** `/srv/komga/logs/komga.log`

**Web Access:** http://komga.teresafares.com or http://<LAN-IP>:8080

---

### Mylar (Metadata Fetcher)
**Purpose:** Fetches comic metadata and automates comic management.  
**Port:** 8090  
**Service File:** `/etc/systemd/system/mylar.service`

**Commands:**
```bash
sudo systemctl start mylar
sudo systemctl stop mylar
sudo systemctl enable mylar
sudo systemctl status mylar
```

**Logs:** `/srv/mylar/logs/mylar.log`

**Web Access:** http://mylar.teresafares.com or http://<LAN-IP>:8090

---

### qBittorrent-nox (Headless Torrent Client)
**Purpose:** Handles torrent downloads for comics and media.  
**Ports:** 8999 (WebUI), 6881 (DHT/Peers)

**Service File:** `/etc/systemd/system/qbittorrent-nox.service`

**Commands:**
```bash
sudo systemctl start qbittorrent-nox
sudo systemctl stop qbittorrent-nox
sudo systemctl enable qbittorrent-nox
sudo systemctl status qbittorrent-nox
```

**Logs:** `/srv/qbittorrent/logs/qbittorrent.log`

**Web Access:** http://<LAN-IP>:8999 or via Cloudflare tunnel (secured access)

**Configuration File:** `~/.config/qBittorrent/qBittorrent.conf`

**Common Fix for 'Downloading metadata' issue:**
```bash
sudo systemctl stop qbittorrent-nox
cd ~/.local/share/data/qBittorrent/BT_backup/
rm -f *.fastresume *.torrent
sudo systemctl start qbittorrent-nox
```

---

### Prowlarr (Indexer Manager)
**Purpose:** Centralized indexer management and integration with qBittorrent/Mylar.  
**Port:** 8081  
**Service File:** `/etc/systemd/system/prowlarr.service`

**Commands:**
```bash
sudo systemctl start prowlarr
sudo systemctl stop prowlarr
sudo systemctl enable prowlarr
sudo systemctl status prowlarr
```

**Web Access:** http://prowlarr.teresafares.com or http://<LAN-IP>:8081

---

### Cloudflare Tunnel
**Purpose:** Provides secure HTTPS access to all local services using Cloudflare DNS.  
**Tunnel Config File:** `/etc/cloudflared/config.yml`

**Example Configuration:**
```yaml
tunnel: <tunnel-id>
credentials-file: /etc/cloudflared/<tunnel-id>.json

ingress:
  - hostname: komga.teresafares.com
    service: http://localhost:8080
  - hostname: mylar.teresafares.com
    service: http://localhost:8090
  - hostname: qbittorrent.teresafares.com
    service: http://localhost:8999
  - hostname: prowlarr.teresafares.com
    service: http://localhost:8081
  - service: http_status:404
```

**Commands:**
```bash
sudo systemctl start cloudflared
sudo systemctl stop cloudflared
sudo systemctl enable cloudflared
sudo systemctl status cloudflared
```

---

## 5. Maintenance & Backups
**Backup Configurations:**
```bash
rsync -av /srv/configs/ /mnt/backup/configs/
```

**Update Packages:**
```bash
sudo apt update && sudo apt upgrade -y
```

**Restart All Services:**
```bash
sudo systemctl restart komga mylar qbittorrent-nox prowlarr cloudflared
```

---

## 6. Troubleshooting

### qBittorrent stuck on "Downloading metadata"
1. Ensure DHT, PeX, and Local Peer Discovery are enabled.
2. Verify firewall allows UDP 6881.
3. Clear `BT_backup` folder as shown above.
4. Check VPN or Cloudflare tunnel isn’t blocking UDP.

### Mylar not fetching metadata
1. Verify ComicVine API key.
2. Clear Mylar cache: `rm -rf ~/.mylar/cache/`
3. Restart service.

### Cloudflare Tunnel not connecting
1. Check Cloudflare service: `sudo systemctl status cloudflared`
2. Verify DNS record matches subdomain in config.
3. Restart: `sudo systemctl restart cloudflared`

---

## 7. Security
- SSH protected with key authentication.
- UFW firewall configured with specific ports only.
- Cloudflare Access can optionally restrict external logins.
- Regular system updates scheduled weekly.

---

## 8. Summary Table
| Service         | Port  | Access URL                       | Config Path                                  |
|-----------------|-------|----------------------------------|----------------------------------------------|
| Komga           | 8080  | komga.teresafares.com            | /srv/komga                                   |
| Mylar           | 8090  | mylar.teresafares.com            | /srv/mylar                                   |
| qBittorrent-nox | 8999  | qbittorrent.teresafares.com      | ~/.config/qBittorrent/qBittorrent.conf       |
| Prowlarr        | 8081  | prowlarr.teresafares.com         | /srv/configs/prowlarr                        |
| Cloudflared     | N/A   | N/A (Manages HTTPS tunnels)      | /etc/cloudflared/config.yml                  |

---

## 9. Future Plans
- Integrate backup automation via cron.
- Add Prometheus + Grafana for monitoring.
- Implement automatic Cloudflare certificate renewal logs.
- Optionally deploy containers (Komga/Mylar/qBittorrent) using Docker Compose for consistency.

