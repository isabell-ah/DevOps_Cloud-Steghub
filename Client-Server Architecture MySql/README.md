# MySQL Client-Server Architecture on AWS EC2


## Overview

This project demonstrates the implementation of a **Client-Server Architecture** using MySQL as the Database Management System (DBMS), hosted on two separate Linux-based EC2 instances on AWS.

### Architecture Components
- **MySQL Server**: EC2 instance running MySQL database server
- **MySQL Client**: EC2 instance with MySQL client tools
- **Network**: Secure VPC communication over port 3306

## Quick Start

### Prerequisites
- AWS Account with EC2 access
- Basic knowledge of Linux commands
- Understanding of networking concepts

### Setup Summary
1. Launch two EC2 instances (`mysql-server` and `mysql-client`)
2. Install MySQL server and client respectively
3. Configure security groups for port 3306
4. Establish remote connection

## Project Structure

```
Client-Server Architecture MySql/
├── README.md              # This file
├── SETUP-GUIDE.md         # Detailed setup instructions
├── CONCEPTS.md            # Architecture concepts and theory
```

## Learning Objectives

- Understand client-server communication model
- Configure MySQL for remote access
- Implement secure database connections
- Practice AWS EC2 and VPC networking
- Learn database security best practices

## Technologies Used

- **Database**: MySQL 8.0
- **Cloud Platform**: AWS EC2
- **Operating System**: Ubuntu Linux
- **Networking**: VPC, Security Groups
- **Protocol**: TCP/IP over port 3306

## Documentation

- **[Setup Guide](SETUP-GUIDE.md)** - Complete step-by-step implementation
- **[Concepts](CONCEPTS.md)** - Technical architecture and theory

## Security Features

- Private IP communication within VPC
- Restricted security group rules
- MySQL user authentication
- Firewall configuration with UFW

##  Testing

Verify the setup by connecting from client to server:

```bash
mysql -h <mysql-server-private-ip> -u root -p
```



**Next Steps**: Follow the [Setup Guide](SETUP-GUIDE.md) for detailed implementation instructions.