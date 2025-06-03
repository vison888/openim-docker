# OpenIM 中间件部署说明

本项目已配置了五个核心中间件，支持远程访问和密码认证。

## 中间件列表

### 1. MongoDB (数据库)
- **镜像**: mongo:7.0
- **端口**: 27017
- **管理员账户**: 
  - 用户名: `root`
  - 密码: `openIM123`
- **应用账户**: 
  - 用户名: `openIM`
  - 密码: `openIM123`
  - 数据库: `openim_v3`

**连接示例**:
```bash
# 使用 mongosh 连接
mongosh "mongodb://root:openIM123@localhost:27017/admin"

# 连接应用数据库
mongosh "mongodb://openIM:openIM123@localhost:27017/openim_v3"
```

### 2. Redis (缓存)
- **镜像**: redis:7.0.0
- **端口**: 6379
- **密码**: `openIM123`

**连接示例**:
```bash
# 使用 redis-cli 连接
redis-cli -h localhost -p 6379 -a openIM123

# 或者
redis-cli -h localhost -p 6379
AUTH openIM123
```

### 3. Etcd (服务发现)
- **镜像**: quay.io/coreos/etcd:v3.5.13
- **客户端端口**: 2379
- **对等端口**: 2380
- **密码**: `etcdPassword123`

**连接示例**:
```bash
# 使用 etcdctl 连接
export ETCDCTL_API=3
etcdctl --endpoints=localhost:2379 \
        --user=root:etcdPassword123 \
        put key1 value1
```

### 4. Kafka (消息队列)
- **镜像**: bitnami/kafka:3.5.1
- **内部端口**: 9092
- **外部端口**: 9094
- **用户名**: `kafka`
- **密码**: `kafkaPassword123`

**连接示例**:
```bash
# 创建主题
kafka-topics.sh --create \
  --bootstrap-server localhost:9094 \
  --topic test-topic \
  --command-config client.properties

# client.properties 内容:
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="kafka" password="kafkaPassword123";
```

### 5. MinIO (对象存储)
- **镜像**: minio/minio:RELEASE.2024-01-11T07-46-16Z
- **API 端口**: 10005
- **控制台端口**: 10004
- **访问密钥**: `root`
- **秘密密钥**: `openIM123`

**访问方式**:
- Web 控制台: http://localhost:10004
- API 端点: http://localhost:10005

## 部署和启动

### 启动所有中间件
```bash
docker-compose up -d
```

### 启动特定中间件
```bash
# 只启动 MongoDB
docker-compose up -d mongo

# 只启动 Redis
docker-compose up -d redis

# 只启动 Etcd
docker-compose up -d etcd

# 只启动 Kafka
docker-compose up -d kafka

# 只启动 MinIO
docker-compose up -d minio
```

### 查看运行状态
```bash
docker-compose ps
```

### 查看日志
```bash
# 查看所有服务日志
docker-compose logs

# 查看特定服务日志
docker-compose logs mongo
docker-compose logs redis
docker-compose logs etcd
docker-compose logs kafka
docker-compose logs minio
```

## 环境配置

### 修改外部访问地址
在 `env` 文件中修改以下配置：

```bash
# Kafka 外部访问地址（改为你的服务器 IP）
KAFKA_EXTERNAL_HOST=your_server_ip

# MinIO 外部访问地址（改为你的服务器 IP）
MINIO_EXTERNAL_ADDRESS="http://your_server_ip:10005"
```

### 修改密码
在 `env` 文件中修改相应的密码配置：

```bash
# MongoDB 密码
MONGO_ROOT_PASSWORD=your_new_password
MONGO_PASSWORD=your_new_password

# Redis 密码
REDIS_PASSWORD=your_new_password

# Etcd 密码
ETCD_ROOT_PASSWORD=your_new_password

# Kafka 密码
KAFKA_PASSWORD=your_new_password

# MinIO 密码
MINIO_SECRET_ACCESS_KEY=your_new_password
```

## 数据持久化

所有数据都存储在 `./components/` 目录下：

- MongoDB: `./components/mongodb/data/`
- Redis: `./components/redis/data/`
- Etcd: `./components/etcd/data/`
- Kafka: `./components/kafka/`
- MinIO: `./components/mnt/data/`

## 安全建议

1. **更改默认密码**: 修改所有中间件的默认密码
2. **防火墙配置**: 只开放必要的端口
3. **网络隔离**: 在生产环境中使用内部网络
4. **定期备份**: 定期备份重要数据
5. **监控日志**: 监控服务日志以发现异常

## 故障排除

### 常见问题

1. **端口被占用**: 检查端口是否被其他服务占用
2. **权限问题**: 确保 Docker 有足够的权限访问数据目录
3. **内存不足**: 确保服务器有足够的内存运行所有服务
4. **网络问题**: 检查 Docker 网络配置

### 重置服务

```bash
# 停止所有服务
docker-compose down

# 删除数据（谨慎操作）
rm -rf ./components/

# 重新启动
docker-compose up -d
``` 