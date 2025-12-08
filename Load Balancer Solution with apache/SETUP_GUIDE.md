# Apache Load Balancer Setup Guide

Complete step-by-step guide to configure Apache as a load balancer on AWS EC2.

##  Table of Contents

1. [Prerequisites](#prerequisites)
2. [Infrastructure Setup](#infrastructure-setup)
3. [Apache Installation](#apache-installation)
4. [Load Balancer Configuration](#load-balancer-configuration)
5. [Backend Server Setup](#backend-server-setup)
6. [Testing and Validation](#testing-and-validation)
7. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### AWS Resources Required
- AWS account with EC2 permissions
- VPC with public and private subnets
- Security groups configured appropriately
- SSH key pair for EC2 access

### Knowledge Requirements
- Basic Linux command line
- Understanding of HTTP and networking
- Familiarity with AWS EC2 and VPC concepts

---

## Infrastructure Setup

### Step 1: Launch Load Balancer EC2 Instance

1. **Navigate to EC2 Dashboard** in AWS Console

2. **Launch Instance** with the following specifications:
   - **Name**: project-8-apache-lb
   - **AMI**: Ubuntu Server 22.04 LTS or similar
   - **Instance Type**: t3.micro
   - **VPC**: training-vpc (vpc-0753e631ccbdee65e)
   - **Subnet**: public-subnet-2 (subnet-059ceac7c777946df)
   - **Auto-assign Public IP**: Enable
   - **Security Group**: Create or select with rules below

3. **Security Group Configuration** for Load Balancer:
   ```
   Inbound Rules:
   - Type: HTTP, Port: 80, Source: 0.0.0.0/0 (or your IP range)
   - Type: SSH, Port: 22, Source: Your IP
   
   Outbound Rules:
   - Type: All traffic, Destination: 0.0.0.0/0
   ```

4. **Launch** the instance and note the following:
   - Instance ID: i-05dd8decd3b71b02e
   - Private IP: 12.0.1.210
   - Public IP: 52.3.239.93

### Step 2: Launch Backend Web Server Instances

1. **Launch two EC2 instances** for backend servers:
   - **Names**: web-server-1, web-server-2
   - **AMI**: Ubuntu Server 22.04 LTS
   - **Instance Type**: t3.micro
   - **VPC**: training-vpc
   - **Subnet**: private-subnet (or public if testing)
   - **Private IPs**: 12.0.3.101, 12.0.3.102

2. **Security Group Configuration** for Backend Servers:
   ```
   Inbound Rules:
   - Type: HTTP, Port: 80, Source: Load Balancer Security Group
   - Type: SSH, Port: 22, Source: Your IP (for management)
   
   Outbound Rules:
   - Type: All traffic, Destination: 0.0.0.0/0
   ```

---

## Apache Installation

### On Load Balancer Instance

1. **Connect to the load balancer instance**:
   ```bash
   ssh -i your-key.pem ubuntu@52.3.239.93
   ```

2. **Update system packages**:
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

3. **Install Apache**:
   ```bash
   sudo apt install apache2 -y
   ```

4. **Enable required Apache modules**:
   ```bash
   sudo a2enmod proxy
   sudo a2enmod proxy_http
   sudo a2enmod proxy_balancer
   sudo a2enmod lbmethod_bytraffic
   sudo a2enmod headers
   ```

5. **Verify Apache is running**:
   ```bash
   sudo systemctl status apache2
   ```

---

## Load Balancer Configuration

### Step 1: Create Apache Configuration File

1. **Create a new virtual host configuration**:
   ```bash
   sudo nano /etc/apache2/sites-available/load-balancer.conf
   ```

2. **Add the following configuration**:
   ```apache
   <VirtualHost *:80>
       ServerAdmin admin@example.com
       ServerName 52.3.239.93
       
       # Load Balancer Configuration
       <Proxy "balancer://mycluster">
           BalancerMember http://12.0.3.101:80 loadfactor=5 timeout=1
           BalancerMember http://12.0.3.102:80 loadfactor=5 timeout=1
           ProxySet lbmethod=bytraffic
       </Proxy>
       
       # Preserve host header
       ProxyPreserveHost On
       
       # Forward all requests to the cluster
       ProxyPass / balancer://mycluster/
       ProxyPassReverse / balancer://mycluster/
       
       # Logging
       ErrorLog ${APACHE_LOG_DIR}/lb_error.log
       CustomLog ${APACHE_LOG_DIR}/lb_access.log combined
   </VirtualHost>
   ```

3. **Save and exit** (Ctrl+X, then Y, then Enter)

### Step 2: Configuration Explanation

- **BalancerMember**: Defines each backend server
  - `loadfactor=5`: Weight for traffic distribution (higher = more traffic)
  - `timeout=1`: Connection timeout in seconds

- **ProxySet lbmethod=bytraffic**: Load balancing algorithm
  - Distributes requests based on traffic volume (bytes transferred)
  - Alternative methods: `byrequests`, `bybusyness`, `heartbeat`

- **ProxyPreserveHost On**: Forwards original Host header to backend servers

- **ProxyPass**: Maps incoming requests to the backend cluster

- **ProxyPassReverse**: Rewrites response headers from backend servers

### Step 3: Enable Configuration

1. **Disable default site**:
   ```bash
   sudo a2dissite 000-default.conf
   ```

2. **Enable load balancer site**:
   ```bash
   sudo a2ensite load-balancer.conf
   ```

3. **Test configuration syntax**:
   ```bash
   sudo apache2ctl configtest
   ```
   Expected output: `Syntax OK`

4. **Restart Apache**:
   ```bash
   sudo systemctl restart apache2
   ```

5. **Verify Apache is running**:
   ```bash
   sudo systemctl status apache2
   ```

---

## Backend Server Setup

### On Each Backend Server (12.0.3.101 and 12.0.3.102)

1. **Connect to backend server**:
   ```bash
   ssh -i your-key.pem ubuntu@<backend-server-ip>
   ```

2. **Install Apache**:
   ```bash
   sudo apt update
   sudo apt install apache2 -y
   ```

3. **Create a unique test page** (to identify which server responds):
   
   **On Server 1 (12.0.3.101)**:
   ```bash
   echo "<h1>Backend Server 1 - IP: 12.0.3.101</h1>" | sudo tee /var/www/html/index.html
   ```
   
   **On Server 2 (12.0.3.102)**:
   ```bash
   echo "<h1>Backend Server 2 - IP: 12.0.3.102</h1>" | sudo tee /var/www/html/index.html
   ```

4. **Ensure Apache is running**:
   ```bash
   sudo systemctl enable apache2
   sudo systemctl start apache2
   ```

5. **Test locally**:
   ```bash
   curl http://localhost
   ```

---

## Testing and Validation

### Test 1: Direct Backend Access (from Load Balancer)

```bash
# From load balancer instance
curl http://12.0.3.101
curl http://12.0.3.102
```

Expected: Each should return its unique test page.

### Test 2: Load Balancer Access (from Internet)

```bash
# From your local machine
curl http://52.3.239.93
```

Expected: Response from one of the backend servers.

### Test 3: Load Distribution

```bash
# Run multiple requests
for i in {1..10}; do
    curl http://52.3.239.93
    echo "---"
done
```

Expected: Responses alternating between backend servers.

### Test 4: DNS Resolution

```bash
nslookup 52.3.239.93
```

Expected: Resolves to ec2-52-3-239-93.compute-1.amazonaws.com

### Test 5: Apache Logs

```bash
# On load balancer
sudo tail -f /var/log/apache2/lb_access.log
```

Make requests and observe traffic being logged.

### Test 6: Balancer Manager (Optional)

1. **Enable balancer-manager**:
   ```bash
   sudo nano /etc/apache2/sites-available/load-balancer.conf
   ```

2. **Add inside VirtualHost**:
   ```apache
   <Location "/balancer-manager">
       SetHandler balancer-manager
       Require ip 12.0.1.210
   </Location>
   ```

3. **Restart Apache**:
   ```bash
   sudo systemctl restart apache2
   ```

4. **Access**: http://52.3.239.93/balancer-manager (configure security appropriately)

---

## Troubleshooting

### Issue: Apache fails to start

**Solution**:
```bash
# Check syntax
sudo apache2ctl configtest

# Check error logs
sudo tail -50 /var/log/apache2/error.log

# Verify modules are enabled
sudo apache2ctl -M | grep proxy
```

### Issue: Cannot reach backend servers

**Solution**:
```bash
# Test connectivity from load balancer
ping 12.0.3.101
telnet 12.0.3.101 80

# Check security group rules
# Ensure backend security group allows traffic from load balancer
```

### Issue: All traffic goes to one server

**Solution**:
- Verify both backend servers are running
- Check load balancing method configuration
- Review Apache logs for errors
- Ensure loadfactor is set correctly

### Issue: 503 Service Unavailable

**Solution**:
```bash
# Check if backend servers are responding
curl http://12.0.3.101
curl http://12.0.3.102

# Increase timeout in configuration
# BalancerMember http://12.0.3.101:80 loadfactor=5 timeout=5

# Check Apache error logs
sudo tail -f /var/log/apache2/lb_error.log
```

### Issue: Host header not preserved

**Solution**:
- Ensure `ProxyPreserveHost On` is in configuration
- Restart Apache after changes

---

## Advanced Configuration

### Health Checks

Add health check parameters to BalancerMember:

```apache
BalancerMember http://12.0.3.101:80 loadfactor=5 timeout=1 retry=60 status=+H
```

### Session Persistence (Sticky Sessions)

```apache
<Proxy "balancer://mycluster">
    BalancerMember http://12.0.3.101:80 route=server1
    BalancerMember http://12.0.3.102:80 route=server2
    ProxySet lbmethod=bytraffic
    ProxySet stickysession=ROUTEID
</Proxy>
```

### SSL/TLS Configuration

```bash
# Enable SSL module
sudo a2enmod ssl

# Create or obtain SSL certificate
sudo certbot --apache -d yourdomain.com
```

---

## Validation Checklist

- [ ] Load balancer instance is running and accessible
- [ ] Apache service is active on load balancer
- [ ] All required Apache modules are enabled
- [ ] Backend servers are running and accessible from load balancer
- [ ] Security groups allow proper traffic flow
- [ ] Load balancer responds to HTTP requests on public IP
- [ ] Traffic is distributed across backend servers
- [ ] Apache logs show successful proxying
- [ ] No errors in Apache error logs
- [ ] Host headers are preserved (if required by application)

---

## Monitoring and Maintenance

### Regular Checks

```bash
# Apache status
sudo systemctl status apache2

# Check listening ports
sudo netstat -tlnp | grep apache2

# Monitor logs
sudo tail -f /var/log/apache2/lb_access.log

# Check disk space
df -h
```

### Performance Tuning

Edit `/etc/apache2/apache2.conf`:

```apache
# Increase max clients
<IfModule mpm_prefork_module>
    StartServers 5
    MinSpareServers 5
    MaxSpareServers 10
    MaxRequestWorkers 150
    MaxConnectionsPerChild 0
</IfModule>
```

---

## Scaling Considerations

### Horizontal Scaling
- Add more backend servers by adding BalancerMember entries
- Adjust loadfactor based on server capacity
- Consider AWS Auto Scaling Groups for dynamic scaling

### Vertical Scaling
- Upgrade instance types (t3.micro → t3.small → t3.medium)
- Increase Apache worker processes
- Optimize Apache configuration for higher throughput

---

## Conclusion

You now have a fully functional Apache load balancer distributing traffic across multiple backend servers. This setup provides:

- High availability through redundancy
- Horizontal scalability
- Traffic distribution for improved performance
- Foundation for further enhancements (SSL, health checks, auto-scaling)


