# Agent-Skill 生态系统

通过定义专用 Agent（角色）和 Skill（能力插件）实现 AI 辅助的软件工程全流程覆盖。

## 架构概览

```
.qoder/
├── agents/           # 角色定义（5个Agent）
├── skills/           # 通用能力插件（4个Skill）
├── projects/         # 项目配置（按项目名分目录）
│   └── {project}/
│       ├── config.md            # 项目专属配置（自行准备/让ai自动补充）
│       └── specifications/      # 项目专属规范（自行准备/让ai自动补充）
|—— CONVENTIONS.md    # 使用约定
```

**核心设计原则**：
- **Agent 定义角色和工作流程**，Skill 定义通用能力模板
- **项目配置与通用能力分离**：Agent/Skill 通用化，项目专属值集中在 `config.md`
- **独立可用**：每个 Agent/Skill 可独立于项目使用，无 config.md 时使用通用默认值
- **纯 Markdown 格式**：所有文件可被任意 LLM 直接消费

---

## Agent 清单（5个）

| Agent | 核心职责 | 触发词 |
|-------|---------|--------|
| [需求分析师](agents/requirements-analyst.md) | 业务需求分析、逻辑校验、AC细化、系统冲突审查 | 需求分析、需求评审、AC细化 |
| [架构师](agents/architect.md) | 系统架构、领域建模、API/数据库设计、性能优化、技术文档、ADR | 系统架构、后端设计、API设计、技术设计、SQL优化、ADR、深度检查 |
| [代码生成工程师](agents/code-generator.md) | 生成DDL/Entity/Mapper/DTO/VO/Service/Controller/测试 | 生成DDL、生成Entity、生成Controller、生成Service |
| [质量保障专家](agents/code-reviewer.md) | 代码审查、测试失败分析、集成测试设计、覆盖率评估 | 代码审查、Code Review、PR评审、测试失败、测试分析、深度检查 |
| [生态治理师](agents/ecosystem-governor.md) | 管理Agent/Skill生态、审查合规性、Skill质量评估 | 创建agent、创建skill、生态治理 |

---

## Skill 清单（4个）

| Skill | 功能 | 使用场景 |
|-------|------|---------|
| [artifact-generator](skills/artifact-generator/SKILL.md) | 成果物代码模板库 | 生成DDL/Entity/Controller/Service代码模板 |
| [parallel-dispatch](skills/parallel-dispatch/SKILL.md) | 并行任务分派 | 多模块并行排查、多接口并行生成 |
| [encoding-constraint](skills/encoding-constraint/SKILL.md) | 通用编码约束 | 所有编码任务（always_on） |
| [skill-evaluation](skills/skill-evaluation/SKILL.md) | Skill 质量评估（8维度评分） | Skill 质量评估与优化建议 |

---

## 典型工作流

### 新需求开发
```
需求分析师（需求分析/AC细化）
    → 架构师（系统架构/技术设计/数据库设计）
    → 代码生成工程师（DDL/Entity/Mapper/Service/Controller/测试）
    → 质量保障专家（PR审查/测试分析）
```

### 代码优化
```
架构师（诊断/优化方案）
    → 代码生成工程师（实施优化）
    → 质量保障专家（验证优化效果）
```

---

## Agent → Skill 依赖关系

各 Agent 在正文 `## 前置加载` 章节中声明依赖的 Skill：

| Agent | 依赖的 Skill |
|-------|-------------|
| 需求分析师 | encoding-constraint |
| 架构师 | encoding-constraint, parallel-dispatch |
| 代码生成工程师 | artifact-generator, encoding-constraint, parallel-dispatch |
| 质量保障专家 | parallel-dispatch, encoding-constraint |
| 生态治理师 | encoding-constraint, skill-evaluation |

---

## 独立使用指南

### 无项目配置使用

Agent/Skill 可直接拷贝到任意项目使用：

```
1. 拷贝 agents/architect.md → 任意 AI 平台
2. Agent 自动使用通用默认值：
   - 包路径：{basePackage}.{serviceName}.{layer}
   - 表名前缀：t_
   - 日志：SLF4J
   - 注入：@Autowired
```

### 有项目配置使用

```
1. 创建 .qoder/projects/{project}/config.md
2. Agent 读取 config.md 获取项目专属值：
   - 包路径：{basePackage}.{serviceName}.{layer}
   - 表名前缀：{baseTablePrefix}
   - 日志：{baseLogFramework}
   - 注入：{baseInjectAnnotation}
```

---

## 扩展指南

### 新增 Agent
1. 在 `agents/` 下创建 `{agent-name}.md`
2. frontmatter：name、description（含触发词）、color、trigger、glob、skills
3. 正文：角色定义、插件依赖表、项目配置说明、工作流程、异常处理、检查清单

### 新增 Skill
1. 在 `skills/` 下创建 `{skill-name}/SKILL.md`
2. 保持单一职责，不与现有 Skill 重叠
3. 通用 Skill 不包含项目专属内容

### 新增项目规范
1. 在 `projects/` 下创建新项目目录
2. 创建 `config.md`（项目配置）和 `specifications/`（项目规范）

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
| 2026-03 | 建立 Agent-Skill 插件化架构 |
| 2026-06 | Skill 与具体项目解耦 |
| 2026-09 | 架构重构：删除 project-context Skill，改为 projects/{project}/config.md 直接配置 |
| 2026-09 | 激进优化：9 Agent → 5 Agent，7 Skill → 4 Skill |
| 2026-09 | 合并：软件架构师+后端架构师+技术文档工程师→架构师 |
| 2026-09 | 合并：数据层工程师+接口层工程师→代码生成工程师 |
| 2026-09 | 合并：代码审查员+测试分析师→质量保障专家 |
| 2026-09 | 内联：需求分析Skill→需求分析师，代码审查Skill→质量保障专家 |
| 2026-09 | darwin-skill 精简为 skill-evaluation（184行→74行） |
| 2026-09 | 使用频率精简：移除 DevOps 工程师、安全审计师、git-workflow（低频/无引用） |
| 2026-09 | 新增深度检查模式：架构师、质量保障专家支持可选多轮检查 |
| 2026-09 | 质量修复：统一依赖声明为正文「前置加载」方式，移除 frontmatter skills 字段 |
| 2026-06 | 质量修复：artifact-generator/parallel-dispatch 引用更新（旧 Agent 名、平台特定 API） |
| 2026-06 | 质量修复：skill-evaluation 补充评分示例 |
