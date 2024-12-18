# Keycloak Server Setup Guide

This guide provides step-by-step instructions for setting up Keycloak 26.0.6 with SSL on Ubuntu 24.04 using Let's Encrypt certificates.

## Prerequisites
- Ubuntu 24.04 LTS server
- Root or sudo access
- Domain name pointed to your server
- Open ports: 80 (for SSL setup), 8443 (for Keycloak)

## Installation Steps

### 1. Update System and Install Dependencies
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Java
sudo apt install default-jdk -y
```

### 2. Install Keycloak
```bash
# Download and extract Keycloak
cd /opt
sudo wget https://github.com/keycloak/keycloak/releases/download/26.0.6/keycloak-26.0.6.tar.gz
sudo tar xzf keycloak-26.0.6.tar.gz
sudo mv keycloak-26.0.6 keycloak

# Create dedicated user and set permissions
sudo groupadd keycloak
sudo useradd -r -g keycloak -d /opt/keycloak -s /sbin/nologin keycloak
sudo chown -R keycloak:keycloak /opt/keycloak

# Clean up
sudo rm keycloak-26.0.6.tar.gz
```

### 3. Set Up SSL Certificates with Let's Encrypt
```bash
# Install Certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

# Generate SSL certificate (replace with your domain)
sudo certbot certonly --standalone -d your-domain.com
```

### 4. Configure Certificate Permissions
```bash
# Set ownership and permissions for archive directory
sudo chown -R root:keycloak /etc/letsencrypt/archive/your-domain.com/
sudo chmod 750 /etc/letsencrypt/archive/your-domain.com/

# Set permissions for certificate files
sudo chmod 640 /etc/letsencrypt/archive/your-domain.com/cert1.pem
sudo chmod 640 /etc/letsencrypt/archive/your-domain.com/chain1.pem
sudo chmod 640 /etc/letsencrypt/archive/your-domain.com/fullchain1.pem
sudo chmod 640 /etc/letsencrypt/archive/your-domain.com/privkey1.pem

# Set permissions for live directory
sudo chown -R root:keycloak /etc/letsencrypt/live/your-domain.com/
sudo chmod 750 /etc/letsencrypt/live/your-domain.com/

# Set parent directory permissions
sudo chmod 755 /etc/letsencrypt/archive
sudo chmod 755 /etc/letsencrypt/live
```

### 5. Configure Keycloak
```bash
# Create configuration directory
sudo mkdir /opt/keycloak/conf

# Create and edit configuration file
sudo nano /opt/keycloak/conf/keycloak.conf
```

Add the following configuration (replace with your domain):
```properties
hostname=your-domain.com
https-certificate-file=/etc/letsencrypt/live/your-domain.com/fullchain.pem
https-certificate-key-file=/etc/letsencrypt/live/your-domain.com/privkey.pem
http-enabled=false
https-port=8443
db=dev-file
```

### 6. Create Systemd Service
```bash
# Create service file
sudo nano /etc/systemd/system/keycloak.service
```

Add the following content:
```ini
[Unit]
Description=Keycloak Server
After=network.target

[Service]
Type=simple
User=keycloak
Group=keycloak
Environment=KEYCLOAK_ADMIN=admin
Environment=KEYCLOAK_ADMIN_PASSWORD=your-secure-password
ExecStart=/opt/keycloak/bin/kc.sh start
WorkingDirectory=/opt/keycloak
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### 7. Build and Start Keycloak
```bash
# Build optimized distribution
cd /opt/keycloak
sudo -E ./bin/kc.sh build

# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable keycloak
sudo systemctl start keycloak
```

### 8. Verify Installation
```bash
# Check service status
sudo systemctl status keycloak

# View logs
sudo journalctl -u keycloak -f
```

Access your Keycloak instance at:
- Admin Console: https://your-domain.com:8443/admin
- Account Console: https://your-domain.com:8443/realms/master/account

## Troubleshooting

### Certificate Permission Issues
If you encounter certificate access errors, verify permissions:
```bash
ls -la /etc/letsencrypt/live/your-domain.com/
ls -la /etc/letsencrypt/archive/your-domain.com/
```

### Service Start Issues
If the service fails to start:
1. Check logs: `sudo journalctl -u keycloak -f`
2. Verify configuration file syntax
3. Ensure certificate paths are correct
4. Confirm port 8443 is not in use: `sudo lsof -i:8443`

## Security Considerations
1. Use strong passwords for admin accounts
2. Configure firewall rules to restrict access to port 8443
3. Set up automatic certificate renewal
4. Regularly update Keycloak and system packages
5. Consider implementing a reverse proxy for additional security

## Certificate Renewal
Let's Encrypt certificates expire after 90 days. Test automatic renewal:
```bash
sudo certbot renew --dry-run
```

## Contributing
Feel free to submit issues and enhancement requests!

## License
This project is licensed under the MIT License - see the LICENSE file for details.
