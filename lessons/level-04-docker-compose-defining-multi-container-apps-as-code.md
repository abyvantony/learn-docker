# Level 4: Docker Compose: defining multi-container apps as code

**Track:** Docker · **Level:** 4/12 · **Cycle:** 1 · **Difficulty:** `intermediate`

📚 **Today's lesson** — published 2026-07-15

## TL;DR

Docker Compose lets you describe your whole app — all its services, networks, volumes, environment variables — in a single YAML file. One command brings the whole stack up or down. The standard tool for local dev and small deployments.

## Real-world analogies

- Docker Compose is a conductor's score. Each musician (container) plays their part. The conductor (Compose) starts them all, keeps them in sync, stops them when done.
- A docker-compose.yml is like a recipe for a multi-course meal. Each dish is a service. The shopping list is the volumes. The kitchen layout is the network.

## Key concepts

### `Compose v2`

Modern Compose is a Docker CLI plugin: `docker compose` (no hyphen). Old syntax: `docker-compose`. Always use v2.

### `Services`

Each service = one container. You define the image (or build context), ports, environment, volumes, networks, dependencies.

### `Depends_on`

Tells Compose to start services in order. Doesn't wait for the service to be *ready* (just for the container to start). Use healthchecks for true readiness.

### `Profiles`

Group services that should only start in certain situations (e.g., `dev` profile for debugging tools, `prod` for production-only services).

## Code with comments

Every line has a comment. Read it slowly.

```
# === docker-compose.yml for a Laravel app ===

# version: '3.8'  # No longer needed in Compose v2

# services:
#   app:
#     build:
#       context: .
#       dockerfile: Dockerfile
#     container_name: laravel-app
#     volumes:
#       - ./src:/var/www/html          # Live code reload
#     ports:
#       - "8000:80"
#     environment:
#       - DB_HOST=db
#       - DB_DATABASE=laravel
#       - DB_USERNAME=laravel
#       - DB_PASSWORD=secret
#     depends_on:
#       db:
#         condition: service_healthy
#     networks:
#       - laravel-net
#
#   db:
#     image: mysql:8
#     container_name: laravel-db
#     volumes:
#       - db-data:/var/lib/mysql
#     environment:
#       - MYSQL_ROOT_PASSWORD=root-secret
#       - MYSQL_DATABASE=laravel
#       - MYSQL_USER=laravel
#       - MYSQL_PASSWORD=secret
#     healthcheck:
#       test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
#       interval: 10s
#       timeout: 5s
#       retries: 5
#     networks:
#       - laravel-net
#
#   redis:
#     image: redis:7-alpine
#     container_name: laravel-redis
#     networks:
#       - laravel-net
#
# volumes:
#   db-data:
#
# networks:
#   laravel-net:
#     driver: bridge

# === Common commands ===
# docker compose up              # Start in foreground
# docker compose up -d           # Start detached (background)
# docker compose down            # Stop and remove
# docker compose down -v         # Also remove volumes (CAREFUL)
# docker compose ps              # List running services
# docker compose logs            # All logs
# docker compose logs -f app     # Follow app logs
# docker compose logs --tail=100 app
# docker compose exec app bash   # Shell into a running service
# docker compose exec db mysql -uroot -proot-secret
# docker compose run --rm app php artisan migrate  # One-off command
# docker compose build           # Build images
# docker compose build --no-cache
# docker compose pull            # Pull latest images
# docker compose restart         # Restart all services
# docker compose stop            # Stop without removing
# docker compose start           # Start stopped containers
# docker compose config          # Validate + show final config

# === Healthchecks (so depends_on can wait for "ready") ===
# healthcheck:
#   test: ["CMD", "curl", "-f", "http://localhost"]  # Command to run
#   interval: 30s      # How often
#   timeout: 10s       # Per-test timeout
#   retries: 3         # Fail N times = unhealthy
#   start_period: 40s  # Grace period on startup

# === Override files for different environments ===

# docker-compose.override.yml (auto-merged, dev overrides)
# services:
#   app:
#     environment:
#       - APP_DEBUG=true
#     volumes:
#       - ./src:/var/www/html  # Live reload in dev

# docker-compose.prod.yml
# services:
#   app:
#     environment:
#       - APP_DEBUG=false

# Start with: docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# === Profiles (start conditionally) ===
# services:
#   app:
#     build: .
#   debug-tools:
#     image: portainer/portainer
#     profiles: ["debug"]    # Only starts with: docker compose --profile debug up
#     ports:
#       - "9000:9000"

# === Environment files ===
# services:
#   app:
#     env_file:
#       - .env
#       - .env.local

# === Resource limits (prevent one service eating all RAM) ===
# services:
#   app:
#     deploy:
#       resources:
#         limits:
#           cpus: '0.5'
#           memory: 512M
#         reservations:
#           cpus: '0.25'
#           memory: 256M
```

## Try it yourself

Take a multi-container project you've worked on (or use WordPress + MySQL from level 3). Convert it to docker-compose.yml. Add a database with healthcheck. Add a Redis service. Use profiles to add an adminer (database GUI) under the 'debug' profile.

## Common pitfalls

- ⚠️ Using the old `docker-compose` (with hyphen) — deprecated. Always use `docker compose` (v2, plugin).
- ⚠️ `depends_on` without `condition: service_healthy` — your app starts before the DB is ready, then crashes. Add healthchecks.
- ⚠️ Committing `.env` files with secrets. Use `.env.example` for the template, add `.env` to `.gitignore`.
- ⚠️ Forgetting `-d` on `docker compose up` — your terminal blocks forever. Use `up -d` for background.

## What's next?

**Level 5: Production best practices: slim images, security, layers** — coming in the next drop.

---

_Generated by Hermes · Aby's learning cron · Track: Docker · Level 4 · Cycle 1_