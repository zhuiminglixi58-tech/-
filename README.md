# 每日财经案例精选 · Kimi API 版本

> 每天早上 8 点，通过 **Kimi K2.5 + `$web_search` 联网搜索**，自动从凤凰财经、新浪财经、腾讯财经筛选 5 个案例，深度分析最值得写作的一个，通过**飞书**推送。

## 技术架构

```
GitHub Actions (定时触发)
    └─ scripts/daily_case.py
          ├─ Step 1: Kimi K2.5 + $web_search → 筛选5个候选案例（JSON）
          ├─ Step 2: Kimi K2.5 + $web_search → 选出最佳案例 + 深度分析（JSON）
          └─ Step 3: 飞书 Webhook → 推送富文本卡片消息
```

### Kimi API 关键特性

| 特性 | 说明 |
|---|---|
| 模型 | `kimi-k2.5`（最新，支持多模态与联网） |
| 联网搜索 | 内置 `builtin_function.$web_search`，无需外部搜索 API |
| SDK | OpenAI 兼容，`base_url = https://api.moonshot.cn/v1` |
| 搜索限制 | 使用 `$web_search` 时需关闭 thinking（`enable_thinking: False`） |
| 上下文 | 支持 256K 超长上下文 |

---

## 飞书消息内容

每天推送一张富文本卡片，包含：

- **今日5个候选案例速览**：来源、企业、记者论点、写作空白
- **⭐ 重点案例深度分析**：
  - 推荐理由 & 核心研究问题
  - 一、事件背景与关键数据
  - 二、核心矛盾界定（记者提出了什么 / 没有论证什么）
  - 三、推荐分析框架（2套）
  - 四、写作提纲（5章）
  - 五、管理启示（3条）
  - 原文链接

---

## 部署步骤（约 10 分钟）

### 第一步：获取 Kimi API Key

1. 前往 [platform.moonshot.cn](https://platform.moonshot.cn) 注册/登录
2. 进入 **API Key 管理** → **新建 API Key**
3. 复制 Key（格式：`sk-...`）

### 第二步：配置飞书机器人

1. 打开飞书，进入目标群聊
2. **群设置** → **群机器人** → **添加机器人** → **自定义机器人**
3. 名称填「财经案例精选」，复制 **Webhook URL**

> 💡 建议启用**签名校验**提高安全性（需在代码中添加签名逻辑）

### 第三步：Fork 仓库并配置 GitHub Secrets

在 GitHub 仓库：**Settings → Secrets and variables → Actions → New repository secret**

| Secret 名称 | 值 |
|---|---|
| `MOONSHOT_API_KEY` | Kimi 平台的 API Key |
| `FEISHU_WEBHOOK_URL` | 飞书自定义机器人的 Webhook URL |

### 第四步：启用 Actions 并测试

1. 进入仓库 **Actions** 标签页，确认工作流已启用
2. 点击「每日财经案例精选（Kimi）」→ **Run workflow** 手动触发
3. 等待约 3–8 分钟（Kimi 多轮联网搜索需要一些时间）
4. 检查飞书群是否收到卡片消息

---

## 调整定时时间

修改 `.github/workflows/daily_case.yml` 中的 cron：

```yaml
# 北京时间 = UTC + 8
'0 0 * * 1-5'    # UTC 00:00 → 北京 08:00（当前默认，周一至周五）
'0 1 * * 1-5'    # UTC 01:00 → 北京 09:00
'0 23 * * 0-4'   # UTC 23:00 → 北京 07:00（前一天）
'0 0 * * *'      # 每天包括周末
```

---

## 费用估算

Kimi K2.5 定价（参考 [platform.moonshot.cn](https://platform.moonshot.cn/docs/pricing)）：

| 项目 | 说明 |
|---|---|
| 输入 Token | ¥0.004/千 token（缓存后更低） |
| 输出 Token | ¥0.016/千 token |
| `$web_search` | 约 ¥0.035/次调用 |
| 每日估算 | 约 ¥0.5–1.5 元（含 4–8 次搜索调用） |
| 每月估算（工作日）| 约 ¥10–30 元 |

---

## 文件结构

```
finance-case-bot-kimi/
├── .github/
│   └── workflows/
│       └── daily_case.yml      # GitHub Actions 定时任务
├── scripts/
│   └── daily_case.py           # 核心脚本
└── README.md
```

---

## 常见问题

**Q: JSON 解析失败？**
Kimi 偶尔会在 JSON 外包裹 markdown 代码块，脚本已内置容错处理。若仍失败，重新手动触发即可。

**Q: 搜索结果为空或质量差？**
可在 system prompt 中增加搜索关键词引导（如加入具体行业词），或将模型换为 `moonshot-v1-128k`（稳定性更高）。

**Q: Actions 超时？**
Kimi 多轮 tool_call 在网络慢时可能超时，workflow 已设置 20 分钟上限。如仍超时，可将 Step 1 和 Step 2 的 `max_tokens` 适当减小。

**Q: 如何本地测试？**
```bash
export MOONSHOT_API_KEY="sk-..."
export FEISHU_WEBHOOK_URL="https://open.feishu.cn/open-apis/bot/v2/hook/..."
pip install openai requests
python scripts/daily_case.py
```
