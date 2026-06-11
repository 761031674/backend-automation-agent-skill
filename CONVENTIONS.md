# 使用约定

## Skill 加载规则

Agent 在正文的 `## 前置加载` 章节中声明依赖的 Skill，路径格式：`skills/{skill-name}/SKILL.md`

所有平台统一行为：
- AI 读取 Agent 文件后，主动读取 `## 前置加载` 中列出的 Skill 文件
- 将 Skill 内容作为上下文的一部分
- 执行 Agent 工作流时遵循 Skill 中的规范

## 项目配置

项目专属配置位于 `projects/{project-name}/config.md`，包含：
- 包路径前缀、表名前缀
- 技术栈版本
- 实体规范、日志规范
- 依赖注入规范等

Agent 读取配置时：
1. 优先读取 `projects/{project-name}/config.md`
2. 配置文件不存在时使用通用默认值
3. 详细规范查阅 `projects/{project-name}/specifications/`

## 独立使用降级规则

当文件不存在时，按以下规则处理：

| 文件类型 | 不存在时的行为 |
|---------|---------------|
| `skills/xxx/SKILL.md` | 跳过该 Skill，使用 Agent 正文中的通用规范继续执行 |
| `projects/{project}/config.md` | 使用通用默认值（包路径 `{basePackage}`、表名前缀 `t_`、日志 SLF4J、注入 @Autowired） |
| `projects/{project}/specifications/` | 跳过项目规范引用，使用行业通用最佳实践 |

## 独立使用

拷贝 Agent 到其他项目时：
1. 同时拷贝 `skills/` 目录，或确保目标环境有对应 Skill
2. 创建 `projects/{project}/config.md` 提供项目配置
3. 无 config.md 时使用通用默认值

## 目录结构约定

```
.qoder/
├── agents/           # Agent 定义文件
├── skills/           # Skill 定义文件（每个 Skill 一个目录）
├── projects/         # 项目配置（按项目名分目录）
│   └── {project}/
│       ├── config.md            # 项目专属配置
│       └── specifications/      # 项目专属规范
├── rules/            # 全局规则（qoder 专用）
├── README.md         # 项目说明
└── CONVENTIONS.md    # 使用约定（本文件）
```
