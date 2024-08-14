Here's a step-by-step guide to create a Node.js Express app with TypeScript, suitable for adding to a Markdown file (`README.md`).

---

# Setting Up a Node.js Express App with TypeScript

This guide will walk you through the steps to set up a Node.js Express app using TypeScript. By the end, you'll have a basic Express server running with TypeScript support.

## Prerequisites

Make sure you have the following installed on your system:

- **Node.js** (v14 or higher)
- **npm** (v6 or higher) or **yarn**
- **TypeScript** (globally installed)

## Step 1: Initialize the Project

Start by creating a new directory for your project and initializing it with `npm`:

```bash
mkdir my-express-app
cd my-express-app
npm init -y
```

This will generate a `package.json` file in your project directory.

## Step 2: Install Dependencies

Next, install the necessary dependencies for an Express server:

```bash
npm install express
```

Then, install the TypeScript compiler and other required types:

```bash
npm install --save-dev typescript @types/node @types/express ts-node
```

- **`typescript`**: The TypeScript compiler.
- **`@types/node`**: Type definitions for Node.js.
- **`@types/express`**: Type definitions for Express.
- **`ts-node`**: A TypeScript execution environment for Node.js.

## Step 3: Configure TypeScript

Create a `tsconfig.json` file in the root of your project to configure TypeScript:

```bash
npx tsc --init
```

This command will create a basic `tsconfig.json`. You can customize it as needed, but here’s a simple configuration:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist"
  },
  "include": ["src/**/*.ts"],
  "exclude": ["node_modules"]
}
```

## Step 4: Create the Project Structure

Create the necessary directories and files:

```bash
mkdir src
touch src/index.ts
```

Your project structure should now look like this:

```
my-express-app/
├── node_modules/
├── src/
│   └── index.ts
├── package.json
└── tsconfig.json
```

## Step 5: Write the Express Server Code

Open the `src/index.ts` file and write the basic Express server code:

```typescript
import express, { Request, Response } from 'express';

const app = express();
const port = 3000;

app.get('/', (req: Request, res: Response) => {
  res.send('Hello, world!');
});

app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});
```

This is a simple Express server that listens on port 3000 and responds with "Hello, world!" when the root URL is accessed.

## Step 6: Add Scripts to `package.json`

To run your TypeScript code, you can add the following scripts to your `package.json`:

```json
"scripts": {
  "start": "node dist/index.js",
  "build": "tsc",
  "dev": "ts-node src/index.ts"
}
```

- **`start`**: Compiles the TypeScript code and runs the compiled JavaScript.
- **`build`**: Compiles the TypeScript code to JavaScript.
- **`dev`**: Runs the TypeScript code directly using `ts-node`.

## Step 7: Run the Server

For development, you can run the server using:

```bash
npm run dev
```

For production, first build the project:

```bash
npm run build
```

Then, start the server:

```bash
npm start
```

Your server should now be running on `http://localhost:3000`.

## Step 8: Add Nodemon for Auto-reloading (Optional)

To automatically restart the server when you make changes, install `nodemon`:

```bash
npm install --save-dev nodemon
```

Update the `dev` script in `package.json`:

```json
"scripts": {
  "start": "node dist/index.js",
  "build": "tsc",
  "dev": "nodemon --watch src --exec ts-node src/index.ts"
}
```

Now, when you run `npm run dev`, the server will automatically restart whenever you make changes to the `src` directory.

## Step 9: Linting and Formatting (Optional)

To keep your code clean and consistent, you can set up ESLint and Prettier:

```bash
npm install --save-dev eslint prettier eslint-config-prettier eslint-plugin-prettier @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

Create an `.eslintrc.json` file:

```json
{
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint", "prettier"],
  "extends": ["eslint:recommended", "plugin:@typescript-eslint/recommended", "prettier"],
  "rules": {
    "prettier/prettier": "error"
  }
}
```

Create a `.prettierrc` file:

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 80
}
```

Add linting and formatting scripts to `package.json`:

```json
"scripts": {
  "lint": "eslint 'src/**/*.{js,ts}'",
  "format": "prettier --write 'src/**/*.{js,ts}'"
}
```

You can now run `npm run lint` to check for linting issues and `npm run format` to automatically format your code.

---

This completes the basic setup for a Node.js Express app with TypeScript. You can now start developing your application, taking advantage of TypeScript's powerful type-checking features.

--- 

Feel free to customize this guide according to your project needs!