# 开发协议规范 (DEVELOPMENT_PROTOCOL.md)
> 版本: v1.0
> 用途: 规范"人+AI"协作开发的流程，防止屎山代码

---

## 一、核心原则: 架构先行

### 规则
1. **任何新想法/需求，必须先写入 ARCHITECTURE.md 的"想法池"章节**
2. 架构师AI评估后，将想法移入正式架构章节，更新版本号
3. 只有出现在架构文档中的功能，才能开始写代码
4. 代码结构必须与架构文档的模块划分一致

### 流程
```
玩家新想法 -> 写入想法池 -> AI评估 -> 更新架构文档 -> 写代码 -> 测试 -> 提交
```

---

## 二、版本管理规范

### 2.1 版本号规则
- 格式: v主版本.次版本.修订号
- 主版本: 架构级变更 (如从单机到多人)
- 次版本: 新功能模块完成 (如事件系统上线)
- 修订号: bug修复、参数调整

### 2.2 文件组织
```
airline_sim/
  docs/
    ARCHITECTURE.md          <- 架构主文档（唯一真相来源）
    DEVELOPMENT_PROTOCOL.md  <- 本文件
    CHANGELOG.md             <- 变更日志
  src/
    frontend/                <- React前端
    backend/                 <- Supabase Edge Functions
    algorithms/              <- Python排班算法
    data/                    <- 数据抓取配置
  config/
    env.example              <- 环境变量模板
  tests/
  .gitignore
  README.md
```

### 2.3 对话交接协议（跨对话续开发）

当对话过长需要新开对话时，玩家需提供:

```
[交接清单]
1. 当前架构文档: ARCHITECTURE.md (上传文件)
2. 当前版本号: v0.1.0
3. 已完成的功能: [列表]
4. 正在开发的功能: [列表]
5. 待解决的问题: [列表]
6. 本次对话的目标: [具体任务]
```

AI收到后第一步: 读取架构文档 -> 确认理解 -> 开始工作。

---

## 三、代码编写规范

### 3.1 命名规则
- 文件名: kebab-case (如 flight-legs.tsx)
- 组件名: PascalCase (如 FlightLegsTable)
- 变量/函数: camelCase (如 fetchFlightLegs)
- 数据库表: snake_case (如 flight_legs)
- 常量: UPPER_SNAKE_CASE (如 MAX_PASSENGERS)

### 3.2 文件大小限制
- 单文件不超过 300 行
- 超过则拆分为子模块
- 组件文件: 1个组件 = 1个文件

### 3.3 注释要求
- 每个文件头部: 简述文件用途 + 对应架构文档章节
- 每个函数: 参数说明 + 返回值
- 复杂逻辑: 行内注释说明"为什么这么写"

### 3.4 禁止事项
- 禁止在前端硬编码业务逻辑（如利润计算必须在后端）
- 禁止直接操作数据库（必须通过API或Edge Functions）
- 禁止在架构文档之外随意新增数据库表
- 禁止"先写代码再补架构"

---

## 四、开发工具链推荐

### 4.1 强烈推荐: Cursor IDE

Cursor是基于VS Code的AI编程IDE，优势:
- AI可以直接读取你项目中的所有文件
- 你用自然语言描述需求，AI直接在文件中编写/修改代码
- 不需要复制粘贴
- 支持指定模型 (Claude/GPT等)

### 工作流
```
1. 在Cursor中打开 airline_sim 项目
2. 在Cursor的Chat面板中输入:
   "请阅读 docs/ARCHITECTURE.md，按照Phase 1的要求，
    创建Supabase的数据表SQL迁移脚本"
3. AI直接在你的项目中创建文件并写入代码
4. 你审阅后点击Accept
5. git commit 保存
```

### 4.2 备选方案: 传统IDE + GitHub Copilot
- VS Code + GitHub Copilot
- 需要手动复制AI生成的代码到文件中
- 适合已有编程经验的开发者

### 4.3 不推荐: 纯对话窗口写代码
- 容易丢失上下文
- 代码版本混乱
- 无法直接运行和调试

---

## 五、每次开发会话的检查清单

开始开发前:
- [ ] 确认ARCHITECTURE.md是最新版本
- [ ] 确认本次任务对应架构文档中的哪个Phase
- [ ] 确认上一个任务的代码已提交(git commit)

开发中:
- [ ] 每完成一个功能模块，立即git commit
- [ ] commit message格式: "[Phase X] 功能描述"
- [ ] 如有新想法，先记入想法池，不立即开发

结束开发时:
- [ ] 更新ARCHITECTURE.md的变更记录
- [ ] 更新CHANGELOG.md
- [ ] 确认所有代码已push到GitHub
- [ ] 记录未完成项，供下次对话使用
