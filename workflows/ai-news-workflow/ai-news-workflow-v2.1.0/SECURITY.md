# AI News Workflow v2.1.0 – 安全说明

## 已处理的敏感信息

| 类型 | 处理方式 |
| --- | --- |
| 飞书凭证 (`FEISHU_APP_ID`, `FEISHU_APP_SECRET`, `FEISHU_TABLE_APP_TOKEN`, `FEISHU_TABLE_ID`, `FEISHU_WEBHOOK_URL`, `FEISHU_DOC_ID`) | 均由 `Configuration` 节点从环境变量读取 |
| Weibo / DeepSeek / WeChat 凭证 | 统一通过 `{{ $env.X }}` 注入 |
| Workflow B 出口 IP 白名单 | 新增 `WECHAT_WHITELISTED_IPS` 环境变量，告警时提示更新 |

> `env.template` 仅作示例，勿将填好密钥的 `.env` 提交或共享。

## 最佳实践

1. **环境变量管理**：复制 `env.template` → `.env`，设置权限为 600，并已在 `.gitignore` 中忽略。
2. **凭证轮换**：建议 90 天轮换一次飞书/微信/微博密钥；收到 Workflow B 告警后，优先更新白名单。
3. **网络防护**：开启 n8n Basic Auth 或放在内网，微信 API 必须使用固定出口 IP。
4. **数据安全**：定期备份 `.n8n` 数据卷；飞书表格设置字段级权限；日志中勿打印敏感内容。

## 发布前检查清单

- [ ] `.env` 未被提交。
- [ ] `WECHAT_WHITELISTED_IPS` 与公众平台白名单同步。
- [ ] 工作流 JSON 内无明文密钥或 webhook token。
- [ ] n8n 实例启用认证/HTTPS。
- [ ] 飞书应用和表格权限最小化。

若发现安全问题，请先更换所有密钥，再通过 GitHub Issues 反馈（勿在公开渠道贴出凭证）。***
