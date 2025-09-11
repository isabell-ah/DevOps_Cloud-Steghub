# LEMP Stack Learning Notes

## What I Learned About LEMP Stack

**LEMP** = **L**inux + **E**ngine-X (Nginx) + **M**ySQL + **P**HP

LEMP is a modern web development stack that replaces Apache with Nginx for better performance and scalability.

## Core Components I Worked With

### 1. Linux (Operating System)
- **What I used**: Ubuntu 22.04 LTS on AWS EC2
- **What I learned**: Same foundation as LAMP but optimized for modern workloads
- **Why Linux**: Open source, stable, secure, and excellent container support

### 2. Nginx (Web Server) - The "E" in LEMP
- **Pronunciation**: "Engine-X" (hence the E in LEMP)
- **What it does**: High-performance web server and reverse proxy
- **Key advantages I discovered**:
  - Event-driven architecture (vs Apache's process-based)
  - Better handling of concurrent connections
  - Lower memory footprint
  - Superior static file serving
  - Built-in load balancing capabilities
- **Configuration location**: /etc/nginx/
- **My document root**: /var/www/projectLEMP

### 3. MySQL (Database)
- **Same as LAMP**: Relational database management system
- **What I used**: MySQL 8.0 for better performance and security
- **Key improvements**: Better JSON support, improved security, enhanced performance
- **Configuration**: /etc/mysql/
- **Data storage**: /var/lib/mysql/

### 4. PHP (Programming Language)
- **Key difference**: Uses PHP-FPM (FastCGI Process Manager) instead of mod_php
- **What PHP-FPM provides**:
  - Better process management
  - Improved performance and stability
  - Better resource isolation
  - Process pooling and recycling
- **Configuration**: /etc/php/8.1/fpm/

## How LEMP Stack Works (vs LAMP)

### LEMP Request Flow:
```
1. User requests webpage → Nginx Web Server
2. Nginx identifies PHP file → Forwards to PHP-FPM via FastCGI
3. PHP-FPM processes PHP code → May query MySQL Database
4. MySQL returns data → PHP-FPM processes results
5. PHP-FPM returns HTML → Nginx serves to user
6. User receives complete webpage
```

### Key Difference from LAMP:
- **LAMP**: Apache processes PHP directly with mod_php
- **LEMP**: Nginx forwards PHP requests to separate PHP-FPM process

## Nginx Server Blocks vs Apache Virtual Hosts

### Server Block Concept I Learned:
**Server Block** = Nginx equivalent of Apache Virtual Host

### Key Server Block Elements I Configured:
- **listen**: Port to listen on (80, 443)
- **server_name**: Domain names to respond to
- **root**: Website files location
- **index**: Default files to serve
- **location blocks**: URL pattern matching and handling
- **error_log/access_log**: Separate logs per site

### Location Block Patterns I Used:
```nginx
location / {                    # Root location
    try_files $uri $uri/ =404;
}

location ~ \.php$ {             # PHP files
    fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
}

location ~ /\.ht {              # Hidden files
    deny all;
}


```

## PHP-FPM Architecture I Learned

### Process Management Modes:
- **static**: Fixed number of child processes
- **dynamic**: Variable number based on demand
- **ondemand**: Processes created only when needed

### Configuration I Optimized:
```ini
pm = dynamic
pm.max_children = 50        # Maximum child processes
pm.start_servers = 5        # Initial processes
pm.min_spare_servers = 5    # Minimum idle processes
pm.max_spare_servers = 35   # Maximum idle processes
```

## Performance Advantages I Discovered

### Nginx vs Apache Performance:
- **Concurrent Connections**: Nginx handles 10,000+ vs Apache's ~400
- **Memory Usage**: Nginx uses ~2.5MB vs Apache's ~8MB per connection
- **Static Files**: Nginx serves static content 2-3x faster
- **CPU Usage**: Lower CPU overhead under high load

### Why This Matters:
- Better user experience under high traffic
- Lower hosting costs due to resource efficiency
- Better scalability for growing applications
- Improved mobile performance

## Security Features I Implemented

### Nginx Security Advantages:
- **Smaller Attack Surface**: Fewer modules loaded by default
- **Better DDoS Protection**: Built-in rate limiting
- **Security Headers**: Easy to implement security headers
- **SSL/TLS**: Excellent SSL performance and configuration


## Common Nginx Directives I Learned

### Essential Directives:
- **server**: Defines server block
- **listen**: Port and IP binding
- **server_name**: Domain matching
- **root**: Document root directory
- **index**: Default files
- **location**: URL pattern matching
- **try_files**: File lookup order
- **return**: Redirects and responses
- **rewrite**: URL rewriting
- **proxy_pass**: Reverse proxy configuration

## Directory Structure Logic I Applied

```
/etc/nginx/
├── nginx.conf              # Main configuration
├── sites-available/        # Available server blocks
│   └── projectLEMP       # My site configuration
├── sites-enabled/          # Enabled server blocks (symlinks)
│   └── projectLEMP -> ../sites-available/projectLEMP
└── conf.d/                # Additional configurations

/var/www/projectLEMP/       # My website root
├── index.php              # Main entry point
├── assets/                # Static resources
└── uploads/               # User uploads
```

