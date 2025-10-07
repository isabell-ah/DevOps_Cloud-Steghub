author: Sharon Isabela
Github: https://github.com/isabell-ah/DevOps_Cloud-Steghub/tree/master/MEAN_Stack


# Book Management System - MEAN Stack on AWS

A full-stack web application built with MongoDB, Express.js, AngularJS, and Node.js, deployed on AWS EC2.

## Live Demo
Access the application at: `http://YOUR_EC2_PUBLIC_IP:3000`

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

## Features
- Add new books with name, ISBN, author, and pages
- View all books in a table format
- Delete books by ISBN
- Real-time updates without page refresh
- Responsive web interface

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/book` | Get all books |
| POST | `/book` | Add new book |
| DELETE | `/book/:isbn` | Delete book by ISBN |
| GET | `/` | Serve main page |

## Documentation
- [Setup Guide](SETUP.md) - Complete installation and deployment instructions
- [Learning Journey](LEARNING.md) - Key concepts and troubleshooting experiences

---
*Built and deployed by me as part of my MEAN stack learning journey on AWS*