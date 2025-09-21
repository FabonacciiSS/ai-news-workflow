# AI News Workflow 设置指南

## 概述

这是一个基于 n8n 的 AI 新闻自动化工作流，包含两个主要工作流：

1. **AI News Collector** - 收集和整理 AI 相关新闻
2. **AI News Digest** - 生成每日 AI 资讯摘要

## 环境准备

### 1. 安装 n8n

```bash
# 使用 Docker Compose (推荐)
docker-compose up -d

# 或使用 npm
npm install -g n8n
n8n start
```

### 2. 配置环境变量

1. 复制环境变量模板：
```bash
cp env.template .env
```

2. 编辑 `.env` 文件，填入你的实际配置值

### 3. 获取必要的 API 凭证

#### 飞书 (Feishu) 配置

1. 登录 [飞书开放平台](https://open.feishu.cn/)
2. 创建企业自建应用
3. 获取以下信息：
   - `FEISHU_APP_ID`: 应用ID
   - `FEISHU_APP_SECRET`: 应用密钥
   - `FEISHU_TABLE_APP_TOKEN`: 多维表格应用Token
   - `FEISHU_TABLE_ID`: 多维表格ID
   - `FEISHU_WEBHOOK_URL`: 群聊机器人Webhook URL

#### DeepSeek API 配置

1. 注册 [DeepSeek](https://platform.deepseek.com/) 账号
2. 获取 API Key
3. 在 n8n 中配置 DeepSeek API 凭证
4. 记录凭证ID作为 `DEEPSEEK_API_CREDENTIAL_ID`

#### 微博配置 (可选)

1. 登录 [微博开放平台](https://open.weibo.com/)
2. 创建应用
3. 获取以下信息：
   - `WEIBO_APP_KEY`: 应用Key
   - `WEIBO_APP_SECRET`: 应用密钥
   - `WEIBO_ACCESS_TOKEN`: 访问令牌
   - `WEIBO_UID`: 用户ID

## 工作流导入

### 1. 导入工作流

1. 在 n8n 界面中，点击 "Import from file"
2. 导入以下文件：
   - `workflows/workflow_a_ai_news_collector_final.json`
   - `workflows/workflow_b_ai_news_digest.json`

### 2. 配置工作流

#### AI News Collector 配置

1. 打开 "AI News collector final" 工作流
2. 在 "Configuration" 节点中，确保所有环境变量都正确设置
3. 在 "DeepSeek Reasoner Model" 节点中，选择正确的 DeepSeek API 凭证
4. 调整 RSS 源（可选）

#### AI News Digest 配置

1. 打开 "AI News digest" 工作流
2. 在 "Configuration" 节点中，确保所有环境变量都正确设置
3. 在 "DeepSeek Reasoner Model" 节点中，选择正确的 DeepSeek API 凭证
4. 配置发布渠道（飞书群聊、微博等）

## 飞书多维表格结构

确保你的飞书多维表格包含以下字段：

| 字段名 | 类型 | 说明 |
|--------|------|------|
| title | 文本 | 新闻标题 |
| summary | 文本 | 新闻摘要 |
| url | 超链接 | 原文链接 |
| source | 文本 | 新闻来源 |
| publish_date | 日期 | 发布日期 |
| relevance_score | 数字 | 相关度评分 |
| keywords | 文本 | 关键词 |
| category | 文本 | 分类 |
| status | 文本 | 状态 |
| created_at | 日期 | 创建时间 |

## 定时设置

### AI News Collector
- 默认每天 7:00 执行
- 可在 "Cron Trigger" 节点中修改

### AI News Digest
- 默认每天 8:00 执行
- 可在 "Cron Trigger" 节点中修改

## 故障排除

### 常见问题

1. **飞书 API 调用失败**
   - 检查应用权限设置
   - 确认多维表格字段名称正确
   - 验证访问令牌是否有效

2. **DeepSeek API 调用失败**
   - 检查 API Key 是否正确
   - 确认账户余额充足
   - 验证网络连接

3. **RSS 源无法访问**
   - 检查 RSS 源 URL 是否有效
   - 确认网络连接正常
   - 尝试使用代理或 VPN

### 日志查看

在 n8n 执行历史中查看详细日志，特别关注：
- JavaScript 代码节点的 console.log 输出
- HTTP 请求节点的响应状态
- AI 分析节点的输出结果

## 自定义配置

### 添加新的 RSS 源

1. 在 "AI News Collector" 工作流中添加新的 "RSS Feed Read" 节点
2. 在 "Collect RSS Articles" 节点中添加对应的来源映射
3. 连接到 "Filter Recent Articles" 节点

### 修改 AI 分析提示词

1. 编辑 "AI Analysis" 节点的提示词
2. 调整 "Structured Output Parser" 的 JSON Schema
3. 更新 "Format Output" 节点的字段映射

### 添加新的发布渠道

1. 在 "AI News Digest" 工作流中添加新的发布节点
2. 在 "Format Feishu Card" 节点后添加对应的格式化节点
3. 配置相应的 API 调用

## 安全注意事项

1. **保护敏感信息**
   - 不要将 `.env` 文件提交到版本控制
   - 使用环境变量存储所有敏感配置
   - 定期轮换 API 密钥

2. **权限控制**
   - 限制飞书应用的权限范围
   - 使用最小权限原则
   - 定期审查 API 使用情况

3. **数据安全**
   - 定期备份飞书表格数据
   - 监控异常 API 调用
   - 设置合理的执行频率限制

## 支持

如果遇到问题，请：

1. 查看 n8n 执行日志
2. 检查环境变量配置
3. 验证 API 凭证有效性
4. 参考本文档的故障排除部分

## 更新日志

- v1.0.0: 初始版本，支持基本的新闻收集和摘要功能
- v1.1.0: 添加微博发布功能
- v1.2.0: 优化去重算法，提高处理效率
