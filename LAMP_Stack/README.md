# ProjectLAMP - My LAMP Stack Implementation

This project demonstrates my implementation of a LAMP (Linux, Apache, MySQL, PHP) stack on AWS EC2. Through this project, I learned how to deploy and configure a complete web development environment.

## What I Implemented

### 1. AWS EC2 Instance Setup
I launched an Ubuntu 20.04 LTS instance on AWS EC2 and configured the security groups to allow:
- SSH access (port 22) for server management
- HTTP traffic (port 80) for web access
- MySQL access (port 3306) for database connections

```bash
ssh -i my-key.pem ubuntu@my-ec2-ip
```

### 2. LAMP Stack Installation
I installed each component of the LAMP stack:

**Linux**: Used Ubuntu 20.04 LTS as the operating system
**Apache**: Installed and configured the web server
**MySQL**: Set up the database server with security configurations
**PHP**: Installed PHP with necessary modules for database connectivity

```bash
# System update
sudo apt update && sudo apt upgrade -y

# Apache installation
sudo apt install apache2 -y
sudo systemctl enable apache2

# MySQL installation
sudo apt install mysql-server -y
sudo mysql_secure_installation

# PHP installation
sudo apt install php libapache2-mod-php php-mysql -y
```

### 3. Apache Virtual Host Configuration
I created a virtual host to serve my web application from a custom directory:

```bash
# Created project directory
sudo mkdir /var/www/projectlamp
sudo chown -R $USER:$USER /var/www/projectlamp

# Created virtual host configuration
sudo nano /etc/apache2/sites-available/projectlamp.conf
```

```apache
<VirtualHost *:80>
    ServerName projectlamp
    DocumentRoot /var/www/projectlamp
    <Directory /var/www/projectlamp>
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/projectlamp_error.log
    CustomLog ${APACHE_LOG_DIR}/projectlamp_access.log combined
</VirtualHost>
```

```bash
# Enabled my site and disabled default
sudo a2ensite projectlamp.conf
sudo a2dissite 000-default.conf
sudo a2enmod rewrite
sudo systemctl reload apache2
```

### 4. Database Configuration
I created a dedicated database and user for my application:

```bash
mysql -u root -p
```

```sql
CREATE DATABASE projectlamp_db;
CREATE USER 'projectlamp_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON projectlamp_db.* TO 'projectlamp_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 5. Testing and Validation
I created a test PHP file to verify all components work together:

```bash
# Created test file
nano /var/www/projectlamp/index.php
```

```php
<?php
$conn = new mysqli("localhost", "projectlamp_user", "secure_password", "projectlamp_db");
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "<h1>My ProjectLAMP is Working!</h1>";
echo "<p>PHP Version: " . phpversion() . "</p>";
echo "<p>Database: Connected successfully</p>";
?>
```

## Accessing My Application
I can access my LAMP stack application at:
```
http://my-ec2-public-ip
```

## Key Commands I Used
```bash
# Service management
sudo systemctl restart apache2
sudo systemctl restart mysql

# Log monitoring
sudo tail -f /var/log/apache2/projectlamp_error.log

# File permissions
sudo chown -R www-data:www-data /var/www/projectlamp
sudo chmod -R 755 /var/www/projectlamp
```

## Project Structure I Created
```
/var/www/projectlamp/
├── index.php          # Main application file
├── config/            # Configuration files
│   └── database.php   # Database connection settings
└── assets/            # Static resources
    ├── css/           # Stylesheets
    └── js/            # JavaScript files
```

## What I Learned
- How to set up and configure a complete web server environment
- Apache virtual host configuration for hosting multiple websites
- MySQL database creation and user management
- PHP integration with Apache and MySQL
- AWS EC2 instance management and security group configuration
- Linux system administration and file permissions
- Web application deployment and testing procedures