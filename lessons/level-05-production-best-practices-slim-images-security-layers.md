# Level 5: Production best practices: slim images, security, layers

**Track:** Docker · **Level:** 5/12 · **Cycle:** 1 · **Difficulty:** `intermediate`

📚 **Today's lesson** — published 2026-07-17

## TL;DR

Production Docker is different from dev Docker. Smaller images, non-root users, pinned versions, healthchecks, restart policies, resource limits. Get these right and your deployments are stable and fast.

## Real-world analogies

- A development container is a workshop — messy, full of tools, comfortable. A production container is a delivery truck — minimal, only what's needed, secure.
- Image layers are like geological strata. Each `RUN` adds a layer. Order them so the heavy, slow-changing layers are at the bottom, and the frequently-changing ones at the top. Caching makes rebuilds fast.

## Key concepts

### `Image size matters`

Smaller = faster pulls, faster deploys, smaller attack surface. Use Alpine, multi-stage builds, `.dockerignore`.

### `Layer caching`

Docker caches each layer. Order Dockerfile instructions from least-changed to most-changed. Dependencies before source code.

### `Non-root user`

By default, processes run as root inside containers. Switch to a non-root user for security — if compromised, attacker can't escape easily.

### `Pinned versions`

`node:latest` is a moving target. Use `node:20.11.1-alpine` for reproducible builds. Same image hash every time.

## Code with comments

Every line has a comment. Read it slowly.

```
# === Slim image checklist ===

# 1. Use Alpine or distroless base
# node:20-alpine = ~50MB compressed
# node:20-slim = ~80MB (Debian-based, more compatible)
# node:20 = ~350MB (full Debian, avoid for production)

# 2. Multi-stage builds (separate build from runtime)
# === Dockerfile.production ===
# # Build stage
# FROM node:20-alpine AS builder
# WORKDIR /app
# COPY package*.json ./
# RUN npm ci
# COPY . .
# RUN npm run build  # Compiles to /app/dist
#
# # Runtime stage — only what we need
# FROM node:20-alpine
# WORKDIR /app
# COPY package*.json ./
# RUN npm ci --only=production && npm cache clean --force
# COPY --from=builder /app/dist ./dist
# USER node  # Run as non-root
# EXPOSE 3000
# CMD ["node", "dist/main.js"]

# 3. Non-root user
# Add to Dockerfile:
# RUN addgroup -S app && adduser -S app -G app
# USER app

# Or use built-in (node image has 'node' user, alpine has 'nobody')
# USER node

# 4. .dockerignore (CRITICAL)
# node_modules
# npm-debug.log
# .git
# .gitignore
# .env
# .env.*
# !.env.example
# .vscode
# .idea
# coverage
# *.log
# README.md
# Dockerfile
# .dockerignore
# .git
# .DS_Store
# dist
# build

# 5. Healthcheck
# HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
#   CMD node healthcheck.js || exit 1

# 6. Pin versions (never use :latest in production)
# FROM node:20.11.1-alpine
# Or use digests for absolute pinning:
# FROM node:20.11.1-alpine@sha256:abc123...

# === Image inspection ===
docker images --format "{{.Repository}}:{{.Tag}}	{{.Size}}"
# Find the 10 biggest images
docker images --format "{{.Size}}	{{.Repository}}:{{.Tag}}" | sort -h | tail -10

# Analyze image layers
dive my-app:1.0
# Install: https://github.com/wagoodman/dive
# Interactive tool to see what's in each layer

# Or use docker history
docker history my-app:1.0 --no-trunc --format "{{.CreatedBy}}"

# === Image optimization tools ===

# Use a smaller base
docker-slim build my-app:1.0
# Installs: https://github.com/docker-slim/docker-slim
# Automatically optimizes image size

# Check for vulnerabilities
docker scout cves my-app:1.0
# Or use Trivy:
trivy image my-app:1.0
# Or Snyk: snyk container test my-app:1.0

# === Security checklist ===
# - Use official base images (FROM node, not FROM some-user/node)
# - Pin versions (no :latest)
# - Use non-root user
# - Don't put secrets in Dockerfile (use env vars, secrets management)
# - Scan images for CVEs
# - Use multi-stage builds (don't ship dev tools)
# - Set --read-only filesystem when possible
# - Drop capabilities: --cap-drop=ALL --cap-add=NET_BIND_SERVICE
# - Use --security-opt=no-new-privileges
# - Sign images: docker trust sign my-app:1.0

# === Production-ready run example ===
# docker run -d \
#   --name my-app \
#   --restart unless-stopped \
#   --read-only \
#   --tmpfs /tmp \
#   --tmpfs /run \
#   --memory 512m \
#   --cpus 0.5 \
#   --cap-drop ALL \
#   --cap-add NET_BIND_SERVICE \
#   --security-opt no-new-privileges \
#   -p 8080:3000 \
#   -e NODE_ENV=production \
#   my-app:1.0

# === Restart policies ===
# docker run --restart=always ...
# no              — never restart
# on-failure      — restart only if exit code != 0
# unless-stopped  — restart unless you manually stopped it (DEFAULT for prod)
# always          — always restart, even after daemon restart

# === Logging ===
# By default, Docker logs to a JSON file. Configure log driver:
docker run --log-driver=json-file --log-opt max-size=10m --log-opt max-file=3 ...
# Or send to syslog, journald, fluentd, etc.
```

## Try it yourself

Take one of your existing Dockerfiles. Rewrite it as a multi-stage build. Add a non-root user. Pin all base image versions. Run `trivy` or `docker scout` against it. Compare image size before and after.

## Common pitfalls

- ⚠️ Using `:latest` — you'll get a different image on every pull. Pin to a specific version.
- ⚠️ Running as root — the default. Easy to forget to add `USER`. A single vulnerability in your app = full container compromise.
- ⚠️ Building a single-stage Dockerfile for production — ships npm, dev dependencies, build tools. Multi-stage is essential.
- ⚠️ Not using `.dockerignore` — leaks `.env` files, git history, build artifacts into the image. Audit your build context size with `docker build -t test . 2>&1 | grep 'transferring context'`.

## What's next?

**Level 6: Docker networking deep dive: bridge, host, overlay, and DNS** — coming in the next drop.

---

_Generated by Hermes · Aby's learning cron · Track: Docker · Level 5 · Cycle 1_