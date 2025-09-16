# vnstat Installation Guide

vnstat is a free and open-source network monitor. vnStat provides network traffic monitor with statistics

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
  - RAM: 64MB minimum
  - Storage: 100MB for data
  - Network: CLI/daemon
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port N/A (default vnstat port)
  - None
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

# Install vnstat
sudo dnf install -y vnstat

# Enable and start service
sudo systemctl enable --now vnstat

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
sudo firewall-cmd --reload

# Verify installation
vnstat --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install vnstat
sudo apt install -y vnstat

# Enable and start service
sudo systemctl enable --now vnstat

# Configure firewall
sudo ufw allow N/A

# Verify installation
vnstat --version
```

### Arch Linux

```bash
# Install vnstat
sudo pacman -S vnstat

# Enable and start service
sudo systemctl enable --now vnstat

# Verify installation
vnstat --version
```

### Alpine Linux

```bash
# Install vnstat
apk add --no-cache vnstat

# Enable and start service
rc-update add vnstat default
rc-service vnstat start

# Verify installation
vnstat --version
```

### openSUSE/SLES

```bash
# Install vnstat
sudo zypper install -y vnstat

# Enable and start service
sudo systemctl enable --now vnstat

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
sudo firewall-cmd --reload

# Verify installation
vnstat --version
```

### macOS

```bash
# Using Homebrew
brew install vnstat

# Start service
brew services start vnstat

# Verify installation
vnstat --version
```

### FreeBSD

```bash
# Using pkg
pkg install vnstat

# Enable in rc.conf
echo 'vnstat_enable="YES"' >> /etc/rc.conf

# Start service
service vnstat start

# Verify installation
vnstat --version
```

### Windows

```bash
# Using Chocolatey
choco install vnstat

# Or using Scoop
scoop install vnstat

# Verify installation
vnstat --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/vnstat

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
vnstat --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable vnstat

# Start service
sudo systemctl start vnstat

# Stop service
sudo systemctl stop vnstat

# Restart service
sudo systemctl restart vnstat

# Check status
sudo systemctl status vnstat

# View logs
sudo journalctl -u vnstat -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add vnstat default

# Start service
rc-service vnstat start

# Stop service
rc-service vnstat stop

# Restart service
rc-service vnstat restart

# Check status
rc-service vnstat status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'vnstat_enable="YES"' >> /etc/rc.conf

# Start service
service vnstat start

# Stop service
service vnstat stop

# Restart service
service vnstat restart

# Check status
service vnstat status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start vnstat
brew services stop vnstat
brew services restart vnstat

# Check status
brew services list | grep vnstat
```

### Windows Service Manager

```powershell
# Start service
net start vnstat

# Stop service
net stop vnstat

# Using PowerShell
Start-Service vnstat
Stop-Service vnstat
Restart-Service vnstat

# Check status
Get-Service vnstat
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream vnstat_backend {
    server 127.0.0.1:N/A;
}

server {
    listen 80;
    server_name vnstat.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name vnstat.example.com;

    ssl_certificate /etc/ssl/certs/vnstat.example.com.crt;
    ssl_certificate_key /etc/ssl/private/vnstat.example.com.key;

    location / {
        proxy_pass http://vnstat_backend;
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
    ServerName vnstat.example.com
    Redirect permanent / https://vnstat.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName vnstat.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/vnstat.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/vnstat.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:N/A/
    ProxyPassReverse / http://127.0.0.1:N/A/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend vnstat_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/vnstat.pem
    redirect scheme https if !{ ssl_fc }
    default_backend vnstat_backend

backend vnstat_backend
    balance roundrobin
    server vnstat1 127.0.0.1:N/A check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R vnstat:vnstat /etc/vnstat
sudo chmod 750 /etc/vnstat

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
sudo systemctl status vnstat

# View logs
sudo journalctl -u vnstat -f

# Monitor resource usage
top -p $(pgrep vnstat)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/vnstat"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/vnstat-backup-$DATE.tar.gz" /etc/vnstat /var/lib/vnstat

echo "Backup completed: $BACKUP_DIR/vnstat-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop vnstat

# Restore from backup
tar -xzf /backup/vnstat/vnstat-backup-*.tar.gz -C /

# Start service
sudo systemctl start vnstat
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u vnstat -n 100
sudo tail -f /var/log/vnstat/vnstat.log

# Check configuration
vnstat --version

# Check permissions
ls -la /etc/vnstat
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
top -p $(pgrep vnstat)

# Check disk I/O
iotop -p $(pgrep vnstat)

# Check connections
ss -an | grep N/A
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  vnstat:
    image: vnstat:latest
    ports:
      - "N/A:N/A"
    volumes:
      - ./config:/etc/vnstat
      - ./data:/var/lib/vnstat
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update vnstat

# Debian/Ubuntu
sudo apt update && sudo apt upgrade vnstat

# Arch Linux
sudo pacman -Syu vnstat

# Alpine Linux
apk update && apk upgrade vnstat

# openSUSE
sudo zypper update vnstat

# FreeBSD
pkg update && pkg upgrade vnstat

# Always backup before updates
tar -czf /backup/vnstat-pre-update-$(date +%Y%m%d).tar.gz /etc/vnstat

# Restart after updates
sudo systemctl restart vnstat
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/vnstat

# Clean old logs
find /var/log/vnstat -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/vnstat
```

## Additional Resources

- Official Documentation: https://docs.vnstat.org/
- GitHub Repository: https://github.com/vnstat/vnstat
- Community Forum: https://forum.vnstat.org/
- Best Practices Guide: https://docs.vnstat.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
