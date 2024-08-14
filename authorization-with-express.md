Here’s a `README.md` file detailing how to implement authentication and authorization using JWT (JSON Web Tokens) with Express.js:

---

# Authentication and Authorization with JWT in Express.js

This guide explains how to implement authentication and authorization in an Express.js application using JSON Web Tokens (JWT). JWT is a compact, URL-safe means of representing claims to be transferred between two parties. It’s widely used for securing APIs by authenticating users and verifying their identity.

## Table of Contents

- [Introduction](#introduction)
- [Setting Up the Project](#setting-up-the-project)
- [Installing Dependencies](#installing-dependencies)
- [Setting Up Environment Variables](#setting-up-environment-variables)
- [Creating JWT Utility Functions](#creating-jwt-utility-functions)
- [Implementing User Registration](#implementing-user-registration)
- [Implementing User Login](#implementing-user-login)
- [Protecting Routes with JWT Middleware](#protecting-routes-with-jwt-middleware)
- [Role-Based Authorization](#role-based-authorization)
- [Testing the Implementation](#testing-the-implementation)
- [Conclusion](#conclusion)

## Introduction

JWT is a widely-used standard for token-based authentication. It allows you to secure your APIs by ensuring that only authenticated users can access certain endpoints. This guide will show you how to create an authentication system using JWT in an Express.js application.

## Setting Up the Project

Start by creating a new Express.js project if you haven't already:

```bash
mkdir jwt-auth-example
cd jwt-auth-example
npm init -y
```

## Installing Dependencies

Install the necessary dependencies:

```bash
npm install express bcryptjs jsonwebtoken dotenv
```

- `express`: Web framework for Node.js.
- `bcryptjs`: Library to hash passwords.
- `jsonwebtoken`: Library to sign, verify, and decode JWT tokens.
- `dotenv`: Library to load environment variables from a `.env` file.

## Setting Up Environment Variables

Create a `.env` file in the root of your project to store environment variables:

```
PORT=3000
JWT_SECRET=your_secret_key
JWT_EXPIRES_IN=1h
```

- `JWT_SECRET`: A secret key used to sign the JWT. Keep this secure and do not expose it.
- `JWT_EXPIRES_IN`: The duration for which the JWT is valid.

## Creating JWT Utility Functions

Create a `jwtUtils.js` file to handle JWT creation and verification:

```javascript
const jwt = require('jsonwebtoken');
const dotenv = require('dotenv');

dotenv.config();

const JWT_SECRET = process.env.JWT_SECRET;
const JWT_EXPIRES_IN = process.env.JWT_EXPIRES_IN;

const generateToken = (user) => {
  return jwt.sign({ id: user.id, role: user.role }, JWT_SECRET, {
    expiresIn: JWT_EXPIRES_IN,
  });
};

const verifyToken = (token) => {
  return jwt.verify(token, JWT_SECRET);
};

module.exports = { generateToken, verifyToken };
```

## Implementing User Registration

Create a basic user registration endpoint that stores user data and hashes passwords:

```javascript
const express = require('express');
const bcrypt = require('bcryptjs');
const { generateToken } = require('./jwtUtils');

const app = express();
app.use(express.json());

let users = []; // In-memory user store for demonstration purposes

app.post('/register', async (req, res) => {
  const { username, password, role } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);

  const user = { id: Date.now().toString(), username, password: hashedPassword, role };
  users.push(user);

  const token = generateToken(user);

  res.status(201).json({ token });
});
```

## Implementing User Login

Create a login endpoint that checks the user’s credentials and returns a JWT:

```javascript
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = users.find(u => u.username === username);

  if (!user) {
    return res.status(401).json({ message: 'Invalid credentials' });
  }

  const isMatch = await bcrypt.compare(password, user.password);

  if (!isMatch) {
    return res.status(401).json({ message: 'Invalid credentials' });
  }

  const token = generateToken(user);

  res.json({ token });
});
```

## Protecting Routes with JWT Middleware

Create a middleware function to protect routes by verifying the JWT:

```javascript
const jwt = require('jsonwebtoken');
const { verifyToken } = require('./jwtUtils');

const authenticateJWT = (req, res, next) => {
  const authHeader = req.headers.authorization;

  if (authHeader) {
    const token = authHeader.split(' ')[1];

    try {
      const decoded = verifyToken(token);
      req.user = decoded;
      next();
    } catch (err) {
      return res.status(403).json({ message: 'Invalid or expired token' });
    }
  } else {
    res.status(401).json({ message: 'Authorization header required' });
  }
};
```

Apply the `authenticateJWT` middleware to protect routes:

```javascript
app.get('/protected', authenticateJWT, (req, res) => {
  res.json({ message: 'You have accessed a protected route!', user: req.user });
});
```

## Role-Based Authorization

You can extend JWT-based authentication to support role-based authorization. Here’s an example of a middleware function that checks the user’s role:

```javascript
const authorizeRole = (roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ message: 'Access denied' });
    }
    next();
  };
};

// Example: Only allow access to 'admin' users
app.get('/admin', authenticateJWT, authorizeRole(['admin']), (req, res) => {
  res.json({ message: 'Welcome, Admin!' });
});
```

## Testing the Implementation

You can test your implementation using tools like Postman or Curl. Here’s a summary of how to test:

1. **Register a User:**
   - POST `/register`
   - Body: `{ "username": "test", "password": "password123", "role": "user" }`
2. **Login:**
   - POST `/login`
   - Body: `{ "username": "test", "password": "password123" }`
   - Response: `{ "token": "your_jwt_token" }`
3. **Access Protected Route:**
   - GET `/protected`
   - Header: `Authorization: Bearer your_jwt_token`
4. **Access Admin Route (if applicable):**
   - GET `/admin`
   - Header: `Authorization: Bearer your_jwt_token`

## Conclusion

Implementing JWT for authentication and authorization in Express.js is a powerful way to secure your applications. With JWT, you can easily manage user sessions, protect sensitive routes, and implement role-based access control. This guide provides a foundational understanding, and you can expand on it to meet your application's specific needs.

---

This `README.md` file covers the essential steps for setting up authentication and authorization using JWT with Express.js, providing a solid foundation for building secure Node.js applications.