# AI News Workflow v2.1.0 – 设置指南

## 1. 环境准备

| 组件 | 说明 |
| --- | --- |
| n8n | 推荐 v1.43+（Docker 或 npm 均可） |
| Node.js | 使用 npm 部署时需 Node 18+ |
| 飞书企业账号 | 提供多维表格与机器人权限 |

### 安装 n8n（Docker 示例）

```bash
docker run -it --rm \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  -e TZ=Asia/Shanghai \
  n8nio/n8n:latest
```

### 配置环境变量

```bash
cd ai-news-workflow
cp env.template .env
# 按提示填写飞书 / 微信 / 微博 / DeepSeek 等变量
```

## 2. 飞书配置

1. 登陆开放平台创建企业自建应用，启用多维表格/消息推送权限。
2. 获取 `app_id`、`app_secret`。
3. 在多维表格创建下列字段：`title`、`summary`、`url`、`publish_date`、`relevance_score`、`keywords` 等。
4. 获取 `应用 token` 与 `表格 ID`，填入 `.env`。

## 3. 微信 & 微博（可选）

- **微信公众号**：申请接口权限；在公众平台 “开发者 → 服务器 IP 白名单” 中填入 n8n 出口 IP，同时把这些 IP 写入 `WECHAT_WHITELISTED_IPS`。
- **微博**：创建开放平台应用，获取 `APP_KEY`、`APP_SECRET`、`ACCESS_TOKEN`、`UID`。

## 4. 导入工作流

1. 打开 n8n → `Import from file`。
2. 依次导入：
   - `Workflow A - AI News collector v2.1.0.json`
   - `Workflow B - AI News digest v2.1.0 (Feishu + WeChat Draft).json`
   - `Workflow C - Feishu Data Deduplication Tool v2.1.0.json`
3. 检查每个工作流的 `Configuration` 节点，确保所有变量引用正常（无红色警告）。

## 5. 节点提示

| 工作流 | 节点 | 说明 |
| --- | --- | --- |
| A | `DeepSeek Reasoner Model` | 选择你的 DeepSeek Credential |
| B | `Check Public IP` | 调用微信接口获取真实出口 IP |
| B | `Build IP Alert Message` | IP 不在白名单时向飞书发告警 |
| C | `Has Duplicates?` | Switch 以 `duplicate_count` 数值判断 |

## 6. 定时与验证

- Workflow A 默认 07:00 运行，Workflow B 默认 08:00，可在 Cron 节点调整。
- Workflow C 可设为每周运行，也可通过手动触发执行。

验证流程：
1. **Collector**：运行 Workflow A，飞书表格应新增记录。
2. **Digest**：运行 Workflow B，在 IP 不在白名单的情况下应收到飞书告警；添加 IP 后再次运行，应能创建公众号草稿。
3. **Dedup**：复制表格多条数据，运行 Workflow C，`Summarize Results` 节点应显示删除数量。

## 7. 常见问题

- **飞书 403**：检查应用权限/表格 ID 是否匹配。
- **微信 40164**：未在公众平台白名单或 `WECHAT_WHITELISTED_IPS` 中添加当前出口 IP。
- **DeepSeek 429**：频率过高，考虑在 Workflow B 中增加 `Wait` 节点。

完成以上步骤即可进入部署阶段，详见 `docs/deployment_guide.md`。
