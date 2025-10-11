# MySQL Client-Server Setup Guide

## Objective

Create a basic client-server system using MySQL RDBMS where:
- One EC2 instance acts as **MySQL Server**
- Another EC2 instance acts as **MySQL Client**

## Step-by-Step Implementation

### Step 1: Provision Two EC2 Linux Instances

1. **Launch EC2 Instances**
   ```bash
   # Instance Names:
   - mysql-server
   - mysql-client
   ```

2. **Configuration Requirements**
   - **AMI**: Ubuntu Server 20.04 LTS
   - **Instance Type**: t2.micro (Free Tier eligible)
   - **VPC**: Same VPC for both instances
   - **Security Group**: Create or use existing group
   - **Key Pair**: Same key pair for both instances

### Step 2: Install MySQL Software

#### On mysql-server instance:

```bash
# Update package repository
sudo apt update

# Install MySQL Server
sudo apt install mysql-server -y

# Secure MySQL installation
sudo mysql_secure_installation
```

#### On mysql-client instance:

```bash
# Update package repository
sudo apt update

# Install MySQL Client
sudo apt install mysql-client -y
```

### Step 3: Configure MySQL Server for Remote Access

#### 3.1 Edit MySQL Configuration

```bash
# Open MySQL configuration file
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf

# Find and modify the bind-address line:
# FROM: bind-address = 127.0.0.1
# TO:   bind-address = 0.0.0.0
```

#### 3.2 Restart MySQL Service

```bash
# Restart MySQL to apply changes
sudo systemctl restart mysql

# Verify MySQL is running
sudo systemctl status mysql
```

#### 3.3 Configure Firewall

```bash
# Allow MySQL port through UFW firewall
sudo ufw allow 3306

# Check firewall status
sudo ufw status
```

### Step 4: Configure Security Groups

#### 4.1 Edit Inbound Rules for mysql-server

1. Go to **EC2 Dashboard** → **Security Groups**
2. Select the security group attached to `mysql-server`
3. Click **Edit Inbound Rules**
4. Add new rule:
   - **Type**: MySQL/Aurora
   - **Port**: 3306
   - **Source**: Custom IP (mysql-client's private IP)

```
Type: MySQL/Aurora
Port Range: 3306
Source: <mysql-client-private-ip>/32
Description: MySQL access from client
```

**Security Note**: Never use `0.0.0.0/0` for MySQL port. Always restrict to specific IPs.

### Step 5: Create MySQL User for Remote Access

#### 5.1 Connect to MySQL Server Locally

```bash
# Connect to MySQL as root
sudo mysql -u root -p
```

#### 5.2 Create Remote User

```sql
-- Create user for remote access
CREATE USER 'remote_user'@'<mysql-client-private-ip>' IDENTIFIED BY 'secure_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON *.* TO 'remote_user'@'<mysql-client-private-ip>';

-- Apply changes
FLUSH PRIVILEGES;

-- Exit MySQL
EXIT;
```

### Step 6: Test Remote Connection

#### 6.1 From mysql-client instance:

```bash
# Connect to remote MySQL server
mysql -h <mysql-server-private-ip> -u remote_user -p

# Expected output:
# Welcome to the MySQL monitor. Commands end with ; or \g.
# Your MySQL connection id is X
# Server version: 8.0.x-Ubuntu
# mysql>
```

#### 6.2 Test Database Operations

```sql
-- Show databases
SHOW DATABASES;

-- Create test database
CREATE DATABASE test_db;

-- Use the database
USE test_db;

-- Create test table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

-- Insert test data
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');

-- Query data
SELECT * FROM users;
```

## Troubleshooting

### Connection Issues

**Problem**: Connection timeout or refused
```bash
# Check MySQL service status
sudo systemctl status mysql

# Check if MySQL is listening on port 3306
sudo netstat -tlnp | grep 3306

# Check firewall rules
sudo ufw status
```

**Problem**: Access denied for user
```sql
-- Check user privileges
SELECT user, host FROM mysql.user WHERE user='remote_user';

-- Reset user password
ALTER USER 'remote_user'@'<client-ip>' IDENTIFIED BY 'new_password';
FLUSH PRIVILEGES;
```

### Network Issues

**Problem**: Security group not allowing connection
- Verify inbound rules include port 3306
- Ensure source IP matches client's private IP
- Check VPC and subnet configuration

## Verification Commands

### On MySQL Server:
```bash
# Check MySQL process
ps aux | grep mysql

# Check listening ports
sudo ss -tlnp | grep 3306

# Check MySQL error logs
sudo tail -f /var/log/mysql/error.log
```

### On MySQL Client:
```bash
# Test network connectivity
telnet <mysql-server-private-ip> 3306

# Test MySQL connection
mysql -h <mysql-server-private-ip> -u remote_user -p -e "SELECT VERSION();"
```

## Security Best Practices

1. **Network Security**
   - Use private IPs within VPC
   - Restrict security group rules to specific IPs
   - Never expose MySQL to internet (0.0.0.0/0)

2. **Database Security**
   - Use strong passwords
   - Create specific users with limited privileges
   - Regularly update MySQL
   - Enable SSL/TLS encryption

3. **System Security**
   - Keep OS updated
   - Configure proper firewall rules
   - Monitor access logs
   - Use key-based SSH authentication

## Success Criteria

- Both EC2 instances are running
- MySQL server installed and configured
- MySQL client can connect remotely
- Security groups properly configured
- Database operations work correctly
- Connection is secure and stable

---

**Next**: Review [Concepts](CONCEPTS.md) to understand the architecture theory.