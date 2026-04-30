# Agent-Skill 生态系统

本项目是 Agent-Skill 插件化架构配置，通过定义专用 Agent（角色）和 Skill（能力插件）实现 AI 辅助的软件工程全流程覆盖。

## 架构概览

```
backend-automation-agent-skill/
├── agents/           # 角色定义（6个Agent）
├── skills/           # 通用能力插件（6个Skill）
└── specifications/   # 项目专属规范（按类型分类）
```

**核心设计原则**：
- **Agent 定义角色和工作流程**，Skill 定义具体规范和模板
- **project-context** 作为唯一规范引用入口，Agent/Skill 不直接硬编码 specifications 路径
- **多项目适配**：切换 `specifications/` 下的子目录即可适配不同项目
- **解耦分层**：通用 Skill ↔ project-context 路由层 ↔ specifications 项目专属内容

---

## Agent 清单（6个）

| Agent | 层级 | 核心职责 | 触发词示例 |
|-------|------|---------|-----------|
| [软件架构师](agents/software-architect.md) | 战略层 | DDD领域建模、服务拆分、架构演进、ADR、技术选型 | 系统架构、领域建模、服务拆分 |
| [后端架构师](agents/backend-architect.md) | 战术层 | 后端架构设计、详细技术设计、API/数据库设计、性能优化 | 后端设计、技术设计、详细设计、SQL优化 |
| [交付工程师](agents/delivery-engineer.md) | 实现层 | 根据设计文档生成代码成果物（DDL/Entity/Controller/Service/Mapper/DTO/测试） | 代码生成、生成实体类、生成Controller、脚手架 |
| [技术文档工程师](agents/technical-writer.md) | 文档层 | Swagger注解、JavaDoc、API文档编写 | API文档、JavaDoc、Swagger |
| [代码审查员](agents/code-reviewer.md) | 质量层 | 代码审查、安全漏洞、性能问题、规范检查 | 代码审查、Code Review、PR评审 |
| [生态治理师](agents/ecosystem-governor.md) | 元管理层 | 管理Agent/Skill生态、审查合规性、调用darwin-skill持续优化 | 创建agent、创建skill、优化skill、生态治理 |

---

## Skill 清单（6个）

| Skill | 功能 | 使用场景 |
|-------|------|---------|
| [project-context](skills/project-context/SKILL.md) | 项目规范引用管理 | 所有需要查阅项目规范的场景 |
| [artifact-generator](skills/artifact-generator/SKILL.md) | 成果物生成规范 | 代码生成、脚手架搭建 |
| [code-review](skills/code-review/SKILL.md) | 代码审查规范 | 代码审查、PR评审 |
| [parallel-dispatch](skills/parallel-dispatch/SKILL.md) | 并行任务分派 | 多模块并行设计、多故障并行排查 |
| [encoding-constraint](skills/encoding-constraint/SKILL.md) | 通用编码约束 | 所有编码与输出任务 |
| [darwin-skill](skills/darwin-skill/SKILL.md) | Skill自主优化（8维度评分+hill-climbing） | Skill质量评估与持续优化 |

---

## 规范目录（specifications/project-name/）

规范文件按类型分类存放，支持多项目适配：

```
specifications/project-name/
├── overview/                    # 项目全景与架构
│   ├── project-overview.md      # 项目基本信息、技术栈、微服务列表、错误码
│   └── architecture-design.md   # 架构设计规范、领域建模、服务拆分
├── design/                      # 设计规范
│   ├── api-design.md            # API设计规范、Swagger注解、DTO/VO
│   └── database-design.md       # 数据库设计规范、MyBatis-Plus、SQL优化
└── coding/                      # 编码与质量规范
    ├── coding-standard.md       # 编码规范、实体类、日志、依赖注入
    ├── code-review.md           # 代码审查检查清单、评论示例
    └── security-audit.md        # 安全审计规范、安全框架、XSS/SQL注入防护
```

> **多项目适配**：新增项目时，在 `specifications/` 下创建新的项目目录（如 `specifications/project-b/`），按相同结构放置规范文件即可。Agent 和 Skill 无需任何修改。

---

## 典型工作流

### 1. 新需求开发流程
```
业务需求 → 软件架构师（DDD建模/服务拆分） → 后端架构师（详细技术设计）
→ 交付工程师（生成代码成果物） → 代码审查员（PR审查）
```

### 2. 现有代码优化流程
```
慢SQL/性能问题 → 后端架构师（诊断+优化方案） → 交付工程师（实施优化）
→ 代码审查员（验证优化效果）
```

### 3. 生态维护流程
```
新增Agent/Skill需求 → 生态治理师（审查合规性+生成规范文件）
→ darwin-skill（持续优化现有Skill质量）
```

---

## 扩展指南

### 新增 Agent
1. 在 `agents/` 下创建 `{agent-name}.md`
2. 遵循 frontmatter 规范：name、description（含触发词）、color、trigger、glob、skills
3. 正文包含：角色定义、插件依赖表、工作流程（分场景）、异常处理、检查清单
4. 通过 `project-context` 引用项目规范，禁止直接硬编码 `specifications/**` 路径

### 新增 Skill
1. 在 `skills/` 下创建 `{skill-name}/SKILL.md`
2. 保持单一职责原则，不与现有 Skill 重叠
3. 通用 Skill 不包含项目专属内容（包路径、类名等）
4. 如需项目专属约束，通过 `project-context` 查阅

### 新增项目规范
1. 在 `specifications/` 下创建新项目目录（如 `specifications/new-project/`）
2. 按 `overview/`、`design/`、`coding/` 结构组织规范文件
3. 各项目可根据需要增删规范文件

---

## 参考与致谢

本生态系统的部分设计参考了以下开源项目：

| 项目 | 对应组件 | 说明 |
|------|---------|------|
| [dispatching-parallel-agents](https://github.com/wj4616/dispatching-parallel-agents/blob/master/SKILL.md) | `parallel-dispatch` Skill | 并行任务分派与多智能体并发执行的设计参考 |
| [karpathy-skills](https://github.com/whitesmell/karpathy-skills) | `darwin-skill` Skill | 基于 Karpathy  autoresearch 理念的 Skill 自主优化机制 |
| [agency-agents-zh](https://github.com/jnMetaCode/agency-agents-zh?tab=readme-ov-file) | Agent 生态架构 | 中文 Agent 角色定义与多智能体协作框架参考 |

---

## 架构演进记录

| 时间 | 变更 |
|------|------|
| 初始 | 建立 Agent-Skill 插件化架构，project-context 作为规范路由层 |
| 扩展 | 添加交付工程师 + artifact-generator，实现设计到代码的完整链路 |
| 解耦 | Skill与具体项目解耦，项目专属内容迁移至 specifications |
| 分类 | specifications 按 overview/design/coding 三层分类 |
| 合并 | tech-designer 合并至 backend-architect，消除职责重叠 |
| 治理 | 添加生态治理师 + darwin-skill，实现生态自治与持续优化 |
