# 云谷智慧校园 · 技术架构图

云谷教育 SaaS 系统的端到端技术架构可视化，以单页 HTML 形式呈现完整的系统分层设计。

## 架构分层

```
访问入口层     传统用户端 (教师/学生/家长/管理员) + AI Agent 端 (MCP Client)
网络接入层     CDN · WAF · SLB · API Gateway · BFF | MCP Gateway · Agent API 路由 · 鉴权 · 限流
业务网关层     OpenAPI 网关 | Agent API & MCP
业务中心       教学线 · 育人线 · 教务线 · 校务线
微服务中心     课程 · 评价 · 任务 · 日程 · 通用 · 健康 · 人员
支撑层         基础数据中台 (MDM) | 公共支撑服务
基础设施层     K8S · Nacos · MySQL · Redis · OSS · RocketMQ · ES
```

## 文件说明

| 文件 | 说明 |
|------|------|
| `index.html` | 主架构图 — 云谷智慧校园端到端技术架构 |
| `cloud.html` | 云架构图 — 阿里云基础设施视角 |
| `requirements.md` | 系统需求文档 |
| `architecture-analysis.md` | 架构分析报告 |
| `cloud-architecture-analysis.md` | 云架构分析报告 |

## 本地预览

浏览器直接打开 `index.html` 即可。

## 技术栈

- HTML5 + CSS3 (Grid / Flexbox)
- 纯静态，无构建依赖
- 阿里巴巴普惠体 (CDN)
