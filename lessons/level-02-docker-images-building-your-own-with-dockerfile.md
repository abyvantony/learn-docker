# Level 2: Docker images: building your own with Dockerfile

**Track:** Docker · **Level:** 2/12 · **Cycle:** 1 · **Difficulty:** `beginner`

📚 **Today's lesson** — published 2026-07-14

## TL;DR

A Dockerfile is a recipe file with instructions to build an image. `docker build` reads it and produces an image. Once built, the image is portable — push it to a registry, pull it elsewhere, run it anywhere.

## Real-world analogies

- A Dockerfile is a recipe card. Ingredients = base image, dependencies, source code. Steps = RUN commands. The finished dish = the image.
- Building an image is like baking bread. The recipe is fixed (Dockerfile), but each time you bake (build), you get a fresh loaf (image) you can store and serve.

## Key concepts

### `Dockerfile syntax`

Each line is an instruction: `FROM`, `RUN`, `COPY`, `CMD`, etc. Read top-to-bottom, executed in order.

### `Layers`

Each instruction creates a layer. Layers are cached — if nothing changed, Docker reuses the cached layer. Makes rebuilds fast.

### ``.dockerignore``

Like `.gitignore` for Docker. Exclude files from the build context. Critical for keeping images small and fast.

### `Tags`

Versions of an image: `myapp:1.0`, `myapp:latest`, `myapp:dev`. Default tag is `latest` (avoid in production).

## Code with comments

Every line has a comment. Read it slowly.

```
# === A simple Dockerfile for a Node.js app ===

# === Create project structure ===
mkdir my-node-app && cd my-node-app
# app.js
# package.json
# Dockerfile
# .dockerignore

# === app.js ===
# const http = require('http');
# const server = http.createServer((req, res) => {
#   res.end('Hello from Docker!');
# });
# server.listen(3000, () => console.log('Listening on 3000'));

# === package.json ===
# {
#   "name": "my-node-app",
#   "version": "1.0.0",
#   "main": "app.js",
#   "scripts": { "start": "node app.js" }
# }

# === .dockerignore (IMPORTANT — keep image small) ===
# node_modules
# npm-debug.log
# .git
# .gitignore
# .env
# .DS_Store
# *.md
# coverage
# dist
# .vscode
# .idea

# === Dockerfile ===
# # Use an official Node.js base image — Alpine is tiny (50MB vs 350MB)
# FROM node:20-alpine
#
# # Set the working directory inside the container
# WORKDIR /app
#
# # Copy package files first (leverages Docker layer caching)
# COPY package*.json ./
#
# # Install dependencies
# RUN npm install --production
#
# # Copy the rest of the source code
# COPY . .
#
# # Document which port the app uses (documentation only — doesn't publish)
# EXPOSE 3000
#
# # Run the app
# CMD ["node", "app.js"]

# === Build the image ===
docker build -t my-node-app:1.0 .
# -t = tag (name:version)
# . = build context (current directory)

# === Run it ===
docker run -d --name my-app -p 8080:3000 my-node-app:1.0
# -p 8080:3000 = publish container's port 3000 as host's port 8080
# Visit http://localhost:8080

# === Inspect the image ===
docker images my-node-app          # Size, ID, created
docker history my-node-app:1.0     # Each layer, what created it
docker inspect my-node-app:1.0     # Full JSON

# === Tag for pushing to a registry ===
docker tag my-node-app:1.0 yourusername/my-node-app:1.0

# === Push to Docker Hub ===
docker login                        # One-time
docker push yourusername/my-node-app:1.0

# === Pull from anywhere ===
docker pull yourusername/my-node-app:1.0

# === Multi-stage build (smaller images) ===
# Useful when you have a build step (TypeScript, webpack, etc.)
# === Dockerfile.multi-stage ===
# # Stage 1: build
# FROM node:20-alpine AS builder
# WORKDIR /app
# COPY package*.json ./
# RUN npm ci
# COPY . .
# RUN npm run build
#
# # Stage 2: production (only built files + production deps)
# FROM node:20-alpine
# WORKDIR /app
# COPY package*.json ./
# RUN npm ci --only=production
# COPY --from=builder /app/dist ./dist
# EXPOSE 3000
# CMD ["node", "dist/main.js"]

# === Build with specific Dockerfile ===
docker build -f Dockerfile.multi-stage -t my-app:prod .

# === Dockerfile instructions cheat sheet ===
# FROM <image>           — base image
# WORKDIR <path>         — cd into this directory
# COPY <src> <dest>      — copy files from build context
# ADD <src> <dest>       — like COPY but also handles URLs and tarballs
# RUN <command>          — run a command during build
# CMD ["executable", "arg"] — default command when container starts
# ENTRYPOINT [...]       — always run this (CMD becomes args)
# ENV <key>=<value>      — set environment variable
# ARG <name>[=default]   — build-time variable
# EXPOSE <port>          — document which port is used
# VOLUME <path>          — declare a mount point
# USER <user>            — run as this user (security)
# HEALTHCHECK ...        — how to check if container is healthy
# LABEL <key>=<value>    — metadata
```

## Try it yourself

Take a simple app you've built (any language — Node, Python, PHP, Go). Write a Dockerfile for it. Build it. Run it. Tag it `yourname/yourapp:1.0`. Push to Docker Hub. Pull on a different machine (or remove and re-pull to test).

## Common pitfalls

- ⚠️ Forgetting `.dockerignore` — sends your entire directory (including `node_modules`, `.git`, secrets) to the daemon. Slow + huge images + leaks secrets.
- ⚠️ Using `latest` tag in production. `latest` is mutable — your 'latest' today isn't tomorrow's. Always pin versions.
- ⚠️ `COPY . .` before `npm install` — invalidates the cache every time you change any file. Copy package.json first, install, then copy source.
- ⚠️ Running as root inside the container (the default). Use `USER node` or create a non-root user for security.

## What's next?

**Level 3: Networking and volumes: making containers talk and persist data** — coming in the next drop.

---

_Generated by Hermes · Aby's learning cron · Track: Docker · Level 2 · Cycle 1_