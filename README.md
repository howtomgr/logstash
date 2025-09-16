# logstash Installation Guide

logstash is a free and open-source data processing pipeline. Logstash provides server-side data processing pipeline

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
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 10GB for data
  - Network: Various inputs
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5044 (default logstash port)
  - HTTP on 9600
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

# Install logstash
sudo dnf install -y logstash

# Enable and start service
sudo systemctl enable --now logstash

# Configure firewall
sudo firewall-cmd --permanent --add-port=5044/tcp
sudo firewall-cmd --reload

# Verify installation
logstash --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install logstash
sudo apt install -y logstash

# Enable and start service
sudo systemctl enable --now logstash

# Configure firewall
sudo ufw allow 5044

# Verify installation
logstash --version
```

### Arch Linux

```bash
# Install logstash
sudo pacman -S logstash

# Enable and start service
sudo systemctl enable --now logstash

# Verify installation
logstash --version
```

### Alpine Linux

```bash
# Install logstash
apk add --no-cache logstash

# Enable and start service
rc-update add logstash default
rc-service logstash start

# Verify installation
logstash --version
```

### openSUSE/SLES

```bash
# Install logstash
sudo zypper install -y logstash

# Enable and start service
sudo systemctl enable --now logstash

# Configure firewall
sudo firewall-cmd --permanent --add-port=5044/tcp
sudo firewall-cmd --reload

# Verify installation
logstash --version
```

### macOS

```bash
# Using Homebrew
brew install logstash

# Start service
brew services start logstash

# Verify installation
logstash --version
```

### FreeBSD

```bash
# Using pkg
pkg install logstash

# Enable in rc.conf
echo 'logstash_enable="YES"' >> /etc/rc.conf

# Start service
service logstash start

# Verify installation
logstash --version
```

### Windows

```bash
# Using Chocolatey
choco install logstash

# Or using Scoop
scoop install logstash

# Verify installation
logstash --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/logstash

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
logstash --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable logstash

# Start service
sudo systemctl start logstash

# Stop service
sudo systemctl stop logstash

# Restart service
sudo systemctl restart logstash

# Check status
sudo systemctl status logstash

# View logs
sudo journalctl -u logstash -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add logstash default

# Start service
rc-service logstash start

# Stop service
rc-service logstash stop

# Restart service
rc-service logstash restart

# Check status
rc-service logstash status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'logstash_enable="YES"' >> /etc/rc.conf

# Start service
service logstash start

# Stop service
service logstash stop

# Restart service
service logstash restart

# Check status
service logstash status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start logstash
brew services stop logstash
brew services restart logstash

# Check status
brew services list | grep logstash
```

### Windows Service Manager

```powershell
# Start service
net start logstash

# Stop service
net stop logstash

# Using PowerShell
Start-Service logstash
Stop-Service logstash
Restart-Service logstash

# Check status
Get-Service logstash
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream logstash_backend {
    server 127.0.0.1:5044;
}

server {
    listen 80;
    server_name logstash.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name logstash.example.com;

    ssl_certificate /etc/ssl/certs/logstash.example.com.crt;
    ssl_certificate_key /etc/ssl/private/logstash.example.com.key;

    location / {
        proxy_pass http://logstash_backend;
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
    ServerName logstash.example.com
    Redirect permanent / https://logstash.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName logstash.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/logstash.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/logstash.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5044/
    ProxyPassReverse / http://127.0.0.1:5044/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend logstash_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/logstash.pem
    redirect scheme https if !{ ssl_fc }
    default_backend logstash_backend

backend logstash_backend
    balance roundrobin
    server logstash1 127.0.0.1:5044 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R logstash:logstash /etc/logstash
sudo chmod 750 /etc/logstash

# Configure firewall
sudo firewall-cmd --permanent --add-port=5044/tcp
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
sudo systemctl status logstash

# View logs
sudo journalctl -u logstash -f

# Monitor resource usage
top -p $(pgrep logstash)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/logstash"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/logstash-backup-$DATE.tar.gz" /etc/logstash /var/lib/logstash

echo "Backup completed: $BACKUP_DIR/logstash-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop logstash

# Restore from backup
tar -xzf /backup/logstash/logstash-backup-*.tar.gz -C /

# Start service
sudo systemctl start logstash
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u logstash -n 100
sudo tail -f /var/log/logstash/logstash.log

# Check configuration
logstash --version

# Check permissions
ls -la /etc/logstash
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5044

# Test connectivity
telnet localhost 5044

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep logstash)

# Check disk I/O
iotop -p $(pgrep logstash)

# Check connections
ss -an | grep 5044
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  logstash:
    image: logstash:latest
    ports:
      - "5044:5044"
    volumes:
      - ./config:/etc/logstash
      - ./data:/var/lib/logstash
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update logstash

# Debian/Ubuntu
sudo apt update && sudo apt upgrade logstash

# Arch Linux
sudo pacman -Syu logstash

# Alpine Linux
apk update && apk upgrade logstash

# openSUSE
sudo zypper update logstash

# FreeBSD
pkg update && pkg upgrade logstash

# Always backup before updates
tar -czf /backup/logstash-pre-update-$(date +%Y%m%d).tar.gz /etc/logstash

# Restart after updates
sudo systemctl restart logstash
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/logstash

# Clean old logs
find /var/log/logstash -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/logstash
```

## Additional Resources

- Official Documentation: https://docs.logstash.org/
- GitHub Repository: https://github.com/logstash/logstash
- Community Forum: https://forum.logstash.org/
- Best Practices Guide: https://docs.logstash.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
