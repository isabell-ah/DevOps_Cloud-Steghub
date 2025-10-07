# Setup Guide - MEAN Stack on AWS

Complete installation and deployment guide for the Book Management System.

## Prerequisites
- AWS EC2 instance (Ubuntu)
- SSH access to your instance

## Installation Steps

### 1. SSH into EC2 Instance
```bash
ssh -i <your.pem> ubuntu@<public-IP>
```

### 2. Install Node.js and npm
```bash
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 3. Verify Installation
```bash
node --version
npm --version
```

### 4. Install MongoDB
```bash
sudo apt-get install -y gnupg curl
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
  sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
  --dearmor

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

sudo apt-get update
sudo apt-get install -y mongodb-org
```

### 5. Start MongoDB
```bash
sudo systemctl start mongod
sudo systemctl enable mongod
```

### 6. Navigate to Project Directory
```bash
cd ~/Books
```

### 7. Install Node.js Dependencies
```bash
npm install express body-parser mongoose
```

### 8. Configure AWS Security Group
- Go to AWS Console → EC2 → Security Groups
- Select your instance's security group
- Add Inbound Rule:
  - Type: Custom TCP
  - Port: 3000
  - Source: 0.0.0.0/0 (for public access)

### 9. Run the Application
```bash
node server.js
```

### 10. Access the App
- Open browser: `http://YOUR_EC2_PUBLIC_IP:3000`

## Project Structure
```
Books/
├── server.js              # Main server file
├── package.json           # Dependencies
├── apps/
│   ├── route.js          # API routes
│   └── models/
│       └── book.js       # MongoDB book schema
└── public/
    ├── index.html        # Frontend UI
    └── script.js         # AngularJS controller
```

## Verification
After setup, you should see:
```
Server up: http://0.0.0.0:3000
MongoDB connected
```

Your application will be accessible at `http://YOUR_EC2_PUBLIC_IP:3000`