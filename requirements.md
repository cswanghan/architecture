1. 综合架构描述 (用于研发沟通)
系统名称建议： 云谷 AI 安全编排与治理平台 (AI-SOGP: AI Secure Orchestration & Governance Platform)

核心设计理念：
该架构的核心目标是在将 AI决策应用于敏感的生产环境（学校业务系统）之前，建立一道强制性的安全屏障。它采用 “先仿真，后提交（Simulate-before-Commit）” 模式，并结合严格的**“人在回路（Human-in-the-Loop）”**审批机制，确保操作的确定性、安全性和可审计性。

主要技术模块与分层：

接入层与边缘侧 (Access & Edge Tier):

多模态输入网关 (Multi-modal Ingress): 处理来自钉钉、Web端的文本、语音、表单等非结构化输入。

意图识别引擎 (NLU Service): 将用户自然语言转化为结构化的操作意图 (Intent)。

本地代理 (Local Edge Agent): 部署在学校侧的轻量级网关，负责身份认证、安全隧道建立以及初步的数据预处理，连接云端与本地环境。

核心控制平面 (Core Control Plane) - "大脑":

这是系统的核心中枢，负责状态管理和任务调度，但不直接执行更改。

动作编排器 (Action Orchestrator): 维护工具库和API目录，定义可执行的原子操作。

状态管理器 (State Manager): 维护系统当前状态的数字孪生模型 (Digital Twin) 和快照。

策略引擎 (Policy Engine - OPA): 基于预定义的规则（如风险阈值、审批流程）进行自动化判断。

审计中心 (Audit Trail & Evidence): 不可篡改的日志记录服务，追踪所有意图、仿真结果和最终执行操作。

仿真环境 (Simulation Sandbox) - "沙箱":

关键特性：只读隔离 (Read-Only Isolation)。

该环境拥有生产环境数据的脱敏克隆或快照，并严格实现多租户隔离。

仿真引擎 (Simulation Engine): 接收意图，在沙箱中模拟执行，计算出变更集 (ChangeSet)。

影响分析服务 (Impact Analysis Service): 分析变更集对业务的影响范围、潜在冲突，并计算风险等级 (Risk Score)。

治理与交互层 (Governance & Interface Tier):

控制塔/运维中心 (Control Tower UI): 面向管理员/审批者的可视化界面。

提供仿真结果的可视化仪表盘（风险图表、冲突报告），并提供“批准/拒绝”的交互工作流。

生产执行层 (Production Execution Tier):

执行引擎 (Execution Engine): 仅接收经批准的、幂等的变更集。

适配器/连接器 (Connectors): 对接存量的教务、德育、健康等异构业务系统 (MySQL, APIs等)。

事务与回滚机制 (Transaction & Rollback): 确保原子性操作，一旦失败可快速恢复到操作前状态。

2. 标准架构图生成提示 (Prompts)
您可以将以下描述输入给专业的绘图工具（如 Visio, Lucidchart, Draw.io），或者输入给高质量的 AI 图片生成模型（如 Midjourney V6, DALL-E 3），来生成专业的架构图。

选项 A：给 AI 图像生成模型的提示词 (Prompt)
风格要求：
Professional enterprise technical architecture diagram, clean isometric style, blueprint aesthetic, distinct color-coded zones, standard IT icons (servers, databases, clouds, firewalls, gears), clear arrows showing data flow with labels.

画面内容描述：
A comprehensive technical architecture diagram structured into five distinct zones separated by dashed boundaries.

Far Left Zone (Blue - "Access & Edge"): Labeled "USER ACCESS LAYER". Contains icons for "DingTalk/Web Client" and "Voice/Form Input" pointing to an "API Gateway / NLU Service". Below them, an icon for "Local Edge Agent (Proxy)" with a secure tunnel icon.

Top Center Zone (Purple - "Control Plane"): A large cloud icon labeled "CLOUD CONTROL PLANE (AI-Ready OS)". Inside are four stacked functional blocks: "Action Orchestrator (API Registry)", "State Manager (Digital Twin Model)", "Policy Engine (OPA Rules)", "Audit Service (Immutable Log)".

Bottom Center Zone (Yellow - "Simulation Sandbox"): A secure boundary labeled "SIMULATION ENVIRONMENT (Read-Only Replica)". Inside, a "Data Cloning Service" icon pulls from "Tenant Isolated Snapshots". An arrow points to a "Simulation Engine & Impact Analyzer" which outputs a document icon labeled "Proposed ChangeSet + Risk Score".

Top Right Zone (Orange - "Governance"): Labeled "GOVERNANCE TIER". A monitor icon labeled "Control Tower Dashboard (UI)" showing small charts for "Risk/Impact Metrics". Below it, approval buttons labeled "Human Approval Workflow".

Bottom Right Zone (Red - "Production"): Labeled "PRODUCTION ENVIRONMENT (Live)". Contains an "Execution Engine" with a gear icon, pointing to icons for "Legacy Business Systems (MySQL/ERP)". A curved arrow next to it is labeled "Rollback Mechanism".

Data Flow Arrows (Must be numbered and clearly labeled):

Arrow from Access Layer to Control Plane labeled: "1) Submit Intent/Task".

Arrow from Control Plane to Simulation Sandbox labeled: "2) Trigger Simulation Request (Read-Only)".

Internal arrow within Simulation labeled: "3) Generate ChangeSet & Analyze Risk".

Arrow from Simulation Sandbox to Control Plane/Governance Tier labeled: "4) Return Simulation Results".

Two-way arrow between Governance Tier and Control Plane labeled: "5) Review & Approve ChangeSet".

Arrow from Control Plane to Production Environment labeled: "6) Trigger Execution (Apply ChangeSet)".

Arrow from Production to Control Plane labeled: "7) Execution Status & Audit Logs".

选项 B：给设计师的结构化指令 (用于手动绘制)
请参考原图的流程逻辑，绘制一张标准的四层技术架构图。

布局结构： 采用左至右，上至下的分层布局。

最左侧列 - 接入层 (Access Layer)：

绘制用户终端（手机/PC），标注“钉钉/Web集成”。

绘制“意图解析网关”和“本地边缘代理”组件。

中间上层 - 控制平面层 (Control Plane Layer)：

画一个大的云框，标题为“云谷 AI-OS 控制中枢”。

内部包含四个核心微服务方块：动作编排、状态管理、策略引擎、审计日志。

中间下层 - 仿真层 (Simulation Layer) - 重点突出安全边界：

画一个带有明显锁标志和虚线边界的区域，标题为“仿真沙箱环境 (只读)”。

包含组件：数据快照（租户隔离）、仿真计算引擎、影响分析服务。

输出物图标：变更集(ChangeSet)与风险报告。

右侧上层 - 治理层 (Governance Layer)：

绘制“控制塔运维台”界面。

强调可视化报表和审批流（Approval Flow）组件。

右侧下层 - 生产执行层 (Production Layer)：

标题为“生产业务环境”。

包含“幂等执行器”组件。

连接到后端的“核心业务数据库/应用集群”。

增加“事务回滚”模块。

数据流连线（关键）：
严格按照原图的 1-7 顺序连接各层级，并使用技术语言标注连线（例如，将“请求预演”改为“触发只读仿真请求”；将“触发执行”改为“下发已批准变更集”）。
