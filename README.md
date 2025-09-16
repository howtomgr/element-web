# element-web Installation Guide

element-web is a free and open-source Matrix web client. Element provides web-based Matrix client (formerly Riot)

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
  - RAM: 256MB minimum
  - Storage: 100MB for app
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default element-web port)
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

# Install element-web
sudo dnf install -y element-web

# Enable and start service
sudo systemctl enable --now nginx

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
element-web --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install element-web
sudo apt install -y element-web

# Enable and start service
sudo systemctl enable --now nginx

# Configure firewall
sudo ufw allow 80

# Verify installation
element-web --version
```

### Arch Linux

```bash
# Install element-web
sudo pacman -S element-web

# Enable and start service
sudo systemctl enable --now nginx

# Verify installation
element-web --version
```

### Alpine Linux

```bash
# Install element-web
apk add --no-cache element-web

# Enable and start service
rc-update add nginx default
rc-service nginx start

# Verify installation
element-web --version
```

### openSUSE/SLES

```bash
# Install element-web
sudo zypper install -y element-web

# Enable and start service
sudo systemctl enable --now nginx

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
element-web --version
```

### macOS

```bash
# Using Homebrew
brew install element-web

# Start service
brew services start element-web

# Verify installation
element-web --version
```

### FreeBSD

```bash
# Using pkg
pkg install element-web

# Enable in rc.conf
echo 'nginx_enable="YES"' >> /etc/rc.conf

# Start service
service nginx start

# Verify installation
element-web --version
```

### Windows

```bash
# Using Chocolatey
choco install element-web

# Or using Scoop
scoop install element-web

# Verify installation
element-web --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/element-web

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
element-web --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable nginx

# Start service
sudo systemctl start nginx

# Stop service
sudo systemctl stop nginx

# Restart service
sudo systemctl restart nginx

# Check status
sudo systemctl status nginx

# View logs
sudo journalctl -u nginx -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add nginx default

# Start service
rc-service nginx start

# Stop service
rc-service nginx stop

# Restart service
rc-service nginx restart

# Check status
rc-service nginx status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'nginx_enable="YES"' >> /etc/rc.conf

# Start service
service nginx start

# Stop service
service nginx stop

# Restart service
service nginx restart

# Check status
service nginx status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start element-web
brew services stop element-web
brew services restart element-web

# Check status
brew services list | grep element-web
```

### Windows Service Manager

```powershell
# Start service
net start nginx

# Stop service
net stop nginx

# Using PowerShell
Start-Service nginx
Stop-Service nginx
Restart-Service nginx

# Check status
Get-Service nginx
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream element-web_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name element-web.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name element-web.example.com;

    ssl_certificate /etc/ssl/certs/element-web.example.com.crt;
    ssl_certificate_key /etc/ssl/private/element-web.example.com.key;

    location / {
        proxy_pass http://element-web_backend;
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
    ServerName element-web.example.com
    Redirect permanent / https://element-web.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName element-web.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/element-web.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/element-web.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend element-web_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/element-web.pem
    redirect scheme https if !{ ssl_fc }
    default_backend element-web_backend

backend element-web_backend
    balance roundrobin
    server element-web1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R element-web:element-web /etc/element-web
sudo chmod 750 /etc/element-web

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status nginx

# View logs
sudo journalctl -u nginx -f

# Monitor resource usage
top -p $(pgrep element-web)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/element-web"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/element-web-backup-$DATE.tar.gz" /etc/element-web /var/lib/element-web

echo "Backup completed: $BACKUP_DIR/element-web-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop nginx

# Restore from backup
tar -xzf /backup/element-web/element-web-backup-*.tar.gz -C /

# Start service
sudo systemctl start nginx
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u nginx -n 100
sudo tail -f /var/log/element-web/element-web.log

# Check configuration
element-web --version

# Check permissions
ls -la /etc/element-web
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep element-web)

# Check disk I/O
iotop -p $(pgrep element-web)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  element-web:
    image: element-web:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/element-web
      - ./data:/var/lib/element-web
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update element-web

# Debian/Ubuntu
sudo apt update && sudo apt upgrade element-web

# Arch Linux
sudo pacman -Syu element-web

# Alpine Linux
apk update && apk upgrade element-web

# openSUSE
sudo zypper update element-web

# FreeBSD
pkg update && pkg upgrade element-web

# Always backup before updates
tar -czf /backup/element-web-pre-update-$(date +%Y%m%d).tar.gz /etc/element-web

# Restart after updates
sudo systemctl restart nginx
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/element-web

# Clean old logs
find /var/log/element-web -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/element-web
```

## Additional Resources

- Official Documentation: https://docs.element-web.org/
- GitHub Repository: https://github.com/element-web/element-web
- Community Forum: https://forum.element-web.org/
- Best Practices Guide: https://docs.element-web.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
