# LAMP Stack Setup Guide

Complete step-by-step guide to implement LAMP stack on AWS EC2.

![EC2 Setup](images/ec2-setup.png)

## Table of Contents
1. [AWS EC2 Setup](#aws-ec2-setup)
2. [Install LAMP Components](#install-lamp-components)
3. [Configure Apache Virtual Host](#configure-apache-virtual-host)
4. [MySQL Database Setup](#mysql-database-setup)
5. [PHP Configuration](#php-configuration)
6. [Testing & Validation](#testing--validation)
7. [Production Considerations](#production-considerations)

## AWS EC2 Setup

### 1. Launch EC2 Instance

**Instance Configuration:**
- **AMI**: Ubuntu Server 20.04 LTS (Free Tier)
- **Instance Type**: t2.micro (Free Tier)
- **Key Pair**: Create or use existing key pair
- **Storage**: 8 GB gp2 (Free Tier)

**Security Group Configuration:**
```
Type        Protocol    Port Range    Source
SSH         TCP         22           Your IP/0.0.0.0/0
HTTP        TCP         80           0.0.0.0/0
HTTPS       TCP         443          0.0.0.0/0
MySQL       TCP         3306         Your IP (optional)
```

### 2. Connect to Instance

```bash
# Update your key permissions
chmod 400 your-key.pem

# Connect to instance
ssh -i your-key.pem ubuntu@your-ec2-public-ip
```

### 3. Initial Server Setup

```bash
# Update package index
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install curl wget unzip software-properties-common -y

# Configure timezone (optional)
sudo timedatectl set-timezone America/New_York
```

## Install LAMP Components

### 1. Install Apache Web Server

```bash
# Install Apache
sudo apt install apache2 -y

# Enable Apache to start on boot
sudo systemctl enable apache2

# Start Apache service
sudo systemctl start apache2

# Check Apache status
sudo systemctl status apache2

# Test Apache installation
curl http://localhost
```

**Verify Apache Installation:**
- Open browser: `http://your-ec2-public-ip`
- Should see "Apache2 Ubuntu Default Page"

### 2. Install MySQL Database Server

```bash
# Install MySQL server
sudo apt install mysql-server -y

# Start MySQL service
sudo systemctl start mysql
sudo systemctl enable mysql

# Secure MySQL installation
sudo mysql_secure_installation
```

**MySQL Secure Installation Prompts:**
```
- Set root password: YES (choose strong password)
- Remove anonymous users: YES
- Disallow root login remotely: YES
- Remove test database: YES
- Reload privilege tables: YES
```

### 3. Install PHP

```bash
# Install PHP and required modules
sudo apt install php libapache2-mod-php php-mysql php-cli php-curl php-json php-mbstring -y

# Enable PHP module
sudo a2enmod php8.1

# Restart Apache
sudo systemctl restart apache2

# Check PHP version
php -v
```

## Configure Apache Virtual Host

### 1. Create Project Directory

```bash
# Create project directory
sudo mkdir -p /var/www/projectlamp

# Set ownership
sudo chown -R $USER:$USER /var/www/projectlamp

# Set permissions
sudo chmod -R 755 /var/www/projectlamp
```

### 2. Create Virtual Host Configuration

```bash
# Create virtual host config file
sudo nano /etc/apache2/sites-available/projectlamp.conf
```

**Virtual Host Configuration:**
```apache
<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp
    DocumentRoot /var/www/projectlamp
    
    <Directory /var/www/projectlamp>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/projectlamp_error.log
    CustomLog ${APACHE_LOG_DIR}/projectlamp_access.log combined
    
    # Optional: Enable compression
    <IfModule mod_deflate.c>
        AddOutputFilterByType DEFLATE text/plain
        AddOutputFilterByType DEFLATE text/html
        AddOutputFilterByType DEFLATE text/xml
        AddOutputFilterByType DEFLATE text/css
        AddOutputFilterByType DEFLATE application/xml
        AddOutputFilterByType DEFLATE application/xhtml+xml
        AddOutputFilterByType DEFLATE application/rss+xml
        AddOutputFilterByType DEFLATE application/javascript
        AddOutputFilterByType DEFLATE application/x-javascript
    </IfModule>
</VirtualHost>
```

### 3. Enable Virtual Host

```bash
# Enable the new site
sudo a2ensite projectlamp.conf

# Disable default Apache site
sudo a2dissite 000-default.conf

# Enable required modules
sudo a2enmod rewrite
sudo a2enmod deflate

# Test Apache configuration
sudo apache2ctl configtest

# Reload Apache
sudo systemctl reload apache2
```

## MySQL Database Setup

### 1. Create Database and User

```bash
# Login to MySQL as root
sudo mysql -u root -p
```

**MySQL Commands:**
```sql
-- Create database
CREATE DATABASE projectlamp_db;

-- Create user with password
CREATE USER 'projectlamp_user'@'localhost' IDENTIFIED BY 'your_secure_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON projectlamp_db.* TO 'projectlamp_user'@'localhost';

-- Flush privileges
FLUSH PRIVILEGES;

-- Show databases
SHOW DATABASES;

-- Show users
SELECT User, Host FROM mysql.user;

-- Exit MySQL
EXIT;
```

### 2. Test Database Connection

```bash
# Test connection with new user
mysql -u projectlamp_user -p projectlamp_db
```

### 3. Create Sample Table (Optional)

```sql
-- Create a sample table
USE projectlamp_db;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert sample data
INSERT INTO users (name, email) VALUES 
('John Doe', 'john@example.com'),
('Jane Smith', 'jane@example.com');

-- Verify data
SELECT * FROM users;
```

## PHP Configuration

### 1. Configure PHP Settings

```bash
# Edit PHP configuration
sudo nano /etc/php/8.1/apache2/php.ini
```

**Key PHP Settings to Modify:**
```ini
# Increase memory limit
memory_limit = 256M

# Increase upload file size
upload_max_filesize = 64M
post_max_size = 64M

# Increase execution time
max_execution_time = 300

# Enable error reporting (development only)
display_errors = On
error_reporting = E_ALL

# Set timezone
date.timezone = America/New_York
```

### 2. Restart Apache

```bash
sudo systemctl restart apache2
```

## Testing & Validation

### 1. Create PHP Info Page

```bash
# Create PHP info file
sudo nano /var/www/projectlamp/info.php
```

```php
<?php
phpinfo();
?>
```

**Test:** Visit `http://your-ec2-public-ip/info.php`

### 2. Create Database Connection Test

```bash
# Create database test file
sudo nano /var/www/projectlamp/db_test.php
```

```php
<?php
$servername = "localhost";
$username = "projectlamp_user";
$password = "your_secure_password";
$dbname = "projectlamp_db";

try {
    $pdo = new PDO("mysql:host=$servername;dbname=$dbname", $username, $password);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    
    echo "<h2>Database Connection Successful!</h2>";
    
    // Test query
    $stmt = $pdo->query("SELECT * FROM users");
    $users = $stmt->fetchAll(PDO::FETCH_ASSOC);
    
    echo "<h3>Users in Database:</h3>";
    foreach($users as $user) {
        echo "<p>ID: " . $user['id'] . " - Name: " . $user['name'] . " - Email: " . $user['email'] . "</p>";
    }
    
} catch(PDOException $e) {
    echo "Connection failed: " . $e->getMessage();
}
?>
```

### 3. Create Main Application Page

```bash
# Create main index file
sudo nano /var/www/projectlamp/index.php
```

```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ProjectLAMP - LAMP Stack Application</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; background: #f4f4f4; }
        .container { background: white; padding: 30px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        .success { color: #27ae60; }
        .info { background: #ecf0f1; padding: 15px; border-radius: 4px; margin: 10px 0; }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 ProjectLAMP Application</h1>
        <p class="success">✅ LAMP Stack is working successfully!</p>
        
        <div class="info">
            <h3>System Information:</h3>
            <p><strong>Server:</strong> <?php echo $_SERVER['SERVER_SOFTWARE']; ?></p>
            <p><strong>PHP Version:</strong> <?php echo phpversion(); ?></p>
            <p><strong>Server Time:</strong> <?php echo date('Y-m-d H:i:s'); ?></p>
        </div>
        
        <?php
        // Database connection test
        $servername = "localhost";
        $username = "projectlamp_user";
        $password = "your_secure_password";
        $dbname = "projectlamp_db";
        
        try {
            $pdo = new PDO("mysql:host=$servername;dbname=$dbname", $username, $password);
            $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
            echo '<div class="info"><h3>Database Status:</h3><p class="success">✅ MySQL Connection Successful</p></div>';
        } catch(PDOException $e) {
            echo '<div class="info"><h3>Database Status:</h3><p style="color: red;">❌ Connection Failed: ' . $e->getMessage() . '</p></div>';
        }
        ?>
        
        <div class="info">
            <h3>Quick Links:</h3>
            <p><a href="info.php">PHP Info</a> | <a href="db_test.php">Database Test</a></p>
        </div>
    </div>
</body>
</html>
```

### 4. Set Proper Permissions

```bash
# Set ownership to Apache user
sudo chown -R www-data:www-data /var/www/projectlamp

# Set proper permissions
sudo chmod -R 755 /var/www/projectlamp

# Make files readable
sudo chmod -R 644 /var/www/projectlamp/*.php
```

### 5. Final Testing

**Test URLs:**
- Main application: `http://your-ec2-public-ip`
- PHP info: `http://your-ec2-public-ip/info.php`
- Database test: `http://your-ec2-public-ip/db_test.php`

## Production Considerations

### 1. Security Hardening

```bash
# Remove PHP info file (security risk)
sudo rm /var/www/projectlamp/info.php

# Configure firewall
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 'Apache Full'

# Hide Apache version
echo "ServerTokens Prod" | sudo tee -a /etc/apache2/apache2.conf
echo "ServerSignature Off" | sudo tee -a /etc/apache2/apache2.conf
```

### 2. SSL Certificate (Let's Encrypt)

```bash
# Install Certbot
sudo apt install certbot python3-certbot-apache -y

# Get SSL certificate (replace with your domain)
sudo certbot --apache -d yourdomain.com -d www.yourdomain.com

# Test auto-renewal
sudo certbot renew --dry-run
```

### 3. Performance Optimization

```bash
# Enable Apache modules
sudo a2enmod expires
sudo a2enmod headers
sudo a2enmod deflate

# Restart Apache
sudo systemctl restart apache2
```

### 4. Monitoring and Logs

```bash
# Monitor Apache logs
sudo tail -f /var/log/apache2/projectlamp_access.log
sudo tail -f /var/log/apache2/projectlamp_error.log

# Monitor MySQL logs
sudo tail -f /var/log/mysql/error.log

# Check system resources
htop
df -h
free -h
```

### 5. Backup Strategy

```bash
# Database backup
mysqldump -u projectlamp_user -p projectlamp_db > backup_$(date +%Y%m%d).sql

# Files backup
tar -czf website_backup_$(date +%Y%m%d).tar.gz /var/www/projectlamp
```

## Troubleshooting Common Issues

### Apache Issues
```bash
# Check Apache status
sudo systemctl status apache2

# Test configuration
sudo apache2ctl configtest

# Check error logs
sudo tail -f /var/log/apache2/error.log
```

### MySQL Issues
```bash
# Check MySQL status
sudo systemctl status mysql

# Reset MySQL root password
sudo mysql_secure_installation

# Check MySQL logs
sudo tail -f /var/log/mysql/error.log
```

### PHP Issues
```bash
# Check PHP version
php -v

# Check loaded modules
php -m

# Test PHP configuration
php -i | grep -i error
```

### Permission Issues
```bash
# Fix ownership
sudo chown -R www-data:www-data /var/www/projectlamp

# Fix permissions
sudo chmod -R 755 /var/www/projectlamp
sudo chmod -R 644 /var/www/projectlamp/*.php
```

---

**🎉 Congratulations!** You now have a fully functional LAMP stack running on AWS EC2. Your application is ready for development and can be scaled for production use.

**Next Steps:**
- Build your web application
- Implement proper error handling
- Add user authentication
- Optimize for performance
- Set up monitoring and backups
