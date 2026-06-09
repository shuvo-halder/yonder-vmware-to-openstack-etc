# Yonder Platform Installation Guide

## Overview

Yonder is an AI-powered infrastructure and cloud management platform।

By following this guide, you can easily deploy Yonder using Docker Compose.

---

# System Requirements

## Minimum

* CPU: 2 Core
* RAM: 4 GB
* Storage: 20 GB SSD
* Ubuntu 22.04 LTS / Debian 12
* Docker 24+
* Docker Compose v2+

## Recommended

* CPU: 4+ Core
* RAM: 8 GB+
* Storage: 50 GB SSD
* Public IP Address
* Domain Name (Optional)

---

# Install Docker

```bash
curl -fsSL https://get.docker.com | bash

systemctl enable docker
systemctl start docker

docker --version
docker compose version
```

---

# Create Working Directory

```bash
mkdir -p /opt/yonder
cd /opt/yonder
```

---

# Create Environment File

```bash
nano .env
```

Paste:

```env
# Version
YONDER_VERSION=0.11.0

# Django
DJANGO_SETTINGS_MODULE=yonder_platform.settings.production
SECRET_KEY=CHANGE_ME_LONG_RANDOM_STRING
DEBUG=False

# URL
SITE_URL=http://SERVER_IP:8077
ALLOWED_HOSTS=*

# Database
POSTGRES_DB=yonder
POSTGRES_USER=yonder
POSTGRES_PASSWORD=StrongPassword123

# Redis
REDIS_URL=redis://localhost:6379/0

# Security
FERNET_KEY=CHANGE_ME_FERNET_KEY
```

---

# Generate Secret Key

```bash
python3 -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

Example:

```env
SECRET_KEY=your_generated_secret_key
```

---

# Generate Fernet Key

```bash
python3 -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
```

Example:

```env
FERNET_KEY=your_generated_fernet_key
```

---

# Create Docker Compose File

Create:

```bash
nano docker-compose.yml
```

Paste:

```yaml
version: "3.9"

services:

  db:
    image: postgres:16
    container_name: yonder-db
    restart: unless-stopped

    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

    volumes:
      - postgres_data:/var/lib/postgresql/data

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: yonder-redis
    restart: unless-stopped

    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    image: ecintelligence/yonder:${YONDER_VERSION}
    container_name: yonder-backend
    restart: unless-stopped

    env_file:
      - .env

    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy

    network_mode: host

volumes:
  postgres_data:
```

---

# Pull Latest Image

```bash
docker pull ecintelligence/yonder:${YONDER_VERSION}
```

---

# Start Yonder

```bash
docker compose up -d
```

Check status:

```bash
docker compose ps
```

---

# View Logs

Backend logs:

```bash
docker logs -f yonder-backend
```

Database logs:

```bash
docker logs -f yonder-db
```

Redis logs:

```bash
docker logs -f yonder-redis
```

---

# Verify Containers

```bash
docker ps
```

Expected:

```text
yonder-backend
yonder-db
yonder-redis
```

---

# Access Web UI

Open browser:

```text
http://SERVER_IP:8077
```

Example:

```text
http://103.131.144.102:8077
```

---

# Restart Services

```bash
docker compose restart
```

Restart backend only:

```bash
docker restart yonder-backend
```

---

# Update Yonder

Pull latest image:

```bash
docker pull ecintelligence/yonder:${YONDER_VERSION}
```

Recreate container:

```bash
docker compose down
docker compose up -d
```

---

# Backup Database

```bash
docker exec yonder-db pg_dump -U yonder yonder > backup.sql
```

Restore:

```bash
cat backup.sql | docker exec -i yonder-db psql -U yonder yonder
```

---

# Troubleshooting

## Backend Keeps Restarting

Check logs:

```bash
docker logs yonder-backend
```

Common reasons:

* Invalid SECRET_KEY
* Invalid FERNET_KEY
* PostgreSQL not reachable
* Redis not reachable

---

## Database Connection Error

Check database:

```bash
docker exec -it yonder-db psql -U yonder
```

---

## Redis Connection Error

```bash
docker exec -it yonder-redis redis-cli ping
```

Expected:

```text
PONG
```

---

## Port Already In Use

Check:

```bash
ss -tulpn | grep 8077
```

Kill process:

```bash
kill -9 PID
```

---

# Remove Everything

⚠️ Warning: This deletes all data.

```bash
docker compose down -v
```

Remove images:

```bash
docker rmi ecintelligence/yonder:${YONDER_VERSION}
```

---

# Useful Commands

```bash
docker compose ps
docker compose logs -f
docker compose restart
docker compose down
docker compose up -d
docker system prune -f
```

---

# Support

If deployment fails:

```bash
docker compose logs > logs.txt
```

Collect:

* docker-compose.yml
* .env
* logs.txt

and share them with the support team for investigation.
