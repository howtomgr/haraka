# Haraka Installation Guide

Haraka is a free and open-source Mail Server. A highly scalable node.js email server

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
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 25/587 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 25/587 (default haraka port)
- **Dependencies**:
  - nodejs, npm
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

# Install haraka
sudo dnf install -y haraka nodejs, npm

# Enable and start service
sudo systemctl enable --now haraka

# Configure firewall
sudo firewall-cmd --permanent --add-service=haraka
sudo firewall-cmd --reload

# Verify installation
haraka --version || systemctl status haraka
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install haraka
sudo apt install -y haraka nodejs, npm

# Enable and start service
sudo systemctl enable --now haraka

# Configure firewall
sudo ufw allow 25/587

# Verify installation
haraka --version || systemctl status haraka
```

### Arch Linux

```bash
# Install haraka
sudo pacman -S haraka

# Enable and start service
sudo systemctl enable --now haraka

# Verify installation
haraka --version || systemctl status haraka
```

### Alpine Linux

```bash
# Install haraka
apk add --no-cache haraka

# Enable and start service
rc-update add haraka default
rc-service haraka start

# Verify installation
haraka --version || rc-service haraka status
```

### openSUSE/SLES

```bash
# Install haraka
sudo zypper install -y haraka nodejs, npm

# Enable and start service
sudo systemctl enable --now haraka

# Configure firewall
sudo firewall-cmd --permanent --add-service=haraka
sudo firewall-cmd --reload

# Verify installation
haraka --version || systemctl status haraka
```

### macOS

```bash
# Using Homebrew
brew install haraka

# Start service
brew services start haraka

# Verify installation
haraka --version
```

### FreeBSD

```bash
# Using pkg
pkg install haraka

# Enable in rc.conf
echo 'haraka_enable="YES"' >> /etc/rc.conf

# Start service
service haraka start

# Verify installation
haraka --version || service haraka status
```

### Windows

```powershell
# Using Chocolatey
choco install haraka

# Or using Scoop
scoop install haraka

# Verify installation
haraka --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/haraka

# Set up basic configuration
sudo tee /etc/haraka/haraka.conf << 'EOF'
# Haraka Configuration
max_connections=1000
EOF

# Test configuration
sudo haraka -t || sudo haraka configtest

# Reload service
sudo systemctl reload haraka
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R haraka:haraka /etc/haraka
sudo chmod 750 /etc/haraka

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable haraka

# Start service
sudo systemctl start haraka

# Stop service
sudo systemctl stop haraka

# Restart service
sudo systemctl restart haraka

# Reload configuration
sudo systemctl reload haraka

# Check status
sudo systemctl status haraka

# View logs
sudo journalctl -u haraka -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add haraka default

# Start service
rc-service haraka start

# Stop service
rc-service haraka stop

# Restart service
rc-service haraka restart

# Check status
rc-service haraka status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'haraka_enable="YES"' >> /etc/rc.conf

# Start service
service haraka start

# Stop service
service haraka stop

# Restart service
service haraka restart

# Check status
service haraka status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start haraka
brew services stop haraka
brew services restart haraka

# Check status
brew services list | grep haraka
```

### Windows Service Manager

```powershell
# Start service
net start haraka

# Stop service
net stop haraka

# Using PowerShell
Start-Service haraka
Stop-Service haraka
Restart-Service haraka

# Check status
Get-Service haraka
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/haraka/haraka.conf << 'EOF'
max_connections=1000
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart haraka
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream haraka_backend {
    server 127.0.0.1:25/587;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name haraka.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name haraka.example.com;

    ssl_certificate /etc/ssl/certs/haraka.example.com.crt;
    ssl_certificate_key /etc/ssl/private/haraka.example.com.key;

    location / {
        proxy_pass http://haraka_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName haraka.example.com
    Redirect permanent / https://haraka.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName haraka.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/haraka.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/haraka.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:25/587/
    ProxyPassReverse / http://127.0.0.1:25/587/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:25/587/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend haraka_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/haraka.pem
    redirect scheme https if !{ ssl_fc }
    default_backend haraka_backend

backend haraka_backend
    balance roundrobin
    option httpchk GET /health
    server haraka1 127.0.0.1:25/587 check
    server haraka2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R haraka:haraka /etc/haraka
sudo chmod 750 /etc/haraka

# Configure firewall
sudo firewall-cmd --permanent --add-service=haraka
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/haraka.conf << 'EOF'
[haraka]
enabled = true
port = 25/587
filter = haraka
logpath = /var/log/haraka/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/haraka.key \
    -out /etc/ssl/certs/haraka.crt

# Configure SSL in haraka
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE haraka_db;
CREATE USER haraka_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE haraka_db TO haraka_user;
EOF

# Configure haraka to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE haraka_db;
CREATE USER 'haraka_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON haraka_db.* TO 'haraka_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Haraka specific tuning
max_connections=1000
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
haraka soft nofile 65535
haraka hard nofile 65535
haraka soft nproc 32768
haraka hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'haraka'
    static_configs:
      - targets: ['localhost:25/587']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet haraka; then
    echo "Haraka is running"
    exit 0
else
    echo "Haraka is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/haraka << 'EOF'
/var/log/haraka/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 haraka haraka
    postrotate
        systemctl reload haraka > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Haraka backup script
BACKUP_DIR="/backup/haraka"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop haraka

# Backup configuration
tar -czf "$BACKUP_DIR/haraka-config-$DATE.tar.gz" /etc/haraka

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/haraka-data-$DATE.tar.gz" /var/lib/haraka

# Start service
systemctl start haraka

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop haraka

# Restore configuration
sudo tar -xzf /backup/haraka/haraka-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/haraka/haraka-data-*.tar.gz -C /

# Set permissions
sudo chown -R haraka:haraka /etc/haraka
sudo chown -R haraka:haraka /var/lib/haraka

# Start service
sudo systemctl start haraka
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u haraka -n 100
sudo tail -f /var/log/haraka/*.log

# Check configuration
sudo haraka -t || sudo haraka configtest

# Check permissions
ls -la /etc/haraka
ls -la /var/lib/haraka
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 25/587
sudo netstat -tlnp | grep 25/587

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 25/587
nc -zv localhost 25/587
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep haraka)
htop -p $(pgrep haraka)

# Check connections
ss -ant | grep :25/587 | wc -l

# Monitor I/O
iotop -p $(pgrep haraka)
```

### Debug Mode

```bash
# Run in debug mode
sudo haraka -d
# or
sudo haraka debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  haraka:
    image: haraka:latest
    container_name: haraka
    ports:
      - "25/587:25/587"
    volumes:
      - ./config:/etc/haraka
      - ./data:/var/lib/haraka
    environment:
      - haraka_CONFIG=/etc/haraka/haraka.conf
    restart: unless-stopped
    networks:
      - haraka_net

networks:
  haraka_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: haraka
spec:
  replicas: 3
  selector:
    matchLabels:
      app: haraka
  template:
    metadata:
      labels:
        app: haraka
    spec:
      containers:
      - name: haraka
        image: haraka:latest
        ports:
        - containerPort: 25/587
        volumeMounts:
        - name: config
          mountPath: /etc/haraka
      volumes:
      - name: config
        configMap:
          name: haraka-config
---
apiVersion: v1
kind: Service
metadata:
  name: haraka
spec:
  selector:
    app: haraka
  ports:
  - port: 25/587
    targetPort: 25/587
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Haraka
  hosts: all
  become: yes
  tasks:
    - name: Install haraka
      package:
        name: haraka
        state: present
    
    - name: Configure haraka
      template:
        src: haraka.conf.j2
        dest: /etc/haraka/haraka.conf
        owner: haraka
        group: haraka
        mode: '0640'
      notify: restart haraka
    
    - name: Start and enable haraka
      systemd:
        name: haraka
        state: started
        enabled: yes
  
  handlers:
    - name: restart haraka
      systemd:
        name: haraka
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update haraka

# Debian/Ubuntu
sudo apt update && sudo apt upgrade haraka

# Arch Linux
sudo pacman -Syu haraka

# Alpine Linux
apk update && apk upgrade haraka

# openSUSE
sudo zypper update haraka

# FreeBSD
pkg update && pkg upgrade haraka

# Always backup before updates
tar -czf /backup/haraka-pre-update-$(date +%Y%m%d).tar.gz /etc/haraka

# Restart after updates
sudo systemctl restart haraka
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/haraka -name "*.log" -mtime +30 -delete

# Verify integrity
sudo haraka --verify || sudo haraka check

# Update databases (if applicable)
sudo haraka-update-db

# Optimize performance
sudo haraka-optimize

# Check for security updates
sudo haraka --security-check
```

## Additional Resources

- Official Documentation: https://docs.haraka.org/
- GitHub Repository: https://github.com/haraka/haraka
- Community Forum: https://forum.haraka.org/
- Wiki: https://wiki.haraka.org/
- Comparison vs Postfix, Exim, PowerMTA, MailerQ: https://docs.haraka.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
