# AI News Workflow

🤖 基于 n8n 的 AI 新闻自动化工作流，实现新闻收集、智能分析和多渠道发布。

## ✨ 功能特性

- 📰 **多源新闻收集**: 支持多个 RSS 源的 AI 相关新闻收集
- 🧠 **智能分析**: 使用 DeepSeek AI 进行新闻分析和评分
- 🔄 **自动去重**: 智能去重算法，避免重复内容
- 📊 **数据存储**: 使用飞书多维表格存储和管理新闻数据
- 📱 **多渠道发布**: 支持飞书群聊、微博等平台发布
- ⏰ **定时执行**: 可配置的定时任务，自动化运行

## 🏗️ 架构概览

```
RSS Sources → News Collection → AI Analysis → Data Storage
     ↓              ↓              ↓           ↓
  Filtering → Deduplication → Formatting → Publishing
```

### 工作流组件

1. **AI News Collector** (`workflow_a_ai_news_collector_final.json`)
   - 定时收集多个 RSS 源的新闻
   - 智能去重和过滤
   - AI 分析和评分
   - 存储到飞书多维表格

2. **AI News Digest** (`workflow_b_ai_news_digest.json`)
   - 从飞书表格读取新闻数据
   - AI 精选和摘要生成
   - 多渠道发布（飞书、微博等）

## 🚀 快速开始

### 环境要求

- n8n (推荐使用 Docker)
- 飞书企业账号
- DeepSeek API 账号
- 微博开放平台账号（可选）

### 安装步骤

1. **克隆项目**
```bash
git clone <repository-url>
cd ai-news-workflow
```

2. **配置环境变量**
```bash
cp env.template .env
# 编辑 .env 文件，填入你的配置
```

3. **启动 n8n**
```bash
docker-compose up -d
# 或
n8n start
```

4. **导入工作流**
   - 在 n8n 界面中导入 `workflows/` 目录下的 JSON 文件
   - 配置相应的 API 凭证

5. **配置飞书多维表格**
   - 按照文档创建表格结构
   - 获取表格 Token 和 ID

详细设置指南请参考 [docs/setup_guide.md](docs/setup_guide.md)

## 📋 配置说明

### 必需配置

| 配置项 | 说明 | 获取方式 |
|--------|------|----------|
| `FEISHU_APP_ID` | 飞书应用ID | 飞书开放平台 |
| `FEISHU_APP_SECRET` | 飞书应用密钥 | 飞书开放平台 |
| `FEISHU_TABLE_APP_TOKEN` | 飞书表格Token | 飞书多维表格 |
| `FEISHU_TABLE_ID` | 飞书表格ID | 飞书多维表格 |
| `DEEPSEEK_API_CREDENTIAL_ID` | DeepSeek API 凭证ID | n8n 凭证管理 |

### 可选配置

| 配置项 | 说明 | 用途 |
|--------|------|------|
| `FEISHU_WEBHOOK_URL` | 飞书群聊机器人URL | 发送摘要到群聊 |
| `WEIBO_APP_KEY` | 微博应用Key | 发布到微博 |
| `WEIBO_ACCESS_TOKEN` | 微博访问令牌 | 微博API调用 |

## 📊 数据流程

### 新闻收集流程

1. **RSS 源读取**: 从多个 RSS 源获取新闻
2. **时间过滤**: 只处理最近的文章
3. **去重处理**: 基于 URL、标题和内容相似度去重
4. **AI 分析**: 使用 DeepSeek 分析新闻内容
5. **数据存储**: 保存到飞书多维表格

### 摘要生成流程

1. **数据读取**: 从飞书表格读取新闻数据
2. **筛选排序**: 按相关度评分筛选和排序
3. **AI 精选**: 使用 AI 精选最有价值的新闻
4. **格式化**: 生成适合不同平台的格式
5. **发布**: 发送到各个平台

## 🔧 自定义配置

### 添加新的 RSS 源

1. 在收集工作流中添加新的 RSS 节点
2. 在 JavaScript 代码中添加来源映射
3. 连接到过滤节点

### 修改 AI 分析提示词

编辑 "AI Analysis" 节点的提示词，可以调整：
- 分析标准
- 输出格式
- 评分规则

### 添加新的发布渠道

1. 在摘要工作流中添加新的发布节点
2. 创建对应的格式化节点
3. 配置 API 调用

## 📁 项目结构

```
ai-news-workflow/
├── workflows/                    # n8n 工作流文件
│   ├── workflow_a_ai_news_collector_final.json
│   └── workflow_b_ai_news_digest.json
├── docs/                        # 文档
│   ├── setup_guide.md          # 设置指南
│   └── ...
├── scripts/                     # 脚本文件
├── env.template                 # 环境变量模板
├── docker-compose.yml          # Docker 配置
└── README.md                   # 项目说明
```

## 🛠️ 开发指南

### 本地开发

1. 安装依赖
```bash
npm install
```

2. 启动开发环境
```bash
n8n start --tunnel
```

3. 测试工作流
   - 使用 n8n 的测试功能
   - 查看执行日志
   - 验证数据流

### 调试技巧

1. **查看日志**: 在 n8n 执行历史中查看详细日志
2. **测试节点**: 使用 "Execute Workflow" 功能测试单个节点
3. **数据验证**: 检查中间节点的输出数据
4. **错误处理**: 配置错误处理节点

## 🔒 安全注意事项

- ⚠️ **保护敏感信息**: 不要将 API 密钥提交到版本控制
- 🔐 **权限最小化**: 只授予必要的 API 权限
- 📊 **监控使用**: 定期检查 API 使用情况
- 🔄 **定期更新**: 及时更新 API 密钥

## 📈 性能优化

- **批量处理**: 合理设置批处理大小
- **缓存机制**: 使用缓存减少重复请求
- **错误重试**: 配置合理的重试策略
- **资源限制**: 设置适当的超时和频率限制

## 🤝 贡献指南

1. Fork 项目
2. 创建功能分支
3. 提交更改
4. 创建 Pull Request

## 📄 许可证

本项目采用 MIT 许可证。详见 [LICENSE](LICENSE) 文件。

## 🙏 致谢

- [n8n](https://n8n.io/) - 工作流自动化平台
- [DeepSeek](https://platform.deepseek.com/) - AI 分析服务
- [飞书](https://open.feishu.cn/) - 数据存储和通知服务

## 📞 支持

如有问题或建议，请：

1. 查看 [设置指南](docs/setup_guide.md)
2. 检查 [常见问题](docs/faq.md)
3. 提交 [Issue](https://github.com/your-repo/issues)
4. 参与 [讨论](https://github.com/your-repo/discussions)

---

⭐ 如果这个项目对你有帮助，请给个 Star！