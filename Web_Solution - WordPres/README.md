#  Project 6: WordPress Web Solution on AWS (3-Tier Architecture)

## Overview
This project demonstrates the deployment of a scalable WordPress web solution using AWS EC2 instances, EBS volumes, and LVM-backed storage. It follows a classic **3-tier architecture**:

- **Client Tier**: Browser access from a laptop or PC
- **Web Tier**: EC2 instance (RedHat) running Apache, PHP, and WordPress
- **Database Tier**: EC2 instance (RedHat) running MySQL

The goal is to gain hands-on experience with Linux storage provisioning, logical volume management, and remote database integration.

---


- **Client Tier**: Browser access from a laptop or PC
- **Web Tier**: EC2 (RedHat) running Apache, PHP, and WordPress
- **Database Tier**: EC2 (RedHat) running MySQL

---

Client (Browser) ↓ Web Server (EC2 - RedHat) ↓ Database Server (EC2 - RedHat)


## Infrastructure Setup

### 1️ Launch EC2 Instances
- Create two EC2 instances in the same Availability Zone:
  - `Web Server`: RedHat OS
  - `DB Server`: RedHat OS
- SSH access:
  ```bash
  ssh ec2-user@<public-ip>

### Storage Configuration
- Created and attached **3×10 GiB EBS volumes** to Web Server
- Partitioned using `gdisk`, formatted with `mkfs.ext4`
- Configured LVM:
  - Volume Group: `webdata-vg`
  - Logical Volumes: `apps-lv`, `logs-lv`
- Mounted `/u01` and `/var/log`, updated `/etc/fstab` for persistence

---

##  Web Server Setup

```bash
sudo yum -y update
sudo yum -y install httpd php php-mysqlnd php-fpm php-json php-mysqli
sudo systemctl start httpd
sudo systemctl enable httpd
```


- Downloaded and extracted WordPress
- Configured SELinux and permissions
- Updated wp-config.php with remote DB credentials

## Database Server Setup
```bash
sudo yum install mysql-server -y
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```

- Created database and user:
```bash
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'%' IDENTIFIED BY 'yourpassword';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'%';
FLUSH PRIVILEGES;
```


- Opened port 3306 for Web Server IP only


## Validation
- Verified remote DB connectivity via mysql -h <DB_IP> -u wordpressuser -p
- Confirmed WordPress loads at:
http://<WEB_SERVER_IP>/wordpress/
- Mounted volumes persist across reboot (df -h confirmed)

## Notes
This project reinforces:
- Linux disk and volume provisioning
- LVM configuration and persistence
- Web/database tier separation
- Remote service integration and security group best practices

## Author
Sharon Isabela

---


