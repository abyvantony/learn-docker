# Level 3: Networking and volumes: making containers talk and persist data

**Track:** Docker · **Level:** 3/12 · **Cycle:** 1 · **Difficulty:** `beginner-plus`

📚 **Today's lesson** — published 2026-07-15

## TL;DR

Containers are isolated by default — they have their own network and filesystem. To make them useful, you need to expose ports, connect them to each other, and persist data outside the container. Volumes and networks are how.

## Real-world analogies

- A container's filesystem is like a hotel room — anything you leave behind when you check out is gone. A Docker volume is your personal storage locker in the basement — your stuff stays even when the room is cleaned.
- A Docker network is like a phone system. Containers on the same network can call each other by name. Containers on different networks can't.

## Key concepts

### `Port mapping (`-p`)`

Maps a host port to a container port. `-p 8080:80` means 'send host port 8080 traffic to container port 80'.

### `Volume vs bind mount`

Volumes are managed by Docker (stored in `/var/lib/docker/volumes`). Bind mounts link a host directory to a container directory. Use volumes for production data, bind mounts for development.

### `Docker networks`

Virtual networks containers attach to. Default is `bridge` (containers can talk to each other by name). Use `host` for max performance, `none` for total isolation.

### `Container DNS`

On a user-defined network, containers can resolve each other by name (not by IP). `web` container can hit `db:5432` directly.

## Code with comments

Every line has a comment. Read it slowly.

```
# === Port mapping ===
docker run -d --name web -p 8080:80 nginx
# Host port 8080 → Container port 80
# Visit http://localhost:8080

# Multiple ports
docker run -d --name app -p 8080:80 -p 8443:443 my-app

# Bind to specific host IP
docker run -d -p 127.0.0.1:8080:80 nginx  # Only accessible locally

# === Volumes: named and anonymous ===

# Named volume (Docker manages it)
docker volume create my-data
docker run -d --name db -v my-data:/var/lib/postgresql/data postgres
# Data persists in my-data even if container is removed

# Anonymous volume (Docker auto-names it)
docker run -d --name app -v /app/data my-app
# Volume gets a random hash. Avoid — hard to manage.

# Bind mount (link a host directory)
docker run -d --name dev -v $(pwd)/src:/app/src my-app
# Edit files on host, see changes in container immediately

# Read-only bind mount (security)
docker run -d -v $(pwd)/config:/app/config:ro my-app

# === Inspect volumes ===
docker volume ls                    # List all volumes
docker volume inspect my-data       # Details
docker volume prune                 # Remove unused (CAREFUL)

# === Networks ===

# List networks
docker network ls

# Create a user-defined network
docker network create my-net

# Run containers on the same network
docker run -d --name db --network my-net -e POSTGRES_PASSWORD=secret postgres
docker run -d --name app --network my-net -p 8080:3000 my-app

# Now 'app' can reach 'db' by name: postgres://db:5432

# Inspect network (see which containers are attached)
docker network inspect my-net

# Connect an existing container
docker network connect my-web other-container

# Disconnect
docker network disconnect my-net container

# === Real-world example: WordPress + MySQL ===
# Create a network
docker network create wp-net

# Start MySQL
docker run -d \
  --name mysql \
  --network wp-net \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=wordpress \
  -v mysql-data:/var/lib/mysql \
  mysql:8

# Start WordPress (it finds 'mysql' by name)
docker run -d \
  --name wordpress \
  --network wp-net \
  -p 8080:80 \
  -e WORDPRESS_DB_HOST=mysql \
  -e WORDPRESS_DB_USER=root \
  -e WORDPRESS_DB_PASSWORD=secret \
  -e WORDPRESS_DB_NAME=wordpress \
  -v wp-data:/var/www/html \
  wordpress

# === Cleanup ===
docker stop $(docker ps -aq)        # Stop all
docker rm $(docker ps -aq)          # Remove all
docker network prune                # Remove unused networks
docker volume prune                 # Remove unused volumes
```

## Try it yourself

Build a simple 2-container app: a Node.js API on one container, a Postgres database on another. Connect them via a user-defined network. Use a named volume for the database. Verify the API can talk to the DB by name. Stop and remove the containers — verify the data persists in the volume.

## Common pitfalls

- ⚠️ Using the default `bridge` network — containers can only reach each other by IP, not by name. Always create a user-defined network for multi-container apps.
- ⚠️ Putting important data inside the container filesystem — gone when the container is removed. Use volumes.
- ⚠️ `docker volume prune` without thinking — removes all unused volumes. If you forgot to attach a container, the volume is 'unused'.
- ⚠️ Bind mounting `node_modules` — breaks if the host OS is different from the container's. Let the container install its own.

## What's next?

**Level 4: Docker Compose: defining multi-container apps as code** — coming in the next drop.

---

_Generated by Hermes · Aby's learning cron · Track: Docker · Level 3 · Cycle 1_