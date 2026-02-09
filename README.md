# HandsUp Kafka Broker

HandsUp 프로젝트의 Kafka 브로커 인프라. Apache Kafka KRaft 모드 사용 (Zookeeper 불필요).

## Prerequisites

- Docker Engine 24+
- Docker Compose v2

## Local Development

```bash
# 1. 환경변수 파일 생성 (최초 1회)
cp .env.local.example .env.local

# 2. 브로커 + Kafka UI 시작
docker compose -f docker-compose.local.yml --env-file .env.local up -d

# 3. Kafka UI 접속
# http://localhost:8080

# 4. 애플리케이션 연결
# bootstrap-servers: localhost:29092

# 5. 중지
docker compose -f docker-compose.local.yml --env-file .env.local down

# 데이터까지 삭제하려면
docker compose -f docker-compose.local.yml --env-file .env.local down -v
```

## Production

main 브랜치에 push 시 GitHub Actions CD 파이프라인이 자동으로 EC2에 배포.

### GitHub Secrets 설정 필요

| Secret | 설명 |
|--------|------|
| `ENV_PROD_CONTENT` | .env.prod 파일 내용 |
| `EC2_HOST` | EC2 퍼블릭 IP |
| `EC2_USERNAME` | EC2 SSH 사용자 |
| `EC2_SSH_PRIVATE_KEY` | EC2 SSH 키 |
| `EC2_PORT` | EC2 SSH 포트 |

## Architecture

- **Image**: `apache/kafka` (KRaft 모드)
- **Local**: CONTROLLER + HOST(9092) + DOCKER(9093) 리스너, kafka-ui 포함
- **Production**: CONTROLLER + INTERNAL(9092) + EXTERNAL(9094) 리스너, 메모리 제한
