# Nginx Reverse Proxy Demo

This project demonstrates how to set up nginx as a reverse proxy for multiple web applications. It includes two sample applications:

- **Budget Tracker** (`/budget`) - A personal finance management application
- **Home Roster** (`/roster`) - A household chore management system

> **🔧 Configuration Note**: This guide uses example IP address `35.183.32.162` and DNS `ec2-35-183-32-162.ca-central-1.compute.amazonaws.com` for demonstration purposes. **You must replace these with your own server's actual public IP address and domain name** throughout the configuration files and commands.

## Project Structure

```
nginx-proxy/
├── budget-app/
│   └── index.html          # Budget Tracker application
├── roster-app/
│   └── index.html          # Home Roster application
├── index.html              # Main landing page
├── nginx.conf              # Nginx reverse proxy configuration
└── README.md               # This file
```

## Installation on Ubuntu Server

Follow these steps to manually install and configure nginx on an Ubuntu server.

> **Note**: The `index.html` file is designed to work both locally (for testing) and through nginx. When opened locally, it will link directly to the app files. When served through nginx, the links will work with the reverse proxy paths (`/budget` and `/roster`).

### Prerequisites

- Ubuntu 20.04 LTS or later
- Root or sudo access
- **Your own public IP address** (replace `35.183.32.162` with your actual server IP)
- **Your own public DNS** (replace `ec2-35-183-32-162.ca-central-1.compute.amazonaws.com` with your actual domain)

> **⚠️ Important**: The IP address `35.183.32.162` and DNS `ec2-35-183-32-162.ca-central-1.compute.amazonaws.com` used in this guide are **example values only**. You must replace these with your own server's actual public IP address and domain name.

### Step 1: Update System Packages

```bash
# Update package list
sudo apt update

# Upgrade existing packages
sudo apt upgrade -y

# Install essential packages
sudo apt install -y curl wget unzip software-properties-common
```

### Step 2: Install Nginx

```bash
# Install nginx
sudo apt install -y nginx

# Start nginx service
sudo systemctl start nginx

# Enable nginx to start on boot
sudo systemctl enable nginx

# Check nginx status
sudo systemctl status nginx
```

### Step 3: Configure Firewall

```bash
# Check if ufw is active
sudo ufw status

# Allow HTTP traffic
sudo ufw allow 'Nginx Full'

# Allow SSH (if not already allowed)
sudo ufw allow ssh

# Enable firewall if not already enabled
sudo ufw --force enable

# Check firewall status
sudo ufw status
```

### Step 4: Create Directory Structure

```bash
# Create directories for web applications in nginx default location
sudo mkdir -p /usr/share/nginx/html/budget-app
sudo mkdir -p /usr/share/nginx/html/roster-app

# Set proper permissions
sudo chown -R www-data:www-data /usr/share/nginx/html/
sudo chmod -R 755 /usr/share/nginx/html/
```

### Step 5: Deploy Web Applications

```bash
# Copy budget app files
sudo cp -r budget-app/* /usr/share/nginx/html/budget-app/

# Copy roster app files
sudo cp -r roster-app/* /usr/share/nginx/html/roster-app/

# Copy main landing page
sudo cp index.html /usr/share/nginx/html/

# Set proper ownership
sudo chown -R www-data:www-data /usr/share/nginx/html/
```

### Step 6: Configure Nginx Reverse Proxy

```bash
# Backup original nginx configuration
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.backup

# Copy the nginx configuration file
sudo cp nginx.conf /etc/nginx/sites-available/default
```

### Step 7: Test and Reload Nginx

```bash
# Test nginx configuration
sudo nginx -t

# If test passes, reload nginx
sudo systemctl reload nginx

# Check nginx status
sudo systemctl status nginx
```

### Step 8: Verify Installation

```bash
# Check if nginx is running
sudo systemctl status nginx

# Test the health endpoint
curl http://localhost/health

# Test the applications
curl http://localhost/budget
curl http://localhost/roster
```

### Step 9: Access Your Applications

Once everything is set up, you can access your applications at:

> **📝 Replace with your actual server details:**
- **Main Landing Page**: `http://YOUR_PUBLIC_IP/`
- **Budget Tracker**: `http://YOUR_PUBLIC_IP/budget`
- **Home Roster**: `http://YOUR_PUBLIC_IP/roster`
- **Health Check**: `http://YOUR_PUBLIC_IP/health`

**Example** (using the placeholder values from this guide):
- **Main Landing Page**: `http://35.183.32.162/`
- **Budget Tracker**: `http://35.183.32.162/budget`
- **Home Roster**: `http://35.183.32.162/roster`
- **Health Check**: `http://35.183.32.162/health`

## Optional: SSL/HTTPS Configuration

To add SSL/HTTPS support, you'll need SSL certificates. Here's how to set it up with Let's Encrypt:

### Install Certbot

```bash
# Install snapd
sudo apt install -y snapd

# Install certbot
sudo snap install --classic certbot

# Create symlink
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### Obtain SSL Certificate

```bash
# Get SSL certificate
sudo certbot --nginx -d 35.183.32.162 -d ec2-35-183-32-162.ca-central-1.compute.amazonaws.com

# Test automatic renewal
sudo certbot renew --dry-run
```

## Troubleshooting

### Common Issues

1. **Nginx won't start**
   ```bash
   # Check nginx configuration
   sudo nginx -t
   
   # Check nginx logs
   sudo journalctl -u nginx
   ```

2. **403 Forbidden Error**
   ```bash
   # Check file permissions
   sudo chown -R www-data:www-data /var/www/
   sudo chmod -R 755 /var/www/
   ```

3. **502 Bad Gateway**
   ```bash
   # Check if nginx is running
   sudo systemctl status nginx
   
   # Check nginx error logs
   sudo tail -f /var/log/nginx/error.log
   ```

4. **Can't access from external IP**
   ```bash
   # Check firewall status
   sudo ufw status
   
   # Check if port 80 is open
   sudo netstat -tlnp | grep :80
   ```

### Useful Commands

```bash
# Restart nginx
sudo systemctl restart nginx

# Reload nginx configuration
sudo systemctl reload nginx

# Check nginx configuration
sudo nginx -t

# View nginx access logs
sudo tail -f /var/log/nginx/access.log

# View nginx error logs
sudo tail -f /var/log/nginx/error.log

# Check nginx status
sudo systemctl status nginx
```

## Security Considerations

1. **Keep system updated**: Regularly update Ubuntu and nginx
2. **Configure firewall**: Only open necessary ports
3. **Use HTTPS**: Implement SSL/TLS for production
4. **Regular backups**: Backup your configuration and data
5. **Monitor logs**: Regularly check nginx logs for issues

## Performance Optimization

1. **Enable gzip compression**: Already configured in the nginx config
2. **Set up caching**: Static files are cached for 1 year
3. **Use CDN**: Consider using a CDN for static assets
4. **Monitor performance**: Use tools like `htop` and `nginx-status`

## Next Steps

1. **Add more applications**: Extend the reverse proxy for additional services
2. **Implement load balancing**: Add multiple backend servers
3. **Set up monitoring**: Use tools like Prometheus and Grafana
4. **Add authentication**: Implement basic auth or OAuth
5. **Database integration**: Connect applications to databases

## Support

For issues or questions:
1. Check nginx logs: `/var/log/nginx/`
2. Verify configuration: `sudo nginx -t`
3. Check system resources: `htop`, `df -h`
4. Review firewall settings: `sudo ufw status`

---

**Note**: Remember to replace the IP address `35.183.32.162` and DNS name `ec2-35-183-32-162.ca-central-1.compute.amazonaws.com` with your actual server's IP address and domain name.
