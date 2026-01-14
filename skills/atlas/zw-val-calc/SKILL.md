---
name: atlas-zw-val-calc
description: 🗺️ Project atlas for zw-val-calc (fund valuation calculation system v2.2). Use when writing code, adding features, fixing bugs, or running workflows in this project. Covers trade analysis, transfer calculation, valuation workflows, and Prefect orchestration.
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
  - Bash
  - StrReplace
---

# 🗺️ Atlas - ZW Val Calc 项目地图

基金估值计算系统 v2.2，用于日常净值估算与验证。

## 核心功能

| 模块 | 功能 | 入口脚本 |
|------|------|----------|
| **Trade** | 日收益、周收益、换手率、成交率 | `run_trade_workflow.py` |
| **Transfer** | 盘前资金调拨（基于指数权重） | `run_transfer_workflow.py` |
| **Valuation** | 净值计算、保证金监控、持仓验证 | `run_valuation_workflow.py` |
| **Notification** | 飞书消息卡片推送 | 各 workflow 内置 |

## 技术栈

| 组件 | 用途 |
|------|------|
| Python 3.12+ | 运行环境 |
| PostgreSQL | 主数据源（via `PostgresClient`） |
| ClickHouse | 行情/CTP 数据 |
| Polars | 高性能数据处理 |
| Prefect | 工作流编排 |
| FastAPI | API 服务 |

## 项目结构

```
zw-val-calc/
├── scripts/                    # Prefect 工作流入口
│   ├── run_trade_workflow.py
│   ├── run_transfer_workflow.py
│   ├── run_valuation_workflow.py
│   └── deploy_workflow.py      # 部署到 Prefect Server
├── tools/                      # 运维/调试/对账脚本（非 Prefect）
├── zw_val_calc/
│   ├── api/                    # FastAPI 接口
│   ├── config/
│   │   ├── settings.py         # Pydantic Settings（核心配置）
│   │   ├── products.py         # 产品配置枚举
│   │   └── custodians.py       # 托管方配置
│   ├── io/
│   │   ├── clients/            # 数据库客户端
│   │   │   ├── postgres.py     # PostgresClient（高性能）
│   │   │   └── clickhouse.py   # ClickHouse 客户端
│   │   ├── loaders/            # 数据加载器
│   │   └── parsers/            # 文件解析器
│   ├── models/                 # Pydantic 模型 & 枚举
│   ├── services/
│   │   ├── trade/              # 交易分析服务
│   │   │   ├── calculators/    # 收益/换手/成交计算器
│   │   │   └── tasks.py        # Prefect tasks
│   │   ├── transfer/           # 转账服务
│   │   │   ├── calculators/    # 指数权重/转账金额
│   │   │   └── orchestrator.py
│   │   └── valuation/          # 估值服务
│   │       ├── calculators/    # NAV/保证金/赎回
│   │       ├── parsers/        # 估值表解析
│   │       └── verifiers/      # 净值/持仓验证
│   ├── notification/           # 飞书通知
│   └── utils/                  # 工具函数
├── sql/                        # DDL 初始化脚本
├── tests/                      # pytest 测试
└── docs/                       # 文档
```

## 快速开始

### 获取配置和客户端

```python
from zw_val_calc.config.settings import get_settings

settings = get_settings()

# 数据库客户端
pg_client = settings.get_postgres_client()      # PostgreSQL 生产库
bt_client = settings.get_backtest_client()      # PostgreSQL 回测库
ch_client = settings.get_clickhouse_client()    # ClickHouse

# 数据加载器
fund_loader = settings.get_fund_account_loader()
product_loader = settings.get_product_loader()
api_loader = settings.get_api_loader()          # 转账 API
```

### 运行工作流

```bash
# 使用 uv 运行（必须）
uv run python scripts/run_trade_workflow.py
uv run python scripts/run_transfer_workflow.py
uv run python scripts/run_valuation_workflow.py

# 部署到 Prefect Server
uv run python scripts/deploy_workflow.py
```

### 运行测试

```bash
uv run pytest
uv run pytest tests/unit/ -v
uv run pytest --cov=zw_val_calc
```

## 环境变量配置

通过 `.env` 文件或环境变量配置，使用 `__` 分隔嵌套：

```bash
# PostgreSQL
POSTGRES__HOST=10.242.0.16
POSTGRES__PORT=5432
POSTGRES__USER=myuser
POSTGRES__PASSWORD=mypassword
POSTGRES_PRD_DATABASE=zouwu
POSTGRES_BT_DATABASE=backtest

# ClickHouse
CLICKHOUSE__HOST=10.242.0.127

# 飞书 Webhook
WEBHOOKS__NAV_URL=https://open.feishu.cn/...
```

## 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 数据库表名 | PascalCase (带引号) | `"FundAccount"`, `"MarketValue"` |
| 数据库列名 | camelCase | `fundAccount`, `tradeDay` |
| 时间列 | snake_case | `trade_day` |
| Python 变量 | snake_case | `fund_account`, `trade_day` |
| Prefect Task | snake_case 函数 | `@task def load_position_data()` |

## 工作流模式

### Orchestrator Pattern

```python
class TradeAnalysisOrchestrator:
    def __init__(self, settings: Settings):
        self.pg_client = settings.get_postgres_client()
        self.fund_loader = settings.get_fund_account_loader()
    
    def run(self, target_date: date) -> dict:
        # 1. 加载数据
        positions = self.pg_client.read_table(...)
        # 2. 计算
        returns = ReturnsCalculator(...).calculate(positions)
        # 3. 通知
        send_feishu_notification(returns)
```

### Prefect Task 模式

```python
from prefect import task, flow

@task
def load_position_data(client, date: date) -> pl.DataFrame:
    return client.read_table("public", "Position", date, date)

@flow
def trade_workflow(target_date: date):
    settings = get_settings()
    client = settings.get_postgres_client()
    positions = load_position_data(client, target_date)
    # ...
```

## 关键规则

- **必须用 `uv run`** 执行所有 Python 命令
- **禁止 `pip install`** - 使用 `uv add` 添加依赖
- **NUMERIC 列用 `binary=False`** - 金融精度数据必须
- **业务逻辑在 Polars** - 数据库只负责存取

## 相关资源

- **PostgreSQL 模式**: 使用 `postgres-client` skill
- **企业知识库**: 使用 `oracle` skill
- **Git 规范**: 使用 `git-workflow` skill
- **项目文档**: 见 `docs/getting_started.md`
