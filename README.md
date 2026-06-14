# DevOps from Zero to Hero — Notes
**Source:** JS Mastery | YouTube (Sep 2025)  
**Runtime:** ~5 hours | **Stack:** Node.js · Express · Docker · Kubernetes · GitHub Actions · Neon DB · Arcjet

---

## Introduction

- Course covers the **full DevOps lifecycle** — theory + a production API build
- Tech stack used in the practical section: **Node.js + Express**, **Neon (Postgres)**, **Docker**, **Arcjet** (security), **Warp** (terminal)
- Goal: understand DevOps concepts AND ship a real, deployable API

---

## What is DevOps?

### The Problem DevOps Solves
- Traditional teams had a **wall between Dev and Ops**: developers wrote code, operations deployed it — separately, slowly
- Handoffs caused delays, "works on my machine" bugs, and slow releases

### DevOps Definition
> DevOps is a **culture + set of practices** that unifies software development (Dev) and IT operations (Ops) to shorten the development lifecycle and deliver high-quality software continuously.

### Core Pillars
| Pillar | What it means |
|---|---|
| **Automation** | Eliminate manual steps in build, test, deploy |
| **CI/CD** | Continuously integrate and ship code |
| **Collaboration** | Dev + Ops share responsibility |
| **Monitoring** | Observe systems in production |
| **Feedback Loops** | Fast detection → fast fixes |

### DevOps Lifecycle (Infinite Loop)
```
Plan → Code → Build → Test → Release → Deploy → Operate → Monitor → (back to Plan)
```

---

## What You Need to Learn in DevOps

Core skill areas (in order of priority):

1. **Version Control** — Git & GitHub
2. **CI/CD Pipelines** — GitHub Actions, Jenkins
3. **Containerization** — Docker
4. **Orchestration** — Kubernetes
5. **Infrastructure as Code (IaC)** — Terraform, Pulumi
6. **Cloud Providers** — AWS / GCP / Azure
7. **Monitoring & Logging** — Prometheus, Grafana, Datadog
8. **Security (DevSecOps)** — Secrets management, rate limiting, auth

---

## Version Control: Git & GitHub

### Why Git?
- Tracks every change with a **commit history**
- Enables **branching** for feature isolation
- Foundation for all CI/CD pipelines

### Core Git Concepts
```bash
git init                     # Initialize repo
git clone <url>              # Clone remote repo
git add .                    # Stage changes
git commit -m "message"      # Snapshot staged changes
git push origin main         # Push to remote
git pull origin main         # Fetch + merge remote
git branch feature/x         # Create branch
git checkout feature/x       # Switch branch
git merge feature/x          # Merge into current branch
git rebase main              # Rebase onto main
```

### Branching Strategy (Gitflow)
```
main        ← production-ready only
develop     ← integration branch
feature/*   ← individual features
hotfix/*    ← urgent production fixes
release/*   ← pre-release staging
```

### GitHub Features Used
- **Pull Requests** — code review before merge
- **Issues** — task/bug tracking
- **Actions** — CI/CD automation (YAML workflows)
- **Secrets** — store API keys securely for pipelines
- **Environments** — staging vs production deployment targets

### Best Practices
- Write **meaningful commit messages** (`feat:`, `fix:`, `chore:` prefixes — Conventional Commits)
- Never commit secrets or `.env` files
- Use `.gitignore` to exclude `node_modules`, `.env`, build artifacts
- Keep PRs small and focused

---

## CI/CD Pipelines

### What is CI/CD?
- **Continuous Integration (CI):** Every code push triggers automated build + tests. Catches bugs early.
- **Continuous Delivery (CD):** After CI passes, code is automatically deployed to staging/production.

### Why CI/CD?
- Removes "deploy day" anxiety
- Every change is small, tested, and rollback-able
- Faster feedback → faster iteration

### GitHub Actions Anatomy
```yaml
name: CI Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run lint
      - run: npm test
```

### Key Concepts
| Term | Meaning |
|---|---|
| **Workflow** | A YAML file in `.github/workflows/` |
| **Trigger (`on:`)** | What event starts the pipeline (push, PR, schedule) |
| **Job** | A set of steps running on one runner |
| **Step** | Single command or Action |
| **Runner** | The machine that runs the job (`ubuntu-latest`) |
| **Secrets** | Encrypted env vars stored in GitHub Settings |
| **Artifact** | Output files (build, test reports) saved between jobs |

---

## Build Your First Pipeline

### Simple Node.js CI Pipeline
```yaml
# .github/workflows/ci.yml
name: Node.js CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run build --if-present
      - run: npm test
```

### Adding Environment Variables from Secrets
```yaml
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
  JWT_SECRET: ${{ secrets.JWT_SECRET }}
```

---

## Docker

### What is Docker?
Docker **packages your app + its dependencies** into a lightweight, portable **container** that runs identically everywhere.

### Key Concepts
| Concept | Definition |
|---|---|
| **Image** | Blueprint / snapshot of your app (immutable) |
| **Container** | Running instance of an image |
| **Dockerfile** | Instructions to build an image |
| **Docker Hub** | Registry to store/share images |
| **Volume** | Persistent storage mounted into a container |
| **Network** | Virtual network connecting containers |
| **Docker Compose** | Tool to run multi-container apps with one file |

### Dockerfile (Node.js API)
```dockerfile
# Use official Node base image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Copy package files and install deps first (layer caching)
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# Start app
CMD ["node", "src/index.js"]
```

### Common Docker Commands
```bash
docker build -t my-api .           # Build image from Dockerfile
docker run -p 3000:3000 my-api     # Run container, map port
docker ps                          # List running containers
docker stop <id>                   # Stop container
docker rm <id>                     # Remove container
docker images                      # List images
docker rmi <image>                 # Remove image
docker logs <id>                   # View container logs
docker exec -it <id> sh            # Shell into container
```

### Docker Compose (Multi-container)
```yaml
# docker-compose.yml
version: '3.8'
services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
    depends_on:
      - db

  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: password

volumes:
  pgdata:
```

### Best Practices
- Use **multi-stage builds** to keep images small
- Use `.dockerignore` to exclude `node_modules`, `.git`
- Pin image versions (`node:20-alpine`, not `node:latest`)
- Run as **non-root user** in production
- One process per container

### Multi-stage Build (Smaller Production Image)
```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:20-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

---

## Kubernetes

### What is Kubernetes (K8s)?
Kubernetes is a **container orchestration platform** — it automates deploying, scaling, and managing containerized applications across a cluster of machines.

### Why Kubernetes?
- Docker alone: runs containers on one machine
- K8s: runs containers across **many machines**, handles failures, scales automatically

### Core K8s Objects
| Object | Purpose |
|---|---|
| **Pod** | Smallest unit — one or more containers together |
| **Deployment** | Manages pods, handles rollouts + rollbacks |
| **Service** | Stable network endpoint to reach pods |
| **Ingress** | HTTP routing rules (like a load balancer) |
| **ConfigMap** | Non-secret config data |
| **Secret** | Sensitive data (base64-encoded) |
| **Namespace** | Virtual cluster for isolation |
| **Node** | A machine (VM or physical) in the cluster |

### Key Architecture
```
Control Plane (Master)
  ├── API Server       ← everything talks to this
  ├── Scheduler        ← decides which node runs a pod
  ├── etcd             ← cluster state database
  └── Controller Mgr  ← maintains desired state

Worker Nodes
  ├── kubelet          ← talks to API server, manages pods
  ├── kube-proxy       ← handles networking
  └── Container Runtime (Docker/containerd)
```

### Deployment YAML
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-api
  template:
    metadata:
      labels:
        app: my-api
    spec:
      containers:
        - name: my-api
          image: myrepo/my-api:latest
          ports:
            - containerPort: 3000
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: DATABASE_URL
```

### Service YAML
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-api-service
spec:
  selector:
    app: my-api
  ports:
    - port: 80
      targetPort: 3000
  type: LoadBalancer
```

### Essential kubectl Commands
```bash
kubectl get pods                      # List pods
kubectl get deployments               # List deployments
kubectl describe pod <name>           # Debug a pod
kubectl logs <pod-name>               # View logs
kubectl apply -f deployment.yaml      # Apply a config
kubectl delete -f deployment.yaml     # Delete resources
kubectl scale deploy my-api --replicas=5   # Scale
kubectl rollout status deploy/my-api  # Check rollout
kubectl rollout undo deploy/my-api    # Rollback
```

---

## Infrastructure as Code (IaC)

### What is IaC?
Define your **infrastructure (servers, networks, databases) in code** — version-controlled, repeatable, automated.

### Why IaC?
- No more clicking around cloud consoles manually
- Infrastructure is reproducible across environments
- Changes are reviewed via PRs like application code

### Popular IaC Tools
| Tool | Language | Works With |
|---|---|---|
| **Terraform** | HCL | Any cloud |
| **Pulumi** | TypeScript/Python | Any cloud |
| **AWS CDK** | TypeScript | AWS only |
| **Ansible** | YAML | Config management |

### Terraform Example (Provision a VPS)
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "api_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  tags = {
    Name = "my-api-server"
  }
}
```

---

## Practical Section: Build & Deploy an API

### Tech Stack
- **Runtime:** Node.js 20 + Express
- **Database:** Neon (serverless Postgres)
- **ORM:** Drizzle ORM
- **Auth:** JWT + bcrypt
- **Security:** Arcjet (rate limiting, bot protection, shield)
- **Logging:** Pino
- **Linting:** ESLint + Prettier
- **Containerization:** Docker
- **Testing:** Node's built-in test runner
- **Terminal:** Warp

### Project Structure
```
my-api/
├── src/
│   ├── config/         # DB, env config
│   ├── middleware/      # auth, logger, error handler
│   ├── routes/          # route definitions
│   ├── controllers/     # request handlers
│   ├── models/          # DB schemas
│   └── index.js         # Entry point
├── .env
├── .eslintrc
├── .prettierrc
├── Dockerfile
├── docker-compose.yml
└── package.json
```

---

## ESLint & Prettier

### Purpose
- **ESLint** — catches code errors and enforces coding rules
- **Prettier** — auto-formats code consistently

### Setup
```bash
npm install -D eslint prettier eslint-config-prettier
npx eslint --init
```

### `.eslintrc.json`
```json
{
  "env": { "node": true, "es2022": true },
  "extends": ["eslint:recommended", "prettier"],
  "rules": {
    "no-unused-vars": "warn",
    "no-console": "off"
  }
}
```

### `.prettierrc`
```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5"
}
```

### Add to `package.json` scripts
```json
"scripts": {
  "lint": "eslint src/",
  "lint:fix": "eslint src/ --fix",
  "format": "prettier --write src/"
}
```

---

## Database Setup (Neon + Drizzle)

### Neon
- Serverless Postgres — scales to zero, no infrastructure to manage
- Get `DATABASE_URL` from Neon dashboard

### Drizzle ORM Setup
```bash
npm install drizzle-orm @neondatabase/serverless
npm install -D drizzle-kit
```

### Schema Definition (`src/models/users.js`)
```javascript
import { pgTable, serial, text, timestamp } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
  password: text('password').notNull(),
  createdAt: timestamp('created_at').defaultNow(),
});
```

### DB Connection (`src/config/db.js`)
```javascript
import { neon } from '@neondatabase/serverless';
import { drizzle } from 'drizzle-orm/neon-http';

const sql = neon(process.env.DATABASE_URL);
export const db = drizzle(sql);
```

### Migrations
```bash
npx drizzle-kit generate    # Generate SQL migration files
npx drizzle-kit push        # Push schema to DB (dev)
```

---

## Logger & Middleware

### Pino Logger Setup
```bash
npm install pino pino-pretty
```

```javascript
// src/config/logger.js
import pino from 'pino';

export const logger = pino({
  transport: {
    target: 'pino-pretty',
    options: { colorize: true },
  },
});
```

### HTTP Request Logger Middleware
```javascript
// src/middleware/logger.js
import { logger } from '../config/logger.js';

export const requestLogger = (req, res, next) => {
  logger.info({ method: req.method, url: req.url }, 'Incoming request');
  next();
};
```

### Global Error Handler Middleware
```javascript
// src/middleware/errorHandler.js
export const errorHandler = (err, req, res, next) => {
  const status = err.status || 500;
  const message = err.message || 'Internal Server Error';
  res.status(status).json({ success: false, error: message });
};
```

### Registering Middleware
```javascript
// src/index.js
import express from 'express';
import { requestLogger } from './middleware/logger.js';
import { errorHandler } from './middleware/errorHandler.js';

const app = express();
app.use(express.json());
app.use(requestLogger);

// routes here...

app.use(errorHandler);   // Must be last
```

---

## Authentication

### JWT Auth Flow
```
Register → Hash password (bcrypt) → Store in DB
Login    → Verify password → Issue JWT
Protected routes → Verify JWT in Authorization header
```

### Setup
```bash
npm install bcryptjs jsonwebtoken
```

### Auth Controller
```javascript
import bcrypt from 'bcryptjs';
import jwt from 'jsonwebtoken';
import { db } from '../config/db.js';
import { users } from '../models/users.js';
import { eq } from 'drizzle-orm';

export const register = async (req, res, next) => {
  try {
    const { name, email, password } = req.body;
    const hashed = await bcrypt.hash(password, 10);
    const [user] = await db.insert(users).values({ name, email, password: hashed }).returning();
    res.status(201).json({ success: true, user: { id: user.id, email: user.email } });
  } catch (err) {
    next(err);
  }
};

export const login = async (req, res, next) => {
  try {
    const { email, password } = req.body;
    const [user] = await db.select().from(users).where(eq(users.email, email));
    if (!user || !(await bcrypt.compare(password, user.password))) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    const token = jwt.sign({ userId: user.id }, process.env.JWT_SECRET, { expiresIn: '7d' });
    res.json({ success: true, token });
  } catch (err) {
    next(err);
  }
};
```

### Auth Middleware (Protect Routes)
```javascript
// src/middleware/auth.js
import jwt from 'jsonwebtoken';

export const protect = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token provided' });
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
};
```

---

## Security Middleware (Arcjet)

### What is Arcjet?
Arcjet is a **security-as-code SDK** that adds:
- **Rate limiting** — prevent API abuse
- **Bot protection** — block scrapers/bots
- **Shield** — protect against common attacks (SQL injection, XSS)
- **Email validation** — catch disposable/invalid emails on signup

### Setup
```bash
npm install @arcjet/node
```

### Configuration
```javascript
// src/config/arcjet.js
import arcjet, { shield, rateLimit, detectBot } from '@arcjet/node';

export const aj = arcjet({
  key: process.env.ARCJET_KEY,
  rules: [
    shield({ mode: 'LIVE' }),            // Block common attacks
    detectBot({ mode: 'LIVE', allow: [] }),  // Block bots
    rateLimit({
      mode: 'LIVE',
      window: '1m',
      max: 60,                           // 60 requests/minute
    }),
  ],
});
```

### Using Arcjet in Routes
```javascript
import { aj } from '../config/arcjet.js';

export const securityMiddleware = async (req, res, next) => {
  const decision = await aj.protect(req);
  if (decision.isDenied()) {
    return res.status(429).json({ error: 'Too many requests' });
  }
  next();
};
```

---

## Dockerization

### `.dockerignore`
```
node_modules
.env
.git
*.log
dist
```

### Production Dockerfile
```dockerfile
FROM node:20-alpine AS base
WORKDIR /app

FROM base AS deps
COPY package*.json ./
RUN npm ci --only=production

FROM base AS production
COPY --from=deps /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
ENV NODE_ENV=production
CMD ["node", "src/index.js"]
```

### Build & Run
```bash
docker build -t my-api .
docker run -p 3000:3000 --env-file .env my-api
```

### Push to Docker Hub
```bash
docker tag my-api username/my-api:latest
docker push username/my-api:latest
```

### CI/CD: Auto-build + Push on Merge
```yaml
# .github/workflows/docker.yml
name: Build & Push Docker Image

on:
  push:
    branches: [main]

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/my-api:latest
```

---

## User CRUD

### Route Definition
```javascript
// src/routes/users.js
import express from 'express';
import { protect } from '../middleware/auth.js';
import { getUsers, getUserById, updateUser, deleteUser } from '../controllers/users.js';

const router = express.Router();

router.get('/', protect, getUsers);
router.get('/:id', protect, getUserById);
router.put('/:id', protect, updateUser);
router.delete('/:id', protect, deleteUser);

export default router;
```

### Controller Pattern
```javascript
export const getUsers = async (req, res, next) => {
  try {
    const allUsers = await db.select({ id: users.id, name: users.name, email: users.email })
      .from(users);
    res.json({ success: true, data: allUsers });
  } catch (err) {
    next(err);
  }
};

export const updateUser = async (req, res, next) => {
  try {
    const { id } = req.params;
    const { name } = req.body;
    const [updated] = await db.update(users)
      .set({ name })
      .where(eq(users.id, parseInt(id)))
      .returning();
    if (!updated) return res.status(404).json({ error: 'User not found' });
    res.json({ success: true, data: updated });
  } catch (err) {
    next(err);
  }
};
```

---

## Testing

### Node.js Built-in Test Runner (Node 20+)
No extra library needed:

```javascript
// src/tests/auth.test.js
import { test, describe } from 'node:test';
import assert from 'node:assert';

describe('Auth', () => {
  test('returns 400 if email is missing', async () => {
    const res = await fetch('http://localhost:3000/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ password: '123' }),
    });
    assert.strictEqual(res.status, 400);
  });

  test('returns 401 on wrong credentials', async () => {
    const res = await fetch('http://localhost:3000/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email: 'bad@test.com', password: 'wrong' }),
    });
    assert.strictEqual(res.status, 401);
  });
});
```

### Run Tests
```bash
node --test src/tests/         # Run all tests
node --test --watch            # Watch mode
```

### Add to CI Pipeline
```yaml
- run: npm test
```

---

## Next Steps

### Immediate Next Steps
- Add **input validation** (Zod or Joi) to all routes
- Add **refresh tokens** for better auth UX
- Set up **monitoring** — Prometheus + Grafana, or Datadog
- Add **health check endpoint** (`GET /health`)
- Configure **Kubernetes deployment** on a cloud provider

### Production Readiness Checklist
- [ ] All secrets in env vars, never hardcoded
- [ ] Rate limiting enabled (Arcjet or express-rate-limit)
- [ ] Input validation on all POST/PUT routes
- [ ] Error messages don't leak stack traces
- [ ] Logs are structured (Pino JSON output in prod)
- [ ] Docker image uses non-root user
- [ ] CI/CD pipeline runs lint + tests on every PR
- [ ] Database has connection pooling enabled
- [ ] HTTPS enforced (via reverse proxy or Ingress)

### Deeper Learning Paths
- **Docker Deep Dive** → multi-stage builds, security scanning, compose v2
- **Kubernetes** → Helm charts, Horizontal Pod Autoscaler, Ingress controllers
- **Terraform** → provision cloud infra (VPC, RDS, EC2/ECS) as code
- **Monitoring** → Prometheus metrics, Grafana dashboards, alerting
- **DevSecOps** → SAST (CodeQL), secret scanning, SBOM generation

---

*Notes compiled from: "DevOps from Zero to Hero" — JavaScript Mastery (Sep 2025)*
