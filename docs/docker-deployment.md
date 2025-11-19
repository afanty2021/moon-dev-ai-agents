# Docker 容器化部署指南

## 概述

本指南详细介绍如何使用 Docker 容器化部署 Moon Dev AI Agents 系统。容器化部署提供了环境一致性、可移植性、可扩展性和简化的运维管理。

## 目录

1. [Docker 架构概览](#docker-架构概览)
2. [容器化设计原则](#容器化设计原则)
3. [Dockerfile 配置](#dockerfile-配置)
4. [Docker Compose 配置](#docker-compose-配置)
5. [多阶段构建](#多阶段构建)
6. [环境变量管理](#环境变量管理)
7. [数据持久化](#数据持久化)
8. [网络配置](#网络配置)
9. [监控和日志](#监控和日志)
10. [生产环境最佳实践](#生产环境最佳实践)

## Docker 架构概览

### 容器化架构图

```mermaid
graph TB
    subgraph "负载均衡层"
        LB[Nginx/HAProxy]
    end

    subgraph "应用层"
        APP1[moon-dev-app-1]
        APP2[moon-dev-app-2]
        APP3[moon-dev-app-3]
    end

    subgraph "服务层"
        REDIS[Redis Cache]
        POSTGRES[PostgreSQL]
        PROMETHEUS[Prometheus]
        GRAFANA[Grafana]
    end

    subgraph "数据层"
        DB_DATA[(PostgreSQL Data)]
        REDIS_DATA[(Redis Data)]
        LOGS[(Log Data)]
    end

    LB --> APP1
    LB --> APP2
    LB --> APP3

    APP1 --> REDIS
    APP1 --> POSTGRES
    APP2 --> REDIS
    APP2 --> POSTGRES
    APP3 --> REDIS
    APP3 --> POSTGRES

    PROMETHEUS --> APP1
    PROMETHEUS --> APP2
    PROMETHEUS --> APP3
    GRAFANA --> PROMETHEUS

    POSTGRES --> DB_DATA
    REDIS --> REDIS_DATA
    APP1 --> LOGS
    APP2 --> LOGS
    APP3 --> LOGS
```

### 服务组件

| 服务 | 镜像 | 端口 | 描述 |
|------|------|------|------|
| moon-dev-app | moon-dev-ai-agents:latest | 8000 | 主应用服务 |
| nginx | nginx:alpine | 80, 443 | 负载均衡和反向代理 |
| redis | redis:alpine | 6379 | 缓存和会话存储 |
| postgres | postgres:15 | 5432 | 主数据库 |
| prometheus | prometheus:latest | 9090 | 监控数据收集 |
| grafana | grafana/grafana:latest | 3000 | 监控可视化 |

## 容器化设计原则

### 1. 单一职责原则

每个容器只负责一个特定的功能：

```dockerfile
# 主应用容器 - 只运行 Moon Dev AI Agents
FROM python:3.9-slim

# 数据库容器 - 只运行 PostgreSQL
FROM postgres:15

# 缓存容器 - 只运行 Redis
FROM redis:alpine
```

### 2. 无状态设计

应用容器设计为无状态，便于水平扩展：

```python
# src/config.py
class Config:
    # 配置存储在环境变量或外部配置服务中
    REDIS_URL = os.getenv('REDIS_URL', 'redis://redis:6379')
    DATABASE_URL = os.getenv('DATABASE_URL', 'postgresql://user:pass@postgres:5432/moondev')

    # 会话存储在 Redis 中
    SESSION_TYPE = 'redis'
    SESSION_REDIS = redis.from_url(REDIS_URL)
```

### 3. 健康检查

为所有容器定义健康检查：

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:8000/health')" || exit 1
```

## Dockerfile 配置

### 主应用 Dockerfile

```dockerfile
# Dockerfile
FROM python:3.9-slim as base

# 设置工作目录
WORKDIR /app

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    curl \
    && rm -rf /var/lib/apt/lists/*

# 创建非 root 用户
RUN groupadd -r moondev && useradd -r -g moondev moondev

# 多阶段构建 - 构建阶段
FROM base as builder

# 复制依赖文件
COPY requirements.txt .
COPY requirements-dev.txt .

# 安装 Python 依赖
RUN pip install --no-cache-dir --user -r requirements.txt

# 生产阶段
FROM base as production

# 复制已安装的包
COPY --from=builder /root/.local /home/moondev/.local

# 设置 Python 路径
ENV PYTHONPATH=/home/moondev/.local/lib/python3.9/site-packages
ENV PATH=/home/moondev/.local/bin:$PATH

# 复制应用代码
COPY --chown=moondev:moondev . /app

# 切换到非 root 用户
USER moondev

# 暴露端口
EXPOSE 8000

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# 启动命令
CMD ["python", "src/main.py"]
```

### 开发环境 Dockerfile

```dockerfile
# Dockerfile.dev
FROM python:3.9-slim

WORKDIR /app

# 安装开发和运行时依赖
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    git \
    vim \
    curl \
    && rm -rf /var/lib/apt/lists/*

# 安装 Python 依赖
COPY requirements.txt .
COPY requirements-dev.txt .
RUN pip install --no-cache-dir -r requirements-dev.txt

# 复制代码
COPY . .

# 开发环境变量
ENV PYTHONPATH=/app
ENV FLASK_ENV=development
ENV DEBUG=true

# 暴露端口
EXPOSE 8000 5000

# 开发模式启动
CMD ["python", "src/main.py", "--debug"]
```

### 测试环境 Dockerfile

```dockerfile
# Dockerfile.test
FROM python:3.9-slim as base

WORKDIR /app

# 安装测试依赖
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    git \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
COPY requirements-dev.txt .
RUN pip install --no-cache-dir -r requirements-dev.txt

# 测试阶段
FROM base as test

COPY . .

# 运行测试
CMD ["python", "-m", "pytest", "tests/", "-v", "--cov=src", "--cov-report=html"]
```

## Docker Compose 配置

### 开发环境配置

`docker-compose.dev.yml`:
```yaml
version: '3.8'

services:
  # 主应用服务
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    container_name: moon-dev-app
    ports:
      - "8000:8000"
    volumes:
      - .:/app
      - /app/__pycache__
    environment:
      - REDIS_URL=redis://redis:6379/0
      - DATABASE_URL=postgresql://postgres:password@postgres:5432/moondev_dev
      - LOG_LEVEL=DEBUG
    depends_on:
      - redis
      - postgres
    networks:
      - moon-dev-network
    restart: unless-stopped

  # Redis 缓存
  redis:
    image: redis:alpine
    container_name: moon-dev-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    networks:
      - moon-dev-network
    restart: unless-stopped

  # PostgreSQL 数据库
  postgres:
    image: postgres:15
    container_name: moon-dev-postgres
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=moondev_dev
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init_db.sql:/docker-entrypoint-initdb.d/init_db.sql
    networks:
      - moon-dev-network
    restart: unless-stopped

  # Jupyter Notebook (开发工具)
  jupyter:
    image: jupyter/scipy-notebook
    container_name: moon-dev-jupyter
    ports:
      - "8888:8888"
    volumes:
      - ./notebooks:/home/jovyan/work
    environment:
      - JUPYTER_ENABLE_LAB=yes
    networks:
      - moon-dev-network
    restart: unless-stopped

volumes:
  redis_data:
  postgres_data:

networks:
  moon-dev-network:
    driver: bridge
```

### 生产环境配置

`docker-compose.prod.yml`:
```yaml
version: '3.8'

services:
  # Nginx 负载均衡器
  nginx:
    image: nginx:alpine
    container_name: moon-dev-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
      - ./logs/nginx:/var/log/nginx
    depends_on:
      - app
    networks:
      - moon-dev-network
    restart: unless-stopped

  # 主应用服务 (多实例)
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: moon-dev-ai-agents:latest
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1.0'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 1G
    environment:
      - REDIS_URL=redis://redis:6379/0
      - DATABASE_URL=postgresql://postgres:${POSTGRES_PASSWORD}@postgres:5432/moondev
      - LOG_LEVEL=INFO
      - PROMETHEUS_ENABLED=true
    depends_on:
      - redis
      - postgres
    networks:
      - moon-dev-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  # Redis 集群
  redis:
    image: redis:alpine
    container_name: moon-dev-redis
    command: redis-server --appendonly yes --maxmemory 512mb --maxmemory-policy allkeys-lru
    volumes:
      - redis_data:/data
      - ./redis/redis.conf:/usr/local/etc/redis/redis.conf
    networks:
      - moon-dev-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 3s
      retries: 3

  # PostgreSQL 主数据库
  postgres:
    image: postgres:15
    container_name: moon-dev-postgres
    environment:
      - POSTGRES_DB=moondev
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./postgres/postgresql.conf:/etc/postgresql/postgresql.conf
      - ./backups:/backups
    networks:
      - moon-dev-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 30s
      timeout: 5s
      retries: 5

  # Prometheus 监控
  prometheus:
    image: prom/prometheus:latest
    container_name: moon-dev-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=200h'
      - '--web.enable-lifecycle'
    networks:
      - moon-dev-network
    restart: unless-stopped

  # Grafana 可视化
  grafana:
    image: grafana/grafana:latest
    container_name: moon-dev-grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
      - ./grafana/dashboards:/var/lib/grafana/dashboards
    depends_on:
      - prometheus
    networks:
      - moon-dev-network
    restart: unless-stopped

  # 日志聚合
  loki:
    image: grafana/loki:latest
    container_name: moon-dev-loki
    ports:
      - "3100:3100"
    volumes:
      - ./loki/loki.yml:/etc/loki/local-config.yaml
      - loki_data:/loki
    command: -config.file=/etc/loki/local-config.yaml
    networks:
      - moon-dev-network
    restart: unless-stopped

volumes:
  redis_data:
    driver: local
  postgres_data:
    driver: local
  prometheus_data:
    driver: local
  grafana_data:
    driver: local
  loki_data:
    driver: local

networks:
  moon-dev-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

## 多阶段构建

### 优化的 Dockerfile

```dockerfile
# Dockerfile.optimized
FROM python:3.9-slim as base

# 设置构建参数
ARG BUILD_DATE
ARG VCS_REF

# 标签
LABEL maintainer="moon-dev-team@example.com" \
      org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.name="Moon Dev AI Agents" \
      org.label-schema.description="AI-powered trading system" \
      org.label-schema.url="https://github.com/afanty2021/moon-dev-ai-agents" \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.vcs-url="https://github.com/afanty2021/moon-dev-ai-agents.git" \
      org.label-schema.vendor="Moon Dev" \
      org.label-schema.version="1.0.0" \
      org.label-schema.schema-version="1.0"

# 构建阶段
FROM base as builder

# 安装构建依赖
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    libc6-dev \
    && rm -rf /var/lib/apt/lists/*

# 创建虚拟环境
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 升级 pip
RUN pip install --upgrade pip

# 复制并安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 生产阶段
FROM base as production

# 安装运行时依赖
RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*

# 创建应用用户
RUN groupadd -r moondev && useradd -r -g moondev moondev

# 复制虚拟环境
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 设置工作目录
WORKDIR /app

# 复制应用代码
COPY --chown=moondev:moondev . /app

# 创建必要的目录
RUN mkdir -p /app/logs /app/data /app/config && \
    chown -R moondev:moondev /app

# 切换到非 root 用户
USER moondev

# 暴露端口
EXPOSE 8000

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/api/health || exit 1

# 默认环境变量
ENV PYTHONPATH=/app
ENV LOG_LEVEL=INFO

# 启动脚本
COPY docker/entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["python", "src/main.py"]
```

### 启动脚本

`docker/entrypoint.sh`:
```bash
#!/bin/bash
set -e

# 等待依赖服务启动
wait_for_service() {
    local host=$1
    local port=$2
    local service=$3

    echo "等待 $service 启动..."

    while ! nc -z $host $port; do
        echo "等待 $service ($host:$port) 可用..."
        sleep 2
    done

    echo "$service 已启动!"
}

# 等待 Redis
if [ -n "$REDIS_HOST" ]; then
    wait_for_service $REDIS_HOST ${REDIS_PORT:-6379} "Redis"
fi

# 等待 PostgreSQL
if [ -n "$DATABASE_HOST" ]; then
    wait_for_service $DATABASE_HOST ${DATABASE_PORT:-5432} "PostgreSQL"
fi

# 运行数据库迁移
if [ "$RUN_MIGRATIONS" = "true" ]; then
    echo "运行数据库迁移..."
    python scripts/migrate.py
fi

# 收集静态文件
if [ "$COLLECT_STATIC" = "true" ]; then
    echo "收集静态文件..."
    python scripts/collect_static.py
fi

# 启动应用
echo "启动 Moon Dev AI Agents..."
exec "$@"
```

## 环境变量管理

### 环境变量模板

`.env.template`:
```bash
# 应用配置
APP_NAME=moon-dev-ai-agents
APP_VERSION=1.0.0
DEBUG=false
LOG_LEVEL=INFO

# 数据库配置
DATABASE_HOST=postgres
DATABASE_PORT=5432
DATABASE_NAME=moondev
DATABASE_USER=postgres
DATABASE_PASSWORD=your_secure_password_here
DATABASE_URL=postgresql://postgres:${DATABASE_PASSWORD}@postgres:5432/moondev

# Redis 配置
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_DB=0
REDIS_PASSWORD=
REDIS_URL=redis://redis:6379/0

# API 配置
ANTHROPIC_KEY=your_anthropic_key_here
OPENAI_KEY=your_openai_key_here
GROQ_API_KEY=your_groq_key_here
GEMINI_KEY=your_gemini_key_here
DEEPSEEK_KEY=your_deepseek_key_here

# 交易配置
EXCHANGE=solana
USD_SIZE=100
MAX_POSITION_PERCENTAGE=30
LEVERAGE=1
STOP_LOSS_PERCENTAGE=5.0
TAKE_PROFIT_PERCENTAGE=5.0

# 安全配置
SECRET_KEY=your_very_secure_secret_key_here
JWT_SECRET_KEY=your_jwt_secret_key_here
SESSION_SECRET_KEY=your_session_secret_key_here

# 监控配置
PROMETHEUS_ENABLED=true
PROMETHEUS_PORT=8000
GRAFANA_PASSWORD=your_grafana_password_here

# 外部服务
BIRDEYE_API_KEY=your_birdeye_key_here
MOONDEV_API_KEY=your_moondev_key_here
COINGECKO_API_KEY=your_coingecko_key_here

# 网络配置
NGINX_PORT=80
NGINX_SSL_PORT=443
SSL_CERT_PATH=/etc/nginx/ssl/cert.pem
SSL_KEY_PATH=/etc/nginx/ssl/key.pem

# 备份配置
BACKUP_ENABLED=true
BACKUP_SCHEDULE=0 2 * * *
BACKUP_RETENTION_DAYS=30

# 性能配置
WORKERS=4
MAX_REQUESTS=1000
MAX_REQUESTS_JITTER=100
PRELOAD_APP=true
```

### 生产环境配置

`docker/.env.prod`:
```bash
# 生产环境配置
DEBUG=false
LOG_LEVEL=WARNING

# 数据库连接池
DATABASE_POOL_SIZE=20
DATABASE_MAX_OVERFLOW=30
DATABASE_POOL_TIMEOUT=30

# Redis 连接池
REDIS_POOL_SIZE=10
REDIS_MAX_CONNECTIONS=50

# 性能优化
WORKERS=4
MAX_REQUESTS=1000
MAX_REQUESTS_JITTER=100
PRELOAD_APP=true

# 安全配置
SECURE_SSL_REDIRECT=true
SECURE_HSTS_SECONDS=31536000
SECURE_CONTENT_TYPE_NOSNIFF=true
SECURE_BROWSER_XSS_FILTER=true

# 监控
PROMETHEUS_ENABLED=true
SENTRY_DSN=your_sentry_dsn_here
```

## 数据持久化

### 数据卷配置

```yaml
# docker-compose.volumes.yml
version: '3.8'

volumes:
  # PostgreSQL 数据
  postgres_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /data/postgres

  # Redis 数据
  redis_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /data/redis

  # 应用数据
  app_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /data/app

  # 日志数据
  logs_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /data/logs

  # 备份数据
  backups_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /data/backups

  # 配置数据
  config_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /data/config
```

### 数据备份脚本

`scripts/backup.sh`:
```bash
#!/bin/bash

# 备份配置
BACKUP_DIR="/data/backups"
DATE=$(date +%Y%m%d_%H%M%S)
POSTGRES_CONTAINER="moon-dev-postgres"
REDIS_CONTAINER="moon-dev-redis"

# 创建备份目录
mkdir -p $BACKUP_DIR

# PostgreSQL 备份
echo "开始 PostgreSQL 备份..."
docker exec $POSTGRES_CONTAINER pg_dump -U postgres moondev > $BACKUP_DIR/postgres_backup_$DATE.sql

# Redis 备份
echo "开始 Redis 备份..."
docker exec $REDIS_CONTAINER redis-cli BGSAVE
docker cp $REDIS_CONTAINER:/data/dump.rdb $BACKUP_DIR/redis_backup_$DATE.rdb

# 压缩备份文件
echo "压缩备份文件..."
tar -czf $BACKUP_DIR/backup_$DATE.tar.gz $BACKUP_DIR/*_backup_$DATE.*

# 清理临时文件
rm $BACKUP_DIR/*_backup_$DATE.*

# 删除旧备份 (保留30天)
find $BACKUP_DIR -name "backup_*.tar.gz" -mtime +30 -delete

echo "备份完成: backup_$DATE.tar.gz"
```

## 网络配置

### 自定义网络

```yaml
# docker-compose.network.yml
version: '3.8'

networks:
  # 前端网络 (负载均衡器到应用)
  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.1.0/24
          gateway: 172.20.1.1

  # 后端网络 (应用到数据库)
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.2.0/24
          gateway: 172.20.2.1

  # 监控网络
  monitoring:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.3.0/24
          gateway: 172.20.3.1

  # 数据网络
  data:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.4.0/24
          gateway: 172.20.4.1
```

### Nginx 配置

`nginx/nginx.conf`:
```nginx
upstream moon_dev_app {
    least_conn;
    server app:8000 max_fails=3 fail_timeout=30s;
    # 如果有多个应用实例
    # server app_1:8000 max_fails=3 fail_timeout=30s;
    # server app_2:8000 max_fails=3 fail_timeout=30s;
}

server {
    listen 80;
    server_name api.moondev.ai;

    # 重定向到 HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.moondev.ai;

    # SSL 配置
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    ssl_prefer_server_ciphers off;

    # 安全头
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";

    # 日志配置
    access_log /var/log/nginx/api_access.log;
    error_log /var/log/nginx/api_error.log;

    # 限制请求大小
    client_max_body_size 10M;

    # API 代理
    location /api/ {
        proxy_pass http://moon_dev_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 超时配置
        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;

        # 缓冲配置
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
    }

    # 健康检查
    location /health {
        proxy_pass http://moon_dev_app;
        access_log off;
    }

    # 静态文件
    location /static/ {
        alias /app/static/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

## 监控和日志

### Prometheus 配置

`prometheus/prometheus.yml`:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "rules/*.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  # Prometheus 自监控
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Moon Dev 应用监控
  - job_name: 'moon-dev-app'
    static_configs:
      - targets: ['app:8000']
    metrics_path: '/metrics'
    scrape_interval: 30s
    scrape_timeout: 10s

  # Redis 监控
  - job_name: 'redis'
    static_configs:
      - targets: ['redis:6379']

  # PostgreSQL 监控
  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres:5432']

  # Nginx 监控
  - job_name: 'nginx'
    static_configs:
      - targets: ['nginx:80']
```

### 日志配置

`docker/logging.py`:
```python
import logging
import sys
from typing import Any, Dict

import structlog


def configure_logging(log_level: str = "INFO") -> None:
    """配置结构化日志。"""

    # 配置 structlog
    structlog.configure(
        processors=[
            structlog.stdlib.filter_by_level,
            structlog.stdlib.add_logger_name,
            structlog.stdlib.add_log_level,
            structlog.stdlib.PositionalArgumentsFormatter(),
            structlog.processors.TimeStamper(fmt="iso"),
            structlog.processors.StackInfoRenderer(),
            structlog.processors.format_exc_info,
            structlog.processors.UnicodeDecoder(),
            structlog.processors.JSONRenderer(),
        ],
        context_class=dict,
        logger_factory=structlog.stdlib.LoggerFactory(),
        wrapper_class=structlog.stdlib.BoundLogger,
        cache_logger_on_first_use=True,
    )

    # 配置标准库 logging
    logging.basicConfig(
        format="%(message)s",
        stream=sys.stdout,
        level=getattr(logging, log_level.upper()),
    )


class StructuredLogger:
    """结构化日志记录器。"""

    def __init__(self, name: str) -> None:
        self.logger = structlog.get_logger(name)

    def info(self, message: str, **kwargs: Any) -> None:
        """记录信息日志。"""
        self.logger.info(message, **kwargs)

    def error(self, message: str, **kwargs: Any) -> None:
        """记录错误日志。"""
        self.logger.error(message, **kwargs)

    def warning(self, message: str, **kwargs: Any) -> None:
        """记录警告日志。"""
        self.logger.warning(message, **kwargs)

    def debug(self, message: str, **kwargs: Any) -> None:
        """记录调试日志。"""
        self.logger.debug(message, **kwargs)

    def trading_event(
        self,
        event_type: str,
        symbol: str,
        action: str,
        amount: float,
        price: float,
        **kwargs: Any
    ) -> None:
        """记录交易事件。"""
        self.logger.info(
            "Trading event",
            event_type=event_type,
            symbol=symbol,
            action=action,
            amount=amount,
            price=price,
            **kwargs
        )


# 使用示例
logger = StructuredLogger(__name__)
logger.info("Application started", version="1.0.0", environment="production")
logger.trading_event("order_placed", "BTC-USDT", "BUY", 0.1, 45000.0, order_id="12345")
```

## 生产环境最佳实践

### 1. 安全配置

```dockerfile
# 安全加固的 Dockerfile
FROM python:3.9-slim as base

# 移除不必要的软件包
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get autoremove -y \
    && apt-get clean

# 创建非 root 用户
RUN groupadd -r moondev && useradd -r -g moondev moondev

# 设置文件权限
COPY --chown=moondev:moondev . /app
RUN chmod -R 755 /app \
    && chmod -R 644 /app/config/*

# 切换到非 root 用户
USER moondev
```

### 2. 资源限制

```yaml
# docker-compose.resources.yml
version: '3.8'

services:
  app:
    deploy:
      resources:
        # 资源限制
        limits:
          cpus: '1.0'
          memory: 2G
          pids: 100
        # 资源预留
        reservations:
          cpus: '0.5'
          memory: 1G
    # 重启策略
    restart: unless-stopped
    # 健康检查
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### 3. 自动扩展

`docker-compose.scale.yml`:
```yaml
version: '3.8'

services:
  app:
    image: moon-dev-ai-agents:latest
    environment:
      - REDIS_URL=redis://redis:6379/0
      - DATABASE_URL=postgresql://postgres:${POSTGRES_PASSWORD}@postgres:5432/moondev
    depends_on:
      - redis
      - postgres
    networks:
      - moon-dev-network
    restart: unless-stopped

  # 使用 docker-compose up --scale app=3 来扩展应用实例
```

### 4. 滚动更新

```bash
#!/bin/bash
# scripts/rolling-update.sh

set -e

NEW_IMAGE=$1
if [ -z "$NEW_IMAGE" ]; then
    echo "使用方法: $0 <new-image-tag>"
    exit 1
fi

echo "开始滚动更新到 $NEW_IMAGE"

# 拉取新镜像
docker pull $NEW_IMAGE

# 更新服务
docker-compose -f docker-compose.prod.yml up -d --no-deps app

# 等待健康检查
echo "等待服务健康检查..."
sleep 30

# 验证更新
curl -f http://localhost:8000/health || {
    echo "健康检查失败，回滚..."
    docker-compose -f docker-compose.prod.yml rollback
    exit 1
}

echo "滚动更新完成!"
```

### 5. 部署脚本

`scripts/deploy.sh`:
```bash
#!/bin/bash

set -e

# 配置
DOCKER_REGISTRY="your-registry.com"
PROJECT_NAME="moon-dev-ai-agents"
ENVIRONMENT="production"

# 颜色输出
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

log() {
    echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] $1${NC}"
}

warn() {
    echo -e "${YELLOW}[$(date +'%Y-%m-%d %H:%M:%S')] WARNING: $1${NC}"
}

error() {
    echo -e "${RED}[$(date +'%Y-%m-%d %H:%M:%S')] ERROR: $1${NC}"
}

# 检查环境
check_environment() {
    log "检查部署环境..."

    if ! command -v docker &> /dev/null; then
        error "Docker 未安装"
        exit 1
    fi

    if ! command -v docker-compose &> /dev/null; then
        error "Docker Compose 未安装"
        exit 1
    fi

    # 检查环境变量
    if [ ! -f ".env.prod" ]; then
        error ".env.prod 文件不存在"
        exit 1
    fi

    log "环境检查完成"
}

# 构建镜像
build_image() {
    log "构建 Docker 镜像..."

    docker build -t $PROJECT_NAME:latest .
    docker tag $PROJECT_NAME:latest $PROJECT_NAME:$(date +%Y%m%d_%H%M%S)

    log "镜像构建完成"
}

# 运行测试
run_tests() {
    log "运行测试套件..."

    docker-compose -f docker-compose.test.yml up --build --abort-on-container-exit
    test_result=$?

    docker-compose -f docker-compose.test.yml down

    if [ $test_result -ne 0 ]; then
        error "测试失败"
        exit 1
    fi

    log "测试通过"
}

# 部署应用
deploy_application() {
    log "部署应用..."

    # 备份当前版本
    docker-compose -f docker-compose.prod.yml down

    # 部署新版本
    docker-compose -f docker-compose.prod.yml up -d

    log "等待服务启动..."
    sleep 30

    # 健康检查
    if curl -f http://localhost/health; then
        log "应用部署成功"
    else
        error "健康检查失败"
        docker-compose -f docker-compose.prod.yml down
        exit 1
    fi
}

# 清理旧镜像
cleanup() {
    log "清理旧镜像..."

    docker image prune -f
    docker volume prune -f

    log "清理完成"
}

# 主函数
main() {
    log "开始部署 $PROJECT_NAME 到 $ENVIRONMENT 环境"

    check_environment
    build_image
    run_tests
    deploy_application
    cleanup

    log "部署完成!"
}

# 执行主函数
main "$@"
```

---

这个 Docker 容器化部署指南提供了完整的容器化解决方案，包括开发、测试和生产环境的配置，以及监控、日志、安全等最佳实践。通过遵循这个指南，您可以实现 Moon Dev AI Agents 系统的可靠、可扩展的容器化部署。