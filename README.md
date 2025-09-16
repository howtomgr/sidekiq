# sidekiq Installation Guide

sidekiq is a free and open-source background jobs. Sidekiq provides simple, efficient background processing for Ruby

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
  - Storage: 5GB for jobs
  - Network: Redis connection
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 6379 (default sidekiq port)
  - Web UI on 3000
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

# Install sidekiq
sudo dnf install -y sidekiq

# Enable and start service
sudo systemctl enable --now sidekiq

# Configure firewall
sudo firewall-cmd --permanent --add-port=6379/tcp
sudo firewall-cmd --reload

# Verify installation
sidekiq --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install sidekiq
sudo apt install -y sidekiq

# Enable and start service
sudo systemctl enable --now sidekiq

# Configure firewall
sudo ufw allow 6379

# Verify installation
sidekiq --version
```

### Arch Linux

```bash
# Install sidekiq
sudo pacman -S sidekiq

# Enable and start service
sudo systemctl enable --now sidekiq

# Verify installation
sidekiq --version
```

### Alpine Linux

```bash
# Install sidekiq
apk add --no-cache sidekiq

# Enable and start service
rc-update add sidekiq default
rc-service sidekiq start

# Verify installation
sidekiq --version
```

### openSUSE/SLES

```bash
# Install sidekiq
sudo zypper install -y sidekiq

# Enable and start service
sudo systemctl enable --now sidekiq

# Configure firewall
sudo firewall-cmd --permanent --add-port=6379/tcp
sudo firewall-cmd --reload

# Verify installation
sidekiq --version
```

### macOS

```bash
# Using Homebrew
brew install sidekiq

# Start service
brew services start sidekiq

# Verify installation
sidekiq --version
```

### FreeBSD

```bash
# Using pkg
pkg install sidekiq

# Enable in rc.conf
echo 'sidekiq_enable="YES"' >> /etc/rc.conf

# Start service
service sidekiq start

# Verify installation
sidekiq --version
```

### Windows

```bash
# Using Chocolatey
choco install sidekiq

# Or using Scoop
scoop install sidekiq

# Verify installation
sidekiq --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/sidekiq

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
sidekiq --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable sidekiq

# Start service
sudo systemctl start sidekiq

# Stop service
sudo systemctl stop sidekiq

# Restart service
sudo systemctl restart sidekiq

# Check status
sudo systemctl status sidekiq

# View logs
sudo journalctl -u sidekiq -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add sidekiq default

# Start service
rc-service sidekiq start

# Stop service
rc-service sidekiq stop

# Restart service
rc-service sidekiq restart

# Check status
rc-service sidekiq status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'sidekiq_enable="YES"' >> /etc/rc.conf

# Start service
service sidekiq start

# Stop service
service sidekiq stop

# Restart service
service sidekiq restart

# Check status
service sidekiq status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start sidekiq
brew services stop sidekiq
brew services restart sidekiq

# Check status
brew services list | grep sidekiq
```

### Windows Service Manager

```powershell
# Start service
net start sidekiq

# Stop service
net stop sidekiq

# Using PowerShell
Start-Service sidekiq
Stop-Service sidekiq
Restart-Service sidekiq

# Check status
Get-Service sidekiq
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream sidekiq_backend {
    server 127.0.0.1:6379;
}

server {
    listen 80;
    server_name sidekiq.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name sidekiq.example.com;

    ssl_certificate /etc/ssl/certs/sidekiq.example.com.crt;
    ssl_certificate_key /etc/ssl/private/sidekiq.example.com.key;

    location / {
        proxy_pass http://sidekiq_backend;
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
    ServerName sidekiq.example.com
    Redirect permanent / https://sidekiq.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName sidekiq.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/sidekiq.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/sidekiq.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:6379/
    ProxyPassReverse / http://127.0.0.1:6379/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend sidekiq_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/sidekiq.pem
    redirect scheme https if !{ ssl_fc }
    default_backend sidekiq_backend

backend sidekiq_backend
    balance roundrobin
    server sidekiq1 127.0.0.1:6379 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R sidekiq:sidekiq /etc/sidekiq
sudo chmod 750 /etc/sidekiq

# Configure firewall
sudo firewall-cmd --permanent --add-port=6379/tcp
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
sudo systemctl status sidekiq

# View logs
sudo journalctl -u sidekiq -f

# Monitor resource usage
top -p $(pgrep sidekiq)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/sidekiq"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/sidekiq-backup-$DATE.tar.gz" /etc/sidekiq /var/lib/sidekiq

echo "Backup completed: $BACKUP_DIR/sidekiq-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop sidekiq

# Restore from backup
tar -xzf /backup/sidekiq/sidekiq-backup-*.tar.gz -C /

# Start service
sudo systemctl start sidekiq
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u sidekiq -n 100
sudo tail -f /var/log/sidekiq/sidekiq.log

# Check configuration
sidekiq --version

# Check permissions
ls -la /etc/sidekiq
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 6379

# Test connectivity
telnet localhost 6379

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep sidekiq)

# Check disk I/O
iotop -p $(pgrep sidekiq)

# Check connections
ss -an | grep 6379
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  sidekiq:
    image: sidekiq:latest
    ports:
      - "6379:6379"
    volumes:
      - ./config:/etc/sidekiq
      - ./data:/var/lib/sidekiq
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update sidekiq

# Debian/Ubuntu
sudo apt update && sudo apt upgrade sidekiq

# Arch Linux
sudo pacman -Syu sidekiq

# Alpine Linux
apk update && apk upgrade sidekiq

# openSUSE
sudo zypper update sidekiq

# FreeBSD
pkg update && pkg upgrade sidekiq

# Always backup before updates
tar -czf /backup/sidekiq-pre-update-$(date +%Y%m%d).tar.gz /etc/sidekiq

# Restart after updates
sudo systemctl restart sidekiq
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/sidekiq

# Clean old logs
find /var/log/sidekiq -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/sidekiq
```

## Additional Resources

- Official Documentation: https://docs.sidekiq.org/
- GitHub Repository: https://github.com/sidekiq/sidekiq
- Community Forum: https://forum.sidekiq.org/
- Best Practices Guide: https://docs.sidekiq.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
