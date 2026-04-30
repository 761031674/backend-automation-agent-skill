---
name: project-context
description: 项目规范引用管理层，统一维护所有 specifications 文件的引用目录和内容摘要。Agent 通过本 Skill 了解可用规范及其查阅路径，不再自行硬编码 specifications 路径。当需要查阅项目专属规范、了解项目架构约束、获取代码规范时使用。触发词：项目上下文、项目规范、查阅规范、规范引用、项目配置
---

# 项目上下文

> 本 Skill 是项目规范引用的**统一管理层**。
> 所有项目专属详细规范存放在 `specifications/{project-name}/` 目录下，本 Skill 维护引用目录的结构模板和查阅指引。
> **原则**：Agent 引用规范时，应通过本 Skill 查找对应规范，避免自行硬编码 specifications 路径。

## 如何确定当前项目

Agent 按以下优先级确定当前项目：

1. **用户显式指定**（如用户说"切换到 project-b"）
2. **上下文推断**（如当前正在处理 `project-b` 的代码文件）
3. **默认项目**：`specifications/` 目录下唯一的子目录
4. 如有多个项目且无法确定，**询问用户**当前应使用哪个项目的规范

## 规范引用目录模板

当确定项目名为 `{project-name}` 时，规范文件路径如下：

| 规范领域 | 文件路径 | 内容摘要 |
|---------|---------|---------|
| 项目全景 | `specifications/{project-name}/overview/project-overview.md` | 项目基本信息、微服务列表、技术栈版本、业务领域、错误码规范、响应格式、架构约束 |
| 架构设计 | `specifications/{project-name}/overview/architecture-design.md` | 架构原则、服务拆分、通信方式、领域建模 |
| API 设计 | `specifications/{project-name}/design/api-design.md` | 接口设计规范、URL 规范、版本管理、Swagger 注解 |
| 数据库设计 | `specifications/{project-name}/design/database-design.md` | 表设计规范、索引规范、SQL 规范、ORM框架最佳实践 |
| 编码规范 | `specifications/{project-name}/coding/coding-standard.md` | 实体类规范、日志规范、代码风格、依赖注入 |
| 安全审计 | `specifications/{project-name}/coding/security-audit.md` | 安全审查要点、漏洞检查清单、安全框架使用 |
| 代码审查 | `specifications/{project-name}/coding/code-review.md` | 审查流程、评论格式、成功指标 |

> **注意**：具体规范文件以 `specifications/{project-name}/` 目录下实际存在的文件为准。各项目可根据需要调整子目录结构和增删规范文件。

## 使用指引

Agent 应在以下场景查阅对应规范：

| 场景 | 查阅规范 |
|-----|---------|
| 了解项目技术栈、微服务划分、功能归属服务 | `specifications/{project-name}/overview/project-overview.md` |
| 设计接口、确定 URL/参数/响应规范 | `specifications/{project-name}/design/api-design.md` + `overview/project-overview.md` |
| 设计数据库表、字段、索引、约束 | `specifications/{project-name}/design/database-design.md` + `coding/coding-standard.md` |
| 编写/审查代码（实体类、日志、安全） | `specifications/{project-name}/coding/coding-standard.md` |
| 检查安全漏洞、输入验证、鉴权 | `specifications/{project-name}/coding/security-audit.md` |
| 进行代码审查、输出审查报告 | `specifications/{project-name}/coding/code-review.md` + `coding/coding-standard.md` |
| 确定服务拆分、架构决策、ADR | `specifications/{project-name}/overview/architecture-design.md` + `overview/project-overview.md` |

## 边界条件

### 规范文件缺失
**处理**：如果 `specifications/{project-name}/` 下某文件（含子目录内）不存在，告知用户该规范尚未定义，建议参考已有规范或项目代码推断。

### 规范冲突
**处理**：如果 specifications 之间出现矛盾（如 `coding-standard.md` 与 `security-audit.md` 对同一问题有不同要求），以 `security-audit.md` 为准，并提醒用户更新规范。

### 多项目共存
**处理**：`specifications/` 下可存在多个项目目录（如 `specifications/project-b/`）。Agent 必须通过上述"如何确定当前项目"的流程明确当前项目，避免混用不同项目的规范。
