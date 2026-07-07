# ✈️ Airline Simulation - 架构主文档 (ARCHITECTURE.md)
> **版本**: v0.3.0  
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
| 开局与成长 | 固定起点，单一模式 | 双模式（畅玩/挑战），多资金渠道与董事会机制 |
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
预算+融资+燃油套期保值”]
A --> B4[“👥 人力资源部
高管聘用+员工培训”]
A --> B5[“🌐 外部关系与机队采购
飞机交易+联盟合作”]

B1 --> C1[审批航季计划]
B2 --> C2[处理P0/P1事件]
B3 --> C3[财务监控与董事会汇报]
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
- **资金机制**：无限资金（或极高初始资金），无破产风险。
- **时间系统**：
  - 支持三档快进：1x（1分钟=1分钟）、60x（1分钟=1小时）、180x（1分钟=3小时）。
  - 仅显示 UTC 时间（无当地时间换算，简化心智负担）。
- **外部数据**：依然联动现实世界天气和新闻，但时间流速不同步。

#### 💼 挑战模式
- **目标**：模拟真实航司创业，面临高门槛、重资产、董事会业绩压力。
- **资金机制**：有限资金，需通过多渠道融资，有破产风险。
- **时间系统**：严格 1:1 现实同步，双显 UTC 和当地时间。
- **董事会与资金来源**（核心玩法）：
  - 开局自动生成董事会结构和初始股权分配。
  - **初始资金来源**（基于现实航空业）：
    - 创始团队自有资金（占比小，体现决心）
    - 政府引导基金 / 地方国资（获取地方政策支持）
    - 风险投资 (VC) / 私募股权 (PE)（要求高增长和退出机制）
    - 银行贷款 / 出口信贷（用于特定机型采购，需还本付息）
    - 飞机租赁（经营性租赁降低前期资本支出）
  - **破产机制**：现金流转负且无法短期融资时，触发破产保护，需寻找白衣骑士（战略投资者）被收购，或重组清算。

### 3.3 总部所在地与货币系统
- **总部设定**：总部所在机场即为航司主基地，影响初期航线审批难度和地方政府补贴。
- **多币种结算**：
  - 默认根据总部所在国设定结算货币（如总部在上海默认CNY，在纽约默认USD）。
  - 涉及国际航线收入、飞机美元租赁费、油价波动时，系统内部计算汇率折算损益。
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
players (
id UUID PRIMARY KEY,
username TEXT,
created_at TIMESTAMPTZ
)
airlines (
id UUID PRIMARY KEY,
player_id UUID REFERENCES players(id),
name TEXT,
ceo_name TEXT,
game_mode TEXT, – sandbox / challenge
hub_airport TEXT, – 基地机场ICAO码
headquarters_city TEXT, – 总部所在城市
base_currency TEXT, – 结算货币 (CNY/USD/EUR等)
cash BIGINT, – 现金（以分为单位，按base_currency计）
reputation INT, – 声誉值
season_plan JSONB, – 当前航季计划快照
settings JSONB, – 自动化策略配置
founded_date DATE – 创立日期
)

### 5.2 融资与董事会 (挑战模式专属)
sql
shareholders (
id UUID PRIMARY KEY,
airline_id UUID REFERENCES airlines(id),
name TEXT, – 股东名称（如：创始团队、红杉资本、合肥政府引导基金）
share_percentage FLOAT, – 持股比例 (0-100)
invested_amount BIGINT, – 投资金额
expected_roi FLOAT, – 期望年化回报率（影响KPI压力）
seat_count INT – 董事会席位
)
loans (
id UUID PRIMARY KEY,
airline_id UUID REFERENCES airlines(id),
lender_name TEXT, – 贷款方（如：中国银行、美国进出口银行）
principal BIGINT, – 贷款本金
interest_rate FLOAT, – 年利率
term_months INT, – 期限（月）
monthly_payment BIGINT, – 月供
remaining_balance BIGINT, – 剩余本金
loan_type TEXT – bank_loan / export_credit
)

### 5.3 机队与飞机市场
sql
aircraft (
id UUID PRIMARY KEY,
airline_id UUID REFERENCES airlines(id),
tail_number TEXT,
aircraft_type TEXT,
status TEXT, – idle/flying/maintenance/grounded
flight_hours INT,
cycles INT,
next_maintenance_at INT,
config JSONB,
acquisition_type TEXT, – purchase / operating_lease / finance_lease
acquisition_date DATE,
acquisition_cost BIGINT
)
aircraft_market (
id UUID PRIMARY KEY,
source TEXT, – controller/buyplane/avcap
aircraft_type TEXT,
year INT,
price BIGINT,
location TEXT,
seller TEXT,
specs JSONB,
fetched_at TIMESTAMPTZ,
url TEXT
)
lease_contracts (
id UUID PRIMARY KEY,
airline_id UUID REFERENCES airlines(id),
aircraft_id UUID REFERENCES aircraft(id),
lessor TEXT, – 出租方（如AerCap）
monthly_rent BIGINT, – 月租金（通常以USD计）
lease_term INT, – 租期（月）
start_date DATE,
end_date DATE,
security_deposit BIGINT,
terms JSONB
)

### 5.4 航季计划与航班环
sql
flight_schedules (
id UUID PRIMARY KEY,
airline_id UUID REFERENCES airlines(id),
season TEXT,
status TEXT, – draft/approved/executing
plan_data JSONB,
created_at TIMESTAMPTZ,
approved_at TIMESTAMPTZ
)
flight_legs (
id UUID PRIMARY KEY,
schedule_id UUID REFERENCES flight_schedules(id),
flight_number TEXT,
dep_airport TEXT,
arr_airport TEXT,
dep_time_utc TIMESTAMPTZ,
arr_time_utc TIMESTAMPTZ,
aircraft_id UUID REFERENCES aircraft(id),
status TEXT,
actual_dep TIMESTAMPTZ,
actual_arr TIMESTAMPTZ,
load_factor FLOAT,
revenue BIGINT,
cost BIGINT
)

### 5.5 事件系统
sql
events (
id UUID PRIMARY KEY,
airline_id UUID REFERENCES airlines(id),
priority INT, – 0=紧急 1=重要 2=常规
type TEXT, – weather/maintenance/fuel_price/airspace/news/financial
title TEXT,
description TEXT,
related_leg_id UUID,
status TEXT, – pending/resolved/ignored/auto_resolved
options JSONB,
player_choice TEXT,
triggered_at TIMESTAMPTZ,
resolved_at TIMESTAMPTZ,
auto_resolved_by_ai BOOLEAN DEFAULT FALSE
)

### 5.6 外部数据缓存
sql
external_data (
id UUID PRIMARY KEY,
source TEXT,
data_type TEXT,
location TEXT,
payload JSONB,
fetched_at TIMESTAMPTZ
)

---
## 六、核心业务流程
### 6.1 开局初始化流程
mermaid
flowchart TD
A[玩家点击新建游戏] --> B{选择模式}
B -->|畅玩模式| C[给予无限资金]
C --> D[选择总部机场与货币]
D --> F[生成空机队，游戏开始]
B -->|挑战模式| E[生成初始董事会与股东结构]
E --> D
D --> G[根据融资方案计算初始现金]
G --> F

### 6.2 航季规划流程
[系统触发: 航季结束前7天]
→ 系统收集: 现有aircraft + 外部需求数据
→ 调用Python OR-Tools微服务
→ 写入 flight_schedules (status=draft)
→ 生成P2事件 “请审阅下航季排班”
→ 玩家审阅 → 选择方案 → status=approved
→ 系统展开plan_data → 写入 flight_legs 表

### 6.3 航班自动执行流程
[Supabase Cron: 每分钟执行]
→ 查询 flight_legs WHERE dep_time_utc <= NOW() AND status=‘scheduled’
→ 检查: 关联的外部天气数据 → 判定是否可飞
→ 可飞: status=‘departed’ → 推送Realtime
→ 查询 flight_legs WHERE arr_time_utc <= NOW() AND status=‘departed’
→ 计算利润 → 更新玩家cash → status=‘arrived’
→ 检查飞机flight_hours → 判定是否触发维修事件

### 6.4 事件触发流程
[Supabase Cron: 每10分钟执行]
→ 扫描 external_data 表最新数据
→ 规则引擎匹配生成事件
→ 写入 events 表
→ Realtime推送到前端 → 待办中心显示红点

### 6.5 财务结算与破产判定流程 (挑战模式)
[Supabase Cron: 每月1日执行]
→ 查询 loans 表，扣除当月需偿还的本息
→ 查询 lease_contracts 表，扣除当月租金
→ 结算上月总营收与总成本
→ 判定: 若 cash < 0 且无未动用信用额度
→ 生成 P0 级 “破产危机” 事件
→ 玩家必须选择: 寻找白衣骑士收购 / 出售核心资产重组 / 宣布破产清算

---
## 七、待办事项优先级定义
| 级别 | 颜色 | 触发示例 | 超时后果 |
|------|------|---------|---------|
| P0 紧急 | 🔴 | 极端天气/机械故障/破产危机 | 航班取消, 声誉暴跌, 强制清算 |
| P1 重要 | 🟡 | 油价暴涨/罢工/董事会质询 | 成本暴增, 股价下跌 |
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
- [ ] 航班自动执行 + 利润计算
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
### Phase 5: 机队市场 (v0.5)
- [ ] 飞机市场数据抓取
- [ ] 采购/租赁界面
- [ ] 财务管理模块
### Phase 6: 开局与模式 (v0.6)
- [ ] 开局引导流程开发
- [ ] 畅玩模式时间加速系统
- [ ] 挑战模式董事会与融资系统
- [ ] 破产判定逻辑
### Phase 7: 打磨发布 (v0.7+)
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
---
## 十二、玩家想法池 (待评估，未纳入架构)
> 此区域记录玩家提出的想法，但尚未经过设计评估。
> 每次评估后，移入正式架构章节或标记为"不采纳"。
- (空)