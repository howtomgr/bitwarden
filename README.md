# bitwarden Installation Guide

bitwarden is a free and open-source open source password manager. Bitwarden provides secure password management for individuals and teams, serving as a FOSS alternative to LastPass, 1Password, or Dashlane

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 1 core minimum (2+ recommended)
  - RAM: 512MB minimum (2GB+ recommended)
  - Storage: 1GB for installation
  - Network: HTTPS for secure access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80/443 (default bitwarden port)
  - Port 3012 for WebSocket
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install bitwarden
sudo dnf install -y bitwarden

# Enable and start service
sudo systemctl enable --now bitwarden

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
./bitwarden.sh version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install bitwarden
sudo apt install -y bitwarden

# Enable and start service
sudo systemctl enable --now bitwarden

# Configure firewall
sudo ufw allow 80/443

# Verify installation
./bitwarden.sh version
```

### Arch Linux

```bash
# Install bitwarden
sudo pacman -S bitwarden

# Enable and start service
sudo systemctl enable --now bitwarden

# Verify installation
./bitwarden.sh version
```

### Alpine Linux

```bash
# Install bitwarden
apk add --no-cache bitwarden

# Enable and start service
rc-update add bitwarden default
rc-service bitwarden start

# Verify installation
./bitwarden.sh version
```

### openSUSE/SLES

```bash
# Install bitwarden
sudo zypper install -y bitwarden

# Enable and start service
sudo systemctl enable --now bitwarden

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
./bitwarden.sh version
```

### macOS

```bash
# Using Homebrew
brew install bitwarden

# Start service
brew services start bitwarden

# Verify installation
./bitwarden.sh version
```

### FreeBSD

```bash
# Using pkg
pkg install bitwarden

# Enable in rc.conf
echo 'bitwarden_enable="YES"' >> /etc/rc.conf

# Start service
service bitwarden start

# Verify installation
./bitwarden.sh version
```

### Windows

```bash
# Using Chocolatey
choco install bitwarden

# Or using Scoop
scoop install bitwarden

# Verify installation
./bitwarden.sh version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/bitwarden

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
./bitwarden.sh version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable bitwarden

# Start service
sudo systemctl start bitwarden

# Stop service
sudo systemctl stop bitwarden

# Restart service
sudo systemctl restart bitwarden

# Check status
sudo systemctl status bitwarden

# View logs
sudo journalctl -u bitwarden -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add bitwarden default

# Start service
rc-service bitwarden start

# Stop service
rc-service bitwarden stop

# Restart service
rc-service bitwarden restart

# Check status
rc-service bitwarden status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'bitwarden_enable="YES"' >> /etc/rc.conf

# Start service
service bitwarden start

# Stop service
service bitwarden stop

# Restart service
service bitwarden restart

# Check status
service bitwarden status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start bitwarden
brew services stop bitwarden
brew services restart bitwarden

# Check status
brew services list | grep bitwarden
```

### Windows Service Manager

```powershell
# Start service
net start bitwarden

# Stop service
net stop bitwarden

# Using PowerShell
Start-Service bitwarden
Stop-Service bitwarden
Restart-Service bitwarden

# Check status
Get-Service bitwarden
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream bitwarden_backend {
    server 127.0.0.1:80/443;
}

server {
    listen 80;
    server_name bitwarden.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name bitwarden.example.com;

    ssl_certificate /etc/ssl/certs/bitwarden.example.com.crt;
    ssl_certificate_key /etc/ssl/private/bitwarden.example.com.key;

    location / {
        proxy_pass http://bitwarden_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName bitwarden.example.com
    Redirect permanent / https://bitwarden.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName bitwarden.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/bitwarden.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/bitwarden.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/443/
    ProxyPassReverse / http://127.0.0.1:80/443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend bitwarden_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/bitwarden.pem
    redirect scheme https if !{ ssl_fc }
    default_backend bitwarden_backend

backend bitwarden_backend
    balance roundrobin
    server bitwarden1 127.0.0.1:80/443 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R bitwarden:bitwarden /etc/bitwarden
sudo chmod 750 /etc/bitwarden

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status bitwarden

# View logs
sudo journalctl -u bitwarden -f

# Monitor resource usage
top -p $(pgrep bitwarden)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/bitwarden"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/bitwarden-backup-$DATE.tar.gz" /etc/bitwarden /var/lib/bitwarden

echo "Backup completed: $BACKUP_DIR/bitwarden-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop bitwarden

# Restore from backup
tar -xzf /backup/bitwarden/bitwarden-backup-*.tar.gz -C /

# Start service
sudo systemctl start bitwarden
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u bitwarden -n 100
sudo tail -f /var/log/bitwarden/bitwarden.log

# Check configuration
./bitwarden.sh version

# Check permissions
ls -la /etc/bitwarden
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80/443

# Test connectivity
telnet localhost 80/443

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep bitwarden)

# Check disk I/O
iotop -p $(pgrep bitwarden)

# Check connections
ss -an | grep 80/443
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  bitwarden:
    image: bitwarden:latest
    ports:
      - "80/443:80/443"
    volumes:
      - ./config:/etc/bitwarden
      - ./data:/var/lib/bitwarden
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update bitwarden

# Debian/Ubuntu
sudo apt update && sudo apt upgrade bitwarden

# Arch Linux
sudo pacman -Syu bitwarden

# Alpine Linux
apk update && apk upgrade bitwarden

# openSUSE
sudo zypper update bitwarden

# FreeBSD
pkg update && pkg upgrade bitwarden

# Always backup before updates
tar -czf /backup/bitwarden-pre-update-$(date +%Y%m%d).tar.gz /etc/bitwarden

# Restart after updates
sudo systemctl restart bitwarden
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/bitwarden

# Clean old logs
find /var/log/bitwarden -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/bitwarden
```

## Additional Resources

- Official Documentation: https://docs.bitwarden.org/
- GitHub Repository: https://github.com/bitwarden/bitwarden
- Community Forum: https://forum.bitwarden.org/
- Best Practices Guide: https://docs.bitwarden.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
