---
name: project-context
description: 项目规范引用管理层，统一维护所有 specifications 文件的引用目录和内容摘要。Agent 通过本 Skill 了解可用规范及其查阅路径，不再自行硬编码 specifications 路径。当需要查阅项目专属规范、了解项目架构约束、获取代码规范时使用。触发词：项目上下文、项目规范、查阅规范、规范引用、项目配置
trigger: model_decision
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

> **检查点**：当通过步骤1-3仍无法确定当前项目时，**必须暂停并向用户确认**，避免混用不同项目的规范。

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

## 标准使用流程

Agent 引用项目规范时，遵循以下步骤：

1. **确定项目**：按「如何确定当前项目」的优先级流程，明确当前使用的项目名
2. **匹配场景**：对照「使用指引」表格，找到当前任务对应的规范文件
3. **验证存在**：确认该规范文件在 `specifications/{project-name}/` 下实际存在
4. **应用规范**：读取规范内容并应用到当前任务中
5. **引用来源**：在输出中注明引用了哪份规范（如"根据 `coding-standard.md`..."）

> **检查点**：如果步骤3发现规范文件不存在，或规范内容与其他来源冲突，**告知用户并请求确认**，不要基于不完整或矛盾的信息继续执行。

## 任务启动前自检

每次需要查阅项目规范时，先回答以下问题，确认找对规范、用对场景：

- [ ] **项目已确定**：我能明确说出当前项目名（如 `cls`），且不是猜测的
- [ ] **场景已匹配**：我能从「使用指引」表格中指出当前任务对应的 1~3 个规范文件
- [ ] **文件已验证**：我确认了这些规范文件在 `specifications/{project-name}/` 下**实际存在**
- [ ] **内容已核对**：规范文件的内容摘要与当前任务目标**基本匹配**（如不匹配，可能存在规范过时或任务理解偏差）

> **自检结果**：
> - 全部勾选 → 继续应用规范
> - 任一未勾选 → 暂停执行，告知用户具体情况并请求确认

## 使用示例

### 示例1：生成代码时查阅规范

```
用户：帮我生成用户管理模块的代码
→ 1. 确定项目：cls（specifications/下唯一目录）
→ 2. 匹配场景：代码生成 → 需查阅 coding-standard.md + database-design.md + api-design.md
→ 3. 验证存在：确认 specifications/cls/coding/coding-standard.md 存在
→ 4. 应用规范：按规范生成Entity（继承BaseEntity）、Service（不互相注入）等
→ 5. 引用来源：输出"根据 coding-standard.md 的实体类规范..."
```

### 示例2：代码审查时查阅规范

```
用户：审查这个PR
→ 1. 确定项目：cls（从PR涉及的文件路径推断）
→ 2. 匹配场景：代码审查 → 需查阅 code-review.md + coding-standard.md + security-audit.md
→ 3. 验证存在：确认 specifications/cls/coding/code-review.md 存在
→ 4. 应用规范：按规范执行分层审查（安全性→规范性→性能→可维护性）
→ 5. 引用来源：输出"根据 security-audit.md 的安全审查要点..."
```

### 示例3：数据库设计时查阅规范

```
用户：帮我设计用户表和订单表
→ 1. 确定项目：cls
→ 2. 匹配场景：数据库设计 → 需查阅 database-design.md + coding-standard.md
→ 3. 验证存在：确认 specifications/cls/design/database-design.md 存在
→ 4. 应用规范：按规范设计表结构（表名前缀、逻辑删除字段、索引命名）
→ 5. 引用来源：输出"根据 database-design.md 的表设计规范..."
```

### 示例4：安全审计时查阅规范

```
用户：检查这段登录代码有没有安全问题
→ 1. 确定项目：cls
→ 2. 匹配场景：安全审计 → 需查阅 security-audit.md + coding-standard.md
→ 3. 验证存在：确认 specifications/cls/coding/security-audit.md 存在
→ 4. 应用规范：按规范检查 SQL 注入、XSS、输入校验、鉴权绕过
→ 5. 引用来源：输出"根据 security-audit.md 的安全审查要点..."
```

## 边界条件

### 规范文件缺失
**处理**：如果 `specifications/{project-name}/` 下某文件（含子目录内）不存在：
1. 告知用户该规范尚未定义
2. **推断指引**：按以下优先级从已有信息推断：
   - 参考同项目其他规范文件（如 `coding-standard.md` 缺失时，参考 `code-review.md` 中的规范要求）
   - 参考项目代码中的现有实现（如查看已有 Entity 类推断基础实体规范）
   - 使用行业通用最佳实践作为临时替代
3. 在输出中标注"该规范尚未定义，以下为基于项目代码/行业最佳实践的推断"
4. 建议用户补充该规范文件

### 规范冲突
**处理**：如果 specifications 之间出现矛盾（如 `coding-standard.md` 与 `security-audit.md` 对同一问题有不同要求），以 `security-audit.md` 为准，并提醒用户更新规范。

### 多项目共存
**处理**：`specifications/` 下可存在多个项目目录（如 `specifications/cls/`、`specifications/project-b/`）。Agent 必须通过上述"如何确定当前项目"的流程明确当前项目，避免混用不同项目的规范。

### 规范文件已过时
**处理**：如果规范文件内容明显与项目代码现状不符（如规范要求使用旧版本框架，代码已升级），以项目代码现状为准，并提醒用户更新规范文件。

### 规范文件存在但内容为空/不完整
**处理**：如果文件存在但内容极少（如仅标题、无实质规范），视为"规范尚未定义"：
1. 告知用户该规范文件当前为空
2. 按「规范文件缺失」的推断优先级处理
3. 在输出中标注"该规范文件内容为空，以下为基于推断的临时规范"
4. 建议用户补充该规范文件
