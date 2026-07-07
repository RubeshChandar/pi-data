# Database Infrastructure Stack

This repository contains the configuration for a centralized database infrastructure running on a Raspberry Pi.

## Architecture Overview

- **Infrastructure:** A single `docker-compose.yml` orchestrating Postgres, Redis, and MongoDB.
- **Role:** This is a **shared data layer only** — no application code runs here. All apps (regardless of which machine or repo they live in) connect to these three DB instances remotely via the Pi's IP.
- **Networking:** Accessible locally and remotely via Tailscale (using Tailscale IP or MagicDNS).

## Logical Isolation (Multi-Environment & Multi-Project)

To maximize resource efficiency on the Raspberry Pi, we use a single instance of each service and enforce **strict logical isolation**:

- **Postgres:** Separate databases (e.g., `splitsy_dev`, `splitsy_prod`).
- **Redis:** Separate database indices (e.g., `0` for dev, `1` for prod).
- **MongoDB:** Separate databases per project/environment.

Future applications will share this exact infrastructure using the same logical isolation strategy.

## Getting Started

1. Ensure your external SSD is mounted and Docker is configured to use it for volume storage.
2. Connect the Raspberry Pi to your Tailscale network.
3. Run `docker compose up -d` to start the stack.

---

## Connecting to Services

> **Pi LAN IP:** `192.168.68.71`

> [!IMPORTANT]
> Apps running in their **own Docker Compose file** (even on the same Pi) are on a separate network and **cannot use container names** like `postgres` or `redis`. Always use the Pi's LAN IP (`192.168.68.71`) as the host in your app's connection strings. `host.docker.internal` does not work on Linux/Pi.

---

### PostgreSQL

| | Value |
|---|---|
| Port | `5432` |
| User | `admin` |
| Password | `password` |
| Default DB | `defaultdb` |

**From outside the Pi (your Mac / another machine / your apps):**
```bash
psql -h 192.168.68.71 -p 5432 -U admin -d defaultdb
```

**From the Pi host itself (not inside a container):**
```bash
psql -h localhost -p 5432 -U admin -d defaultdb
```

**pgAdmin UI:** http://192.168.68.71:5050
- Email: `admin@admin.com`
- Password: `password`

---

### Redis

| | Value |
|---|---|
| Port | `6379` |

**From outside the Pi:**
```bash
redis-cli -h 192.168.68.71 -p 6379 ping
# → PONG
```

**Without redis-cli installed (using Docker):**
```bash
docker run --rm redis:7-alpine redis-cli -h 192.168.68.71 -p 6379 ping
```

**From the Pi host itself (not inside a container):**
```bash
redis-cli -h localhost -p 6379 ping
```

**Select a database index (for isolation):**
```bash
redis-cli -h 192.168.68.71 -p 6379
> SELECT 0   # dev
> SELECT 1   # prod
```

---

### MongoDB

| | Value |
|---|---|
| Port | `27017` |
| Root User | `admin` |
| Root Password | `password` |

**From outside the Pi (your Mac / your apps):**
```bash
mongosh --host 192.168.68.71 --port 27017 -u admin -p password
```

**Without mongosh installed (using Docker):**
```bash
docker run --rm mongo:7 mongosh --host 192.168.68.71 -u admin -p password --eval "db.adminCommand('ping')"
```

**From the Pi host itself (not inside a container):**
```bash
mongosh --host localhost --port 27017 -u admin -p password
```

**Connection string format:**
```
mongodb://admin:password@192.168.68.71:27017/
```

---

## Verifying All Services Are Healthy

Run from the Pi or your Mac (replace IP with `localhost` if on the Pi):

```bash
# Postgres
pg_isready -h 192.168.68.71 -p 5432 -U admin

# Redis
redis-cli -h 192.168.68.71 -p 6379 ping

# MongoDB
docker run --rm mongo:7 mongosh --host 192.168.68.71 -u admin -p password --eval "db.adminCommand('ping')" --quiet
```

Or check Docker health status directly on the Pi:
```bash
docker ps --format "table {{.Names}}\t{{.Status}}"
```
