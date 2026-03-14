# Apache Installation and Domain Configuration Guide

CentOS

Ubuntu/Debian

## Installing Apache (httpd) on CentOS

The Apache package is already included in the CentOS base repository and can be easily installed using the 'yum' package manager.

### Installation

```
# Check for updates
sudo yum update httpd

# Install Apache
sudo yum install httpd

# Start Apache
sudo systemctl start httpd

# Check status
sudo systemctl status httpd
```

### Firewall Configuration

```
# Allow Apache through firewall (HTTP and HTTPS)
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https

# Reload firewall
sudo firewall-cmd --reload
```

### Apache Management Commands

| Command | Description |
| --- | --- |
| `sudo systemctl start httpd` | Start Apache server |
| `sudo systemctl stop httpd` | Stop Apache server |
| `sudo systemctl restart httpd` | Restart Apache server |
| `sudo systemctl enable httpd` | Enable Apache to start on boot |
| `sudo systemctl disable httpd` | Disable Apache from starting on boot |
| `sudo systemctl status httpd` | Check Apache status |

## Installing Apache2 on Ubuntu/Debian

```
# Update package index
sudo apt update

# Install Apache2
sudo apt install apache2

# Check status
sudo systemctl status apache2
```

### Firewall Configuration (UFW)

```
# Allow Apache through firewall
sudo ufw allow 'Apache'
sudo ufw allow 'Apache Full'
sudo ufw allow 'Apache Secure'  # For HTTPS

# Check firewall status
sudo ufw status
```

## Adding Domains to Apache (VirtualHost Configuration)

**📌 Important:** Before configuring VirtualHosts, you need to set up DNS on the domain registrar side, otherwise no one will see your site.

### Where to Find VirtualHost Settings

| OS  | Configuration Location |
| --- | --- |
| **Ubuntu/Debian** | `/etc/apache2/sites-available/` |
| **CentOS/RHEL** | `/etc/httpd/conf/httpd.conf` (or `/etc/httpd/conf.d/` for separate files) |
| **Other Unix-like** | VirtualHost settings are often in the main `apache2.conf` file |

### VirtualHost Configuration Syntax

Use the `<VirtualHost *:80>` container to add a new virtual host. You can add as many as you need.

#### Basic VirtualHost Structure

<VirtualHost \*:80> # Virtual host settings tag, 80 is the port ServerAdmin admin@domain.com # Administrator's contact email DocumentRoot /var/www/domain.com # Website directory ServerName domain.com # Primary domain name ServerAlias www.domain.com # Alternative names (subdomains) # Error and access logs ErrorLog logs/domain.com-error\_log CustomLog logs/domain.com-access\_log common # Directory rules <Directory /> Options FollowSymLinks AllowOverride All </Directory> </VirtualHost>

### Example: Multiple Domains Configuration

#### Example 1: Two Domains on Ubuntu/Debian

Create configuration files in `/etc/apache2/sites-available/`:

##### First domain: /etc/apache2/sites-available/domain1.com.conf

<VirtualHost \*:80> ServerAdmin admin@domain1.com DocumentRoot /var/www/domain1.com ServerName domain1.com ServerAlias www.domain1.com ErrorLog ${APACHE\_LOG\_DIR}/domain1.com-error.log CustomLog ${APACHE\_LOG\_DIR}/domain1.com-access.log combined <Directory /var/www/domain1.com> Options FollowSymLinks AllowOverride All Require all granted </Directory> </VirtualHost>

##### Second domain: /etc/apache2/sites-available/domain2.com.conf

<VirtualHost \*:80> ServerAdmin admin@domain2.com DocumentRoot /var/www/domain2.com ServerName domain2.com ServerAlias www.domain2.com ErrorLog ${APACHE\_LOG\_DIR}/domain2.com-error.log CustomLog ${APACHE\_LOG\_DIR}/domain2.com-access.log combined <Directory /var/www/domain2.com> Options FollowSymLinks AllowOverride All Require all granted </Directory> </VirtualHost>

##### Enable the sites on Ubuntu/Debian:

```
# Create website directories
sudo mkdir -p /var/www/domain1.com
sudo mkdir -p /var/www/domain2.com

# Set proper permissions
sudo chown -R www-data:www-data /var/www/domain1.com
sudo chown -R www-data:www-data /var/www/domain2.com

# Enable sites
sudo a2ensite domain1.com.conf
sudo a2ensite domain2.com.conf

# Disable default site (optional)
sudo a2dissite 000-default.conf

# Restart Apache
sudo systemctl restart apache2
```

#### Example 2: Multiple Domains on CentOS

Add to `/etc/httpd/conf/httpd.conf` or create separate files in `/etc/httpd/conf.d/`:

\# First domain <VirtualHost \*:80> ServerAdmin user1@domain1.com DocumentRoot /var/www/domain1.com ServerName domain1.com ServerAlias www.domain1.com ErrorLog logs/domain1.com-error\_log CustomLog logs/domain1.com-access\_log common <Directory /var/www/domain1.com> Options FollowSymLinks AllowOverride All Require all granted </Directory> </VirtualHost> # Second domain <VirtualHost \*:80> ServerAdmin user2@domain2.com DocumentRoot /var/www/domain2.com ServerName domain2.com ServerAlias www.domain2.com ErrorLog logs/domain2.com-error\_log CustomLog logs/domain2.com-access\_log common <Directory /var/www/domain2.com> Options FollowSymLinks AllowOverride All Require all granted </Directory> </VirtualHost>

```
# Create website directories
sudo mkdir -p /var/www/domain1.com
sudo mkdir -p /var/www/domain2.com

# Set proper permissions
sudo chown -R apache:apache /var/www/domain1.com
sudo chown -R apache:apache /var/www/domain2.com

# Restart Apache
sudo systemctl restart httpd
```

### VirtualHost Parameters Explained

| Parameter | Description | Example |
| --- | --- | --- |
| `ServerAdmin` | Administrator's email address (shown in error messages) | `admin@domain.com` |
| `DocumentRoot` | Directory where website files are stored | `/var/www/domain.com` |
| `ServerName` | Primary domain name | `domain.com` |
| `ServerAlias` | Alternative domain names (subdomains, www, etc.) | `www.domain.com` |
| `ErrorLog` | Location of error logs | `logs/domain.com-error.log` |
| `CustomLog` | Location of access logs | `logs/domain.com-access.log` |
| `<Directory>` | Directory-specific configuration | Options, AllowOverride, etc. |

### Directory Options Explained

| Option | Description |
| --- | --- |
| `Options FollowSymLinks` | Allows following symbolic links in the directory |
| `AllowOverride All` | Allows .htaccess files to override configuration |
| `Require all granted` | Allows access from all sources (Apache 2.4+) |

### Testing Your Configuration

```
# Test Apache configuration syntax
sudo apachectl configtest

# Or on Ubuntu/Debian
sudo apache2ctl configtest

# Create a test file for each domain
echo "<h1>Domain 1 Website</h1>" | sudo tee /var/www/domain1.com/index.html
echo "<h1>Domain 2 Website</h1>" | sudo tee /var/www/domain2.com/index.html

# Restart Apache after changes
sudo systemctl restart apache2  # Ubuntu/Debian
sudo systemctl restart httpd    # CentOS
```

### HTTPS Configuration (SSL)

For HTTPS sites, use port 443 and include SSL certificate information:

<VirtualHost \*:443> ServerAdmin admin@domain.com DocumentRoot /var/www/domain.com ServerName domain.com SSLEngine on SSLCertificateFile /path/to/certificate.crt SSLCertificateKeyFile /path/to/private.key SSLCertificateChainFile /path/to/chain.crt ErrorLog ${APACHE\_LOG\_DIR}/domain.com-ssl-error.log CustomLog ${APACHE\_LOG\_DIR}/domain.com-ssl-access.log combined <Directory /var/www/domain.com> Options FollowSymLinks AllowOverride All Require all granted </Directory> </VirtualHost>

## Troubleshooting

| Issue | Solution |
| --- | --- |
| Apache won't start | Check syntax: `apachectl configtest`  <br>Check ports: `sudo netstat -tulpn \| grep :80` |
| Wrong site showing | Check the order of VirtualHosts - first match is used  <br>Use `ServerAlias` to specify all domain variations |
| Permission denied | Ensure DocumentRoot is readable by Apache user:  <br>`sudo chown -R www-data:www-data /var/www/domain.com` (Ubuntu)  <br>`sudo chown -R apache:apache /var/www/domain.com` (CentOS) |
| 404 Not Found | Check if index.html exists in DocumentRoot  <br>Check DirectoryIndex settings |

## Quick Reference: Complete Setup Workflow

1.  **Install Apache** (see installation sections above)
2.  **Configure firewall** to allow HTTP/HTTPS
3.  **Create website directories** for each domain
4.  **Create VirtualHost configuration** for each domain
5.  **Enable sites** (a2ensite on Ubuntu or include files on CentOS)
6.  **Test configuration** with `apachectl configtest`
7.  **Restart Apache** to apply changes
8.  **Configure DNS** at your domain registrar
9.  **Test domains** in browser
