# LAMP Stack Learning Notes

## What I Learned About LAMP Stack

**LAMP** = **L**inux + **A**pache + **M**ySQL + **P**HP

Through this project, I discovered that LAMP is a complete web development stack for building dynamic websites and applications.

## Core Components I Worked With

### 1. Linux (Operating System)
- **What I used**: Ubuntu 20.04 LTS on AWS EC2
- **What I learned**: Linux serves as the foundation OS that runs all other components
- **Why Linux**: I found it to be open source, stable, secure, and widely supported

### 2. Apache (Web Server)
- **What it does**: Serves web pages to users' browsers
- **Key features I implemented**:
  - Handles HTTP requests from browsers
  - Serves static files (HTML, CSS, JS, images)
  - Processes PHP scripts
  - Virtual hosts for multiple websites
- **Configuration location**: /etc/apache2/
- **My document root**: /var/www/projectlamp (custom setup)

### 3. MySQL (Database)
- **Purpose**: Stores and manages my application data
- **Key features I used**:
  - Relational database management system
  - SQL queries for data operations
  - User authentication and permissions
  - ACID compliance for data integrity
- **Configuration**: /etc/mysql/
- **Data storage**: /var/lib/mysql/

### 4. PHP (Programming Language)
- **Purpose**: Server-side scripting for dynamic content
- **What I implemented**:
  - Processes business logic
  - Connects to MySQL database
  - Generates HTML dynamically
  - Handles user input and sessions
- **Configuration**: /etc/php/8.1/apache2/php.ini

## How I Understand LAMP Stack Works

```
1. User requests my webpage → Apache Web Server
2. Apache identifies PHP file → PHP Processor
3. PHP executes my code → May query MySQL Database
4. MySQL returns data → PHP processes results
5. PHP generates HTML → Apache serves to user
6. User receives complete webpage
```

## Virtual Hosts Concept I Learned

**Virtual Host** = Multiple websites on one server

### Why I Used Virtual Hosts:
- Host multiple domains on single server
- Separate configurations per site
- Better organization and security
- Cost-effective hosting solution

### Key Virtual Host Elements I Configured:
- **ServerName**: Domain name (projectlamp)
- **DocumentRoot**: Website files location (/var/www/projectlamp)
- **Directory**: Access permissions and settings
- **Logs**: Separate error and access logs per site

## Directory Structure Logic I Applied

```
/var/www/projectlamp/          # My website root
├── index.php                  # Main entry point
├── config/                    # Configuration files
│   └── database.php          # DB connection settings
├── includes/                  # Reusable PHP functions
│   └── functions.php         # Common functions
├── assets/                   # Static resources
│   ├── css/                  # Stylesheets
│   ├── js/                   # JavaScript files
│   └── images/               # Image files
└── uploads/                  # User uploaded files
```

## Security Concepts I Implemented

### File Permissions I Set
- **755 for directories**: Owner can read/write/execute, others can read/execute
- **644 for files**: Owner can read/write, others can read only
- **www-data**: Apache user that needs access to my web files

### Database Security I Applied
- **Separate user**: Did not use root for my application
- **Limited privileges**: Granted only necessary permissions
- **Strong passwords**: Used complex passwords
- **Prepared statements**: Prevent SQL injection

## Common Issues I Encountered and Solved

### 1. 403 Forbidden Error
**What happened**: Wrong file permissions
**How I fixed it**: `sudo chmod -R 755 /var/www/projectlamp`

### 2. PHP Not Executing
**What happened**: PHP module not enabled
**How I fixed it**: `sudo a2enmod php8.1`

### 3. Database Connection Failed
**What happened**: Wrong credentials or user permissions
**How I fixed it**: Checked MySQL user privileges

### 4. Virtual Host Not Working
**What happened**: Site not enabled or conflicting configurations
**How I fixed it**: `sudo a2ensite projectlamp.conf`

## Best Practices I Learned

### Development Practices
- Use version control (Git)
- Separate development/production environments
- Regular backups of database and files
- Monitor error logs regularly

### Security Practices I Applied
- Keep software updated
- Use strong passwords
- Implement input validation
- Enable firewall (UFW)
- Use HTTPS in production

### Performance Optimization I Discovered
- Enable Apache compression (mod_deflate)
- Optimize MySQL queries
- Use caching mechanisms
- Monitor resource usage

## Key Commands I Memorized

```bash
# Service management
sudo systemctl restart apache2
sudo systemctl restart mysql

# File permissions
sudo chown -R www-data:www-data /var/www/projectlamp
sudo chmod -R 755 /var/www/projectlamp

# Apache configuration
sudo a2ensite projectlamp.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest

# MySQL access
mysql -u projectlamp_user -p projectlamp_db

# View logs
sudo tail -f /var/log/apache2/error.log
sudo tail -f /var/log/mysql/error.log
```

## Why I Chose This Stack

### Advantages I Discovered
- **Open Source**: No licensing costs
- **Mature**: Well-established and documented
- **Flexible**: Highly customizable
- **Community**: Large support community
- **Scalable**: Can handle small to large applications

### Use Cases I Can Apply This To
- Content Management Systems (WordPress, Drupal)
- E-commerce websites
- Web applications
- APIs and web services
- Development and testing environments

## My Learning Journey

1. **Understanding each component individually**
2. **Learning how components interact**
3. **Practicing with simple PHP applications**
4. **Implementing database operations**
5. **Adding security measures**
6. **Optimizing for performance**
7. **Deploying to production environment**

## Skills I Developed

- Linux system administration
- Web server configuration
- Database design and management
- Server-side programming with PHP
- Cloud infrastructure management (AWS EC2)
- Security implementation
- Troubleshooting and problem-solving
- Documentation and project management