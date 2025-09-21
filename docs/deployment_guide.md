# 部署指南

## 🚀 部署选项

### 选项 1: Docker Compose (推荐)

#### 1. 准备环境

```bash
# 克隆项目
git clone <repository-url>
cd ai-news-workflow

# 配置环境变量
cp env.template .env
# 编辑 .env 文件，填入你的配置
```

#### 2. 启动服务

```bash
# 启动 n8n 和相关服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f n8n
```

#### 3. 访问 n8n

打开浏览器访问: http://localhost:5678

### 选项 2: 云服务器部署

#### 1. 服务器要求

- **最低配置**: 2 CPU, 4GB RAM, 20GB 存储
- **推荐配置**: 4 CPU, 8GB RAM, 50GB 存储
- **操作系统**: Ubuntu 20.04+ / CentOS 8+ / Debian 11+

#### 2. 安装 Docker

```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# CentOS/RHEL
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
```

#### 3. 安装 Docker Compose

```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

#### 4. 部署应用

```bash
# 克隆项目
git clone <repository-url>
cd ai-news-workflow

# 配置环境变量
cp env.template .env
nano .env  # 编辑配置文件

# 启动服务
docker-compose up -d

# 设置开机自启
sudo systemctl enable docker
```

### 选项 3: Kubernetes 部署

#### 1. 创建命名空间

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ai-news-workflow
```

#### 2. 创建 ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: n8n-config
  namespace: ai-news-workflow
data:
  N8N_BASIC_AUTH_ACTIVE: "true"
  N8N_BASIC_AUTH_USER: "admin"
  N8N_BASIC_AUTH_PASSWORD: "your-password"
  N8N_HOST: "0.0.0.0"
  N8N_PORT: "5678"
  N8N_PROTOCOL: "http"
  WEBHOOK_URL: "https://your-domain.com"
```

#### 3. 创建 Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: n8n-secrets
  namespace: ai-news-workflow
type: Opaque
data:
  FEISHU_APP_ID: <base64-encoded-value>
  FEISHU_APP_SECRET: <base64-encoded-value>
  # ... 其他敏感配置
```

#### 4. 创建 Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: n8n
  namespace: ai-news-workflow
spec:
  replicas: 1
  selector:
    matchLabels:
      app: n8n
  template:
    metadata:
      labels:
        app: n8n
    spec:
      containers:
      - name: n8n
        image: n8nio/n8n:latest
        ports:
        - containerPort: 5678
        envFrom:
        - configMapRef:
            name: n8n-config
        - secretRef:
            name: n8n-secrets
        volumeMounts:
        - name: n8n-data
          mountPath: /home/node/.n8n
      volumes:
      - name: n8n-data
        persistentVolumeClaim:
          claimName: n8n-pvc
```

## 🔧 配置说明

### 环境变量配置

#### 必需配置

```bash
# 飞书配置
FEISHU_APP_ID=your_app_id
FEISHU_APP_SECRET=your_app_secret
FEISHU_TABLE_APP_TOKEN=your_table_token
FEISHU_TABLE_ID=your_table_id

# DeepSeek 配置
DEEPSEEK_API_CREDENTIAL_ID=your_credential_id
```

#### 可选配置

```bash
# 飞书群聊通知
FEISHU_WEBHOOK_URL=https://open.feishu.cn/open-apis/bot/v2/hook/your_webhook_id

# 微博发布
WEIBO_APP_KEY=your_weibo_app_key
WEIBO_APP_SECRET=your_weibo_app_secret
WEIBO_ACCESS_TOKEN=your_weibo_access_token
WEIBO_UID=your_weibo_uid

# 系统配置
TIMEZONE=Asia/Shanghai
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=your_password
```

### 网络配置

#### 端口映射

- **5678**: n8n Web 界面
- **5679**: n8n Webhook 端口（可选）

#### 防火墙设置

```bash
# Ubuntu/Debian
sudo ufw allow 5678/tcp
sudo ufw allow 5679/tcp

# CentOS/RHEL
sudo firewall-cmd --permanent --add-port=5678/tcp
sudo firewall-cmd --permanent --add-port=5679/tcp
sudo firewall-cmd --reload
```

## 📊 监控和维护

### 健康检查

```bash
# 检查服务状态
docker-compose ps

# 检查日志
docker-compose logs -f n8n

# 检查资源使用
docker stats
```

### 备份策略

#### 1. 数据备份

```bash
# 备份 n8n 数据
docker-compose exec n8n tar -czf /tmp/n8n-backup-$(date +%Y%m%d).tar.gz /home/node/.n8n

# 备份到本地
docker cp <container_id>:/tmp/n8n-backup-$(date +%Y%m%d).tar.gz ./backups/
```

#### 2. 自动备份脚本

```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/opt/backups/n8n"
DATE=$(date +%Y%m%d_%H%M%S)
CONTAINER_NAME="ai-news-workflow_n8n_1"

# 创建备份目录
mkdir -p $BACKUP_DIR

# 执行备份
docker exec $CONTAINER_NAME tar -czf /tmp/n8n-backup-$DATE.tar.gz /home/node/.n8n
docker cp $CONTAINER_NAME:/tmp/n8n-backup-$DATE.tar.gz $BACKUP_DIR/

# 清理旧备份（保留30天）
find $BACKUP_DIR -name "n8n-backup-*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR/n8n-backup-$DATE.tar.gz"
```

#### 3. 设置定时备份

```bash
# 添加到 crontab
crontab -e

# 每天凌晨2点执行备份
0 2 * * * /opt/scripts/backup.sh >> /var/log/n8n-backup.log 2>&1
```

### 日志管理

#### 1. 日志轮转

```bash
# 创建 logrotate 配置
sudo nano /etc/logrotate.d/n8n

# 内容：
/var/log/n8n/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 644 root root
    postrotate
        docker-compose restart n8n
    endscript
}
```

#### 2. 日志监控

```bash
# 实时监控日志
docker-compose logs -f n8n | grep -E "(ERROR|WARN|FATAL)"

# 统计错误日志
docker-compose logs n8n | grep -c ERROR
```

## 🔄 更新和维护

### 更新 n8n

```bash
# 停止服务
docker-compose down

# 拉取最新镜像
docker-compose pull

# 启动服务
docker-compose up -d

# 清理旧镜像
docker image prune -f
```

### 更新工作流

1. 在 n8n 界面中导入新的工作流文件
2. 更新环境变量配置
3. 测试工作流执行
4. 监控运行状态

### 性能优化

#### 1. 资源限制

```yaml
# docker-compose.yml
services:
  n8n:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 4G
        reservations:
          cpus: '1.0'
          memory: 2G
```

#### 2. 数据库优化

```bash
# 清理 n8n 数据库
docker-compose exec n8n n8n db:cleanup

# 重建索引
docker-compose exec n8n n8n db:rebuild
```

## 🚨 故障排除

### 常见问题

#### 1. 服务无法启动

```bash
# 检查端口占用
netstat -tlnp | grep 5678

# 检查 Docker 状态
docker ps -a
docker logs <container_id>
```

#### 2. 工作流执行失败

- 检查环境变量配置
- 验证 API 凭证有效性
- 查看执行日志
- 测试单个节点

#### 3. 内存不足

```bash
# 增加交换空间
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### 恢复步骤

#### 1. 从备份恢复

```bash
# 停止服务
docker-compose down

# 恢复数据
docker run --rm -v n8n_data:/data -v $(pwd)/backups:/backup alpine tar -xzf /backup/n8n-backup-YYYYMMDD.tar.gz -C /data

# 启动服务
docker-compose up -d
```

#### 2. 重置配置

```bash
# 删除数据卷
docker-compose down -v

# 重新配置
cp env.template .env
# 编辑 .env 文件

# 启动服务
docker-compose up -d
```

## 📞 技术支持

如遇到部署问题，请：

1. 查看本文档的故障排除部分
2. 检查 [GitHub Issues](https://github.com/your-repo/issues)
3. 提交新的 Issue 并提供：
   - 错误日志
   - 系统环境信息
   - 配置信息（隐藏敏感数据）

---

**注意**: 部署前请确保已完成所有必要的 API 配置，并测试了所有功能。
