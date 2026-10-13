# MCP 接入说明

## 使用的 MCP Server

| 项目 | 内容 |
|---|---|
| 名称 | 麦当劳中国 MCP Server（mcd-mcp） |
| 接入地址 | https://mcp.mcd.cn |
| 传输协议 | Streamable HTTP |
| 鉴权方式 | `Authorization: Bearer {MCP Token}` |
| Token 申请 | https://open.mcd.cn/mcp （手机号登录 → 控制台 → 激活） |
| 官方指南 | https://github.com/M-China/mcd-mcp-server |

> 安全说明：本仓库不保存任何真实 Token；所有凭证仅通过环境变量 / 占位符（`YOUR_MCP_TOKEN`）形式出现。

## 实际调用的 Tool（11 个）

| Tool | 用途 | 在本 Skill 中的使用场景 |
|---|---|---|
| `query-meals` | 当前门店可售菜单 | 点餐骰子、精准推荐、程序员套餐、1+1 随心配的餐品池 |
| `query-meal-detail` | 套餐组成与可替换项 | 肠胃优化的**真实可替换方案**、1+1 组合替换 |
| `list-nutrition-foods` | 餐品营养信息 | 性价比评估与肠胃风险提示 |
| `query-nearby-stores` | 附近可用门店 | 定位当前门店，确保菜单 / 优惠券为门店实时数据 |
| `query-store-coupons` | 当前门店可用优惠券 | 优惠券自动匹配 |
| `available-coupons` | 麦麦省可领取的券 | 建议用户领哪张券（只推荐、不代领） |
| `query-my-coupons` | 我的优惠券 | 避免重复推荐用户已领的券 |
| `calculate-price` | 商品价格计算 | **真实券后实付价**计算（不手写数字） |
| `campaign-calendar` | 当月活动日历 | 感知 1024 程序员节等活动，活动驱动推荐 |
| `now-time-info` | 当前时间 | 时段感知：深夜 / 早餐 / 下午茶的肠胃提示差异 |
| `order-list` | 历史订单 | 可选：自动同步订单进用餐日志 |

## 调用流程（以「程序员套餐推荐」为例）

1. 用户输入「给我推荐程序员节麦麦套餐，预算30」；
2. 调用 `now-time-info` + `campaign-calendar` 确认当前时间与当月活动（1024 活动在期内则优先程序员套餐）；
3. Skill 判定进入「程序员套餐模式」，从内置程序员热门套餐池挑选候选组合；
4. 调用 `query-nearby-stores` 定位当前门店；
5. 调用 `query-meals` + `query-meal-detail` + `list-nutrition-foods` 获取菜单、套餐组成 / 可替换项、营养信息；
6. 调用 `query-store-coupons` + `available-coupons` + `query-my-coupons` 汇总「门店可用 / 可领 / 已领」优惠券；
7. 用 `calculate-price` 计算各候选组合的**真实券后实付价**，选最低实付方案；
8. 结合用户本地历史记录，给出肠胃风险提示与**基于真实套餐结构**的替换优化（如把饮料换成热红茶）；
9. 输出：推荐套餐、原价、优惠券、券后价、美味预期、肠胃风险提示。

## 订单自动同步流程（可选）

1. 用户说「同步我的订单」→ 调用 `order-list`；
2. 拉取近期到店 / 外送订单，生成用餐记录草稿（日期、餐品、实付金额）；
3. 逐条询问美味评分与是否腹痛；
4. 保存后给出统计摘要。

## 业务价值

- **省钱**：自动匹配门店优惠券并计算**真实**券后价，把"领券"变成"一句提醒"，明确告知用哪张券、省多少钱。
- **避雷**：结合个人用餐记录（金额 / 美味 / 是否腹痛）做肠胃风险学习，形成个性化推荐；替换建议基于真实套餐结构，可执行。
- **省事**：历史订单可自动同步进日志，免逐条手录。
- **趣味**：点餐骰子 + 二选一抛硬币，解决选择困难。
- **合规**：MCP 仅用于查询与推荐，**不涉及自动领券或下单**；所有菜单 / 价格 / 优惠券数据实时来自官方 MCP，仓库内不硬编码。

## 合规边界：刻意不使用的工具

| 工具 | 类别 | 不使用的理由 |
|---|---|---|
| `create-order` / `cancel-order` / `query-order` / `party-order-create` / `mall-create-order` | 下单类 | 大赛限制：不做真实下单能力，只做推荐 |
| `auto-bind-coupons` | 代领券 | 只推荐用户领哪张券，不代替用户领取 |
| `draw-lottery` / `query-lottery-info` / `query-my-prizes` | 积分抽奖 | 博彩性质，与「点餐推荐 + 肠胃避雷」定位无关 |
| `mall-points-products` / `mall-product-detail` / `mall-order-list` / `mall-order-detail` / `query-my-account` | 商城 / 积分 | 与项目主题无关，刻意不做 |
| `delivery-query-addresses` / `delivery-create-address` / `delivery-query-stores` | 配送地址管理 | 仅用 `query-nearby-stores` 定位门店，不管理用户地址 |
| `query-meal-assistance` | 企业团餐助餐 | 面向 C 端个人用户，暂不覆盖 |
