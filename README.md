# 云谷智慧校园 · 技术架构图

云谷教育 SaaS 系统的端到端技术架构可视化，以单页 HTML 形式呈现完整的系统分层设计。

## 架构视图

本项目包含四张架构图，覆盖从技术实现到 AI 战略的不同视角：

```
index.html          研发内视图 — "系统有什么"
cloud.html          平台外视图 — "平台能给谁用"
cognitive-arch.html AI 原生架构 — "脑手分离 + 双螺旋"
deployment-arch.html部署架构   — "实际跑在哪里"
```

### 架构分层 (index.html)

```
访问入口层     传统用户端 (教师/学生/家长/管理员) + AI Agent 端 (MCP Client)
网络接入层     CDN · WAF · SLB · API Gateway · BFF | MCP Gateway · Agent API 路由 · 鉴权 · 限流
业务网关层     OpenAPI 网关 | Agent API & MCP
业务中心       教学线 · 育人线 · 教务线 · 校务线
微服务中心     课程 · 评价 · 任务 · 日程 · 通用 · 健康 · 人员
支撑层         基础数据中台 (MDM) | 公共支撑服务
基础设施层     K8S · Nacos · MySQL · Redis · OSS · RocketMQ · ES
```

### AI 原生架构 (cognitive-arch.html)

```
应用层         官方应用 | 第三方 | AI Agents (教师工具)
认知决策层     认知引擎 (双螺旋: 学生被动数据 ↔ 教师主动 AI)
               ├ 学生侧: StudentProfile → CognitiveDiagnostics → WorldModel (被动采集)
               └ 教师侧: ContentFactory (内容提效) + TeachingDiagnostics (质量提升)
编排层         TaskGraph Orchestrator (DAG + 并行 + 人工检查点 + 沙箱模拟)
模型层         Model Council (安全等级感知路由, L1私有/L2脱敏/L3禁止)
执行操作层     Open Platform (Domain API + Resource API)
底座能力层     Storage | Compute | Network | Security + 数据主权三道防线
基础设施层     PaaS | 存储 | AI 推理 | 外部集成
```

### 部署架构 (deployment-arch.html)

```
Internet Edge  CDN → WAF → SLB
VPC            cn-hangzhou
├ Gateway      NLB(内网) · Nginx Ingress · BFF · API Gateway(规划)
├ ACK K8S      7 微服务 + agent-api-gateway (82 API)
│ ├ AI Tier    Cognitive Engine · TaskGraph · Model Council (规划)
│ └ GPU Tier   Private Model · Inference · Embedding (规划)
├ Data         MySQL · Redis · OSS · RocketMQ · ES + Vector/Graph DB(规划)
└ External     DingTalk · Qwen API · DeepSeek · IoT
```

## 文件说明

| 文件 | 说明 | 受众 |
|------|------|------|
| `index.html` | 主架构图 — 端到端技术架构 | 研发团队 |
| `cloud.html` | 云架构图 — 开放平台战略蓝图 | 管理层 / CEO |
| `cognitive-arch.html` | AI 原生架构 — 脑手分离 + 双螺旋 | 架构师 / AI 团队 |
| `deployment-arch.html` | 部署架构 — 阿里云基础设施拓扑 | 运维 / SRE |
| `requirements.md` | 系统需求文档 | — |
| `architecture-analysis.md` | 架构分析报告 | — |
| `cloud-architecture-analysis.md` | 云架构分析报告 | — |

## 本地预览

浏览器直接打开任一 `.html` 文件即可。

## 技术栈

- HTML5 + CSS3 (Grid / Flexbox)
- 纯静态，无构建依赖
- 阿里巴巴普惠体 (CDN)
