# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Infrastructure-only repository for the HandsUp project's Kafka broker. No application code — only Docker Compose configurations for running Apache Kafka in KRaft mode.

## Commands

```bash
# Local: start broker + kafka-ui
docker compose -f docker-compose.local.yml --env-file .env.local up -d

# Local: stop
docker compose -f docker-compose.local.yml --env-file .env.local down

# Local: stop and delete all data
docker compose -f docker-compose.local.yml --env-file .env.local down -v

# Check broker health
docker inspect --format='{{.State.Health.Status}}' hands-up-kafka
```

## Architecture

- **Image**: `apache/kafka` in KRaft mode (no Zookeeper)
- **Environment separation**: `docker-compose.local.yml` (dev) / `docker-compose.prod.yml` (prod), each with its own `.env.*` file
- **Local listeners**: CONTROLLER(29093), HOST(9092 → host:29092), DOCKER(9093 for kafka-ui)
- **Prod listeners**: CONTROLLER(29093), INTERNAL(9092), EXTERNAL(9094 → host:29092)
- **Local extras**: `provectuslabs/kafka-ui` at http://localhost:8080
- **Prod extras**: memory limits (768M container / 512M heap), `restart: unless-stopped`, health-check verification in CD

## Deployment

Push to `main` triggers `.github/workflows/cd.yml` which SCPs `docker-compose.prod.yml` + `.env.prod` to EC2 and runs `docker compose up -d`. The pipeline includes a health-check verification step (polls for up to 120s).

GitHub Secret `ENV_PROD_CONTENT` contains the full `.env.prod` contents. The `CLUSTER_ID` in prod must never change after first deployment — it is baked into KRaft metadata.

## Key Constraints

- `CLUSTER_ID` is fixed per environment. Changing it makes existing volume data inaccessible.
- `KAFKA_LOG_DIRS` is explicitly set to `/var/lib/kafka/data` (not the default `/tmp/kraft-combined-logs`) to survive container restarts with volume mounts.
- Environment files (`.env`, `.env.local`, `.env.prod`) are gitignored. Only `.example` templates are committed.
