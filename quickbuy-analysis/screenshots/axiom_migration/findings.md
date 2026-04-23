# Axiom Migration Snipe 竞品分析

**日期**: 2026-04-23 17:21-17:28
**平台**: Axiom (axiom.trade)
**示例 Token**: Red Bull (7fGd...pygc) — SOL, Pump.fun
**Token 状态**: Migrating (Bonding Curve 填满，迁移中)

---

## 截图说明

| # | 文件 | 内容 |
|---|------|------|
| 01 | final_stretch_list.png | Final Stretch 列表，Red Bull 显示 Migrating 标签，右侧有 "0.02 SOL" Snipe 按钮 |
| 02 | token_detail_snipe.png | Token 详情页，B.Curve 显示 "Migrating"，右侧交易面板有 "Snipe RED BULL" 按钮 |
| 03 | adv_migration_order.png | Adv. 高级面板 → Migration 订单类型，可设金额 + 滑点 + Priority Fee + Bribe Fee |
| 04 | bribe_fee_error.png | 错误提示 "Limit order creation failed: Bribe fee must be at least 0.000001 SOL"，列表显示 Migrating 9% |
| 05 | fee_warning.png | 警告 "Your snipe fees are low. Consider increasing to at least 0.000036 priority fee and 0.044 bribe fee"，Migrating 23% |
| 06 | network_polling.png | DevTools Network 面板，显示客户端持续轮询 ping / gethealth / getTipAccounts / aura |

---

## 关键发现

### 1. 三种下单方式并存

Axiom 在 Migrating 状态下提供三种入口：

| Tab | 功能 | 按钮文案 |
|-----|------|----------|
| **Market** | 一键 Snipe，迁移完成自动成交 | "Snipe RED BULL" |
| **Limit** | 限价单，迁移完成后按设定价执行 | "Place Order" |
| **Adv. → Migration** | 专用迁移订单，可配置金额/fee/bribe | "Place Order" |

**注意**: Market tab 的 "Snipe" 按钮和 Adv. → Migration 是两个不同入口，后者提供更多配置选项。

### 2. 双 Fee 结构（Priority Fee + Bribe/Jito Tip）

Axiom 明确区分两种费用：
- **Priority Fee**: Solana 交易优先级费（默认 0.001 SOL）
- **Bribe Fee**: Jito Bundle 小费（最低 0.000001 SOL）

**平台建议值**（截图05）:
- Priority Fee ≥ 0.000036 SOL
- Bribe Fee ≥ **0.044 SOL**（约 $3.7）

→ Axiom 认为要在 migration snipe 中有竞争力，仅 Jito tip 就需要 ~$3.7

### 3. Migrating 进度追踪

列表中显示迁移百分比（截图04: 9%, 截图05: 23%），实时更新。
用户可以看到 Bonding Curve 迁移进度。

### 4. 交易触发机制 — 客户端轮询

DevTools Network 面板（截图06）显示迁移等待期间，客户端持续发起请求：

| 请求 | 频率 | 推测用途 |
|------|------|----------|
| `ping` | 每 ~3-5s | 保活 |
| `gethealth?api-key=Axiom...` | 每 ~3-5s | 检查 RPC 节点健康 + 确认自用 API key |
| `getTipAccounts` | 每 ~3-5s | 获取最新 Jito Tip 账户列表 |
| `aura` | 每 ~5-10s | Axiom 内部服务（可能是状态同步） |

**关键判断**: 这是**客户端轮询模式**，不是纯服务端触发。客户端持续：
1. 检查 RPC 健康状态
2. 预获取 Jito tip 账户（随时准备构建 bundle）
3. 通过 aura 服务同步迁移状态

**但 snipe 订单本身可能已提交到服务端**（截图04 显示 "Limit order creation failed" → 说明订单是提交到 Axiom 服务端的）。

→ **结论**: 可能是混合模式 — 订单提交到服务端，但客户端也在主动监控，做好随时发交易的准备。需要进一步抓包确认迁移完成时交易是从客户端还是服务端发出。

### 5. Axiom 自有 RPC

`gethealth?api-key=Axiom...` 表明 Axiom 使用带 API key 的 RPC 端点。结合 URL 中的域名，可以推断他们使用的 RPC provider。

### 6. 其他观察

- **Final Stretch** 列表名称 = 即将毕业的 Token 专区，对应我们说的 "Migrating" 阶段
- **Migrated** 列表在右侧，是已毕业 Token 的独立区域
- Token 卡片上的 "0.02 SOL" 按钮是快捷 Snipe（Preset 金额），类似我们的 Quickbuy
- 页面同时显示 Bonding Curve 数据（MC $94K, Vol $947, TX 296）

---

## 7. 三平台实时行为对比（关键发现）

**验证时间**: 2026-04-23
**验证方法**: 同时打开 Axiom / gmgn / Binance Web3，查看同一 Token 的交易入口

| Token | Axiom | gmgn | Binance Web3 |
|-------|-------|------|-------------|
| Red Bull (7fGd...pygc) | **Snipe**（Migrating 状态） | Buy（直接买入） | Buy（直接买入） |
| NVDIA (Gnox...SYoD) | **Snipe**（Migrating 状态） | Buy（直接买入） | Buy（直接买入） |

两个 Token 均在 Meteora DBC（Dynamic Bonding Curve）上交易，尚未毕业。

### 差异分析

**gmgn / Binance 的 "Buy"** = 路由到当前 Bonding Curve 内盘直接成交，用户立即买到。

**Axiom 的 "Snipe"** = 检测到 Token 接近毕业，不走内盘，预挂单等外盘开盘第一时间冲入。

### 核心结论：Axiom 在 Migrated 场景的结构性速度优势

Axiom 的 Snipe 不是"同样的买入但更快"，而是**流程上少了两步**：

| 步骤 | Binance Web3 / gmgn | Axiom (Snipe) |
|------|---------------------|---------------|
| 1. 检测到毕业 | 等平台推送 (T_graduated_push) | **毕业前已挂好订单** |
| 2. 用户反应 | 看到 → 点 Buy (T_reaction) | **不需要，自动触发** |
| 3. 构建交易 | 点击后开始构建 | **预构建，毕业瞬间提交** |
| 4. 广播 | 普通 RPC 提交 | **Jito Bundle 优先打包** |

**延迟对比**:
- Binance/gmgn: `T_graduated_push + T_reaction + T_build + T_broadcast` → 数秒级
- Axiom Snipe: `T_server_detect + T_bundle_submit` → 亚秒级，且用户网络不在关键路径上

> **这不是优化能追平的差距，而是功能层缺失。** 我们没有 Migration Snipe 功能，用户在毕业场景只能靠手速。

---

## 对我方的启示

| 维度 | Axiom 做法 | 我方现状 | Gap |
|------|-----------|---------|-----|
| Migrating 标识 | 专门的 "Final Stretch" 列表 + Migrating 标签 + 进度百分比 | 待确认 | 需要自测 |
| Snipe 入口 | 一键 Snipe + 高级 Migration 订单 | 待确认 | 需要自测 |
| Fee 配置 | 双 Fee（Priority + Jito Bribe），平台给建议值 | 待确认 | 关键差异 |
| 迁移进度 | 实时百分比显示 | 待确认 | |
| Jito 集成 | 已确认（getTipAccounts 持续获取） | 待确认 | 可能是核心差异 |
