# Apache Load Balancer on AWS EC2

A production-ready Apache HTTP Server configured as a load balancer using `mod_proxy_balancer` to distribute traffic across multiple backend web servers on AWS EC2.

## Overview

This project implements horizontal scaling using Apache as a reverse proxy load balancer, distributing incoming HTTP traffic across multiple backend servers to improve availability, reliability, and performance.

##  Features

- **Traffic-based load distribution** using Apache's `bytraffic` algorithm
- **Horizontal scaling** across multiple EC2 backend instances
- **Host header preservation** for proper request routing
- **Configurable load factors** and timeouts per backend member
- **High availability** architecture within AWS VPC

## 🏗️ Architecture

### Load Balancer Instance
- **Name**: project-8-apache-lb
- **Instance ID**: i-05dd8decd3b71b02e
- **Instance Type**: t3.micro
- **Private IP**: 12.0.1.210
- **Public IP**: 52.3.239.93
- **Public DNS**: ec2-52-3-239-93.compute-1.amazonaws.com
- **Hostname**: ip-12-0-1-210.ec2.internal
- **VPC**: training-vpc (vpc-0753e631ccbdee65e)
- **Subnet**: public-subnet-2 (subnet-059ceac7c777946df)

### Backend Web Servers
- **Server 1**: 12.0.3.101:80
- **Server 2**: 12.0.3.102:80

### Network Flow
```
Internet → Public IP (52.3.239.93) → Load Balancer (12.0.1.210) → Backend Servers (12.0.3.101, 12.0.3.102)
```

##  Prerequisites

- AWS account with EC2 access
- Ubuntu/Debian-based EC2 instances
- Apache 2.4+ installed
- Security groups configured to allow:
  - Inbound HTTP (port 80) to load balancer
  - Outbound HTTP from load balancer to backend servers
  - Inbound HTTP on backend servers from load balancer

##  Quick Start

1. **Clone or reference this documentation**
2. **Follow the [SETUP_GUIDE.md](SETUP_GUIDE.md)** for detailed installation steps
3. **Configure Apache** with the provided configuration
4. **Validate** the setup using the checklist

## Project Structure

```
Load Balancer Solution with apache/
├── README.md                    # This file
├── SETUP_GUIDE.md              # Detailed setup instructions
└── images/                     # Architecture and configuration screenshots
    ├── ec2-instance-summary.png
    ├── apache-loadbalancer-config.png
    └── loadbalancer-concept.png
```


## ⚙️ Configuration Highlights

- **Load balancing method**: `bytraffic` (distributes based on traffic volume)
- **Load factor**: 5 for each backend (equal weight)
- **Timeout**: 1 second per backend connection
- **Host preservation**: Enabled via `ProxyPreserveHost On`

## ✅ Validation Checklist

- [x] Apache service is running on load balancer
- [x] Load balancer reachable via public IP (52.3.239.93)
- [x] Load balancer reachable via private IP (12.0.1.210)
- [x] Backend servers respond to HTTP requests
- [x] Traffic is distributed across backend members
- [x] Host headers are preserved in forwarded requests

##  Maintenance

### Check Apache Status
```bash
sudo systemctl status apache2
```

### Restart Apache
```bash
sudo systemctl restart apache2
```

### View Apache Logs
```bash
sudo tail -f /var/log/apache2/access.log
sudo tail -f /var/log/apache2/error.log
```

### Test Load Distribution
```bash
# Multiple requests to observe distribution
for i in {1..10}; do curl http://52.3.239.93; done
```

## Security Considerations

- Ensure security groups restrict backend server access to load balancer only
- Consider implementing SSL/TLS termination at the load balancer
- Regularly update Apache and apply security patches
- Use AWS Security Groups and NACLs for network-level protection

## Next Steps

- [ ] Implement health checks for backend servers
- [ ] Add SSL/TLS support with Let's Encrypt
- [ ] Configure Apache mod_status for monitoring
- [ ] Set up CloudWatch monitoring and alarms
- [ ] Implement auto-scaling for backend servers
- [ ] Add session persistence (sticky sessions) if needed

##  Additional Resources

- [Apache mod_proxy_balancer Documentation](https://httpd.apache.org/docs/2.4/mod/mod_proxy_balancer.html)

- [Apache Load Balancing Guide](https://httpd.apache.org/docs/2.4/howto/reverse_proxy.html)



**Authored By**:Sharon Isabela
