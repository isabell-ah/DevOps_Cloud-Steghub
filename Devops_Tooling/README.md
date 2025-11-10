# DevOps Tooling Website Solution

## Table of Contents
- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Key Concepts and Learning Outcomes](#key-concepts-and-learning-outcomes)
- [Technologies Used](#technologies-used)
- [Project Screenshots](#project-screenshots)
- [Implementation](#implementation)
- [Conclusion](#conclusion)

## Project Overview

This project implements a 3-tier DevOps Tooling Website Solution that provides centralized access to essential DevOps tools for team collaboration. The solution demonstrates a scalable, stateless web architecture using shared storage (NFS) and a centralized database (MySQL).

### Business Problem

DevOps teams need unified access to multiple tools including Jenkins, Kubernetes, JFrog Artifactory, Rancher, Grafana, Prometheus, and Kibana without managing separate authentication systems and configurations across different servers.

### Solution

A web-based portal that:
- Provides single sign-on access to DevOps tools
- Uses stateless web servers for horizontal scalability
- Implements shared file storage via Network File System (NFS)
- Centralizes data in a MySQL database
- Enables easy addition or removal of web servers without data loss

## Architecture

### High-Level Architecture Diagram

```
                    ┌─────────────┐
                    │   Client    │
                    │  (Browser)  │
                    └──────┬──────┘
                           │
                           ▼
            ┌──────────────────────────────┐
            │    Load Balancer (Optional)  │
            └──────────────┬───────────────┘
                           │
                           ▼
            ┌──────────────────────────────┐
            │      Web Server Tier         │
            │  ┌────────┐ ┌────────┐ ┌────┐│
            │  │ Web 1  │ │ Web 2  │ │Web3││
            │  │ RHEL 8 │ │ RHEL 8 │ │RHEL││
            │  │ Apache │ │ Apache │ │8   ││
            │  │  PHP   │ │  PHP   │ │PHP ││
            │  └───┬────┘ └───┬────┘ └─┬──┘│
            └──────┼──────────┼────────┼───┘
                   │          │        │
                   └──────────┼────────┘
                              │
                   ┌──────────┴──────────┐
                   │                     │
                   ▼                     ▼
            ┌─────────────┐      ┌─────────────┐
            │ NFS Server  │      │  DB Server  │
            │   RHEL 8    │      │ Ubuntu 24.04│
            │             │      │   MySQL     │
            │ /mnt/apps   │      │             │
            │ /mnt/logs   │      │ tooling DB  │
            │ /mnt/opt    │      │             │
            └─────────────┘      └─────────────┘
```

### Architecture Components

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Web Servers | RHEL 8 + Apache + PHP | Serve web application (stateless) |
| Database Server | Ubuntu 24.04 + MySQL | Store application data |
| Storage Server | RHEL 8 + NFS | Shared file storage for web servers |
| Infrastructure | AWS EC2 | Cloud hosting platform |

## Key Concepts and Learning Outcomes

### 1. Storage Solutions

#### Network File System (NFS)

NFS allows multiple servers to access the same files over a network as if they were local. In this project, NFS is used to share web application files across multiple web servers, enabling centralized file management where updates made once are reflected everywhere.

#### Storage Types Comparison

**Block Storage**
- Description: Raw storage volumes attached directly to servers
- Use Case: Databases, virtual machines
- Examples: AWS EBS, Azure Disk Storage
- Characteristics: High performance, low latency, requires formatting

**File Storage**
- Description: Hierarchical file system shared over network
- Use Case: Shared files, content management systems
- Examples: NFS, SMB/CIFS, AWS EFS
- Characteristics: Easy to share, familiar interface, network dependent

**Object Storage**
- Description: Flat namespace with metadata
- Use Case: Backups, media files, static content
- Examples: AWS S3, Azure Blob Storage
- Characteristics: Highly scalable, cost-effective, HTTP access

#### Storage Protocols

**NFS (Network File System):** Unix/Linux file sharing protocol used in this project for sharing application files across web servers.

**SMB/CIFS:** Windows file sharing protocol, commonly used in Windows environments.

**iSCSI:** Block-level storage protocol over IP networks, provides SAN-like functionality over standard networks.

**FTP/SFTP:** File transfer protocols for moving files between systems.

### 2. Three-Tier Architecture

**Tier 1: Presentation Layer (Web Servers)**
- Handles user requests and responses
- Serves HTML, CSS, and JavaScript to browsers
- Stateless design allows horizontal scaling
- Multiple servers provide redundancy and load distribution

**Tier 2: Application Layer (PHP Application)**
- Contains business logic
- Processes user requests and data
- Communicates with the database
- Runs on the web servers alongside Apache

**Tier 3: Data Layer (MySQL Database)**
- Stores persistent data
- Manages user information and application state
- Handles data integrity and transactions
- Centralized data storage for all web servers

### 3. Stateless vs Stateful Architecture

**Stateless Web Servers (This Project)**

In a stateless architecture, web servers do not store session data locally. All state information is stored either in the database or in shared storage (NFS). This approach provides several benefits:

- No session data stored locally on web servers
- Easy to scale horizontally by adding more servers
- Can add or remove servers without data loss
- All state stored in database or shared storage
- Any web server can handle any request
- Server failure does not cause data loss
- Simplified load balancing and maintenance

**Why This Matters**

Stateless design is crucial for modern web applications because it enables true horizontal scalability. When a user makes a request, it can be handled by any available web server since no server holds unique session data. This architecture is the foundation for cloud-native applications and microservices.

### 4. Logical Volume Management (LVM)

LVM provides flexible disk management by separating physical storage from logical organization. It allows administrators to resize volumes without downtime and manage storage more efficiently.

**LVM Hierarchy:**
```
Physical Volumes (PV) → Volume Groups (VG) → Logical Volumes (LV) → File Systems
```

**In This Project:**
- Created three logical volumes: lv-apps, lv-logs, lv-opt
- Formatted with XFS filesystem (better performance for NFS than ext4)
- Mounted to /mnt/apps, /mnt/logs, /mnt/opt respectively

**Benefits of LVM:**
- Flexible disk management without hardware constraints
- Can resize volumes without downtime
- Snapshot capability for backups
- Ability to span volumes across multiple disks

### 5. File System Types

**XFS**
- Characteristics: High performance, excellent for large files, better metadata performance
- When to Use: NFS exports, large file storage, high-concurrency environments
- Used in this project for NFS shared storage

**ext4**
- Characteristics: General purpose, widely compatible, stable and mature
- When to Use: General Linux use, boot partitions, standard file storage

**NTFS**
- Characteristics: Windows native file system
- When to Use: Windows systems, dual-boot scenarios

**Why XFS for NFS?**
- Better performance with concurrent access from multiple clients
- Efficient handling of large files
- Superior metadata performance
- Better suited for network file sharing workloads

### 6. Linux Permissions and Ownership

**Commands Used in This Project:**
```bash
sudo chown -R nobody: /mnt/apps    # Change ownership to 'nobody' user
sudo chmod -R 777 /mnt/apps        # Give read, write, execute to all
```

**Permission Breakdown:**
- 777 = rwxrwxrwx (Owner, Group, Others all have full access)
- nobody user = Generic user for NFS to prevent privilege escalation
- r (read) = 4, w (write) = 2, x (execute) = 1

**Security Note:** 777 permissions are used here for simplicity in a learning environment but should be restricted in production environments. Production systems should use more restrictive permissions like 755 for directories and 644 for files.

### 7. NFS Export Options

```bash
/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
```

**Option Meanings:**
- **rw:** Read and write access for clients
- **sync:** Write changes to disk before responding (ensures data safety)
- **no_all_squash:** Preserve user IDs from client systems
- **no_root_squash:** Allow root on client to be root on server (use cautiously in production)

**Security Consideration:** The no_root_squash option should be avoided in production as it allows root users on client machines to have root access on the NFS server.

### 8. Database User Management and Security

**Subnet-Based Access Control:**
```sql
CREATE USER 'webaccess'@'172.31.32.0/20' IDENTIFIED BY 'password';
```

**Key Learnings:**
- MySQL can restrict users by IP address or subnet CIDR
- Principle of least privilege: only grant necessary permissions
- Network segmentation improves security by limiting access
- Subnet-based restrictions prevent unauthorized access from outside the network

**Best Practices:**
- Use strong passwords (minimum 12 characters with mixed case, numbers, symbols)
- Grant only necessary privileges, not ALL
- Restrict access by subnet or specific IP addresses
- Regularly audit user permissions

### 9. Persistent Mounts with /etc/fstab

The /etc/fstab file ensures that file systems are automatically mounted at boot time, making mounts persistent across reboots.

**Why /etc/fstab?**
- Ensures mounts survive system reboots
- Automatic mounting at boot time
- Centralized mount configuration
- Prevents manual remounting after restarts

**Entry Format:**
```
<device> <mount-point> <filesystem-type> <options> <dump> <pass>
```

**Example:**
```
172.31.32.10:/mnt/apps /var/www nfs defaults 0 0
```

### 10. Service Management with systemd

systemd is the modern init system and service manager for Linux systems, replacing older systems like SysV init.

**Commands :**
```bash
sudo systemctl start nfs-server.service    # Start service immediately
sudo systemctl enable nfs-server.service   # Enable service at boot
sudo systemctl status nfs-server.service   # Check service status
sudo systemctl restart nfs-server.service  # Restart service
```

## Project Screenshots

### CLI Output
![CLI Commands](./imgs/CLI_Output.png)

### Final Application

**Login Page**
![Application Login](./imgs/Final_Output%201.png)

**Dashboard**
![Application Dashboard](./imgs/Final_Output%202.png)


**systemd Benefits:**
- Parallel service startup for faster boot times
- Dependency management between services
- Service monitoring and automatic restart on failure
- Unified interface for managing all services
- Detailed logging with journalctl

## Technologies Used

### Infrastructure and Cloud
- **AWS EC2:** Virtual servers for hosting all components
- **AWS VPC:** Network isolation and security
- **Security Groups:** Firewall rules for controlling traffic

### Operating Systems
- **Red Hat Enterprise Linux (RHEL) 8:** Web servers and NFS server
- **Ubuntu 24.04 LTS:** Database server

### Storage and File Systems
- **NFS (Network File System):** Shared storage across web servers
- **LVM (Logical Volume Manager):** Flexible disk management
- **XFS File System:** High-performance filesystem for NFS

### Web Stack
- **Apache HTTP Server:** Web server for serving PHP applications
- **PHP 7.4:** Server-side scripting language
- **MySQL 8.0:** Relational database management system


## Implementation

For detailed step-by-step implementation instructions, please refer to [SETUP.md](./SETUP.md).

The setup guide includes:
- NFS Server Configuration
- Database Server Setup
- Web Servers Configuration
- Testing and Verification
- Troubleshooting Guide

## Conclusion

### What Was Accomplished

This project successfully implemented a production-ready 3-tier DevOps tooling website with the following achievements:

- **Scalable Architecture:** Stateless web servers allow horizontal scaling by simply adding more servers
- **Centralized Storage:** NFS provides a shared file system across all web servers for consistent content
- **Data Persistence:** MySQL database ensures data integrity and centralized data management
- **High Availability:** Multiple web servers eliminate single point of failure
- **Cloud Infrastructure:** Leveraged AWS EC2, VPC, and Security Groups for robust hosting

### Skills Developed

- AWS EC2 instance management and configuration
- Linux system administration on RHEL and Ubuntu
- LVM configuration and management
- NFS server setup and client configuration
- MySQL database administration and security
- Apache web server configuration
- PHP application deployment
- Network security with Security Groups
- Troubleshooting distributed systems

### Key Takeaways

**Storage Solutions Matter**

Understanding the differences between NFS, block storage, and object storage is crucial for choosing the right solution for specific use cases. Each storage type has its strengths and appropriate applications.

**Stateless Design Enables Scalability**

By keeping web servers stateless and storing all state in shared storage or databases, we can easily scale horizontally and maintain high availability.

**Infrastructure as Code is the Next Step**

While manual setup helps understand the underlying concepts, automation using tools like Terraform and Ansible is essential for production environments.

**Security Layers Work Together**

Network segmentation, user permissions, and access control must work together to create a secure system. No single security measure is sufficient.

**Monitoring is Essential**

You cannot manage what you do not measure. Proper monitoring and logging are critical for maintaining system health.

### Next Steps and Improvements

**Implement Load Balancer**

Add AWS Application Load Balancer for intelligent traffic distribution across web servers with health checks and automatic failover.

**Automate Deployment**

Use Ansible for configuration management and Terraform for infrastructure as code to enable repeatable, version-controlled deployments.

**Add CI/CD Pipeline**

Implement Jenkins or GitHub Actions for automated testing and deployment of application updates.

**Implement HTTPS**

Add SSL/TLS certificates using Let's Encrypt or AWS Certificate Manager for encrypted communications.

**Set Up Monitoring**

Deploy Prometheus and Grafana for comprehensive monitoring of all system components.

**Database Replication**

Implement MySQL Master-Slave replication for database high availability and read scaling.

**Containerization**

Migrate the application to Docker containers and orchestrate with Kubernetes for improved portability and scalability.

**Implement Caching**

Add Redis or Memcached for session management and caching to improve performance.

### Real-World Applications

This architecture pattern is widely used in production environments:

- **E-commerce Platforms:** Multiple web servers serving the same product catalog and handling transactions
- **Content Management Systems:** WordPress, Drupal, and other CMS platforms in high-traffic scenarios
- **SaaS Applications:** Multi-tenant applications with shared infrastructure
- **Enterprise Portals:** Centralized access to tools and resources for large organizations

## Additional Resources

### Documentation
- [NFS Server Configuration Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/managing_file_systems/exporting-nfs-shares_managing-file-systems)
- [MySQL Documentation](https://dev.mysql.com/doc/)
- [Apache HTTP Server Documentation](https://httpd.apache.org/docs/)
- [AWS EC2 User Guide](https://docs.aws.amazon.com/ec2/)

### Learning Resources
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [DevOps Roadmap](https://roadmap.sh/devops)
- [Linux Academy - LVM Deep Dive](https://linuxacademy.com)

---

**Project Status:** Complete
**Last Updated:** 2025-11-10
**Author:** Sharon Isabela  
**Github:** https://github.com/isabell-ah/DevOps_Cloud-Steghub/tree/master/Devops_Tooling

