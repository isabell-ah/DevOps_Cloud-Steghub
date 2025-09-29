# MERN Stack Todo App Documentation
## Complete Guide: MongoDB Atlas + Express + React + Node.js on AWS EC2

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [AWS EC2 Setup](#aws-ec2-setup)
5. [MongoDB Atlas Configuration](#mongodb-atlas-configuration)
6. [Backend Development (Node.js + Express)](#backend-development)
7. [Frontend Development (React)](#frontend-development)
8. [Monitoring & Maintenance](#monitoring--maintenance)
9. [Troubleshooting](#troubleshooting)

---

## Project Overview

### What is MERN Stack?
- **M**ongoDB: NoSQL database for data storage
- **E**xpress.js: Web framework for Node.js
- **R**eact: Frontend JavaScript library
- **N**ode.js: JavaScript runtime environment

### Todo App Features
- Create, Read, Update, Delete (CRUD) operations
- Real-time updates
- Responsive design
- Cloud database integration

### Technology Stack
```
Frontend: React.js, HTML5, CSS3, JavaScript ES6+
Backend: Node.js, Express.js
Database: MongoDB Atlas (Cloud)
Hosting: AWS EC2
Process Manager: PM2
```

---

## Architecture

### System Architecture Diagram
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   React Client  │────│   AWS EC2       │────│  MongoDB Atlas  │
│   (Frontend)    │    │   (Backend)     │    │   (Database)    │
│                 │    │                 │    │                 │
│ - Components    │    │ - Node.js       │    │ -               │
│ - State Mgmt    │    │ - Express API   │    │ -               │
│ - HTTP Requests │    │ - Middleware    │    │ -               │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Data Flow
```
1. User interacts with React frontend
2. Frontend sends HTTP requests to Express API
3. Express processes requests and validates data
4. Backend connects to MongoDB Atlas
5. Database operations performed
6. Response sent back to frontend
7. UI updates with new data
```

---

## Prerequisites

### Required Software
- AWS Account with EC2 access
- MongoDB Atlas account
- Node.js (v16+ recommended)
- npm or yarn package manager
- Git for version control
- Code editor (VS Code recommended)

### Required Knowledge
- JavaScript fundamentals
- React basics
- Node.js and Express
- MongoDB queries
- AWS EC2 basics
- Linux command line

---

## AWS EC2 Setup

### 1. Launch EC2 Instance
```bash
# Instance Configuration
Instance Type: t2.micro (Free tier eligible)
AMI: Ubuntu Server 22.04 LTS
Storage: 8GB GP2
Security Group: Custom (see below)
```

### 2. Security Group Configuration
```bash
# Inbound Rules
Type        Protocol    Port Range    Source
SSH         TCP         22           Your IP/0.0.0.0/0
HTTP        TCP         80           0.0.0.0/0
HTTPS       TCP         443          0.0.0.0/0
Custom TCP  TCP         3000         0.0.0.0/0  # React dev server
Custom TCP  TCP         5000         0.0.0.0/0  # Express API
```

### 3. Connect to EC2 Instance
```bash
# Using SSH
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip

# Update system packages
sudo apt update && sudo apt upgrade -y
```

### 4. Install Required Software
```bash
# Install Node.js and npm
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node --version
npm --version

# Install PM2 globally
sudo npm install -g pm2

# Install Git
sudo apt install git -y

# Install Nginx (optional)
sudo apt install nginx -y
```

---

## MongoDB Atlas Configuration

### 1. Create MongoDB Atlas Account
```
1. Go to https://www.mongodb.com/atlas
2. Sign up for free account
3. Create new project
4. Build a database (Free M0 cluster)
```

### 2. Cluster Configuration
```bash
# Cluster Settings
Provider: AWS
Region: Same as your EC2 (e.g., us-east-1)
Cluster Tier: M0 Sandbox (Free)
Cluster Name: TodoApp-Cluster
```

### 3. Database Access Setup
```bash
# Create Database User
Username: todoapp-user
Password: [Generate secure password]
Privileges: Read and write to any database
```

### 4. Network Access Configuration
```bash
# Add IP Addresses
1. Add your current IP
2. Add EC2 instance public IP
3. Or use 0.0.0.0/0 (less secure but simpler for development)
```

### 5. Get Connection String
```javascript
// Example connection string
mongodb+srv://todoapp-user:<password>@todoapp-cluster.xxxxx.mongodb.net/todoapp?retryWrites=true&w=majority
```

---

## Backend Development

### 1. Project Structure
```
backend/
├── config/
│   └── database.js
├── models/
│   └── Todo.js
├── routes/
│   └── todos.js
├── .env
├── app.js
└── package.json
```

### 2. Initialize Backend Project
```bash
# Create project directory
mkdir mern-todo-app
cd mern-todo-app
mkdir backend
cd backend

# Initialize npm project
npm init -y

# Install dependencies
npm install express mongoose cors dotenv bcryptjs jsonwebtoken
npm install -D nodemon
```

### 3. Package.json Configuration
```json
{
  "name": "todo-backend",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.5.0",

    "dotenv": "^16.3.1",
 
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

### 4. Environment Variables (.env)
```bash
# Database
MONGODB_URI=mongodb+srv://todoapp-user:password@todoapp-cluster.xxxxx.mongodb.net/todoapp

# Server
PORT=5000
NODE_ENV=production

# JWT
JWT_SECRET=your-super-secret-jwt-key-here
JWT_EXPIRE=7d
```

### 5. Database Configuration
```javascript
// config/database.js
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGODB_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log(`MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error('Database connection error:', error);
    process.exit(1);
  }
};

module.exports = connectDB;
```

### 6. Todo Model
```javascript
// models/Todo.js
const mongoose = require('mongoose');

const todoSchema = new mongoose.Schema({
  action: {
    type: String,
    required: [true, 'Title is required'],
    trim: true,
    maxlength: [100, 'Title cannot exceed 100 characters']
  },
  

module.exports = mongoose.model('Todo', todoSchema);
```
