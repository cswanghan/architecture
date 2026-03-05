# cloud.html 教育云平台架构分析：演进逻辑、落地路径与风险评估

## 一、架构概览

**名称**: 云谷教育云 · 平台架构 (Yungu Education Cloud · Platform Architecture)
**核心原则**: Open API · Agent-Ready · Dogfooding · Simulate-before-Commit
**布局**: 纯纵向分层，自上而下

### 分层结构

```
┌─────────────────────────────────────────────────────────────┐
│ 双流程条: Open API 调用链 (6步) + Agent 调用链 (6步)          │
├─────────────────────────────────────────────────────────────┤
│ 应用层 (Applications)                                        │
│  官方应用套件 │ 第三方/ISV │ AI Agents (Skills/Plugin)        │
├─────────────────────────────────────────────────────────────┤
│ 开放平台 (Open Platform)                                     │
│  开发者工具条 (7): Portal · Docs · SDK · Playground ·         │
│                    MCP Explorer · Skill Market · Webhook      │
│  ┌─────────────────────┬───────────────────────┐             │
│  │ Domain API (8)      │ Resource API (8)       │             │
│  │ 高级业务语义抽象     │ 标准 CRUD 原语          │             │
│  └─────────────────────┴───────────────────────┘             │
├──────────── 底座能力层 Infrastructure Primitives ─────────────┤
│ ┌──────────┬──────────┬──────────┬──────────┐                │
│ │ Storage  │ Compute  │ Network  │ Security │                │
│ │ (7 Store)│ (7 Eng.) │ (6 Comp.)│ (6 Comp.)│                │
│ └──────────┴──────────┴──────────┴──────────┘                │
├─────────────────────────────────────────────────────────────┤
│ 基础设施 (Infrastructure)                                     │
│  PaaS (K8S/Nacos/CI) │ Storage (MySQL/Redis/MQ) │ External   │
└─────────────────────────────────────────────────────────────┘
```

---

## 二、与 index.html 的根本差异

这不是 index.html 的美化版，而是**架构理念的重构**。

| 维度 | index.html | cloud.html |
|------|-----------|------------|
| **核心叙事** | "我们的系统有什么" | "我们的平台能给谁用" |
| **组织原则** | 按技术栈分层（接入→业务→微服务→数据→基础设施） | 按消费关系分层（谁用→用什么→底座什么） |
| **API 观** | 500+ 散乱 API，无统一抽象 | Domain + Resource 双层抽象，API-First |
| **Agent 定位** | 右侧栏附属 | 应用层三分之一，一等公民 |
| **Dogfooding** | 未提及 | 核心原则——官方应用与第三方同等地位 |
| **底座模型** | 微服务中心（按项目划分） | 四原语（Storage/Compute/Network/Security，按能力划分） |
| **演进方向** | 无（静态资产图） | SaaS 功能 → 可分发的 Skills + 能力原语 |

**一句话**: index.html 是面向研发团队的技术资产清单；cloud.html 是面向 CEO/CTO 的平台战略蓝图。

---

## 三、这个架构要解决什么问题

### 问题 1: 从"卖软件"到"卖能力"的商业模式转型

**老问题**: 传统教育 SaaS 按模块卖 license，客户定制需求高，交付重。
**架构回答**:
- 将功能拆解为 **API 原语**（Resource API）和 **业务语义**（Domain API）
- 第三方可以基于 API 自建应用，不再依赖我们的前端
- 最终形态：**Skill Market** 让学校按需选择工作流，而非买整套系统

**落地难点**: Dogfooding 是前提——官方应用自己都不走 Open API 的话，API 质量无法保证。

### 问题 2: AI Agent 的规模化接入

**老问题**: 82 个 MCP Tool 已经跑起来了，但只服务内部 Agent，缺乏开放生态。
**架构回答**:
- **Skill Market**: 将 Agent 工作流打包为可分发的 Plugin（Skills + MCP config + hooks）
- **MCP Explorer**: 让开发者浏览和测试 Tool
- **Playground**: 在线调试 API 调用

**落地难点**: 7 个开发者工具目前只实现了 2 个（API Docs + MCP Explorer）。

### 问题 3: 系统复杂度的降维

**老问题**: 500+ API、70+ 路由、14 个后端模块——开发者面对的复杂度太高。
**架构回答**: 两层降维
- **Domain API**: 8 个高级业务语义接口（`schedule.solve()`, `student.enroll()`），一个调用完成一个业务任务
- **Resource API**: 8 个标准 CRUD 端点（`/v1/students`, `/v1/courses`），按资源分组

**落地难点**: Domain API 已有 82 个实现（agent-api-gateway），但 Resource API **完全未建设**。

### 问题 4: 底座能力的教育领域化

**老问题**: 通用云基础设施（K8S/MySQL/Redis）没有教育语义，开发者需要从数据库表理解业务。
**架构回答**: 四大领域化原语
- **Storage**: 不是 MySQL 表，而是 StudentStore / CourseStore / ScheduleStore 等教育领域对象
- **Compute**: 不是通用计算，而是 SchedulingEngine / SelectionEngine / EvaluationEngine 等教育专属引擎
- **Network**: API Gateway + MCP Gateway 双入口，EventBus + Webhook 双出口
- **Security**: IAM + PolicyEngine + AuditTrail + DataClassification 教育数据安全体系

**落地难点**: 这是**概念层的重新组织**，不是新建系统。现有 7 个微服务需要被重新理解为 Storage + Compute 的组合。

### 问题 5: 三类消费者的架构统一

**老问题**: 官方前端走 BFF/内部 API，Agent 走 MCP Gateway，第三方无统一入口。
**架构回答**:
```
官方应用  ─┐
第三方 ISV ─┼── Open Platform (Domain API + Resource API) ── 底座
AI Agent  ─┘
```
三类消费者走同一套 API，架构对等。

**落地难点**: 这是整个架构最激进的主张。**实现 Dogfooding 需要官方前端全面改造**——从直连后端 API 改为走 Open API，这是一个巨大的存量改造工程。

---

## 四、落地路径分析

### 4.1 当前实现状态

```
应用层         ▓▓▓▓▓▓▓▓░░░░  70%  (官方✅ Agent✅ 第三方API未开放)
开放平台       ▓▓▓▓░░░░░░░░  35%  (82 Domain API✅ Resource API❌ 5/7工具❌)
底座·Storage   ▓▓▓▓▓▓▓▓▓░░░  80%  (数据已有,缺统一Store抽象)
底座·Compute   ▓▓▓▓▓▓▓░░░░░  60%  (排课/评价引擎✅ 选课/分析❌)
底座·Network   ▓▓▓▓▓░░░░░░░  45%  (API+MCP GW✅ EventBus/Webhook❌)
底座·Security  ▓▓▓▓▓▓░░░░░░  50%  (IAM+Policy+Rate✅ Tenant+DataClass❌)
基础设施       ▓▓▓▓▓▓▓▓▓▓▓░  95%  (K8S/MySQL/Redis/MQ全部生产)
```

### 4.2 分阶段落地计划

#### Phase A: API 闭环（当前 → 3个月）

**目标**: 让 82 个 Domain API 真正被用起来

| 任务 | 产出 | 阻塞项 |
|------|------|--------|
| 修复 CAS 登录 BUG | Gateway 稳定连接下游 | BUG-001 |
| 域名绑定 + HTTPS | 公网可访问的 API 端点 | NLB 配置 |
| 审批 UI (轻量版) | 管理员可在钉钉/Web 审批 Agent 写操作 | 前端资源 |
| 内部 Dogfooding | 至少 1 个官方场景走 Open API | 选定试点场景 |
| Skill 试点 | 1-2 个 Skill (scheduling-assistant) 跑通 | Skill 编写 |

**里程碑**: Agent + 审批 + Skill 端到端闭环。

#### Phase B: 开放平台成型（3-6个月）

**目标**: 从"Agent 专用网关"升级为"开放平台"

| 任务 | 产出 | 前置条件 |
|------|------|---------|
| Resource API (v1) | 8 个标准 CRUD 端点 | Phase A 完成 |
| Developer Portal | 开发者注册、API Key 管理、用量看板 | Resource API |
| SDK 生成 | Java / Python / JS SDK (基于 OpenAPI spec 自动生成) | API 稳定 |
| Playground | 在线 API 调试工具 | Portal + SDK |
| Webhook 完整实现 | 事件订阅 + 回调推送 + 签名验证 | EventBus |
| Skill Market (v1) | 3-5 个官方 Skill 上架 | Skills 积累 |

**里程碑**: 第三方开发者可以注册 → 获取 API Key → 查看文档 → 调试 API → 订阅事件。

#### Phase C: 底座原语化（6-12个月）

**目标**: 将现有微服务重组为教育领域原语

| 任务 | 说明 | 复杂度 |
|------|------|--------|
| Store 抽象层 | 在现有数据库之上封装统一的 Store 接口 | 中 |
| Engine 标准化 | 每个 Compute Engine 定义标准输入/输出契约 | 中 |
| EventBus 建设 | 基于 RocketMQ 建设业务事件总线 | 高 |
| TenantIsolation | 多学校数据隔离（当前是单库模式） | 高 |
| DataClassification | 敏感数据分级 + 自动脱敏 | 中 |

**里程碑**: 底座四原语各有至少 3 个组件生产可用。

#### Phase D: Dogfooding 全面落地（12-18个月）

**目标**: 官方应用全面迁移到 Open API

| 任务 | 说明 | 风险 |
|------|------|------|
| 前端 BFF 层改造 | 从直连后端 API 改为走 Open Platform | 极高：影响全部 70+ 路由 |
| 渐进式迁移 | 按模块逐步切换（排课→评价→选课→...） | 需要灰度发布能力 |
| 性能保障 | Open API 多一层网关，需确保 P99 < 200ms | 需要缓存策略 |

**里程碑**: 官方应用 100% 走 Open API，Dogfooding 彻底落地。

---

## 五、风险矩阵

| 风险 | 概率 | 影响 | 缓解 |
|------|------|------|------|
| **Dogfooding 推不动** | 高 | 致命 | 选择 1-2 个低风险模块先试点，证明可行再推广 |
| **Resource API 设计失误** | 中 | 高 | 先发布 alpha 版给内部使用，收集反馈后再开放 |
| **底座原语化停在概念层** | 中 | 中 | 不需要重建系统，只需在现有服务上加统一接口层 |
| **开发者工具链投入不足** | 高 | 中 | SDK 和 Playground 可用开源方案（Swagger UI/Stoplight） |
| **Skill Market 无人发布** | 中 | 中 | 官方先发布 5-10 个 Skill，形成样板效应 |
| **多租户隔离改造困难** | 高 | 高 | 短期用 API 层 school_id 过滤，中期再做存储层隔离 |

---

## 六、架构的隐含假设

这张架构图有几个关键假设，如果假设不成立，架构需要调整：

### 假设 1: "API-First 是正确的方向"
**依据**: Twilio、Stripe、钉钉开放平台的成功案例。教育行业的碎片化需求特别适合 API 化——每个学校的流程不同，但底层数据结构相似。
**风险点**: 教育行业客户的技术能力普遍较弱，可能没有能力基于 API 自建应用。Skill Market 是缓解手段——不需要写代码，选一个 Skill 就能用。

### 假设 2: "三类消费者可以共享同一套 API"
**依据**: 官方前端、第三方 ISV、AI Agent 的底层数据需求是相同的（都需要学生/课程/课表数据）。
**风险点**: 性能要求不同（前端需要低延迟，Agent 可以容忍高延迟但需要更丰富的上下文）；权限模型不同（前端是用户级权限，Agent 可能是代理权限）。需要在统一 API 层内做差异化处理。

### 假设 3: "教育领域原语是稳定的"
**依据**: 学生、课程、课表、评价、组织架构——这些概念在教育行业有数十年历史，极少变化。
**风险点**: 新教育模式（项目制学习、跨校合作、在线教育）可能引入新的原语。架构需要预留扩展点。

### 假设 4: "Skills 会成为新的产品交付形态"
**依据**: Claude Code Skills/Plugin 生态的快速发展；GitHub Copilot Extensions 的模式验证。
**风险点**: Skills 依赖特定 AI 平台（目前是 Claude Code），如果平台策略变化，Skills 分发机制需要调整。应保持 MCP Tools 层的平台无关性。

---

## 七、核心结论

### 结论 1: 这是一张"目标态"架构图，不是"现状"

cloud.html 描述的是系统**应该成为的样子**，而非现在的样子。当前实现覆盖率约 **50%**，关键缺口在 Resource API、开发者工具链和 Dogfooding。

### 结论 2: 最大的赌注是 Dogfooding

"官方应用走 Open API" 是整个架构的基石。如果这一点做不到，那么：
- API 质量无法通过真实流量打磨
- 第三方开发者不会信任一个连官方都不用的 API
- "三类消费者架构对等" 沦为口号

建议: **不要等所有 API 就绪再推 Dogfooding，而是选 1-2 个模块立刻开始**。哪怕只有排课模块走 Open API，也比 82 个 API 无人使用强。

### 结论 3: 演进路径的优先级应该是

```
                高价值
                  ↑
  ┌───────────────┼───────────────┐
  │ Dogfooding    │ Skill Market  │
  │ (试点1-2模块) │ (打包3-5个)    │  ← 先做这些
  ├───────────────┼───────────────┤
  │ Resource API  │ Developer     │
  │ (CRUD 端点)   │ Portal        │  ← 然后这些
  ├───────────────┼───────────────┤
  │ EventBus      │ 底座原语化     │
  │ Webhook       │ Store/Engine  │  ← 最后这些
  └───────────────┼───────────────┘
                  ↑
                低价值
        低复杂度 ←───→ 高复杂度
```

### 结论 4: 两张架构图应该共存

- **index.html** → 面向研发团队，用于日常开发参考（"这个功能在哪个模块？"）
- **cloud.html** → 面向管理层和外部，用于战略沟通（"我们的平台能做什么？"）

它们不矛盾，是**同一个系统的两种视角**：index.html 是内视图（How），cloud.html 是外视图（What & Why）。

### 结论 5: 一句话总结给 CEO

> 构建 API-First 的教育云开放平台——通过 Domain API 与 Resource API 双层抽象统一业务与数据语义，原生支持 MCP/REST 双协议 Agent 接入，实现官方应用、第三方 ISV、AI Agent 三类消费者架构对等；底层提供教育领域内以教学、教务、校务为核心的计算引擎，让每一个 API 调用都带着教育场景的领域语义。最终，现有 SaaS 业务流程将逐步沉淀为可编排、可定制、可分发的 Skill 工作流，从交付软件转向交付能力。

---

*文档生成时间: 2026-03-04*
*基于 cloud.html (教育云平台架构) + agent-api-gateway (82 API 实现) + index.html (现状对照) 综合分析*
