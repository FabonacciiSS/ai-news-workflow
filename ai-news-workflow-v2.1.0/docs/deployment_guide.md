# 部署指南 – AI News Workflow v2.1.0

## 1. Docker Compose

```bash
git clone https://github.com/FabonacciiSS/ai-news-workflow.git
cd ai-news-workflow
cp env.template .env
docker-compose up -d
```

升级：
```bash
docker-compose pull
docker-compose down
docker-compose up -d
```

## 2. 云服务器部署要点

| 项目 | 建议 |
| --- | --- |
| OS | Ubuntu 22.04 / Debian 12 |
| 规格 | 2C4G 起，SSD ≥ 40GB |
| 防火墙 | 仅开放 22/80/443（5678 建议走内网或反代） |

1. 安装 Docker 与 Docker Compose。
2. 拉取项目、填写 `.env`。
3. 通过 Nginx/Traefik 提供 HTTPS + Basic Auth。

## 3. 生产环境运维

- `.env` 权限设为 600，仅管理员可读。
- `WECHAT_WHITELISTED_IPS` 与公众平台同步，Workflow B 告警后及时更新。
- 每日备份 `~/.n8n` 或 Docker 数据卷；飞书表格可导出 CSV 保存。
- 日志查看：
  ```bash
  docker logs -f n8n
  docker-compose logs -f
  ```

## 4. 故障排查

| 场景 | 处理 |
| --- | --- |
| 微信接口 40164 | 检查飞书告警中的 IP，加入 `.env` 与公众平台白名单 |
| 飞书 900010 权限不足 | 确认应用授权了多维表格读写，并使用正确表格 ID |
| 工作流长时间执行 | 查看 Execution Logs，必要时增加 `Wait` 或重试机制 |

## 5. 安全加固

- 启用 HTTPS + Basic Auth。
- 微信 API 一定使用固定出口 IP。
- 参考 `../SECURITY.md` 完成上线前检查清单。

## 6. 发布包结构

```
ai-news-workflow-v2.1.0/
├─ .gitignore
├─ .gitattributes
├─ README.md
├─ SECURITY.md
├─ env.template
├─ docs/
│  ├─ setup_guide.md
│  └─ deployment_guide.md
├─ Workflow A - AI News collector v2.1.0.json
├─ Workflow B - AI News digest v2.1.0 (Feishu + WeChat Draft).json
└─ Workflow C - Feishu Data Deduplication Tool v2.1.0.json
```

压缩该目录后上传到 GitHub Release，用户即可按照 `docs/setup_guide.md` 完成导入与部署。
