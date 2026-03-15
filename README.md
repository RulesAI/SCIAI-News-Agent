# SCI.AI 全球供应链智能编辑部

<p align="center">
  <strong>基于 OpenClaw 框架的多 Agent 新闻生产平台</strong>
</p>

<p align="center">
  <a href="#架构概览">架构</a> · <a href="#agent-团队">Agent 团队</a> · <a href="#核心能力">核心能力</a> · <a href="#技术栈">技术栈</a> · <a href="#部署架构">部署</a> · <a href="#skill-清单">Skills</a>
</p>

---

## 项目简介

SCI.AI 全球供应链智能编辑部是一套**全自动化的供应链行业智能新闻编辑部**。基于 [OpenClaw](https://github.com/openclaw/openclaw) 开源框架，深度定制了多 Agent 协作、内容安全审核、多渠道分发等能力，实现从情报采集 → 内容生成 → 质量审核 → 多平台发布的端到端自动化。

系统当前部署在两台服务器上，运行 **8 个专业化 Agent**，承载 **60+ 定时任务**，每日自动产出供应链行业资讯并分发至 WordPress、微信公众号、知乎、小红书、X (Twitter) 等平台。

## 架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                     SCI.AI Agent 编辑部                       │
├──────────────────────┬──────────────────────────────────────┤
│   NAS (生产服务器)     │      MateBook X (辅助节点)            │
│                      │                                      │
│  赵子龙 — 总编辑       │  林紫涵 — KOL 运营总监                │
│  陆文澜 — 编辑发文     │  Iris — 小红书运营                    │
│  秦鉴远 — 情报研究     │  Stella — SCI.AI 小红书               │
│  智链进化论 — 公众号    │                                      │
├──────────────────────┴──────────────────────────────────────┤
│                    Telegram Bot 通讯层                        │
│  @Simon_Main_bot · @Landon_sciai_bot · @Raymond_SCIAI_bot   │
│  @EvoChain_LinX_bot · @linzihan_bot                        │
└─────────────────────────────────────────────────────────────┘
```

## Agent 团队

| Agent          | 角色         | 职责                               | 服务器     |
| -------------- | ------------ | ---------------------------------- | ---------- |
| 赵子龙 Draco   | 总编辑       | 任务派发、内容审核、团队协调       | NAS        |
| 陆文澜 Landon  | 编辑         | 文章撰写、WordPress 发布、质量把关 | NAS        |
| 秦鉴远 Raymond | 研究员       | 情报采集、行业分析、深度研究       | NAS        |
| 智链进化论     | 公众号编辑   | 微信公众号内容同步                 | NAS        |
| 林紫涵         | KOL 运营总监 | 跨平台内容策略、下属管理           | MateBook X |
| Iris           | XHS 运营     | 小红书内容创作与互动               | MateBook X |
| Stella         | XHS 运营     | SCI.AI 官方小红书运营              | MateBook X |

## 核心能力

### 1. 草稿优先发布流水线

```
情报采集 → LLM 内容生成 → 草稿提交 → 自动质量审核 → 发布/修复/废弃
```

- **四层 LLM 兜底链**：DashScope → DeepSeek → ZhipuAI → Google Gemini，任一层失败自动切换
- **双 API Key 机制**：Agent Key（仅创建草稿）+ Validator Key（审核通过后发布），权限隔离
- **质量门三档**：PASS → 直接发布 | FIXABLE → 自动修复后发布 | FATAL → 废弃

### 2. 内容安全审核体系

- **敏感词检测**：53+ 中文 + 11 英文编码敏感词库，覆盖政治、宗教、争议话题
- **AI 声明规范化**：自动清除 LLM 产生的各类声明格式，统一为标准格式追加到文末
- **高风险话题预警**：涉及特定领域时自动提醒，确保措辞中性客观
- **封面图智能处理**：自动下载 og:image → 格式转换 → 空白检测 → Pexels 备选 → 唯一路径防重复
- **阿里云内容安全 API**：集成阿里云内容安全服务，双重审核

### 3. 多渠道自动分发

| 渠道                        | 能力                | 自动化程度 |
| --------------------------- | ------------------- | ---------- |
| WordPress (news.yrules.com) | 全自动发布 + 封面图 | 全自动     |
| 微信公众号（智链进化论）    | 文章同步 + 格式适配 | 全自动     |
| 知乎                        | 专栏文章发布        | 全自动     |
| 小红书                      | 图文/视频内容       | 半自动     |
| X (Twitter)                 | 推文发布            | 全自动     |

### 4. Agent 自主进化体系

```
实时层: Agent 自主创建 Skill 脚本（Watcher 热加载，250ms 生效）
每日层: 工作反思 → MEMORY.md 自动更新 → 行为模式优化
每周层: Evolver 基因进化引擎 → 代码级变更 → 能力迭代
```

- Agent 可自主创建、注册、测试新 Skill，无需人工干预
- 每日反思 Cron 自动总结经验教训，写入持久化记忆
- 周度进化引擎分析历史数据，提出并实施代码优化

### 5. 跨 Agent 任务协作

- 总编辑 → 编辑/研究员：通过 TASKS.json 直接派发任务
- 下属 Heartbeat 自动拾取新任务
- 兄弟 Agent 间通过 workspace 文件通讯
- 跨服务器任务协调（NAS ↔ MateBook X）

### 6. 生产级运维保障

- **三层容器防护**：Docker 重启策略 + 监控脚本（5 分钟巡检）+ crash loop 保护
- **Provider Failover**：API 账号自动轮换，billing/auth 错误自动切换备用账号
- **Cron Guard**：每小时备份定时任务配置，异常时自动恢复
- **Portainer + Tailscale**：远程容器管理 + 异地网络接入

## 技术栈

| 组件   | 技术                                                               |
| ------ | ------------------------------------------------------------------ |
| 框架   | [OpenClaw](https://github.com/openclaw/openclaw) (TypeScript, ESM) |
| 运行时 | Node.js 22+ / Bun                                                  |
| LLM    | DeepSeek V3/R1, DashScope Qwen, Google Gemini, xAI Grok            |
| 通讯   | Telegram Bot API (Grammy)                                          |
| CMS    | WordPress REST API + 自定义插件                                    |
| 容器   | Docker (NAS) / 原生进程 (MateBook X)                               |
| 代理   | Clash (NAS 容器网络)                                               |
| 监控   | Portainer, Tailscale, 自定义健康检查                               |

## 部署架构

### NAS (ZSpace, 192.168.3.185)

- **容器**: `openclaw-gateway`，单容器单进程
- **网络**: Bridge + openclaw-net (Clash 代理)
- **存储**: `/data_ZR5D8EL4/openclaw/config` → 容器 `/home/node/.openclaw`
- **Skills**: 45 个，覆盖发文、研究、运营、监控、自主进化

### MateBook X (Ubuntu 24.04, 192.168.3.119)

- **运行**: 原生 OpenClaw Gateway 进程
- **Skills**: 63 个，含金融监控、小红书专属、深度研究等

## Skill 清单

### 内容生产

`supply-news-publisher` · `daily-supply-news-digest` · `weekly-report` · `wechat-article-sync` · `zhihu-publisher` · `xiaohongshu-publisher` · `x-twitter` · `video-producer` · `evochain-publish` · `evochain-topics`

### 情报研究

`exa` · `sci-report-analyst` · `youtube-transcript` · `pdf-extract` · `deep-research-pro` · `news-summary`

### 运营监控

`site-analytics` · `site-traffic-booster` · `cover-image-guard` · `cover-patrol` · `user-advocate` · `user-census` · `ops-health-check` · `server-ops`

### 自主进化

`evolver` · `self-improving` · `self-heal` · `skill-creator` · `workspace-cleanup`

### 小红书专属

`xiaohongshu-engager` · `xiaohongshu-publisher-sciai` · `xiaohongshu-cover-image-generator` · `xiaohongshu-data-fetcher` · `xhs-engager-trigger`

## 目录结构

```
deploy/
  nas/skills/           # NAS 部署的 Skill 快照 (45 个)
  matebook/skills/      # MateBook X 部署的 Skill 快照 (63 个)
  README.md             # 本文件
```

## 许可

基于 [OpenClaw](https://github.com/openclaw/openclaw) 开源框架（MIT License）构建。业务定制部分为 SCI.AI 内部使用。
