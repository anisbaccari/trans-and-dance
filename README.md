
# Basic example 

A simple example on how to connect a frontend javascript vanilla index.js to a backend nodejs fastify endpoint. 




# Installation

- the frontend will be launch with the vscode extension "live-server"
- the backend will be launch in a separete terminal with nodejs 

## Backend 
1.1 Install Node.js and npm

 - Step 1: Set Up the Backend (Fastify)
First, ensure you have Node.js and npm installed. Open a terminal and run:
```bash
node -v
npm -v
```
- If not installed, install Node.js (LTS version recommended):
```bash
sudo apt update && sudo apt install -y nodejs npm
```

1.2 Initialize a New Fastify Project

- Create a directory for your backend:

```bash
mkdir fastify-backend && cd fastify-backend
npm init -y
```

1.3 Install Fastify
```bash
npm install fastify dotenv  @fastify/cors @fastifyjwt
```

- fastify → Main backend framework.

- dotenv → Loads environment variables. ( not necessary )

 - fastify-cors → Enables CORS (important for frontend communication).

 - fastify-jwt → JSON Web Token authentication. ( not necessary )


 1.4 Project Structure

Run:
```bash
mkdir src routes plugins
touch src/index.js  routes/users.js  plugins/auth.js
```
Your project structure should look like this:

    fastify-backend/
    │── node_modules/
    │── src/
    │   ├── index.js
    │── routes/
    │   ├── users.js
    │── plugins/
    │   ├── auth.js
    │── .env
    │── package.json
    │── package-lock.json

1.5 Configure Fastify Server (src/index.js)

- Create a .env File
```bash
touch .env
```
- Edit to .env
```bash
JWT_SECRET=your-secret-key
```


- Edit src/index.js:
```bash
    import Fastify from 'fastify';
    import dotenv from 'dotenv';
    import cors from '@fastify/cors';
    import jwt from '@fastify/jwt';

    dotenv.config();

    const fastify = Fastify({ logger: true });

    // Register plugins
    fastify.register(cors, { origin: '*' });
    fastify.register(jwt, { secret: process.env.JWT_SECRET });

    // Sample route
    fastify.get('/', async () => {
    return { message: 'Fastify backend is running!' };
    });

    // Import routes
    import userRoutes from '../routes/users.js';
    fastify.register(userRoutes, { prefix: '/api/users' });

    // Start server
    const start = async () => {
    try {
        await fastify.listen({ port: 3000, host: '0.0.0.0' });
        console.log('🚀 Server running on http://localhost:3000');
    } catch (err) {
        fastify.log.error(err);
        process.exit(1);
    }
    };
    start();
```

1.7 Create Authentication Plugin (plugins/auth.js)

- Edit plugins/auth.js:
```bash
export async function authenticate(request, reply) {
  try {
    await request.jwtVerify();
  } catch (err) {
    reply.send(err);
  }
}
```
1.8 Create User Routes (routes/users.js)

- Edit routes/users.js:
```bash
import { authenticate } from '../plugins/auth.js';

export default async function (fastify, options) {
  fastify.post('/login', async (request, reply) => {
    const { username, password } = request.body;

    if (username === 'admin' && password === 'password') {
      const token = fastify.jwt.sign({ username });
      return { token };
    }
    reply.code(401).send({ error: 'Unauthorized' });
  });

  fastify.get('/profile', { preHandler: authenticate }, async (request, reply) => {
    return { message: 'This is a protected route', user: request.user };
  });
}
```
1.9 package.json 
since index.js file uses ES module syntax (`import`/`export`) and  
 Node.js is treating it as a CommonJS module by default.  
 We need to modify the package.json to resolve it 
 by adding the fild
```bash
  {
  "type": "module"
  }
```
- it should be like this : 
```bash
{
  "name": "fastify-backend",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "type": "module",
  "keywords": [],
  "author": "",
  "license": "ISC",
  "description": "",
  "dependencies": {
    "@fastify/cors": "^11.0.1",
    "@fastify/jwt": "^9.1.0",
    "dotenv": "^16.4.7",
    "fastify": "^5.2.2",
    "fastify-cors": "^6.1.0",
    "fastify-jwt": "^4.2.0"
  }
}

```

1.9 Run the Backend

 

```bash
node src/index.js
```
    Visit:
    ➡️ http://localhost:3000/ → Should return { message: "Fastify backend is running!" }
    ➡️ POST http://localhost:3000/api/users/login with { "username": "admin", "password": "password" } should return a JWT token.

-------------


# Step 2: Build the Vanilla JS Frontend

- Now, let’s create a simple frontend.

## 2.1  Create Frontend Directory 

```bash
mkdir -p ../fastify-frontend && cd ../fastify-frontend
touch index.html script.js
```

## 2.2 Project Structure
```bash
fastify-frontend/
│── index.html
│── script.js
```
## 2.3 Create index.html

- Edit index.html:

```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fastify Frontend</title>
</head>
<body>
    <h1>Fastify + Vanilla JS</h1>
    <button onclick="login()">Login</button>
    <button onclick="getProfile()">Get Profile</button>
    <pre id="output"></pre>

    <script src="script.js"></script>
</body>
</html>
```
## 2.4 Create script.js

- Edit script.js:

```bash
const API_URL = 'http://localhost:3000/api/users';
let token = '';

async function login() {
    const response = await fetch(`${API_URL}/login`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username: 'admin', password: 'password' })
    });
    const data = await response.json();
    token = data.token;
    document.getElementById('output').innerText = JSON.stringify(data, null, 2);
}

async function getProfile() {
    const response = await fetch(`${API_URL}/profile`, {
        headers: { 'Authorization': `Bearer ${token}` }
    });
    const data = await response.json();
    document.getElementById('output').innerText = JSON.stringify(data, null, 2);
}
```

