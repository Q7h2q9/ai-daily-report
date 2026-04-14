# VS_AI — 运行仓库

## 双仓库架构

本项目采用代码与运行分离的双仓库架构：

- **源码仓库** (`Q7h2q9/AI-and-webhook`) — 代码开发、本地测试、文档维护
- **本仓库** (`Q7h2q9/ai-daily-report`) — GitHub Actions 运行、报告数据存储、Pages 站点托管

源码在 AI-and-webhook 开发测试通过后，同步到本仓库，由 Actions 自动执行每日流水线。

## 本仓库的职责

- 运行 GitHub Actions 每日流水线（采集→分析→渲染→推送→归档→Pages）
- 存储报告产物（`data/reports/`、`data/archives/`、`data/state/`）
- 托管 GitHub Pages 公开站（`docs/`）
- 管理 Secrets（API key、推送 token 等）

## 从源码仓库同步

当 AI-and-webhook 有代码改动时，需同步以下文件到本仓库：
- `src/`、`config/`、`tests/` — 运行时代码和配置
- `requirements.txt`、`pyproject.toml` — 依赖定义
- `.github/workflows/` — workflow 定义（如有改动）

源码仓库的 `examples/workflows/` 目录保存了 workflow 文件的参考副本，修改 workflow 时以此为基础。

同步后提交推送即可，Actions 会按 cron 自动执行。

## 项目定位

VS_AI 是一个每日 AI 情报生产工程。自动采集多源 AI 信号 → 去重/评分/聚类 → LLM 三阶段分析 → 多渠道推送 → 归档到 GitHub Pages。

覆盖领域：AI / LLM / Agent、AI × Electronics。

## 数据流

```
collect (RSS/GitHub/HN/Websites)
→ dedup / score / topic builder
→ AI pipeline (analysis → summary → forecast)
→ intelligence layer
→ render (MD/JSON/HTML)
→ publish (auto / reviewed)
→ push (PushPlus / Telegram / Feishu)
→ archive → pages / ops dashboard / review queue
```

## 关键目录

- `src/auto_report/app.py` — 主编排入口
- `src/auto_report/cli.py` — CLI 命令定义
- `src/auto_report/settings.py` — 环境变量与配置
- `src/auto_report/sources/` — 数据采集（RSS/GitHub/HN/Websites）
- `src/auto_report/pipeline/` — 分析管线（去重/评分/AI/主题/情报）
- `src/auto_report/integrations/` — 外部服务（LLM/PushPlus/Telegram/Feishu）
- `src/auto_report/outputs/` — 报告渲染、Pages 构建、归档
- `config/sources/` — 数据源配置
- `config/domains/` — 领域关键词配置
- `config/ai_reading/` — Prompt 模板与 registry
- `config/providers.yaml` — LLM 和推送 provider 配置
- `config/schedules.yaml` — 调度 cron 配置
- `config/prompt_eval/` — Prompt 评估基准数据集

## CLI 命令

```bash
PYTHONPATH=src python -m auto_report.cli <command>
```

- `run-once` — 完整流水线（采集→分析→渲染→推送）
- `backfill --date YYYY-MM-DD` — 指定日期补报
- `render-report` — 仅渲染，不重新采集
- `diagnose-delivery` — 测试推送通道连通性
- `collect-only` — [CI] 仅采集+去重+分类+评分
- `analyze-only` — [CI] 仅跑 AI 三阶段
- `render-and-push` — [CI] 渲染+推送+归档
- `build-pages` — [CI] 构建 GitHub Pages
- `evaluate-prompts` — 离线 Prompt 评估

## 发布模式

- `auto` — 默认，自动发布
- `reviewed` — 人工审核后发布，需传 `--publication-mode reviewed --reviewer <name>`

## GitHub Actions

主入口：`.github/workflows/collect-report.yml`，每天北京时间 07:00 自动触发。

串联子 workflow：guard → test → collect → analyze → report → pages → ops dashboard → review queue

其他 workflow：
- `backfill-report.yml` — 补报
- `compensate-report.yml` — 失败补偿
- `delivery-canary.yml` — 通道健康检查

## 关键状态文件

`data/state/run-status.json` 是最重要的运行状态文件，包含：
- `publication_mode` / `review` — 发布轨和审核信息
- `delivery_results` — 各通道推送结果
- `risk_level` — 本轮风险等级
- `stage_status` — 各阶段执行状态
- `ai_metrics` — LLM provider/model/token 用量
- `source_health` — 数据源健康状态

## 环境变量

必填项见 `.env.example`。关键变量：
- `DEEPSEEK_API_KEY` / `AI_API_KEY` — LLM 密钥
- `PUSHPLUS_TOKEN` / `TELEGRAM_BOT_TOKEN` / `FEISHU_APP_ID` — 推送通道
- `REPORT_REPO_URL` — 本仓库 GitHub 地址
- `PUBLIC_SITE_URL` — Pages 公开站地址

## 测试

```bash
PYTHONPATH=src python -m pytest tests -q
```

基线：191 passed

## 扩展指南

- 新增数据源：改 `config/sources/*.yaml` + `src/auto_report/sources/*.py` + 测试
- 新增推送通道：改 `integrations/` + `renderers.py` + `app.py` + 测试
- 修改 Prompt：改 `config/ai_reading/registry.json` + 对应 `.md` + eval dataset

## 本仓库的职责

- 运行 GitHub Actions 每日流水线（采集→分析→渲染→推送→归档→Pages）
- 存储报告产物（`data/reports/`、`data/archives/`、`data/state/`）
- 托管 GitHub Pages 公开站（`docs/`）
- 管理 Secrets（API key、推送 token 等）

## 从源码仓库同步

当 AI-and-webhook 有代码改动时，需同步以下文件到本仓库：
- `src/`、`config/`、`tests/` — 运行时代码和配置
- `requirements.txt`、`pyproject.toml` — 依赖定义
- `.github/workflows/` — workflow 定义（如有改动）

同步后提交推送即可，Actions 会按 cron 自动执行。

## 工程约束

- 本地改完 workflow 必须先 push 再触发线上验收（workflow_dispatch 执行远端版本）
- 出问题优先看 `data/state/run-status.json`
- 页面、推送、状态文件三者要一起看
- 不要在本仓库直接改源码，改动应在源码仓库 (`AI-and-webhook`) 完成后同步过来
