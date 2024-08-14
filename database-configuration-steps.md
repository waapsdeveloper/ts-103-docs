Here’s the updated guide to include your previous setup using the `db.ts` file in `src/config/db.ts`:

---

# Setting Up MySQL Database Connection in Express App Using `mysql2`, Connection Pool, and `executeQuery`

This guide will walk you through configuring your Node.js Express application to connect to a MySQL database using the `mysql2` package with a connection pool. We'll also include an `executeQuery` function to simplify query execution.

## Step 1: Install `mysql2` and `dotenv` Packages

First, you need to install the `mysql2` package for MySQL connectivity and `dotenv` for environment variable management.

```bash
npm install mysql2 dotenv
```

## Step 2: Create a Database Configuration File (`db.ts`)

Create a configuration file in `src/config/db.ts` to manage your database connection settings using environment variables.

```typescript
// src/config/db.ts

import mysql from "mysql2/promise";
require('dotenv').config();

const port = process.env.DB_PORT;

const pool = mysql.createPool({
  host: process.env.HOST,
  user: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  port: parseInt(port as any),
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0,
});

export default pool;

export function executeQuery<T>(query: string, params: any[]): Promise<T> {
  return new Promise((resolve, reject) => {
    pool
      .query(query, params)
      .then((result) => {
        resolve(result[0] as any);
      })
      .catch((error) => {
        console.log("err", error);
        reject(new Error(`Error executing query: ${error.message}`));
      });
  });
}
```

### Environment Variables Configuration

Ensure you have a `.env` file at the root of your project with the following variables:

```plaintext
HOST=localhost
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=root
DB_NAME=presentation-1
```

## Step 3: Use `executeQuery` in Your Express Application

Now that you’ve set up the database connection, you can use the `executeQuery` function to interact with the MySQL database in your Express routes.

### Example: Fetching Data from the Database

Here’s how to use `executeQuery` in a controller to fetch data.

```typescript
// src/controllers/dataController.ts
import { Request, Response } from 'express';
import { executeQuery } from '../config/db';

export const getData = async (req: Request, res: Response) => {
  try {
    const query = 'SELECT * FROM your_table_name';
    const data = await executeQuery(query, []);
    res.json(data);
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch data from the database' });
  }
};
```

### Example Route Setup

Create a route that uses the controller to serve data.

```typescript
// src/routes/dataRoutes.ts
import express from 'express';
import { getData } from '../controllers/dataController';

const router = express.Router();

router.get('/data', getData);

export default router;
```

### Integrate the Route into Your Express App

Integrate the route into your main Express application.

```typescript
// src/app.ts
import express from 'express';
import dataRoutes from './routes/dataRoutes';

const app = express();

app.use(express.json());

app.use('/api', dataRoutes);

export default app;
```

```typescript
// src/index.ts
import app from './app';

const PORT = process.env.PORT || 3600;

app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
```

## Step 4: Test the Database Connection

To test the connection and ensure your Express app is correctly communicating with the MySQL database, start your server:

```bash
npm start
```

Then, you can send a request to the `/api/data` endpoint, which should return data from the `presentation-1` database.

### Example Test with `curl`:

```bash
curl http://localhost:3600/api/data
```

You should see the data from your MySQL table in the terminal output.

---

This guide helps you set up a MySQL database connection in your Express application using the `mysql2` package, connection pooling, and the `executeQuery` function for streamlined database operations.