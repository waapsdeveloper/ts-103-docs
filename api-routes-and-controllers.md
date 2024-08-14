Here’s a step-by-step guide to set up API routes and controllers in a Node.js Express app with TypeScript, following an MVC structure similar to .NET, including login, registration, and user authentication/authorization. You can add this guide to your `README.md` file.

---

# Setting Up API Routes and Controllers with MVC Structure in Node.js Express with TypeScript

This guide will walk you through setting up a structured API using the MVC (Model-View-Controller) pattern in a Node.js Express app with TypeScript. We'll create routes for login, registration, and user authentication/authorization.

## Prerequisites

Ensure you have the following installed:

- **Node.js** (v14 or higher)
- **npm** (v6 or higher) or **yarn**
- **TypeScript** (globally installed)

## Step 1: Project Structure

Start by setting up a directory structure that follows the MVC pattern. Here’s an example structure:

```
my-express-app/
├── src/
│   ├── controllers/
│   │   ├── authController.ts
│   │   └── userController.ts
│   ├── middlewares/
│   │   └── authMiddleware.ts
│   ├── models/
│   │   └── userModel.ts
│   ├── routes/
│   │   ├── authRoutes.ts
│   │   └── userRoutes.ts
│   ├── services/
│   │   └── authService.ts
│   ├── utils/
│   │   └── generateToken.ts
│   ├── index.ts
│   └── app.ts
├── package.json
└── tsconfig.json
```

## Step 2: Create the `User` Model

Define the `User` model to interact with the database. For simplicity, we'll assume you're using a simple in-memory structure, but this can be extended to use a database like MongoDB, MySQL, etc.

```typescript
// src/models/userModel.ts

export interface User {
  id: string;
  username: string;
  password: string;
  email: string;
  isAdmin: boolean;
}

const users: User[] = [];

export default users;
```

## Step 3: Create Controllers

### Authentication Controller

This controller will handle login and registration logic.

```typescript
// src/controllers/authController.ts
import { Request, Response } from 'express';
import users, { User } from '../models/userModel';
import { v4 as uuidv4 } from 'uuid';
import bcrypt from 'bcryptjs';
import generateToken from '../utils/generateToken';

export const registerUser = (req: Request, res: Response) => {
  const { username, password, email } = req.body;

  const userExists = users.find((user) => user.email === email);
  if (userExists) {
    return res.status(400).json({ message: 'User already exists' });
  }

  const hashedPassword = bcrypt.hashSync(password, 10);
  const newUser: User = {
    id: uuidv4(),
    username,
    password: hashedPassword,
    email,
    isAdmin: false,
  };

  users.push(newUser);

  res.status(201).json({
    id: newUser.id,
    username: newUser.username,
    email: newUser.email,
    token: generateToken(newUser.id),
  });
};

export const loginUser = (req: Request, res: Response) => {
  const { email, password } = req.body;

  const user = users.find((user) => user.email === email);
  if (user && bcrypt.compareSync(password, user.password)) {
    res.json({
      id: user.id,
      username: user.username,
      email: user.email,
      token: generateToken(user.id),
    });
  } else {
    res.status(401).json({ message: 'Invalid credentials' });
  }
};
```

### User Controller

This controller will handle user-specific actions.

```typescript
// src/controllers/userController.ts
import { Request, Response } from 'express';
import users from '../models/userModel';

export const getUserProfile = (req: Request, res: Response) => {
  const user = users.find((user) => user.id === req.user?.id);
  if (user) {
    res.json({
      id: user.id,
      username: user.username,
      email: user.email,
      isAdmin: user.isAdmin,
    });
  } else {
    res.status(404).json({ message: 'User not found' });
  }
};
```

## Step 4: Create Authentication Middleware

This middleware will protect routes that require authentication.

```typescript
// src/middlewares/authMiddleware.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import users from '../models/userModel';

declare module 'express-serve-static-core' {
  interface Request {
    user?: any;
  }
}

export const protect = (req: Request, res: Response, next: NextFunction) => {
  const token = req.headers.authorization?.split(' ')[1];

  if (token) {
    try {
      const decoded: any = jwt.verify(token, process.env.JWT_SECRET || 'secret');
      req.user = users.find((user) => user.id === decoded.id);
      next();
    } catch (error) {
      res.status(401).json({ message: 'Not authorized, token failed' });
    }
  } else {
    res.status(401).json({ message: 'Not authorized, no token' });
  }
};
```

## Step 5: Create Utility for Token Generation

Generate JWT tokens for authentication.

```typescript
// src/utils/generateToken.ts
import jwt from 'jsonwebtoken';

const generateToken = (id: string) => {
  return jwt.sign({ id }, process.env.JWT_SECRET || 'secret', {
    expiresIn: '30d',
  });
};

export default generateToken;
```

## Step 6: Create Routes

### Authentication Routes

Define routes for registration and login.

```typescript
// src/routes/authRoutes.ts
import express from 'express';
import { registerUser, loginUser } from '../controllers/authController';

const router = express.Router();

router.post('/register', registerUser);
router.post('/login', loginUser);

export default router;
```

### User Routes

Define routes for user-specific actions.

```typescript
// src/routes/userRoutes.ts
import express from 'express';
import { getUserProfile } from '../controllers/userController';
import { protect } from '../middlewares/authMiddleware';

const router = express.Router();

router.get('/profile', protect, getUserProfile);

export default router;
```

## Step 7: Set Up the Express App

Finally, wire everything together in the main app file.

```typescript
// src/app.ts
import express from 'express';
import authRoutes from './routes/authRoutes';
import userRoutes from './routes/userRoutes';

const app = express();

app.use(express.json());

app.use('/api/auth', authRoutes);
app.use('/api/user', userRoutes);

export default app;
```

```typescript
// src/index.ts
import app from './app';

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
```

## Step 8: Add Environment Variables

To handle sensitive data like JWT secrets, use environment variables. Create a `.env` file in the root of your project:

```plaintext
JWT_SECRET=your_jwt_secret_key
PORT=5000
```

Load these variables in your application by installing `dotenv`:

```bash
npm install dotenv
```

Then, add the following at the top of your `src/index.ts` file:

```typescript
import dotenv from 'dotenv';

dotenv.config();
```

## Step 9: Testing the API

With everything set up, you can now test your API routes:

1. **Register a User:**

   - Endpoint: `POST /api/auth/register`
   - Body: `{ "username": "your_username", "email": "your_email", "password": "your_password" }`

2. **Login a User:**

   - Endpoint: `POST /api/auth/login`
   - Body: `{ "email": "your_email", "password": "your_password" }`

3. **Get User Profile (Protected Route):**

   - Endpoint: `GET /api/user/profile`
   - Headers: `{ "Authorization": "Bearer <your_jwt_token>" }`

---

This guide sets up a basic MVC structure with authentication and authorization using Node.js, Express, and TypeScript. You can expand on this foundation to include more features like role-based access control, password reset, etc.