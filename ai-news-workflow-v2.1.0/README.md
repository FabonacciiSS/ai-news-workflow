# AI News Workflow v2.1.0

面向终端使用者的 n8n 工作流合集（采集 → Digest → 去重）。本版本在 v2.0 基础上完成了微信白名单自动检测、飞书凭证安全化、去重分支修复等更新，开箱即用。

## 🎯 功能概览

| 工作流 | 作用 | 亮点 |
| --- | --- | --- |
| Workflow A – AI News Collector | 各渠道资讯采集 + 清洗后写入飞书多维表格 | 定时触发、内容去噪、全程调用飞书 API |
| Workflow B – AI News Digest (Feishu + WeChat) | 读取表格 → 生成 Digest → 推送飞书 → 生成公众号草稿 | 新增出口 IP 校验 + 飞书告警，支持微博/小红书/知乎格式化 |
| Workflow C – Feishu Data Deduplication Tool | 周期性扫描多维表格重复项并批量删除 | Switch 节点修正为数值判断，自动分页删除 |

所有 `.json` 可直接导入 n8n，目录位于 `workflows/ai-news-workflow/ai-news-workflow-v2.1.0/`。

## ⚙️ 环境变量

请将以下变量写入 `.env` 或 n8n Credential：

| 变量 | 用途 |
| --- | --- |
| `FEISHU_APP_ID`, `FEISHU_APP_SECRET` | 飞书自建应用 |
| `FEISHU_TABLE_APP_TOKEN`, `FEISHU_TABLE_ID` | 飞书多维表格 App Token & Table ID |
| `FEISHU_WEBHOOK_URL` | 飞书群机器人（Digest + 告警） |
| `FEISHU_DOC_ID` | Workflow B 写入的飞书文档 |
| `WEIBO_APP_KEY`, `WEIBO_APP_SECRET`, `WEIBO_ACCESS_TOKEN`, `WEIBO_UID` | 微博推送凭证（可选） |
| `WECHAT_APPID`, `WECHAT_APPSECRET` | 公众号草稿 API |
| `WECHAT_WHITELISTED_IPS` | 逗号分隔的允许 IP；支持 `*` / `any` |

> `.env.example` 已新增 `WECHAT_WHITELISTED_IPS`，导入后只需按需填值。

## 🚀 快速部署

1. **导入工作流**：在 n8n UI → *Import from file*，分别导入 A/B/C 三个 JSON。
2. **配置变量**：按照上方表格填写 `.env` 或在 Configuration 节点里引用的 Credential 中填写。
3. **激活触发器**：A/B 默认 Cron 触发，可按需调整；C 可切换为选择性手动触发。
4. **验证**：  
   - 运行 Workflow A，检查飞书表格是否出现新记录；  
   - 运行 Workflow B，确认两种情形：IP 在白名单时创建草稿成功；不在白名单时飞书收到告警；  
   - 运行 Workflow C，看是否正确将重复行归类并删除。

## 🆕 v2.1.0 更新摘要

- **Workflow A**：移除所有硬编码的 `app_id/app_secret`，包括 `Refresh Feishu Token` 的 JSON 体，完全依赖环境变量，避免泄露。
- **Workflow B**：新增 “调用微信 token 接口 → 解析 errcode 中 IP → 与 `WECHAT_WHITELISTED_IPS` 比对 → True/False 分支” 的白名单链路，同时将告警直接推送至飞书；`Format Draft Notification` 的成功/失败提示也与分支保持一致。
- **Workflow C**：`Has Duplicates?` 的第二个条件改为数字比较（`duplicate_count === 0`），不再因为类型不符报错。

## 🧪 建议测试用例

1. **飞书凭证异常**：暂时清空 `FEISHU_APP_SECRET`，确保 Workflow A 的报错能在 n8n Logs 中定位；恢复后再次执行。
2. **微信白名单**：临时从 `WECHAT_WHITELISTED_IPS` 中移除当前出口 IP，运行 Workflow B → 应收到飞书告警；随后将 IP 加回并确认草稿成功创建。
3. **重复数据**：在飞书表格手动复制若干条记录，运行 Workflow C，观察 `Summarize Results` 节点输出的删除数量。

## 📦 发布到 GitHub

1. 将 `workflows/ai-news-workflow/ai-news-workflow-v2.1.0/` 复制到仓库，并更新仓库根 README 以引用该版本。
2. 更新 `env.template`，确保新增变量说明同步。
3. 可在 Release/Changelog 中记录 “Workflow B IP 校验、Workflow A 安全化、Workflow C Switch 修复”等要点。

配套文档：

- `docs/setup_guide.md`：环境准备、凭证申请、导入步骤
- `docs/deployment_guide.md`：Docker/服务器部署、升级、排障

如需更多部署场景或 IP 白名单排障，可参考仓库 `docs` 目录或在 Issues 中反馈。
