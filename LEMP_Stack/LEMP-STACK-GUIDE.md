# ProjectLEMP - My LEMP Stack Implementation

This project demonstrates my implementation of a LEMP (Linux, Nginx, MySQL, PHP) stack on AWS EC2. LEMP is a powerful alternative to LAMP, using Nginx instead of Apache for better performance and resource efficiency.

## What I Implemented

### 1. AWS EC2 Instance Setup
I launched an Ubuntu 22.04 LTS instance on AWS EC2 and configured the security groups to allow:
- SSH access (port 22) for server management
- HTTP traffic (port 80) for web access
- HTTPS traffic (port 443) for secure web access
- MySQL access (port 3306) for database connections (if needed externally)

```bash
ssh -i my-key.pem ubuntu@my-ec2-ip - (dummy)
```

### 2. LEMP Stack Installation
I installed each component of the LEMP stack:

**Linux**: Used Ubuntu 22.04 LTS as the operating system
**Nginx**: Installed and configured the web server
**MySQL**: Set up the database server with security configurations
**PHP**: Installed PHP-FPM with necessary modules for database connectivity

```bash
# System update
sudo apt update && sudo apt upgrade -y

# Nginx installation
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx

# MySQL installation
sudo apt install mysql-server -y
sudo mysql_secure_installation

# PHP installation (PHP-FPM for Nginx)
sudo apt install php-fpm php-mysql php-curl php-gd php-mbstring php-xml php-zip -y
```

### 3. Nginx Server Block Configuration
I created a server block (Nginx equivalent of Apache virtual host) to serve my web application:

```bash
# Created project directory
sudo mkdir /var/www/projectLEMP
sudo chown -R $USER:$USER /var/www/projectLEMP
```

```bash
# Created server block configuration
sudo nano /etc/nginx/sites-available/projectLEMP
```

```nginx
server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    error_log /var/log/nginx/projectLEMP_error.log;
    access_log /var/log/nginx/projectLEMP_access.log;
}
```

```bash
# Enabled my site and tested configuration
sudo ln -s /etc/nginx/sites-available/pprojectLEMP /etc/nginx/sites-enabled/
sudo unlink /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

### 4. PHP-FPM Configuration
I configured PHP-FPM to work optimally with Nginx:

```bash
# Check PHP-FPM version
php --version

# Edit PHP-FPM configuration
sudo nano /etc/php/8.1/fpm/php.ini
```

Key PHP settings I modified:
```ini
cgi.fix_pathinfo=0
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 300
```

```bash
# Restart PHP-FPM
sudo systemctl restart php8.1-fpm
```

### 5. Database Configuration
I created a dedicated database and user for my application:

```bash
mysql -u root -p
```

```sql
CREATE DATABASE projectlemp_db;
CREATE USER 'projectlemp_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON projectlemp_db.* TO 'projectlemp_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 6. Testing and Validation
I created test files to verify all components work together:

```bash
# Created HTML test file
nano /var/www/projectLEMP/index.html
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>My ProjectLEMP</title>
</head>
<body>
    <h1>Welcome to ProjectLEMP!</h1>
    <p>LEMP stack is running successfully on AWS EC2</p>
</body>
</html>
```

```bash
# Created PHP test file
nano /var/www/projectLEMP/info.php
```

```php
<?php
phpinfo();
?>
```

```bash
# Created database connection test
nano /var/www/projectLEMP/test_db.php
```

```php
<?php
$conn = new mysqli("localhost", "projectlemp_user", "secure_password", "projectlemp_db");
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "<h1>My ProjectLEMP is Working!</h1>";
echo "<p>PHP Version: " . phpversion() . "</p>";
echo "<p>Database: Connected successfully</p>";
echo "<p>Server: Nginx</p>";
$conn->close();
?>
```

## Accessing My Application
I can access my LEMP stack application at:
```
http://my-ec2-public-ip
http://my-ec2-public-ip/info.php
http://my-ec2-public-ip/test_db.php
```

## Key Commands I Used
```bash
# Service management
sudo systemctl restart nginx
sudo systemctl restart php8.1-fpm
sudo systemctl restart mysql

# Configuration testing
sudo nginx -t
sudo php-fpm8.1 -t

# Log monitoring
sudo tail -f /var/log/nginx/projectlemp_error.log
sudo tail -f /var/log/nginx/access.log

# File permissions
sudo chown -R www-data:www-data /var/www/projectlemp
sudo chmod -R 755 /var/www/projectlemp
```

## Project Structure I Created
```
/var/www/projectlemp/
├── index.html         # Static homepage
├── index.php          # PHP application entry
├── info.php           # PHP info page
├── test_db.php        # Database test page
├── config/            # Configuration files
│   └── database.php   # Database connection settings
└
```

## LEMP vs LAMP - What I Learned

### Why I Chose Nginx over Apache:
- **Better Performance**: Handles more concurrent connections
- **Lower Memory Usage**: More efficient resource utilization
- **Faster Static Content**: Superior static file serving
- **Better for High Traffic**: Scales better under load
- **Modern Architecture**: Event-driven, non-blocking I/O

### Key Differences I Discovered:
- **Configuration**: Nginx uses server blocks vs Apache's virtual hosts
- **PHP Processing**: PHP-FPM vs mod_php
- **Rewrite Rules**: Different syntax for URL rewriting
- **Modules**: Different module system and availability


## Monitoring and Maintenance

### Log Locations I Monitor:
- Nginx: `/var/log/nginx/`
- PHP-FPM: `/var/log/php8.1-fpm.log`
- MySQL: `/var/log/mysql/`

### Health Check Commands:
```bash
# Check services status
sudo systemctl status nginx
sudo systemctl status php8.1-fpm
sudo systemctl status mysql

# Check resource usage
htop
df -h
free -m
```

## What I Learned
- How to configure Nginx server blocks for multiple sites
- PHP-FPM configuration and optimization
- Nginx performance tuning and security hardening
- SSL certificate management with Let's Encrypt
- LEMP stack advantages over LAMP for high-performance applications
- Modern web server architecture and best practices
- AWS EC2 optimization for web servers
- Monitoring and troubleshooting LEMP stack issues

## Next Steps for Production
- Implement proper backup strategies
- Set up monitoring with tools like Nagios or Zabbix
- Configure load balancing for high availability
- Implement caching with Redis or Memcached
- Set up CI/CD pipelines for automated deployments
- Configure database replication for redundancy