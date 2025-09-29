# Simple MERN Stack Todo App
## Basic CRUD Application: MongoDB + Express + React + Node.js

> **Note**: This is a simple todo CRUD application built for learning purposes. AWS EC2 was used as a development environment instead of local computer.

---

## Quick Start

### Prerequisites
- AWS EC2 instance (Ubuntu 22.04) - *used as development environment*
- MongoDB Atlas account (free tier)
- Node.js 18+
- Git

### Installation

#### 1. Clone Repository
```bash
git clone <your-repo-url>
cd mern-todo-app
```

#### 2. Backend Setup
```bash
cd backend
npm install
```

Create `.env` file:
```bash
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/todoapp
PORT=5000
```

Start backend:
```bash
npm run dev
```

#### 3. Frontend Setup
```bash
cd ../frontend
npm install
```

Create `.env` file:
```bash
REACT_APP_API_URL=http://your-ec2-ip:5000/api
```

Start frontend:
```bash
npm start
```

---

## Project Structure

```
mern-todo-app/
├── backend/
│   ├── config/
│   │   └── database.js
│   ├── models/
│   │   └── Todo.js
│   ├── routes/
│   │   └── todos.js
│   ├── .env
│   ├── app.js
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── context/│  
 │   └── App.js
│   └── package.json
└── README.md
```

---
##  AWS EC2 Configuration

### Security Group Rules
```
Port 22   (SSH)     - Your IP
Port 80   (HTTP)    - 0.0.0.0/0
Port 443  (HTTPS)   - 0.0.0.0/0
Port 5000 (Express) - 0.0.0.0/0
```

### Install Dependencies on EC2
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PM2
sudo npm install -g pm2

# Install Git
sudo apt install git -y
```

---

## MongoDB Atlas Setup

1. **Create Cluster**
   - Provider: AWS
   - Region: Same as EC2
   - Tier: M0 (Free)

2. **Database Access**
   - Create user with read/write permissions
   - Generate secure password

3. **Network Access**
   - Add EC2 public IP
   - Or use 0.0.0.0/0 for development

4. **Get Connection String**
   ```
   mongodb+srv://username:password@cluster.mongodb.net/todoapp
   ```

---

## Running on EC2 (Development Environment)

### Backend Setup
```bash
# Clone and setup
git clone <repo-url>
cd mern-todo-app/backend
npm install

# Create .env file
nano .env

# Run in development mode
npm run dev
```

### Frontend Setup
```bash
cd ../frontend
npm install

# Start React development server
npm start
```

*Note: EC2 was used as a development environment, not for production hosting*

---

##  API Endpoints

### Todos
```
GET    /api/todos     - Get all todos
POST   /api/todos     - Create todo
PUT    /api/todos/:id - Update todo
DELETE /api/todos/:id - Delete todo
```

---
## Testing with Postman



### . Create Todo (with Bearer token)
```
POST http://your-ec2-ip:5000/api/todos
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{
  "title": "Learn MERN Stack",
  "description": "Complete todo app tutorial",
  "priority": "high"
}
```

---

## Troubleshooting

### Common Issues

**MongoDB Connection Failed**
```bash
# Check connection string
# Verify IP whitelist in Atlas
# Test connectivity
ping cluster0.xxxxx.mongodb.net
```

**CORS Errors**
```javascript
// Add to server.js
app.use(cors({
  origin: ['http://localhost:3000', 'http://your-ec2-ip:3000'],
  credentials: true
}));
```

**Port Already in Use**
```bash
# Check what's using the port
sudo netstat -tlnp | grep :5000

# Kill process if needed
sudo kill -9 <PID>
```

### Useful Commands
```bash
# Check PM2 status
pm2 status

# View logs
pm2 logs todo-backend
pm2 logs todo-frontend

# Restart services
pm2 restart all

# Check system resources
htop
df -h
```

---

## Simple Features

- ✅ Basic CRUD Operations (Create, Read, Update, Delete todos)
- ✅ MongoDB Atlas for data storage
- ✅ React frontend with basic components
- ✅ Express.js REST API
- ✅ Postman API testing
- ✅ AWS EC2 as development environment



##  Contributing

1. Fork the repository
2. Create feature branch
3. Commit changes
4. Push to branch
5. Create Pull Request

---

