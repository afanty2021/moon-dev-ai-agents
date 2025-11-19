# 生产环境部署和监控指南

## 概述

本指南详细介绍 Moon Dev AI Agents 系统在生产环境中的部署、监控、运维和最佳实践。生产环境部署需要考虑高可用性、安全性、性能优化和可维护性。

## 目录

1. [生产环境架构](#生产环境架构)
2. [基础设施要求](#基础设施要求)
3. [高可用部署](#高可用部署)
4. [监控体系](#监控体系)
5. [日志管理](#日志管理)
6. [安全配置](#安全配置)
7. [性能优化](#性能优化)
8. [备份策略](#备份策略)
9. [灾难恢复](#灾难恢复)
10. [运维最佳实践](#运维最佳实践)

## 生产环境架构

### 多层架构设计

```mermaid
graph TB
    subgraph "负载均衡层"
        LB1[Load Balancer 1]
        LB2[Load Balancer 2]
    end

    subgraph "Web服务层"
        WEB1[Web Server 1]
        WEB2[Web Server 2]
        WEB3[Web Server 3]
    end

    subgraph "应用服务层"
        APP1[App Server 1]
        APP2[App Server 2]
        APP3[App Server 3]
        APP4[App Server 4]
    end

    subgraph "缓存层"
        REDIS_CLUSTER[Redis Cluster]
        REDIS_SENTINEL[Redis Sentinel]
    end

    subgraph "数据库层"
        DB_MASTER[(PostgreSQL Master)]
        DB_SLAVE1[(PostgreSQL Slave 1)]
        DB_SLAVE2[(PostgreSQL Slave 2)]
    end

    subgraph "监控层"
        PROMETHEUS[Prometheus Cluster]
        GRAFANA[Grafana Cluster]
        ALERTMANAGER[AlertManager]
    end

    LB1 --> WEB1
    LB1 --> WEB2
    LB2 --> WEB2
    LB2 --> WEB3

    WEB1 --> APP1
    WEB1 --> APP2
    WEB2 --> APP2
    WEB2 --> APP3
    WEB3 --> APP3
    WEB3 --> APP4

    APP1 --> REDIS_CLUSTER
    APP2 --> REDIS_CLUSTER
    APP3 --> REDIS_CLUSTER
    APP4 --> REDIS_CLUSTER

    APP1 --> DB_MASTER
    APP2 --> DB_MASTER
    APP3 --> DB_SLAVE1
    APP4 --> DB_SLAVE2

    PROMETHEUS --> APP1
    PROMETHEUS --> APP2
    PROMETHEUS --> APP3
    PROMETHEUS --> APP4
    GRAFANA --> PROMETHEUS
    ALERTMANAGER --> PROMETHEUS
```

### 服务组件

| 层级 | 组件 | 数量 | 用途 |
|------|------|------|------|
| 负载均衡 | Nginx/HAProxy | 2 | 流量分发、SSL终端 |
| Web服务 | Nginx | 3 | 静态文件服务、反向代理 |
| 应用服务 | Moon Dev App | 4+ | 核心业务逻辑 |
| 缓存 | Redis Cluster | 6 | 数据缓存、会话存储 |
| 数据库 | PostgreSQL | 3 | 主从复制、读写分离 |
| 监控 | Prometheus/Grafana | 各2 | 系统监控、告警 |

## 基础设施要求

### 最低硬件配置

| 服务 | CPU | 内存 | 存储 | 网络 |
|------|-----|------|------|------|
| 负载均衡器 | 2核 | 4GB | 50GB SSD | 1Gbps |
| Web服务器 | 4核 | 8GB | 100GB SSD | 1Gbps |
| 应用服务器 | 8核 | 16GB | 100GB SSD | 1Gbps |
| Redis节点 | 4核 | 8GB | 50GB SSD | 1Gbps |
| PostgreSQL主库 | 8核 | 32GB | 500GB SSD | 1Gbps |
| PostgreSQL从库 | 4核 | 16GB | 500GB SSD | 1Gbps |
| 监控服务器 | 4核 | 8GB | 200GB SSD | 1Gbps |

### 推荐配置

```yaml
# infrastructure/terraform/variables.tf
variable "instance_types" {
  description = "EC2 instance types for different services"
  type = object({
    load_balancer = string
    web_server = string
    app_server = string
    redis = string
    postgresql_master = string
    postgresql_slave = string
    monitoring = string
  })
  default = {
    load_balancer = "t3.large"
    web_server = "t3.xlarge"
    app_server = "t3.2xlarge"
    redis = "t3.large"
    postgresql_master = "r6g.2xlarge"
    postgresql_slave = "r6g.xlarge"
    monitoring = "t3.large"
  }
}
```

### 网络配置

```bash
# 网络分段配置
# 管理网络：10.0.1.0/24
# 应用网络：10.0.2.0/24
# 数据网络：10.0.3.0/24
# 监控网络：10.0.4.0/24

# 防火墙规则
# 允许外部流量：80, 443
# 允许内部通信：全部端口
# 拒绝其他所有流量
```

## 高可用部署

### 负载均衡配置

`nginx/nginx-ha.conf`:
```nginx
upstream moon_dev_backend {
    least_conn;
    server app1:8000 max_fails=3 fail_timeout=30s weight=1;
    server app2:8000 max_fails=3 fail_timeout=30s weight=1;
    server app3:8000 max_fails=3 fail_timeout=30s weight=1;
    server app4:8000 max_fails=3 fail_timeout=30s weight=1;
}

upstream moon_dev_web {
    server web1:80 max_fails=3 fail_timeout=30s;
    server web2:80 max_fails=3 fail_timeout=30s;
    server web3:80 max_fails=3 fail_timeout=30s;
}

# HTTP 重定向到 HTTPS
server {
    listen 80;
    server_name api.moondev.ai;
    return 301 https://$server_name$request_uri;
}

# HTTPS 主配置
server {
    listen 443 ssl http2;
    server_name api.moondev.ai;

    # SSL 配置
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # 安全头
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options DENY always;
    add_header X-Content-Type-Options nosniff always;
    add_header X-XSS-Protection "1; mode=block" always;

    # 限制请求
    client_max_body_size 10M;
    client_body_timeout 60s;
    client_header_timeout 60s;

    # API 代理
    location /api/ {
        proxy_pass http://moon_dev_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;

        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
    }

    # 静态文件
    location /static/ {
        proxy_pass http://moon_dev_web;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # 健康检查
    location /health {
        proxy_pass http://moon_dev_backend/api/health;
        access_log off;
    }
}

# 限流配置
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

server {
    listen 443 ssl http2;
    server_name api.moondev.ai;

    location /api/trade/ {
        limit_req zone=api_limit burst=20 nodelay;
        proxy_pass http://moon_dev_backend;
    }
}
```

### 数据库高可用

`postgresql/postgresql.conf`:
```ini
# 连接配置
listen_addresses = '*'
port = 5432
max_connections = 200
superuser_reserved_connections = 3

# 内存配置
shared_buffers = 8GB
effective_cache_size = 24GB
work_mem = 64MB
maintenance_work_mem = 1GB

# WAL 配置
wal_level = replica
max_wal_senders = 3
max_replication_slots = 3
wal_keep_segments = 64
archive_mode = on
archive_command = 'cp %p /backup/archive/%f'

# 检查点配置
checkpoint_completion_target = 0.9
checkpoint_timeout = 15min
wal_buffers = 16MB

# 查询优化
random_page_cost = 1.1
effective_io_concurrency = 200
```

`postgresql/pg_hba.conf`:
```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# 本地连接
local   all             postgres                                peer
local   all             all                                     md5

# IPv4 本地连接
host    all             all             127.0.0.1/32            md5

# 内网连接
host    all             all             10.0.0.0/8              md5

# 复制连接
host    replication     replicator      10.0.3.0/24             md5
```

### Redis 高可用集群

`redis/redis-cluster.conf`:
```conf
# 基本配置
port 7000
cluster-enabled yes
cluster-config-file nodes-7000.conf
cluster-node-timeout 5000
appendonly yes
appendfilename "appendonly-7000.aof"

# 内存配置
maxmemory 2gb
maxmemory-policy allkeys-lru

# 持久化配置
save 900 1
save 300 10
save 60 10000

# 网络配置
bind 0.0.0.0
protected-mode no
tcp-keepalive 300

# 日志配置
loglevel notice
logfile /var/log/redis/redis-7000.log
```

## 监控体系

### Prometheus 配置

`prometheus/prometheus.yml`:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'moon-dev-prod'
    region: 'us-west-2'

# 规则文件
rule_files:
  - "rules/*.yml"

# 告警管理器
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

# 抓取配置
scrape_configs:
  # Prometheus 自监控
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # 应用服务监控
  - job_name: 'moon-dev-app'
    static_configs:
      - targets:
        - 'app1:8000'
        - 'app2:8000'
        - 'app3:8000'
        - 'app4:8000'
    metrics_path: '/metrics'
    scrape_interval: 30s
    scrape_timeout: 10s

  # 数据库监控
  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']

  # Redis 监控
  - job_name: 'redis'
    static_configs:
      - targets:
        - 'redis-exporter:9121'

  # 系统监控
  - job_name: 'node'
    static_configs:
      - targets:
        - 'node-exporter:9100'

  # Nginx 监控
  - job_name: 'nginx'
    static_configs:
      - targets: ['nginx-exporter:9113']
```

### 告警规则

`prometheus/rules/moon-dev.yml`:
```yaml
groups:
  - name: moon-dev-alerts
    rules:
      # 应用服务告警
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }} on {{ $labels.instance }}"

      - alert: HighResponseTime
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High response time detected"
          description: "95th percentile response time is {{ $value }}s on {{ $labels.instance }}"

      - alert: ServiceDown
        expr: up{job="moon-dev-app"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service is down"
          description: "{{ $labels.instance }} has been down for more than 1 minute"

      # 数据库告警
      - alert: DatabaseConnectionsHigh
        expr: pg_stat_activity_count > 150
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High database connections"
          description: "Database has {{ $value }} active connections"

      - alert: DatabaseReplicationLag
        expr: pg_replication_lag_seconds > 30
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Database replication lag"
          description: "Replication lag is {{ $value }} seconds"

      # 系统资源告警
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage"
          description: "CPU usage is {{ $value }}% on {{ $labels.instance }}"

      - alert: HighMemoryUsage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory usage is {{ $value }}% on {{ $labels.instance }}"

      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100 < 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Low disk space"
          description: "Disk space is {{ $value }}% full on {{ $labels.instance }}"
```

### Grafana 仪表板

`grafana/dashboards/moon-dev-overview.json`:
```json
{
  "dashboard": {
    "title": "Moon Dev AI Agents Overview",
    "tags": ["moon-dev", "production"],
    "timezone": "browser",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m])) by (service)",
            "legendFormat": "{{service}}"
          }
        ],
        "yAxes": [
          {
            "label": "Requests/sec"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status=~\"5..\"}[5m])) by (service) / sum(rate(http_requests_total[5m])) by (service)",
            "legendFormat": "{{service}}"
          }
        ],
        "yAxes": [
          {
            "label": "Error Rate",
            "max": 1,
            "min": 0
          }
        ]
      },
      {
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.50, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))",
            "legendFormat": "50th percentile - {{service}}"
          },
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))",
            "legendFormat": "95th percentile - {{service}}"
          }
        ],
        "yAxes": [
          {
            "label": "Response Time (s)"
          }
        ]
      },
      {
        "title": "Active Trading Positions",
        "type": "stat",
        "targets": [
          {
            "expr": "trading_active_positions_total"
          }
        ]
      },
      {
        "title": "Daily P&L",
        "type": "graph",
        "targets": [
          {
            "expr": "increase(trading_pnl_total[24h])",
            "legendFormat": "Daily P&L"
          }
        ]
      }
    ],
    "time": {
      "from": "now-1h",
      "to": "now"
    },
    "refresh": "5s"
  }
}
```

## 日志管理

### 集中式日志架构

```yaml
# docker-compose.logging.yml
version: '3.8'

services:
  # 日志收集
  fluentd:
    image: fluent/fluentd:v1.14
    container_name: moon-dev-fluentd
    volumes:
      - ./fluentd/fluent.conf:/fluentd/etc/fluent.conf
      - /var/log:/var/log:ro
    ports:
      - "24224:24224"
    networks:
      - monitoring

  # 日志存储
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    container_name: moon-dev-elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - monitoring

  # 日志可视化
  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.0
    container_name: moon-dev-kibana
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
    networks:
      - monitoring

volumes:
  elasticsearch_data:

networks:
  monitoring:
    external: true
```

### Fluentd 配置

`fluentd/fluent.conf`:
```xml
<source>
  @type tail
  path /var/log/moon-dev/*.log
  pos_file /var/log/fluentd/moon-dev.log.pos
  tag moon-dev.*
  format json
  time_key timestamp
  time_format %Y-%m-%dT%H:%M:%S.%NZ
</source>

# 解析应用日志
<filter moon-dev.**>
  @type parser
  key_name message
  reserve_data true
  remove_key_name_field true
  <parse>
    @type json
  </parse>
</filter>

# 添加环境信息
<filter moon-dev.**>
  @type record_transformer
  <record>
    environment ${ENVIRONMENT}
    cluster ${CLUSTER_NAME}
  </record>
</filter>

# 输出到 Elasticsearch
<match moon-dev.**>
  @type elasticsearch
  host elasticsearch
  port 9200
  index_name moon-dev-logs-%Y.%m.%d
  type_name _doc

  <buffer>
    @type file
    path /var/log/fluentd/buffer
    flush_mode interval
    flush_interval 5s
    chunk_limit_size 2M
    queue_limit_length 8
    overflow_action block
  </buffer>
</match>
```

## 安全配置

### SSL/TLS 配置

```bash
#!/bin/bash
# scripts/setup-ssl.sh

DOMAIN="api.moondev.ai"
SSL_DIR="/etc/nginx/ssl"

# 创建 SSL 目录
mkdir -p $SSL_DIR

# 生成私钥
openssl genrsa -out $SSL_DIR/key.pem 4096

# 生成证书签名请求
openssl req -new -key $SSL_DIR/key.pem -out $SSL_DIR/cert.csr -subj "/C=US/ST=State/L=City/O=Organization/CN=$DOMAIN"

# 生成自签名证书（开发环境）
openssl x509 -req -days 365 -in $SSL_DIR/cert.csr -signkey $SSL_DIR/key.pem -out $SSL_DIR/cert.pem

# 生产环境使用 Let's Encrypt
if [ "$ENVIRONMENT" = "production" ]; then
    certbot certonly --webroot -w /var/www/html -d $DOMAIN --email admin@moondev.ai --agree-tos --non-interactive
    cp /etc/letsencrypt/live/$DOMAIN/fullchain.pem $SSL_DIR/cert.pem
    cp /etc/letsencrypt/live/$DOMAIN/privkey.pem $SSL_DIR/key.pem
fi

# 设置权限
chmod 600 $SSL_DIR/key.pem
chmod 644 $SSL_DIR/cert.pem
```

### 防火墙配置

```bash
#!/bin/bash
# scripts/setup-firewall.sh

# 清除现有规则
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X

# 设置默认策略
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# 允许本地回环
iptables -A INPUT -i lo -j ACCEPT

# 允许已建立的连接
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# 允许 SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 允许 HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# 允许内部网络通信
iptables -A INPUT -s 10.0.0.0/8 -j ACCEPT

# 保存规则
iptables-save > /etc/iptables/rules.v4
```

### 应用安全配置

```python
# src/security/production_config.py
import os
from typing import Dict, Any

class ProductionSecurityConfig:
    """生产环境安全配置。"""

    # JWT 配置
    JWT_SECRET_KEY = os.getenv('JWT_SECRET_KEY')
    JWT_ACCESS_TOKEN_EXPIRES = 3600  # 1小时
    JWT_REFRESH_TOKEN_EXPIRES = 86400  # 24小时

    # CORS 配置
    CORS_ORIGINS = [
        'https://moondev.ai',
        'https://app.moondev.ai'
    ]
    CORS_METHODS = ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS']
    CORS_ALLOW_CREDENTIALS = True

    # 速率限制
    RATE_LIMITS = {
        'default': '100/hour',
        'trading': '10/minute',
        'analysis': '60/hour'
    }

    # 请求大小限制
    MAX_CONTENT_LENGTH = 16 * 1024 * 1024  # 16MB

    # 会话配置
    SESSION_COOKIE_SECURE = True
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'
    PERMANENT_SESSION_LIFETIME = 3600  # 1小时

    # 安全头
    SECURITY_HEADERS = {
        'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
        'X-Frame-Options': 'DENY',
        'X-Content-Type-Options': 'nosniff',
        'X-XSS-Protection': '1; mode=block',
        'Content-Security-Policy': "default-src 'self'"
    }

    @classmethod
    def get_config(cls) -> Dict[str, Any]:
        """获取安全配置字典。"""
        return {
            'JWT_SECRET_KEY': cls.JWT_SECRET_KEY,
            'JWT_ACCESS_TOKEN_EXPIRES': cls.JWT_ACCESS_TOKEN_EXPIRES,
            'CORS_ORIGINS': cls.CORS_ORIGINS,
            'RATE_LIMITS': cls.RATE_LIMITS,
            'MAX_CONTENT_LENGTH': cls.MAX_CONTENT_LENGTH,
            'SESSION_COOKIE_SECURE': cls.SESSION_COOKIE_SECURE,
            'SESSION_COOKIE_HTTPONLY': cls.SESSION_COOKIE_HTTPONLY,
            'SESSION_COOKIE_SAMESITE': cls.SESSION_COOKIE_SAMESITE,
            'SECURITY_HEADERS': cls.SECURITY_HEADERS
        }
```

## 性能优化

### 数据库优化

```sql
-- 数据库性能优化配置
-- postgresql-performance.sql

-- 创建索引
CREATE INDEX CONCURRENTLY idx_trades_symbol_timestamp
ON trades(symbol, timestamp DESC);

CREATE INDEX CONCURRENTLY idx_positions_user_symbol
ON positions(user_id, symbol);

CREATE INDEX CONCURRENTLY idx_market_data_symbol_timestamp
ON market_data(symbol, timestamp DESC);

-- 分区表（如果数据量大）
CREATE TABLE trades_partitioned (
    LIKE trades INCLUDING ALL
) PARTITION BY RANGE (timestamp);

-- 创建月度分区
CREATE TABLE trades_2024_01 PARTITION OF trades_partitioned
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- 统计信息更新
ANALYZE trades;
ANALYZE positions;
ANALYZE market_data;

-- 查询优化配置
ALTER SYSTEM SET shared_preload_libraries = 'pg_stat_statements';
ALTER SYSTEM SET track_activity_query_size = 2048;
ALTER SYSTEM SET log_min_duration_statement = 1000;  # 记录慢查询

SELECT pg_reload_conf();
```

### 应用性能优化

```python
# src/performance/optimizations.py
import asyncio
import functools
from typing import Any, Callable, Dict, List
import redis
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

class PerformanceOptimizer:
    """性能优化器。"""

    def __init__(self):
        self.redis_pool = redis.ConnectionPool(
            host='redis',
            port=6379,
            db=0,
            max_connections=20,
            retry_on_timeout=True
        )
        self.redis_client = redis.Redis(connection_pool=self.redis_pool)

    # 数据库连接池优化
    def get_database_engine(self):
        """获取优化的数据库引擎。"""
        return create_engine(
            'postgresql://postgres:password@postgres:5432/moondev',
            poolclass=QueuePool,
            pool_size=20,
            max_overflow=30,
            pool_timeout=30,
            pool_recycle=3600,
            echo=False
        )

    # 缓存装饰器
    def cache_result(self, ttl: int = 300):
        """结果缓存装饰器。"""
        def decorator(func: Callable) -> Callable:
            @functools.wraps(func)
            def wrapper(*args, **kwargs):
                # 生成缓存键
                cache_key = f"{func.__name__}:{hash(str(args) + str(kwargs))}"

                # 尝试从缓存获取
                cached_result = self.redis_client.get(cache_key)
                if cached_result:
                    return pickle.loads(cached_result)

                # 执行函数
                result = func(*args, **kwargs)

                # 缓存结果
                self.redis_client.setex(
                    cache_key,
                    ttl,
                    pickle.dumps(result)
                )

                return result
            return wrapper
        return decorator

    # 批量处理装饰器
    def batch_process(self, batch_size: int = 100):
        """批量处理装饰器。"""
        def decorator(func: Callable) -> Callable:
            @functools.wraps(func)
            def wrapper(items: List[Any]) -> List[Any]:
                results = []

                for i in range(0, len(items), batch_size):
                    batch = items[i:i + batch_size]
                    batch_results = func(batch)
                    results.extend(batch_results)

                return results
            return wrapper
        return decorator

    # 异步并发控制
    async def limited_gather(self, tasks: List[asyncio.Task], limit: int = 10):
        """限制并发数量的异步任务收集。"""
        semaphore = asyncio.Semaphore(limit)

        async def limited_task(task):
            async with semaphore:
                return await task

        limited_tasks = [limited_task(task) for task in tasks]
        return await asyncio.gather(*limited_tasks, return_exceptions=True)
```

### 缓存策略

```python
# src/cache/strategy.py
import json
import hashlib
from typing import Any, Optional
import redis
from datetime import timedelta

class CacheStrategy:
    """缓存策略管理。"""

    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client

    def generate_key(self, prefix: str, *args, **kwargs) -> str:
        """生成缓存键。"""
        key_data = f"{prefix}:{args}:{sorted(kwargs.items())}"
        return hashlib.md5(key_data.encode()).hexdigest()

    # 多级缓存
    def multilevel_get(self, key: str) -> Optional[Any]:
        """多级缓存获取。"""
        # L1: 内存缓存（应用实例内）
        # L2: Redis 缓存（集群内）
        # L3: 数据库

        # 尝试从 Redis 获取
        value = self.redis.get(key)
        if value:
            return json.loads(value)

        return None

    def multilevel_set(self, key: str, value: Any, ttl: int = 300) -> None:
        """多级缓存设置。"""
        # 存储到 Redis
        self.redis.setex(key, ttl, json.dumps(value))

    # 智能缓存失效
    def invalidate_pattern(self, pattern: str) -> None:
        """按模式失效缓存。"""
        keys = self.redis.keys(pattern)
        if keys:
            self.redis.delete(*keys)

    # 缓存预热
    def warmup_cache(self, warmup_functions: List[Callable]) -> None:
        """缓存预热。"""
        for func in warmup_functions:
            try:
                func()
            except Exception as e:
                print(f"缓存预热失败: {func.__name__}: {e}")

# 缓存策略配置
CACHE_STRATEGIES = {
    'market_data': {
        'ttl': 60,  # 1分钟
        'pattern': 'market_data:*'
    },
    'analysis_result': {
        'ttl': 300,  # 5分钟
        'pattern': 'analysis:*'
    },
    'user_config': {
        'ttl': 3600,  # 1小时
        'pattern': 'user_config:*'
    }
}
```

## 备份策略

### 自动化备份

```bash
#!/bin/bash
# scripts/backup.sh

set -e

# 配置
BACKUP_DIR="/data/backups"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# 创建备份目录
mkdir -p $BACKUP_DIR/{database,redis,config,logs}

# 数据库备份
echo "开始数据库备份..."
docker exec postgres pg_dump -U postgres moondev | gzip > $BACKUP_DIR/database/postgres_$DATE.sql.gz

# Redis 备份
echo "开始 Redis 备份..."
docker exec redis redis-cli BGSAVE
sleep 5
docker cp redis:/data/dump.rdb $BACKUP_DIR/redis/redis_$DATE.rdb
gzip $BACKUP_DIR/redis/redis_$DATE.rdb

# 配置文件备份
echo "开始配置文件备份..."
tar -czf $BACKUP_DIR/config/config_$DATE.tar.gz \
    docker-compose.*.yml \
    .env.prod \
    nginx/ \
    prometheus/ \
    grafana/

# 日志备份
echo "开始日志备份..."
find /var/log -name "*.log" -mtime -1 -exec cp {} $BACKUP_DIR/logs/ \;
tar -czf $BACKUP_DIR/logs/logs_$DATE.tar.gz -C $BACKUP_DIR/logs .

# 清理旧备份
echo "清理旧备份..."
find $BACKUP_DIR -name "*.gz" -mtime +$RETENTION_DAYS -delete
find $BACKUP_DIR -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete

# 上传到云存储（可选）
if [ "$CLOUD_BACKUP_ENABLED" = "true" ]; then
    echo "上传到云存储..."
    aws s3 sync $BACKUP_DIR s3://moon-dev-backups/
fi

echo "备份完成: $DATE"
```

### 备份验证

```python
# scripts/backup_verify.py
import os
import subprocess
import hashlib
from typing import Dict, List
import requests

class BackupVerifier:
    """备份验证器。"""

    def __init__(self, backup_dir: str):
        self.backup_dir = backup_dir

    def verify_backup_integrity(self, backup_file: str) -> bool:
        """验证备份文件完整性。"""
        if not os.path.exists(backup_file):
            return False

        # 计算文件哈希
        with open(backup_file, 'rb') as f:
            file_hash = hashlib.sha256(f.read()).hexdigest()

        # 验证文件大小
        file_size = os.path.getsize(backup_file)

        # 简单验证规则
        if file_size < 1024:  # 文件太小
            return False

        # 可以添加更多验证逻辑
        return True

    def test_database_restore(self, backup_file: str) -> bool:
        """测试数据库恢复。"""
        try:
            # 创建临时数据库
            temp_db = "moondev_test_restore"

            # 恢复备份
            cmd = f"docker exec postgres createdb -U postgres {temp_db}"
            subprocess.run(cmd, shell=True, check=True)

            cmd = f"gunzip -c {backup_file} | docker exec -i postgres psql -U postgres {temp_db}"
            subprocess.run(cmd, shell=True, check=True)

            # 验证数据
            cmd = f"docker exec postgres psql -U postgres {temp_db} -c 'SELECT COUNT(*) FROM trades;'"
            result = subprocess.run(cmd, shell=True, capture_output=True, text=True)

            # 清理临时数据库
            cmd = f"docker exec postgres dropdb -U postgres {temp_db}"
            subprocess.run(cmd, shell=True)

            # 检查是否有数据
            return int(result.stdout.strip()) > 0

        except Exception as e:
            print(f"数据库恢复测试失败: {e}")
            return False

    def verify_cloud_backup(self) -> bool:
        """验证云备份。"""
        try:
            # 检查 S3 中的备份
            response = requests.get("https://s3.amazonaws.com/moon-dev-backups/latest.txt")

            if response.status_code == 200:
                latest_backup = response.text.strip()
                return self.verify_backup_integrity(latest_backup)

            return False

        except Exception as e:
            print(f"云备份验证失败: {e}")
            return False

    def generate_backup_report(self) -> Dict[str, Any]:
        """生成备份报告。"""
        report = {
            'timestamp': datetime.now().isoformat(),
            'total_backups': 0,
            'valid_backups': 0,
            'invalid_backups': [],
            'cloud_backup_available': False
        }

        # 扫描备份目录
        for backup_file in os.listdir(self.backup_dir):
            if backup_file.endswith('.gz') or backup_file.endswith('.tar.gz'):
                report['total_backups'] += 1

                if self.verify_backup_integrity(os.path.join(self.backup_dir, backup_file)):
                    report['valid_backups'] += 1
                else:
                    report['invalid_backups'].append(backup_file)

        # 验证云备份
        report['cloud_backup_available'] = self.verify_cloud_backup()

        return report
```

## 灾难恢复

### 恢复流程

```bash
#!/bin/bash
# scripts/disaster_recovery.sh

set -e

BACKUP_DIR="/data/backups"
RESTORE_TIMESTAMP=$1

if [ -z "$RESTORE_TIMESTAMP" ]; then
    echo "使用方法: $0 <backup_timestamp>"
    echo "可用备份:"
    ls -la $BACKUP_DIR/database/
    exit 1
fi

echo "开始灾难恢复流程..."

# 停止所有服务
echo "停止服务..."
docker-compose -f docker-compose.prod.yml down

# 恢复数据库
echo "恢复数据库..."
gunzip -c $BACKUP_DIR/database/postgres_$RESTORE_TIMESTAMP.sql.gz | \
    docker exec -i postgres psql -U postgres moondev

# 恢复 Redis
echo "恢复 Redis..."
gunzip -c $BACKUP_DIR/redis/redis_$RESTORE_TIMESTAMP.rdb.gz > \
    docker exec redis docker exec redis cp /dev/stdin /data/dump.rdb
docker restart redis

# 恢复配置文件
echo "恢复配置文件..."
tar -xzf $BACKUP_DIR/config/config_$RESTORE_TIMESTAMP.tar.gz -C /

# 验证恢复
echo "验证恢复..."
python scripts/verify_restore.py

# 启动服务
echo "启动服务..."
docker-compose -f docker-compose.prod.yml up -d

# 等待服务就绪
echo "等待服务就绪..."
sleep 30

# 健康检查
if curl -f http://localhost/health; then
    echo "✅ 灾难恢复完成"
else
    echo "❌ 恢复后健康检查失败"
    exit 1
fi
```

### 故障转移

```python
# scripts/failover.py
import subprocess
import time
import requests
from typing import Dict, List

class FailoverManager:
    """故障转移管理器。"""

    def __init__(self):
        self.services = {
            'app': ['app1:8000', 'app2:8000', 'app3:8000', 'app4:8000'],
            'database': ['postgres-master:5432', 'postgres-slave1:5432', 'postgres-slave2:5432'],
            'redis': ['redis-master:6379', 'redis-slave1:6379', 'redis-slave2:6379']
        }

    def check_service_health(self, service_url: str) -> bool:
        """检查服务健康状态。"""
        try:
            response = requests.get(f"http://{service_url}/health", timeout=5)
            return response.status_code == 200
        except:
            return False

    def promote_database_slave(self):
        """提升数据库从库为主库。"""
        try:
            # 选择最新的从库
            cmd = "docker exec postgres-slave1 pg_controldata | grep 'Time of latest checkpoint'"
            result = subprocess.run(cmd, shell=True, capture_output=True, text=True)

            # 停止主库写入
            cmd = "docker exec postgres-master psql -U postgres -c 'SELECT pg_promote();'"
            subprocess.run(cmd, shell=True)

            # 更新应用配置指向新主库
            # ... 配置更新逻辑

            print("数据库故障转移完成")
            return True

        except Exception as e:
            print(f"数据库故障转移失败: {e}")
            return False

    def handle_redis_failover(self):
        """处理 Redis 故障转移。"""
        try:
            # 使用 Redis Sentinel 进行故障转移
            cmd = "docker exec redis-sentinel redis-sentinel get-master-addr-by-name mymaster"
            result = subprocess.run(cmd, shell=True, capture_output=True, text=True)

            new_master = result.stdout.strip()
            print(f"Redis 新主库: {new_master}")

            # 更新应用配置
            # ... 配置更新逻辑

            return True

        except Exception as e:
            print(f"Redis 故障转移失败: {e}")
            return False

    def execute_failover(self, failed_service: str) -> bool:
        """执行故障转移。"""
        print(f"开始 {failed_service} 故障转移...")

        if failed_service == 'database':
            return self.promote_database_slave()
        elif failed_service == 'redis':
            return self.handle_redis_failover()
        else:
            print(f"不支持的故障转移服务: {failed_service}")
            return False

    def monitor_services(self):
        """监控服务状态。"""
        while True:
            for service, instances in self.services.items():
                healthy_instances = []

                for instance in instances:
                    if self.check_service_health(instance):
                        healthy_instances.append(instance)

                if len(healthy_instances) == 0:
                    print(f"❌ {service} 所有实例都不可用")
                    self.execute_failover(service)
                elif len(healthy_instances) < len(instances):
                    print(f"⚠️ {service} 部分实例不可用: {healthy_instances}")

            time.sleep(30)  # 30秒检查一次
```

## 运维最佳实践

### 部署脚本

```bash
#!/bin/bash
# scripts/deploy.sh

set -e

# 配置
ENVIRONMENT="production"
VERSION=$1
HEALTH_CHECK_URL="http://localhost/health"
ROLLBACK_TIMEOUT=300

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

# 前置检查
pre_deployment_check() {
    log "执行部署前检查..."

    # 检查环境变量
    if [ ! -f ".env.prod" ]; then
        error ".env.prod 文件不存在"
        exit 1
    fi

    # 检查服务状态
    if ! docker-compose -f docker-compose.prod.yml ps | grep -q "Up"; then
        warn "当前没有运行的服务"
    fi

    # 检查磁盘空间
    disk_usage=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
    if [ $disk_usage -gt 80 ]; then
        error "磁盘空间不足: ${disk_usage}%"
        exit 1
    fi

    log "部署前检查完成"
}

# 备份当前版本
backup_current_version() {
    log "备份当前版本..."

    BACKUP_TAG="backup_$(date +%Y%m%d_%H%M%S)"
    docker tag moon-dev-ai-agents:latest moon-dev-ai-agents:$BACKUP_TAG

    log "备份完成: $BACKUP_TAG"
}

# 蓝绿部署
blue_green_deploy() {
    log "开始蓝绿部署..."

    # 构建新版本
    docker build -t moon-dev-ai-agents:$VERSION .
    docker tag moon-dev-ai-agents:$VERSION moon-dev-ai-agents:latest

    # 更新绿环境
    docker-compose -f docker-compose.green.yml up -d

    # 等待绿环境就绪
    log "等待绿环境启动..."
    sleep 30

    # 健康检查
    if curl -f $HEALTH_CHECK_URL; then
        log "绿环境健康检查通过"
    else
        error "绿环境健康检查失败"
        docker-compose -f docker-compose.green.yml down
        exit 1
    fi

    # 切换流量
    log "切换流量到绿环境..."
    # 更新负载均衡器配置

    # 停止蓝环境
    docker-compose -f docker-compose.blue.yml down

    log "蓝绿部署完成"
}

# 滚动更新
rolling_update() {
    log "开始滚动更新..."

    # 更新镜像
    docker build -t moon-dev-ai-agents:$VERSION .
    docker tag moon-dev-ai-agents:$VERSION moon-dev-ai-agents:latest

    # 逐个更新实例
    instances=("app1" "app2" "app3" "app4")

    for instance in "${instances[@]}"; do
        log "更新 $instance..."

        # 停止实例
        docker-compose -f docker-compose.prod.yml stop $instance

        # 更新实例
        docker-compose -f docker-compose.prod.yml up -d --no-deps $instance

        # 等待实例就绪
        sleep 10

        # 健康检查
        if curl -f $HEALTH_CHECK_URL; then
            log "$instance 更新成功"
        else
            error "$instance 更新失败"
            # 回滚
            rollback
            exit 1
        fi
    done

    log "滚动更新完成"
}

# 回滚
rollback() {
    warn "开始回滚..."

    # 获取备份版本
    BACKUP_VERSION=$(docker images --format "table {{.Repository}}:{{.Tag}}" | grep moon-dev-ai-agents | head -2 | tail -1 | cut -d':' -f2)

    if [ -z "$BACKUP_VERSION" ]; then
        error "没有可回滚的版本"
        exit 1
    fi

    log "回滚到版本: $BACKUP_VERSION"

    # 切换到备份版本
    docker tag moon-dev-ai-agents:$BACKUP_VERSION moon-dev-ai-agents:latest
    docker-compose -f docker-compose.prod.yml up -d

    # 等待服务就绪
    sleep 30

    # 健康检查
    if curl -f $HEALTH_CHECK_URL; then
        log "回滚成功"
    else
        error "回滚失败"
        exit 1
    fi
}

# 部署后验证
post_deployment_verification() {
    log "执行部署后验证..."

    # 健康检查
    for i in {1..10}; do
        if curl -f $HEALTH_CHECK_URL; then
            log "健康检查通过"
            break
        else
            warn "健康检查失败，重试 $i/10"
            sleep 10
        fi

        if [ $i -eq 10 ]; then
            error "健康检查最终失败"
            rollback
            exit 1
        fi
    done

    # 功能测试
    python scripts/smoke_test.py

    log "部署后验证完成"
}

# 主函数
main() {
    if [ -z "$VERSION" ]; then
        error "请提供版本号"
        echo "使用方法: $0 <version>"
        exit 1
    fi

    log "开始部署版本: $VERSION"

    pre_deployment_check
    backup_current_version

    # 选择部署策略
    if [ "$DEPLOYMENT_STRATEGY" = "blue_green" ]; then
        blue_green_deploy
    else
        rolling_update
    fi

    post_deployment_verification

    log "部署完成!"
}

# 执行主函数
main "$@"
```

### 监控告警

```python
# scripts/monitoring.py
import psutil
import requests
import json
from datetime import datetime
from typing import Dict, List

class SystemMonitor:
    """系统监控器。"""

    def __init__(self):
        self.metrics = {}
        self.alert_thresholds = {
            'cpu_usage': 80,
            'memory_usage': 85,
            'disk_usage': 90,
            'response_time': 2000,  # ms
            'error_rate': 5  # %
        }

    def collect_system_metrics(self) -> Dict[str, float]:
        """收集系统指标。"""
        return {
            'cpu_usage': psutil.cpu_percent(interval=1),
            'memory_usage': psutil.virtual_memory().percent,
            'disk_usage': psutil.disk_usage('/').percent,
            'network_io': psutil.net_io_counters()._asdict(),
            'timestamp': datetime.now().isoformat()
        }

    def check_application_health(self, url: str) -> Dict[str, Any]:
        """检查应用健康状态。"""
        try:
            start_time = time.time()
            response = requests.get(url, timeout=10)
            response_time = (time.time() - start_time) * 1000

            return {
                'status': 'healthy' if response.status_code == 200 else 'unhealthy',
                'response_time': response_time,
                'status_code': response.status_code,
                'timestamp': datetime.now().isoformat()
            }
        except Exception as e:
            return {
                'status': 'error',
                'error': str(e),
                'timestamp': datetime.now().isoformat()
            }

    def send_alert(self, alert_type: str, message: str, severity: str = 'warning'):
        """发送告警。"""
        alert = {
            'alert_type': alert_type,
            'message': message,
            'severity': severity,
            'timestamp': datetime.now().isoformat(),
            'hostname': psutil.os.uname().nodename
        }

        # 发送到告警系统
        try:
            requests.post(
                'http://alertmanager:9093/api/v1/alerts',
                json=[alert],
                timeout=5
            )
        except Exception as e:
            print(f"发送告警失败: {e}")

    def monitor_loop(self):
        """监控循环。"""
        while True:
            # 收集系统指标
            system_metrics = self.collect_system_metrics()

            # 检查告警阈值
            for metric, value in system_metrics.items():
                if metric in self.alert_thresholds:
                    threshold = self.alert_thresholds[metric]
                    if value > threshold:
                        self.send_alert(
                            f'{metric}_high',
                            f'{metric} is {value}%, threshold is {threshold}%',
                            'warning'
                        )

            # 检查应用健康状态
            app_health = self.check_application_health('http://localhost:8000/health')
            if app_health['status'] != 'healthy':
                self.send_alert(
                    'application_unhealthy',
                    f"Application status: {app_health['status']}",
                    'critical'
                )

            time.sleep(60)  # 1分钟检查一次

if __name__ == "__main__":
    monitor = SystemMonitor()
    monitor.monitor_loop()
```

---

这个生产环境部署和监控指南提供了完整的生产环境解决方案，包括高可用架构、监控体系、安全配置、性能优化、备份策略和运维最佳实践。遵循这个指南可以确保 Moon Dev AI Agents 系统在生产环境中的稳定运行。