# podman Installation Guide

podman is a free and open-source daemonless container engine. Podman provides Docker-compatible container management without a daemon, offering better security and systemd integration

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
  - CPU: 1 core minimum
  - RAM: 512MB minimum
  - Storage: 1GB for installation
  - Network: Container networking
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port N/A (default podman port)
  - API on 8080 if enabled
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

# Install podman
sudo dnf install -y podman

# Enable and start service
sudo systemctl enable --now podman

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
sudo firewall-cmd --reload

# Verify installation
podman --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install podman
sudo apt install -y podman

# Enable and start service
sudo systemctl enable --now podman

# Configure firewall
sudo ufw allow N/A

# Verify installation
podman --version
```

### Arch Linux

```bash
# Install podman
sudo pacman -S podman

# Enable and start service
sudo systemctl enable --now podman

# Verify installation
podman --version
```

### Alpine Linux

```bash
# Install podman
apk add --no-cache podman

# Enable and start service
rc-update add podman default
rc-service podman start

# Verify installation
podman --version
```

### openSUSE/SLES

```bash
# Install podman
sudo zypper install -y podman

# Enable and start service
sudo systemctl enable --now podman

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
sudo firewall-cmd --reload

# Verify installation
podman --version
```

### macOS

```bash
# Using Homebrew
brew install podman

# Start service
brew services start podman

# Verify installation
podman --version
```

### FreeBSD

```bash
# Using pkg
pkg install podman

# Enable in rc.conf
echo 'podman_enable="YES"' >> /etc/rc.conf

# Start service
service podman start

# Verify installation
podman --version
```

### Windows

```bash
# Using Chocolatey
choco install podman

# Or using Scoop
scoop install podman

# Verify installation
podman --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/podman

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
podman --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable podman

# Start service
sudo systemctl start podman

# Stop service
sudo systemctl stop podman

# Restart service
sudo systemctl restart podman

# Check status
sudo systemctl status podman

# View logs
sudo journalctl -u podman -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add podman default

# Start service
rc-service podman start

# Stop service
rc-service podman stop

# Restart service
rc-service podman restart

# Check status
rc-service podman status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'podman_enable="YES"' >> /etc/rc.conf

# Start service
service podman start

# Stop service
service podman stop

# Restart service
service podman restart

# Check status
service podman status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start podman
brew services stop podman
brew services restart podman

# Check status
brew services list | grep podman
```

### Windows Service Manager

```powershell
# Start service
net start podman

# Stop service
net stop podman

# Using PowerShell
Start-Service podman
Stop-Service podman
Restart-Service podman

# Check status
Get-Service podman
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream podman_backend {
    server 127.0.0.1:N/A;
}

server {
    listen 80;
    server_name podman.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name podman.example.com;

    ssl_certificate /etc/ssl/certs/podman.example.com.crt;
    ssl_certificate_key /etc/ssl/private/podman.example.com.key;

    location / {
        proxy_pass http://podman_backend;
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
    ServerName podman.example.com
    Redirect permanent / https://podman.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName podman.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/podman.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/podman.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:N/A/
    ProxyPassReverse / http://127.0.0.1:N/A/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend podman_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/podman.pem
    redirect scheme https if !{ ssl_fc }
    default_backend podman_backend

backend podman_backend
    balance roundrobin
    server podman1 127.0.0.1:N/A check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R podman:podman /etc/podman
sudo chmod 750 /etc/podman

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
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
sudo systemctl status podman

# View logs
sudo journalctl -u podman -f

# Monitor resource usage
top -p $(pgrep podman)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/podman"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/podman-backup-$DATE.tar.gz" /etc/podman /var/lib/podman

echo "Backup completed: $BACKUP_DIR/podman-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop podman

# Restore from backup
tar -xzf /backup/podman/podman-backup-*.tar.gz -C /

# Start service
sudo systemctl start podman
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u podman -n 100
sudo tail -f /var/log/podman/podman.log

# Check configuration
podman --version

# Check permissions
ls -la /etc/podman
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep N/A

# Test connectivity
telnet localhost N/A

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep podman)

# Check disk I/O
iotop -p $(pgrep podman)

# Check connections
ss -an | grep N/A
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  podman:
    image: podman:latest
    ports:
      - "N/A:N/A"
    volumes:
      - ./config:/etc/podman
      - ./data:/var/lib/podman
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update podman

# Debian/Ubuntu
sudo apt update && sudo apt upgrade podman

# Arch Linux
sudo pacman -Syu podman

# Alpine Linux
apk update && apk upgrade podman

# openSUSE
sudo zypper update podman

# FreeBSD
pkg update && pkg upgrade podman

# Always backup before updates
tar -czf /backup/podman-pre-update-$(date +%Y%m%d).tar.gz /etc/podman

# Restart after updates
sudo systemctl restart podman
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/podman

# Clean old logs
find /var/log/podman -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/podman
```

## Additional Resources

- Official Documentation: https://docs.podman.org/
- GitHub Repository: https://github.com/podman/podman
- Community Forum: https://forum.podman.org/
- Best Practices Guide: https://docs.podman.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
