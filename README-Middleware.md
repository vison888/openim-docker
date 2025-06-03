# OpenIM 中间件部署说明

本项目已配置了五个核心中间件，支持远程访问和密码认证，并包含Kafka和Etcd管理界面。

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

## 管理界面

### 6. Kafka UI (Kafka 管理界面)
- **镜像**: provectuslabs/kafka-ui:latest
- **端口**: 9080
- **用户名**: `admin`
- **密码**: `kafkaUI123`

**访问方式**:
- Web 界面: http://localhost:9080
- 功能: 
  - 查看和管理 Kafka 主题
  - 监控消费者组
  - 查看消息内容
  - 集群状态监控

### 7. Etcd Browser (Etcd 管理界面)
- **镜像**: joinsunsoft/etcdv3-browser:0.9.0
- **端口**: 9980
- **认证**: 使用 Etcd 的 root 用户认证

**访问方式**:
- Web 界面: http://localhost:9980
- 功能:
  - 浏览 Etcd 键值存储
  - 查看和编辑键值对
  - 实时监控数据变化
  - 支持 Etcd v3 API

**使用说明**:
1. 打开浏览器访问: http://localhost:9980
2. 在连接配置页面输入:
   - Endpoint: `http://etcd:2379` (容器内已自动配置)
   - Username: `root`
   - Password: `etcdPassword123`

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

# 只启动 Etcd Browser
docker-compose up -d etcd-browser

# 只启动 Kafka
docker-compose up -d kafka

# 只启动 Kafka UI
docker-compose up -d kafka-ui

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
docker-compose logs etcd-browser
docker-compose logs kafka
docker-compose logs kafka-ui
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

# Kafka UI 密码
KAFKA_UI_PASSWORD=your_new_password

# MinIO 密码
MINIO_SECRET_ACCESS_KEY=your_new_password
```

### 修改端口
在 `env` 文件中修改端口配置：

```bash
# 管理界面端口
KAFKA_UI_PORT=9080
ETCD_BROWSER_PORT=9980

# 中间件端口
MONGO_PORT=27017
REDIS_PORT=6379
ETCD_CLIENT_PORT=2379
KAFKA_EXTERNAL_PORT=9094
MINIO_PORT=10005
```

## Kafka UI 使用指南

### 访问 Kafka UI
1. 启动服务后访问: http://localhost:9080
2. 使用用户名 `admin` 和密码 `kafkaUI123` 登录

### 主要功能
- **集群概览**: 查看 Kafka 集群状态和基本信息
- **主题管理**: 创建、删除、配置主题
- **消息浏览**: 查看主题中的消息内容
- **消费者组**: 监控消费者组的消费进度
- **连接器**: 管理 Kafka Connect 连接器（如果有）
- **架构注册**: 管理 Schema Registry（如果有）

### 创建主题示例
1. 在 Kafka UI 中点击 "Topics"
2. 点击 "Add a Topic"
3. 输入主题名称和配置
4. 点击 "Create Topic"

## Etcd Browser 使用指南

### 访问 Etcd Browser
1. 启动服务后访问: http://localhost:9980
2. 系统已自动配置连接参数

### 主要功能
- **键值浏览**: 以树形结构浏览所有键值对
- **数据操作**: 创建、修改、删除键值对
- **实时监控**: 监控键值变化
- **搜索功能**: 快速查找特定的键
- **权限管理**: 查看和管理 Etcd 用户权限

### 操作示例
1. **查看键值**: 在左侧树形结构中点击任意键
2. **添加键值**: 点击 "Add Key" 按钮
3. **编辑值**: 双击值区域进行编辑
4. **删除键**: 选中键后点击删除按钮

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
6. **管理界面访问控制**: 在生产环境中限制管理界面的访问

## 端口汇总

| 服务 | 内部端口 | 外部端口 | 用途 |
|------|----------|----------|------|
| MongoDB | 27017 | 27017 | 数据库连接 |
| Redis | 6379 | 6379 | 缓存连接 |
| Etcd | 2379/2380 | 2379/2380 | 服务发现 |
| Kafka | 9092/9094 | 9092/9094 | 消息队列 |
| Kafka UI | 8080 | 9080 | Kafka 管理界面 |
| Etcd Browser | 80 | 9980 | Etcd 管理界面 |
| MinIO | 9000/9090 | 10005/10004 | 对象存储 |

## 故障排除

### 常见问题

1. **端口被占用**: 检查端口是否被其他服务占用
2. **权限问题**: 确保 Docker 有足够的权限访问数据目录
3. **内存不足**: 确保服务器有足够的内存运行所有服务
4. **网络问题**: 检查 Docker 网络配置
5. **管理界面无法连接**: 确保对应的中间件服务已正常启动

### Kafka UI 故障排除

```bash
# 检查 Kafka UI 日志
docker-compose logs kafka-ui

# 检查 Kafka 连接状态
docker-compose exec kafka-ui curl -f http://localhost:8080/actuator/health

# 重启 Kafka UI
docker-compose restart kafka-ui
```

### Etcd Browser 故障排除

```bash
# 检查 Etcd Browser 日志
docker-compose logs etcd-browser

# 检查 Etcd 连接状态
docker-compose exec etcd-browser curl -f http://localhost:80

# 重启 Etcd Browser
docker-compose restart etcd-browser

# 验证 Etcd 服务状态
docker-compose exec etcd etcdctl --endpoints=localhost:2379 endpoint health
```

### 重置服务

```bash
# 停止所有服务
docker-compose down

# 删除数据（谨慎操作）
rm -rf ./components/

# 重新启动
docker-compose up -d
``` 