---
name: oracle
description: 🔮 Enterprise knowledge base containing internal database schemas, system configurations, and company-specific technical references. Use when querying internal databases, understanding data models, or working with company infrastructure.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# 🔮 Oracle - 企业知识库

内部系统的技术参考文档。

## PostgreSQL 表参考

### 核心配置表

| 表名 | 用途 | 备注 |
|------|------|------|
| `Product` | 产品主表 | 产品基本信息 |
| `FundAccount` | 账号对应表 | 基金账户配置 |
| `SubscriptionRedemptionRecord` | 申赎记录 | 申购/赎回记录 |
| `TransferRecord` | 转账记录 | 资金划转 |

### 预估净值历史序列表

| 表名 | 用途 |
|------|------|
| `nav_custodian` | 托管净值（真实值，从估值表解析） |
| `nav_est` | 预估净值 |
| `margin_custodian` | 托管保证金（真实值，从swap文件解析） |
| `margin_est` | 预估保证金 |
| `option_custodian` | 期货账户结算（真实值，从结算单解析） |
| `position_verify` | 多空市值验证 |

### 交易数据表

| 表名 | 用途 |
|------|------|
| `Trade` | 成交记录 |
| `Order` | 订单记录 |
| `Position` | 持仓数据 |
| `MarketValue` | 市值数据 |
| `InnerFundSnapshot` | 账户资金快照 |
| `FuturesAccountInfo` | 期货账户信息 |

### 行情数据表

| 表名 | 用途 |
|------|------|
| `QuoteBrief` | 证券/期货收盘价 |
| `IndexWeight` | 指数权重 |
| `AlphaIndexWeight` | Alpha指数权重 |

## 常用查询方法

```python
from zw_val_calc.config.settings import get_settings

client = get_settings().get_postgres_client()

# 配置数据
client.read_fund_account(active_only=True)
client.read_product(active_only=True)

# 期货账户
client.read_futures_account_info(trading_day=date(2024, 1, 15))

# 指数权重
client.read_index_weight(index_code="000905.SH")
client.read_alpha_index_weight(index_code="000300.SH")
```
