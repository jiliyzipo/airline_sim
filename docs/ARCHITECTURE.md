# ✈️ Airline Simulation - 架构主文档 (ARCHITECTURE.md)
> **版本**: v0.4.0  
> **最后更新**: 2025-01  
> **状态**: 架构设计阶段  
> **原则**: 架构先行，变更必更新本文档，代码必须遵循本文档
---
## 一、项目愿景
一款连接现实世界实时数据的航空模拟运营网页游戏。玩家扮演**航司总经理**，负责宏观战略规划与突发危机决策，而非微观重复操作。

### 核心差异点（对比AM4）
| 维度 | AM4 | 本游戏 |
|------|-----|--------|
| 核心玩法 | 手动点起飞/维修/买油 | 宏观排班+事件决策 |
| 时间流逝 | 游戏时间加速 | 挑战模式1:1现实同步 / 畅玩模式可加速 |
| 数据来源 | 游戏内静态随机 | 真实API（气象/油价/新闻/飞机市场） |
| 交互方式 | 逐项操作弹窗 | 智能待办中心（分级提醒） |
| 玩家角色 | 航空公司管理者 | 航司总经理（战略决策者） |
| 财务系统 | 简单的收入减支出 | 拟真财务模型（多维度收支核算+年度财报+董事会考核） |
---
## 二、玩家角色与游戏模块
### 2.1 玩家角色：航空公司总经理
玩家扮演航司最高行政负责人，承担以下五大核心职责：
1. **🧭 战略决策者**：制定中长期规划，将董事会战略转化为经营成果
2. **🛡️ 安全第一责任人**：确保符合CCAR-121部等法规，建立SMS体系
3. **💼 经营核心**：对董事会负责，实现年度财务与运营目标（挑战模式）
4. **👥 团队领导者**：组建高效团队，培养人才，建设企业文化
5. **🌐 外部协调者**：与局方、机场、合作伙伴沟通，处理危机公关

### 2.2 游戏核心模块（对应总经理职责）
mermaid
flowchart TD
A[航司总经理仪表盘] --> B1[“🧭 战略决策中心
航季规划+KPI管理”]
A --> B2[“🛡️ 安全运行控制中心
AOC+事件处理”]
A --> B3[“💼 财务资产管理
盈亏模型+融资+年度财报”]
A --> B4[“👥 人力资源部
高管聘用+员工培训”]
A --> B5[“🌐 外部关系与机队采购
飞机交易+联盟合作”]

B1 --> C1[审批航季计划]
B2 --> C2[处理P0/P1事件]
B3 --> C3[控制成本与汇报董事会]
B4 --> C4[团队建设与绩效管理]
B5 --> C5[机队扩张与租赁管理]
---
## 三、开局流程与游戏模式
### 3.1 开局流程
1. **选择模式**：玩家选择“畅玩模式”或“挑战模式”。
2. **选择总部所在地**：在地图或列表中选择一个机场所在城市作为公司总部（基地枢纽）。
3. **设定货币**：默认以总部所在国的货币作为结算货币，玩家可手动修改。
4. **开始游戏**：根据所选模式生成初始资金、机队和董事会结构。

### 3.2 游戏模式定义
#### 🌟 畅玩模式
- **目标**：无压力的沙盒经营，专注于航线网络搭建和机队收集。
- **资金机制**：无限资金，无破产风险，无董事会考核。
- **时间系统**：支持三档快进；仅显示 UTC 时间。

#### 💼 挑战模式
- **目标**：模拟真实航司创业，面临高门槛、重资产、董事会业绩压力。
- **资金机制**：有限资金，需通过多渠道融资，有破产风险。
- **时间系统**：严格 1:1 现实同步，双显 UTC 和当地时间。
- **董事会与资金来源**：
  - 开局自动生成董事会结构和初始股权分配。
  - 资金来源：自有资金、政府引导基金、VC/PE、银行贷款/出口信贷、飞机租赁。
- **破产机制**：现金流转负且无法短期融资时，触发破产保护。

### 3.3 总部所在地与货币系统
- **总部设定**：总部所在机场即为航司主基地，影响初期航线审批和地方政府补贴。
- **多币种结算**：默认根据总部所在国设定结算货币，国际航线与租赁费按汇率折算。
---
## 四、技术栈选型
| 层级 | 技术 | 理由 |
|------|------|------|
| 前端 | React 18 + Ant Design 5 | 数据表格/表单/时间轴组件丰富 |
| 地图 | Leaflet + React-Leaflet | 轻量、开源、航线绘制友好 |
| 后端/BaaS | Supabase (PostgreSQL) | 免运维、Realtime推送、Edge Functions |
| 排班算法 | Python + Google OR-Tools | 运筹学求解器，适合MIP问题 |
| 算法部署 | Vercel Python Serverless | 按需调用，免维护服务器 |
| 数据抓取 | Make.com / n8n | 低代码定时抓取，无需写爬虫 |
| 代码托管 | GitHub | 版本管理+协作 |
| 开发IDE | Cursor | AI内置IDE，直接在编辑器中对话写代码 |
---
## 五、数据库设计 (核心表)
### 5.1 玩家与公司
sql
players (id UUID PRIMARY KEY, username TEXT, created_at TIMESTAMPTZ)
airlines (
id UUID PRIMARY KEY, player_id UUID REFERENCES players(id),
name TEXT, ceo_name TEXT, game_mode TEXT, – sandbox / challenge
hub_airport TEXT, headquarters_city TEXT, base_currency TEXT,
cash BIGINT, reputation INT, season_plan JSONB, settings JSONB, founded_date DATE
)

### 5.2 融资与董事会 (挑战模式专属)
sql
shareholders (id, airline_id, name, share_percentage FLOAT, invested_amount BIGINT, expected_roi FLOAT, seat_count INT)
loans (id, airline_id, lender_name TEXT, principal BIGINT, interest_rate FLOAT, term_months INT, monthly_payment BIGINT, remaining_balance BIGINT, loan_type TEXT)

### 5.3 机队与飞机市场
sql
aircraft (
id UUID PRIMARY KEY, airline_id UUID, tail_number TEXT, aircraft_type TEXT, status TEXT,
flight_hours INT, cycles INT, next_maintenance_at INT, config JSONB,
acquisition_type TEXT, – purchase / operating_lease / finance_lease
acquisition_date DATE, acquisition_cost BIGINT,
– 财务属性
monthly_lease_cost BIGINT, – 若为租赁，月租金
daily_depreciation BIGINT – 若为自有，每日折旧额
)
aircraft_market (id, source, aircraft_type, year, price, location, seller, specs, fetched_at, url)
lease_contracts (id, airline_id, aircraft_id, lessor, monthly_rent, lease_term, start_date, end_date, security_deposit, terms JSONB)

### 5.4 航季计划与航班执行
sql
flight_schedules (id, airline_id, season, status, plan_data JSONB, created_at, approved_at)
flight_legs (
id UUID PRIMARY KEY, schedule_id UUID, flight_number TEXT,
dep_airport TEXT, arr_airport TEXT, dep_time_utc TIMESTAMPTZ, arr_time_utc TIMESTAMPTZ,
aircraft_id UUID, status TEXT, actual_dep TIMESTAMPTZ, actual_arr TIMESTAMPTZ,
– 财务核算字段
load_factor FLOAT, – 客座率
passenger_revenue BIGINT, – 客运收入
cargo_revenue BIGINT, – 货运收入
ancillary_revenue BIGINT, – 辅营收入(行李/选座等)
fuel_cost BIGINT, – 燃油成本
airport_nav_cost BIGINT, – 机场起降与航路费
crew_cost BIGINT, – 机组成本(按航段分摊)
profit BIGINT – 本航段净利润
)

### 5.5 年度财务报告 (挑战模式核心)
sql
annual_financial_reports (
id UUID PRIMARY KEY,
airline_id UUID REFERENCES airlines(id),
fiscal_year INT,
– 收入汇总
total_passenger_revenue BIGINT,
total_cargo_revenue BIGINT,
total_ancillary_revenue BIGINT,
– 成本汇总
total_fuel_cost BIGINT,
total_airport_nav_cost BIGINT,
total_crew_labor_cost BIGINT, – 含固定人力成本分摊
total_depreciation_lease_cost BIGINT, – 折旧与租赁
total_sales_admin_cost BIGINT, – 销管费用
total_finance_cost BIGINT, – 利息支出
– 利润与指标
operating_profit BIGINT, – 营业利润
net_profit BIGINT, – 净利润
avg_load_factor FLOAT, – 平均客座率
yield FLOAT, – 单位收入 (每客公里收入)
cask FLOAT, – 单位成本 (每可用座公里成本)
– 董事会反馈
board_feedback TEXT, – 董事会评价(满意/警告/重组要求)
created_at TIMESTAMPTZ
)

### 5.6 事件系统与外部数据
sql
events (id, airline_id, priority INT, type TEXT, title, description, related_leg_id UUID, status TEXT, options JSONB, player_choice TEXT, triggered_at, resolved_at, auto_resolved_by_ai BOOLEAN)
external_data (id, source, data_type, location, payload JSONB, fetched_at TIMESTAMPTZ)

---
## 六、核心业务流程
### 6.1 航班盈亏核算模型 (每次落地触发)
mermaid
flowchart TD
A[航班落地 status=arrived] --> B[计算总收入]
A --> C[计算总成本]

B --> B1[客运收入 = 票价 * 旅客数]
B --> B2[货运收入 = 腹舱运量 * 运价]
B --> B3[辅营收入 = 旅客数 * 平均辅营消费]

C --> C1[燃油成本 = 飞行小时 * 燃油率 * 实时油价]
C --> C2[机场航路费 = 起降费 + 航路费]
C --> C3[机组成本 = 飞行小时 * 机组时薪]

B1 --> D[航段利润 = 总收入 - 总成本]
B2 --> D
B3 --> D
C1 --> D
C2 --> D
C3 --> D

D --> E[更新玩家现金池]
### 6.2 年度财务结算与董事会汇报流程 (每年1月1日触发)
[Supabase Cron: 每年1月1日执行]
→ 汇总过去一年 flight_legs 表的所有收入和成本数据
→ 计算固定成本：
- 人力成本 (管理层/地勤固定薪酬)
- 折旧与租赁 (飞机每月折旧/租金 * 12)
- 财务费用 (贷款年利息)
→ 生成 annual_financial_reports 记录
→ 董事会KPI考核逻辑:
- 计算 净利润率 与 营业利润率
- 判定是否达到 shareholders 表中的 expected_roi
- 若未达标: 生成 P1 级 “董事会质询” 事件，要求削减成本或调整战略
- 若连续两年严重亏损: 生成 P0 级 “股东大会弹劾/清算” 事件

### 6.3 其他核心流程
- **开局初始化流程**：选择模式 > 设定资金与股东 > 生成基地。
- **航季规划流程**：OR-Tools生成 > 玩家审批 > 展开至flight_legs。
- **事件触发流程**：外部数据扫描 > 规则匹配 > 推送至待办中心。
- **破产判定流程**：现金 < 0 > 触发破产危机事件 > 寻求收购或重组。
---
## 七、待办事项优先级定义
| 级别 | 颜色 | 触发示例 | 超时后果 |
|------|------|---------|---------|
| P0 紧急 | 🔴 | 极端天气/破产危机/股东弹劾 | 航班取消, 强制清算, 游戏结束 |
| P1 重要 | 🟡 | 油价暴涨/董事会质询/罢工 | 股价下跌, 融资成本上升 |
| P2 常规 | 🟢 | 航季审批/维修计划/员工培训 | 系统按保守策略默认执行 |
---
## 八、外部API清单
| 数据源 | API | 免费额度 | 抓取频率 | 用途 |
|--------|-----|---------|---------|------|
| 气象 | OpenWeatherMap | 1000次/天 | 每3小时 | 航班延误判定 |
| 航班 | OpenSky Network | 无限(开源) | 每1小时 | 真实航班密度参考 |
| 新闻 | NewsAPI.org | 100次/天 | 每6小时 | 突发事件触发 |
| 油价 | CommodityAPI/EIA | 100次/月 | 每天1次 | 燃油成本波动 |
| 飞机市场 | Controller/BuyPlane | 网页爬取 | 每周1次 | 二手飞机价格参考 |
| 汇率 | exchangerate.host | 无限(开源) | 每天1次 | 多币种结算折算 |
---
## 九、时间系统设计
- **服务器时间**: UTC (Supabase默认)
- **前端显示**:
  - 挑战模式: UTC + 玩家本地时间双显示
  - 畅玩模式: 仅显示 UTC 时间
- **时间流速**:
  - 挑战模式: 1:1 现实同步
  - 畅玩模式: 支持 1x, 60x, 180x 虚拟加速
- **离线判定**: 比较上次心跳时间与当前时间差
---
## 十、开发路线图 (MVP优先)
### Phase 1: 基础骨架 (v0.1)
- [ ] Supabase项目创建 + 数据表建表
- [ ] React项目脚手架 + AntD布局
- [ ] 基础Dashboard: 显示玩家信息+航班列表
- [ ] Supabase Cron: 航班自动状态流转
### Phase 2: 核心循环 (v0.2)
- [ ] 航季计划生成 (简化版算法, 先用规则不用OR-Tools)
- [ ] 航班自动执行 + 基础利润计算 (收入-成本)
- [ ] 事件系统骨架 (手动创建事件 → 待办中心)
- [ ] 离线报表
### Phase 3: 现实联动 (v0.3)
- [ ] Make.com 抓取OpenWeather → 写入Supabase
- [ ] 天气→航班延误事件触发
- [ ] 油价抓取 → 成本波动
- [ ] 新闻抓取 → 突发事件
### Phase 4: 排班算法 (v0.4)
- [ ] Python OR-Tools FAM模型
- [ ] Vercel部署
- [ ] 前端审批界面
### Phase 5: 机队与财务 (v0.5)
- [ ] 飞机市场数据抓取
- [ ] 采购/租赁界面
- [ ] 完整盈亏模型核算 (单航段)
- [ ] 多币种结算
### Phase 6: 开局与模式 (v0.6)
- [ ] 开局引导流程开发
- [ ] 畅玩模式时间加速系统
- [ ] 挑战模式董事会与融资系统
- [ ] 破产判定逻辑
### Phase 7: 深度财务与董事会 (v0.7)
- [ ] 年度财务报告生成界面
- [ ] 股东KPI考核与反馈机制
- [ ] 燃油套期保值玩法
- [ ] 董事会质询事件链
### Phase 8: 打磨发布 (v0.8+)
- [ ] 地图可视化
- [ ] 多人系统
- [ ] 经济平衡调参
- [ ] 部署上线
---
## 十一、变更记录
| 日期 | 版本 | 变更内容 | 变更人 |
|------|------|---------|--------|
| 2025-01 | v0.1.0 | 初始架构创建 | 架构师AI |
| 2025-01 | v0.2.0 | 引入航司总经理模块和现实飞机交易市场 | 玩家+架构师AI |
| 2025-01 | v0.3.0 | 引入双模式(畅玩/挑战)、开局流程、董事会融资及多币种结算 | 玩家+架构师AI |
| 2025-01 | v0.4.0 | 引入拟真财务盈亏模型、年度财务报告与董事会汇报考核机制 | 玩家+架构师AI |
---
## 十二、玩家想法池 (待评估，未纳入架构)
> 此区域记录玩家提出的想法，但尚未经过设计评估。
> 每次评估后，移入正式架构章节或标记为"不采纳"。
- (空)