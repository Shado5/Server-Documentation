# Teresa Fares - Wyse 5070 Server Documentation (Updated)

## 1. Overview

Name: Multiverse Archives\
Device: Dell Wyse 5070 Thin Client\
Operating System: Ubuntu 24.04 LTS (64-bit)\
Purpose: Home media and automation server for comics, metadata, and
remote access\
Primary User: teresa\
Services: Komga, Mylar, qBittorrent-nox, Prowlarr, Cloudflare Tunnel
(HTTP + SSH)\
Network: Static IP via router DHCP reservation + Cloudflare DNS tunnel
for external access

## 2. System Setup

OS & User Setup:\
- Installed Ubuntu 24.04 LTS via USB image.\
- Created primary user teresa with sudo privileges.\
- Enabled automatic updates via unattended-upgrades.\
- Hostname: wyse5070.\
\
SSH Access:\
sudo apt install openssh-server -y\
sudo systemctl enable ssh \--now\
Accessible via LAN IP (ssh teresa@192.168.x.x) or Cloudflare Tunnel (ssh
teresa@ssh.teresafares.com).\
\
Network Configuration:\
- Static IP via router\
- DNS managed via Cloudflare\
- UFW enabled with required ports only\
\
Firewall Rules:\
sudo ufw allow 22/tcp\
sudo ufw allow 8080/tcp\
sudo ufw allow 8090/tcp\
sudo ufw allow 8081/tcp\
sudo ufw allow 6881/udp\
sudo ufw allow 8999/tcp\
sudo ufw reload && sudo ufw enable

## 3. Directory Structure

/srv\
├── komga\
├── mylar\
├── qbittorrent\
├── downloads\
└── configs\
\
sudo chown -R teresa:teresa /srv\
sudo chmod -R 775 /srv

## 4. Core Services

Includes Komga, Mylar, qBittorrent-nox, Prowlarr, and Cloudflare
Tunnel.\
Each service runs via a systemd unit in /etc/systemd/system/.\
Cloudflare Tunnel config path: /etc/cloudflared/config.yml

## 5. Secure SSH via Cloudflare Access

Desktop Access:\
sudo apt install cloudflared\
cloudflared access ssh \--hostname ssh.teresafares.com\
\
\~/.ssh/config:\
Host multiverse\
HostName ssh.teresafares.com\
User teresa\
ProxyCommand cloudflared access ssh \--hostname %h\
\
Connect with ssh multiverse\
\
Mobile Access (Termius):\
Mobile app lacks ProxyCommand; use Tailscale for secure remote SSH
instead.

## 6. Remote Access Options for Mobile

Option 1 -- Tailscale (Recommended):\
curl -fsSL https://tailscale.com/install.sh \| sh\
sudo tailscale up\
Install Tailscale app on phone → login same account\
tailscale ip -4\
Add IP to Termius\
\
Option 2 -- Local Wi-Fi:\
Connect via 192.168.x.x\
\
Option 3 -- Android Cloudflared (Advanced):\
cloudflared access tcp \--hostname ssh.teresafares.com \--url
localhost:2222\
Termius → localhost:2222

## 7. Maintenance & Backups

Backup:\
rsync -av /srv/configs/ /mnt/backup/configs/\
\
Update:\
sudo apt update && sudo apt upgrade -y\
\
Restart All:\
sudo systemctl restart komga mylar qbittorrent-nox prowlarr cloudflared

## 8. Troubleshooting

qBittorrent stuck:\
sudo systemctl stop qbittorrent-nox\
cd \~/.local/share/data/qBittorrent/BT_backup/\
rm -f \*.fastresume \*.torrent\
sudo systemctl start qbittorrent-nox\
\
Mylar metadata issues:\
Check ComicVine API key\
rm -rf \~/.mylar/cache/\
sudo systemctl restart mylar\
\
Cloudflare Tunnel down:\
sudo systemctl status cloudflared\
sudo systemctl restart cloudflared

## 9. Security

\- SSH key-only, no root login\
- UFW restrictive rules\
- Cloudflare Access for HTTPS and SSH\
- System auto-updates enabled\
- Optional: Fail2Ban for SSH brute-force protection\
\
sudo apt install fail2ban -y\
sudo systemctl enable fail2ban \--now

## 10. Summary Table

\| Service \| Port \| Access \| Config Path \|\
\|\-\-\-\-\-\-\-\-\--\|\-\-\-\-\--\|\-\-\-\-\-\-\--\|\-\-\-\-\-\-\-\-\-\-\-\--\|\
\| Komga \| 8080 \| komga.teresafares.com \| /srv/komga \|\
\| Mylar \| 8090 \| mylar.teresafares.com \| /srv/mylar \|\
\| qBittorrent-nox \| 8999 \| qbittorrent.teresafares.com \|
\~/.config/qBittorrent \|\
\| Prowlarr \| 8081 \| prowlarr.teresafares.com \| /srv/configs/prowlarr
\|\
\| Cloudflared \| --- \| Manages HTTPS/SSH tunnels \|
/etc/cloudflared/config.yml \|\
\| Tailscale \| 22 \| 100.x.x.x (VPN SSH) \| /var/lib/tailscale \|

## 11. Future Plans

\- Automate backups via cron\
- Add Prometheus + Grafana monitoring\
- Dockerize Komga/Mylar/qBittorrent\
- Auto-renew Cloudflare Access certs\
- Version control configs with Git
