# netflow Installation Guide

netflow is a free and open-source flow collection. nfdump provides NetFlow collection and processing

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
  - Storage: 50GB for flows
  - Network: NetFlow protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9995 (default netflow port)
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

# Install netflow
sudo dnf install -y netflow

# Enable and start service
sudo systemctl enable --now netflow

# Configure firewall
sudo firewall-cmd --permanent --add-port=9995/tcp
sudo firewall-cmd --reload

# Verify installation
netflow --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install netflow
sudo apt install -y netflow

# Enable and start service
sudo systemctl enable --now netflow

# Configure firewall
sudo ufw allow 9995

# Verify installation
netflow --version
```

### Arch Linux

```bash
# Install netflow
sudo pacman -S netflow

# Enable and start service
sudo systemctl enable --now netflow

# Verify installation
netflow --version
```

### Alpine Linux

```bash
# Install netflow
apk add --no-cache netflow

# Enable and start service
rc-update add netflow default
rc-service netflow start

# Verify installation
netflow --version
```

### openSUSE/SLES

```bash
# Install netflow
sudo zypper install -y netflow

# Enable and start service
sudo systemctl enable --now netflow

# Configure firewall
sudo firewall-cmd --permanent --add-port=9995/tcp
sudo firewall-cmd --reload

# Verify installation
netflow --version
```

### macOS

```bash
# Using Homebrew
brew install netflow

# Start service
brew services start netflow

# Verify installation
netflow --version
```

### FreeBSD

```bash
# Using pkg
pkg install netflow

# Enable in rc.conf
echo 'netflow_enable="YES"' >> /etc/rc.conf

# Start service
service netflow start

# Verify installation
netflow --version
```

### Windows

```bash
# Using Chocolatey
choco install netflow

# Or using Scoop
scoop install netflow

# Verify installation
netflow --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/netflow

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
netflow --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable netflow

# Start service
sudo systemctl start netflow

# Stop service
sudo systemctl stop netflow

# Restart service
sudo systemctl restart netflow

# Check status
sudo systemctl status netflow

# View logs
sudo journalctl -u netflow -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add netflow default

# Start service
rc-service netflow start

# Stop service
rc-service netflow stop

# Restart service
rc-service netflow restart

# Check status
rc-service netflow status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'netflow_enable="YES"' >> /etc/rc.conf

# Start service
service netflow start

# Stop service
service netflow stop

# Restart service
service netflow restart

# Check status
service netflow status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start netflow
brew services stop netflow
brew services restart netflow

# Check status
brew services list | grep netflow
```

### Windows Service Manager

```powershell
# Start service
net start netflow

# Stop service
net stop netflow

# Using PowerShell
Start-Service netflow
Stop-Service netflow
Restart-Service netflow

# Check status
Get-Service netflow
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream netflow_backend {
    server 127.0.0.1:9995;
}

server {
    listen 80;
    server_name netflow.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name netflow.example.com;

    ssl_certificate /etc/ssl/certs/netflow.example.com.crt;
    ssl_certificate_key /etc/ssl/private/netflow.example.com.key;

    location / {
        proxy_pass http://netflow_backend;
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
    ServerName netflow.example.com
    Redirect permanent / https://netflow.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName netflow.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/netflow.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/netflow.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9995/
    ProxyPassReverse / http://127.0.0.1:9995/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend netflow_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/netflow.pem
    redirect scheme https if !{ ssl_fc }
    default_backend netflow_backend

backend netflow_backend
    balance roundrobin
    server netflow1 127.0.0.1:9995 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R netflow:netflow /etc/netflow
sudo chmod 750 /etc/netflow

# Configure firewall
sudo firewall-cmd --permanent --add-port=9995/tcp
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
sudo systemctl status netflow

# View logs
sudo journalctl -u netflow -f

# Monitor resource usage
top -p $(pgrep netflow)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/netflow"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/netflow-backup-$DATE.tar.gz" /etc/netflow /var/lib/netflow

echo "Backup completed: $BACKUP_DIR/netflow-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop netflow

# Restore from backup
tar -xzf /backup/netflow/netflow-backup-*.tar.gz -C /

# Start service
sudo systemctl start netflow
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u netflow -n 100
sudo tail -f /var/log/netflow/netflow.log

# Check configuration
netflow --version

# Check permissions
ls -la /etc/netflow
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9995

# Test connectivity
telnet localhost 9995

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep netflow)

# Check disk I/O
iotop -p $(pgrep netflow)

# Check connections
ss -an | grep 9995
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  netflow:
    image: netflow:latest
    ports:
      - "9995:9995"
    volumes:
      - ./config:/etc/netflow
      - ./data:/var/lib/netflow
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update netflow

# Debian/Ubuntu
sudo apt update && sudo apt upgrade netflow

# Arch Linux
sudo pacman -Syu netflow

# Alpine Linux
apk update && apk upgrade netflow

# openSUSE
sudo zypper update netflow

# FreeBSD
pkg update && pkg upgrade netflow

# Always backup before updates
tar -czf /backup/netflow-pre-update-$(date +%Y%m%d).tar.gz /etc/netflow

# Restart after updates
sudo systemctl restart netflow
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/netflow

# Clean old logs
find /var/log/netflow -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/netflow
```

## Additional Resources

- Official Documentation: https://docs.netflow.org/
- GitHub Repository: https://github.com/netflow/netflow
- Community Forum: https://forum.netflow.org/
- Best Practices Guide: https://docs.netflow.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
