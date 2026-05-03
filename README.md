# Database Infrastructure Stack

This repository contains the configuration for a centralized database infrastructure running on a Raspberry Pi.

## Architecture Overview

* **Infrastructure:** A single `docker-compose.yml` orchestrating Postgres, Redis, RabbitMQ, and MongoDB.
* **Networking:** Accessible locally and remotely via Tailscale (using Tailscale IP or MagicDNS).

## Logical Isolation (Multi-Environment & Multi-Project)

To maximize resource efficiency on the Raspberry Pi, we use a single instance of each service and enforce **strict logical isolation**:

* **Postgres:** Separate databases (e.g., `splitsy_dev`, `splitsy_prod`).
* **Redis:** Separate database indices (e.g., `0` for dev, `1` for prod).
* **RabbitMQ:** Distinct vhosts.
* **MongoDB:** Separate databases per project/environment. (Future Needs)

Future applications will share this exact infrastructure using the same logical isolation strategy.

## Getting Started

1. Ensure your external SSD is mounted and Docker is configured to use it for volume storage.
2. Connect the Raspberry Pi to your Tailscale network.
3. Run `docker-compose up -d` to start the stack.
