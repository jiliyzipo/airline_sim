# ✈️ Airline Simulation - 架构主文档 (ARCHITECTURE.md)
> **版本**: v0.2.0  
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
| 时间流逝 | 游戏时间加速 | 1:1现实同步 |
| 数据来源 | 游戏内静态随机 | 真实API（气象/油价/新闻/飞机市场） |
| 交互方式 | 逐项操作弹窗 | 智能待办中心（分级提醒） |
| 玩家角色 | 航空公司管理者 | 航司总经理（战略决策者） |

---

## 二、玩家角色与游戏模块

### 2.1 玩家角色：航空公司总经理
玩家扮演航司最高行政负责人，承担以下五大核心职责：

1. **🧭 战略决策者**：制定中长期规划，将董事会战略转化为经营成果
2. **🛡️ 安全第一责任人**：确保符合CCAR-121部等法规，建立SMS体系
3. **💼 经营核心**：对董事会负责，实现年度财务与运营目标
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
预算+燃油套期保值”]
A --> B4[“👥 人力资源部
高管聘用+员工培训”]
A --> B5[“🌐 外部关系与机队采购
飞机交易+联盟合作”]

B1 --> C1[审批航季计划]
B2 --> C2[处理P0/P1事件]
B3 --> C3[财务监控与成本控制]
B4 --> C4[团队建设与绩效管理]
B5 --> C5[机队扩张与租赁管理]

---

## 三、技术栈选型

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

## 四、系统架构图

[用户浏览器: React + AntDesign + Leaflet]
| REST API | WebSocket (Realtime)
v v
[Supabase (BaaS)]
- PostgreSQL (核心数据)
- Edge Functions (业务逻辑)
- Realtime Engine (事件推送)
- Cron Jobs (定时任务)
|
| HTTP调用
v
[Python 排班算法微服务 (Vercel)]
Google OR-Tools: FAM模型 / 航班环生成 / 尾号指派
^
| 定时写入
[外部数据抓取层 (Make.com/n8n)]
OpenWeather | OpenSky | NewsAPI | 油价指数 | 飞机市场数据
-> 定时抓取 -> 写入Supabase的external_data表


---

## 五、数据库设计 (核心表)

### 5.1 玩家与公司
sql
players (
id UUID PRIMARY KEY,
username TEXT,
cash BIGINT, – 现金（分）
reputation INT, – 声誉值
fuel_reserve INT, – 燃油储备（升）
co2_quota INT, – CO2配额
created_at TIMESTAMPTZ
)

airlines (
id UUID PRIMARY KEY,
player_id UUID REFERENCES players(id),
name TEXT,
hub_airport TEXT, – 基地机场ICAO码
season_plan JSONB, – 当前航季计划快照
settings JSONB, – 自动化策略配置
ceo_name TEXT, – 总经理姓名（玩家自定义）
founded_date DATE – 创立日期
)


### 5.2 机队与飞机市场
sql
aircraft (
id UUID PRIMARY KEY,
airline_id UUID REFERENCES airlines(id),
tail_number TEXT, – 机尾号（唯一）
aircraft_type TEXT, – 机型 A320/B737…
status TEXT, – idle/flying/maintenance/grounded
flight_hours INT, – 累计飞行小时
cycles INT, – 累计起落次数
next_maintenance_at INT, – 下次检修阈值（飞行小时）
config JSONB, – 子构型（如8F8布局）
acquisition_type TEXT, – purchase/lease
acquisition_date DATE,
acquisition_cost BIGINT
)

– 现实飞机市场数据缓存
aircraft_market (
id UUID PRIMARY KEY,
source TEXT, – controller/buyplane/avcap
aircraft_type TEXT,
year INT, – 制造年份
price BIGINT, – 价格（分）
location TEXT, – 所在地ICAO
seller TEXT, – 卖家信息
specs JSONB, – 技术规格
fetched_at TIMESTAMPTZ,
url TEXT – 原始链接
)

– 租赁合同表
lease_contracts (
id UUID PRIMARY KEY,
airline_id UUID REFERENCES airlines(id),
aircraft_id UUID REFERENCES aircraft(id),
lessor TEXT, – 出租方（如AerCap）
monthly_rent BIGINT, – 月租金（分）
lease_term INT, – 租期（月）
start_date DATE,
end_date DATE,
security_deposit BIGINT,
terms JSONB – 合同条款
)


### 5.3 航季计划与航班环
sql
flight_schedules (
id UUID PRIMARY KEY,
airline_id UUID REFERENCES airlines(id),
season TEXT, – 如 “2025-SUMMER”
status TEXT, – draft/approved/executing
plan_data JSONB, – 完整的航班环+尾号指派方案
created_at TIMESTAMPTZ,
approved_at TIMESTAMPTZ
)

flight_legs (
id UUID PRIMARY KEY,
schedule_id UUID REFERENCES flight_schedules(id),
flight_number TEXT,
dep_airport TEXT, – ICAO
arr_airport TEXT, – ICAO
dep_time_utc TIMESTAMPTZ,
arr_time_utc TIMESTAMPTZ,
aircraft_id UUID REFERENCES aircraft(id),
status TEXT, – scheduled/boarding/departed/arrived/delayed/cancelled
actual_dep TIMESTAMPTZ,
actual_arr TIMESTAMPTZ,
load_factor FLOAT, – 客座率
revenue BIGINT,
cost BIGINT
)


### 5.4 事件系统（核心！）
sql
events (
id UUID PRIMARY KEY,
airline_id UUID REFERENCES airlines(id),
priority INT, – 0=紧急 1=重要 2=常规
type TEXT, – weather/maintenance/fuel_price/airspace/news
title TEXT,
description TEXT,
related_leg_id UUID, – 关联航班
status TEXT, – pending/resolved/ignored/auto_resolved
options JSONB, – 玩家可选项 [{id, label, consequence}]
player_choice TEXT,
triggered_at TIMESTAMPTZ,
resolved_at TIMESTAMPTZ,
auto_resolved_by_ai BOOLEAN DEFAULT FALSE
)


### 5.5 外部数据缓存
sql
external_data (
id UUID PRIMARY KEY,
source TEXT, – openweather/opensky/newsapi/aircraft_market
data_type TEXT, – weather/flight/oil_price/news/aircraft
location TEXT, – ICAO码或城市名
payload JSONB, – 原始数据
fetched_at TIMESTAMPTZ
)


---

## 六、核心业务流程

### 6.1 航季规划流程
[系统触发: 航季结束前7天]
→ 系统收集: 现有aircraft + 外部需求数据
→ 调用Python OR-Tools微服务
→ 输入: 机队列表, 可飞航线, 维修约束, 时刻约束
→ 输出: 3套候选航班环方案 (JSON)
→ 写入 flight_schedules (status=draft)
→ 生成P2事件 “请审阅下航季排班”
→ 玩家审阅 → 选择方案 → status=approved
→ 系统展开plan_data → 写入 flight_legs 表


### 6.2 航班自动执行流程
[Supabase Cron: 每分钟执行]
→ 查询 flight_legs WHERE dep_time_utc <= NOW() AND status=‘scheduled’
→ 检查: 关联的外部天气数据 → 判定是否可飞
→ 可飞: status=‘departed’ → 推送Realtime
→ 查询 flight_legs WHERE arr_time_utc <= NOW() AND status=‘departed’
→ 计算利润 → 更新玩家cash → status=‘arrived’
→ 检查飞机flight_hours → 判定是否触发维修事件


### 6.3 事件触发流程
[Supabase Cron: 每10分钟执行]
→ 扫描 external_data 表最新数据
→ 规则引擎匹配:
- 某机场天气 == 雷暴 → 生成P0事件 (关联该机场的flight_legs)
- 油价涨幅 > 10% → 生成P1事件
- 新闻含"停飞" → 生成P1事件 (关联对应机型)
→ 写入 events 表 (status=pending)
→ Realtime推送到前端 → 待办中心显示红点


### 6.4 离线补偿流程
[玩家上线时]
→ 查询 events WHERE status=‘pending’ AND triggered_at < 上次下线时间
→ 对P0/P1事件: 检查 airline.settings.auto_strategy
- 若配置了自动策略 → 按策略自动处理 → 标记 auto_resolved_by_ai=true
- 若无策略 → 仍为pending, 等待玩家手动处理
→ 生成离线报表汇总


### 6.5 飞机采购与租赁流程
[玩家访问机队市场]
→ 前端调用 /api/aircraft-market/search
→ 后端查询 aircraft_market 表 + 实时抓取（如超过1天）
→ 返回飞机列表（按价格/机型/位置筛选）
→ 玩家选择飞机 → 检查资金是否足够
→ 生成采购/租赁请求
→ 资金扣除 → 飞机加入airline机队
→ 生成相应财务记录


---

## 七、待办事项优先级定义

| 级别 | 颜色 | 触发示例 | 超时后果 |
|------|------|---------|---------|
| P0 紧急 | 🔴 | 极端天气/机械故障/领空关闭 | 航班强制取消, 声誉-50, 赔付 |
| P1 重要 | 🟡 | 油价暴涨/罢工/机型停飞令 | 成本暴增, 利润受损 |
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

---

## 九、时间系统设计

- **服务器时间**: UTC (Supabase默认)
- **前端显示**: UTC + 玩家本地时间双显示
- **时区校对**: 前端 `Intl.DateTimeFormat().resolvedOptions().timeZone` 自动获取
- **时间流速**: 1:1 现实同步，无加速
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

### Phase 6: 打磨发布 (v0.6+)
- [ ] 地图可视化 (Leaflet)
- [ ] 多人系统
- [ ] 经济平衡调参
- [ ] 部署上线

---

## 十一、变更记录

| 日期 | 版本 | 变更内容 | 变更人 |
|------|------|---------|--------|
| 2025-01 | v0.1.0 | 初始架构创建 | 架构师AI |
| 2025-01 | v0.2.0 | 引入航司总经理模块和现实飞机交易市场 | 玩家+架构师AI |

---

## 十二、玩家想法池 (待评估，未纳入架构)

> 此区域记录玩家提出的想法，但尚未经过设计评估。
> 每次评估后，移入正式架构章节或标记为"不采纳"。

- (空)