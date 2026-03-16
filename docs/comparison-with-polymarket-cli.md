# rs-clob-client vs polymarket-cli 对比

## 项目定位

| | rs-clob-client | polymarket-cli |
|---|---|---|
| **类型** | Rust SDK 库（发布在 crates.io） | 命令行工具（可执行二进制） |
| **版本** | 0.4.3 | 0.1.4 |
| **代码量** | ~34,585 行 | ~3,620 行 |
| **MSRV** | Rust 1.88.0 | Rust 1.88.0 |
| **仓库** | https://github.com/polymarket/rs-clob-client | https://github.com/Polymarket/polymarket-cli |

**核心关系：polymarket-cli 依赖 `polymarket-client-sdk = "0.4"`，是基于 rs-clob-client 构建的上层应用。**

## 功能差异

### rs-clob-client 独有

- WebSocket 实时流（订单簿、价格、中点价格、用户事件）
- 自动心跳（`heartbeats` feature）
- AWS KMS 远程签名
- RFQ 报价接口（`rfq` feature）
- 实时数据流 RTDS（加密货币价格、评论，`rtds` feature）
- 类型状态编译期安全（`Client<Unauthenticated>` vs `Client<Authenticated<K>>`）
- 并发缓存（DashMap 缓存 tick_size / neg_risk / fee_rate_bps）
- 结构化日志与未知字段告警（`tracing` feature）

### polymarket-cli 独有

- 交互式 REPL（`shell` 命令）
- 钱包管理（`wallet create` / `import` / `show` / `reset`）
- 引导式设置向导（`setup` 命令）
- 自动升级（`upgrade` 命令）
- 本地配置文件持久化（`~/.config/polymarket/config.json`）
- 表格 / JSON 双输出格式（`--output table|json`）
- 三级配置优先级（CLI 标志 > 环境变量 > 配置文件）

### WebSocket 支持

| 特性 | rs-clob-client | polymarket-cli |
|---|---|---|
| WebSocket 客户端 | 有（`ws` feature） | **无** |
| 订单簿实时流 | 有 | 无 |
| 用户事件流 | 有 | 无 |
| 实时价格推送 | 有 | 无 |
| 心跳自动化 | 有（`heartbeats` feature） | 无 |
| 断线自动重连 | 有（指数退避） | 无 |

polymarket-cli 是纯 HTTP 请求/响应模式，所有数据为一次性拉取，不支持实时推送。

## 架构差异

### rs-clob-client

- 单 crate 库，无二进制目标
- Feature-gated 模块化设计（clob / gamma / data / bridge / ctf / ws / rtds）
- 类型安全的构建器模式（`OrderBuilder<Limit, K>` / `OrderBuilder<Market, K>`）
- 类型级状态机防止未认证调用
- 分层 WebSocket 架构：通用 `ConnectionManager<M, P>` + CLOB 特化的 `InterestTracker` + `SubscriptionManager`
- 零成本抽象，编译期安全检查

### polymarket-cli

- 独立可执行二进制
- clap 命令解析 + 每命令一模块（`commands/`）+ 独立输出格式化层（`output/`）
- 轻量化编译（strip = true, LTO = thin, panic = abort）
- 本地配置管理与钱包生命周期

## 认证方式

| 方面 | rs-clob-client | polymarket-cli |
|---|---|---|
| LocalSigner（本地私钥） | 有 | 有 |
| AWS KMS | 有 | 无 |
| Proxy 钱包 | 有 | 有（默认） |
| Gnosis Safe | 有 | 有 |
| 配置持久化 | 无（调用者管理） | 有（配置文件） |

## CLOB 操作对比

| 功能 | rs-clob-client | polymarket-cli |
|---|---|---|
| 获取价格 | `client.get_price()` | `clob price TOKEN_ID --side buy` |
| 批量价格 | `client.get_prices()` | `clob batch-prices "T1,T2" --side buy` |
| 中点价格 | `client.get_midpoint()` | `clob midpoint TOKEN_ID` |
| 订单簿 | `client.get_order_book()` | `clob book TOKEN_ID` |
| 限价单 | `client.create_order()` | `clob create-order --token T --side buy --price 0.50 --size 10` |
| 市价单 | `client.market_order()` | `clob market-order --token T --side buy --amount 5` |
| 取消订单 | `client.cancel_order()` | `clob cancel ORDER_ID` |
| 查询余额 | `client.get_balance()` | `clob balance --asset-type collateral` |
| 查询订单 | `client.get_orders()` | `clob orders` |
| 查询交易 | `client.get_trades()` | `clob trades` |

## Feature Flags（rs-clob-client）

| Feature | 用途 | polymarket-cli 是否启用 |
|---|---|---|
| `clob` | 核心 CLOB 客户端 | 是 |
| `data` | 数据 API | 是 |
| `gamma` | Gamma 市场发现 API | 是 |
| `bridge` | 跨链存款 | 是 |
| `ctf` | 条件代币框架 | 是 |
| `ws` | WebSocket 实时流 | **否** |
| `rtds` | 实时数据流 | **否** |
| `rfq` | RFQ 报价 | **否** |
| `heartbeats` | 自动心跳 | **否** |
| `tracing` | 结构化日志 | **否** |

## 适用场景

| 场景 | 推荐使用 |
|---|---|
| 自动交易机器人 | rs-clob-client |
| 实时订单簿监控 | rs-clob-client（WebSocket） |
| 自定义交易界面 | rs-clob-client |
| 程序化市场分析 | rs-clob-client |
| AWS KMS 远程签名 | rs-clob-client |
| 命令行手动交易 | polymarket-cli |
| Shell 脚本自动化 | polymarket-cli |
| 快速原型验证 | polymarket-cli |
| 不需要写代码的用户 | polymarket-cli |
| 交互式探索市场 | polymarket-cli（REPL） |
