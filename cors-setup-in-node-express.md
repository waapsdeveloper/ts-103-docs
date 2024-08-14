Here’s a step-by-step guide to set up API CORS (Cross-Origin Resource Sharing) for your Node.js server running on port 3600, allowing connections from your Ionic Angular project running on port 8100. You can add this guide to your `README.md` file.

---

# Setting Up CORS in a Node.js API

This guide will walk you through setting up CORS in your Node.js Express application to allow requests from an Ionic Angular project running on a different port.

## Step 1: Install CORS Middleware

First, you need to install the `cors` package, which provides an Express middleware to enable CORS.

```bash
npm install cors
```

## Step 2: Configure CORS in Your Express App

Next, you need to configure the CORS middleware in your Express application. You’ll allow requests from your Ionic Angular project running on port 8100.

### Option 1: Allow Specific Origin

You can configure CORS to allow requests only from the Ionic Angular project running on port 8100.

```typescript
// src/app.ts
import express from 'express';
import cors from 'cors';

const app = express();

app.use(cors({
  origin: 'http://localhost:8100', // Ionic Angular project URL
  methods: 'GET,POST,PUT,DELETE', // Allowed HTTP methods
  credentials: true // Enable credentials (e.g., cookies)
}));

// Add your routes and other middlewares here

export default app;
```

### Option 2: Allow Multiple Origins (If Needed)

If you have multiple origins (e.g., development and production), you can set up CORS to allow them.

```typescript
// src/app.ts
import express from 'express';
import cors from 'cors';

const app = express();

const allowedOrigins = ['http://localhost:8100', 'https://your-production-url.com'];

app.use(cors({
  origin: function (origin, callback) {
    if (!origin || allowedOrigins.indexOf(origin) !== -1) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  methods: 'GET,POST,PUT,DELETE',
  credentials: true
}));

// Add your routes and other middlewares here

export default app;
```

### Option 3: Allow All Origins (Not Recommended for Production)

For development purposes, you might want to allow all origins. This is not recommended for production environments.

```typescript
// src/app.ts
import express from 'express';
import cors from 'cors';

const app = express();

app.use(cors({
  origin: '*', // Allow all origins
  methods: 'GET,POST,PUT,DELETE',
  credentials: true
}));

// Add your routes and other middlewares here

export default app;
```

## Step 3: Set Up the Express App

Make sure your Express app is configured correctly and listening on port 3600.

```typescript
// src/index.ts
import app from './app';

const PORT = process.env.PORT || 3600;

app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
```

## Step 4: Test CORS Configuration

To test if CORS is configured correctly, try making a request from your Ionic Angular project running on port 8100 to your Node.js server running on port 3600.

If configured correctly, your Ionic Angular app should be able to make API requests without running into CORS-related issues.

### Example Request from Angular:

```typescript
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root',
})
export class ApiService {
  private apiUrl = 'http://localhost:3600/api';

  constructor(private http: HttpClient) {}

  getData() {
    return this.http.get(`${this.apiUrl}/data`);
  }
}
```

---

This guide ensures that your Node.js API server allows cross-origin requests from your Ionic Angular project. You can further customize the CORS configuration as per your needs.