---
name: artifact-generator
description: 成果物生成规范，定义从技术设计文档生成数据库DDL、Entity、Controller、Service、Mapper/Repository、DTO/VO、单元测试的完整规范、代码模板和验证检查清单。当执行代码生成、脚手架搭建、根据设计文档产出代码时使用。触发词：代码生成、生成DDL、生成实体类、生成Controller、生成Service、生成Mapper、生成repository、生成单元测试、脚手架、代码骨架
---

> **项目上下文**：当前项目的包路径、技术栈版本、ORM框架、架构约束通过 `project-context` 查阅。
>
> **编码规范**：实体类规范、日志规范、代码风格通过 `project-context` 查阅。

# 成果物生成

## 支持的成果物类型

| 类型 | 文件扩展名 | 生成依据 |
|------|-----------|---------|
| 数据库 DDL | `.sql` | 数据库设计（表结构、字段、索引、约束） |
| 迁移脚本 | `.sql` | 数据库设计 + 版本号 |
| Entity 实体类 | `.java` | 数据库设计 + 字段映射 |
| DTO / VO | `.java` | 接口设计（请求/响应参数） |
| Controller | `.java` | 接口设计（URL/Method/参数/响应） |
| Service | `.java` | 核心流程 |
| Mapper 接口 + XML | `.java` + `.xml` | 数据库设计 + 复杂查询 |
| Repository | `.java` | 数据库设计 + 复杂查询 |
| 单元测试 | `.java` | Service/Controller 逻辑 |

## 生成顺序

```
数据库DDL → 迁移脚本 → Entity → DTO/VO → Mapper/Repository → Service → Controller → 单元测试
```

> **原则**：先生成数据层，再生成业务层，最后生成表现层。每层生成后验证通过再继续下一层。

## 生成规范

各类成果物的具体代码模板、项目专属约束和命名规范，详见 `specifications/cls/artifact-template.md`（通过 `project-context` 查阅）。

本 Skill 仅定义通用生成框架，具体实现模板和项目专属约束均引用上述规范文件。

### 各类型通用约束摘要

| 类型 | 包路径占位 | 核心约束 |
|------|-----------|---------|
| 数据库 DDL | — | 表名前缀、索引命名、基础实体字段、字符集通过 `project-context` 查阅 |
| Entity | `{basePackage}.{serviceName}.entity` | 继承基础实体类、禁用 `@Data`、ORM 注解通过 `project-context` 查阅 |
| DTO / VO | `{basePackage}.{serviceName}.dto` / `.vo` | API 文档注解、校验注解、日期格式通过 `project-context` 查阅 |
| Mapper | `{basePackage}.{serviceName}.mapper` | 继承 `BaseMapper<T>`、防 SQL 注入、逻辑删除条件通过 `project-context` 查阅 |
| Repository | `{basePackage}.{serviceName}.repository` | 封装 Mapper、Service 通过 Repository 访问数据 |
| Service | `{basePackage}.{serviceName}.service` | 禁止互相注入、`@Transactional`、日志规范通过 `project-context` 查阅 |
| Controller | `{basePackage}.{serviceName}.controller` | 返回统一响应类型、参数校验、安全校验通过 `project-context` 查阅 |
| 单元测试 | `{basePackage}.{服务名}.service` | JUnit 5 + Mockito、命名规范、覆盖路径要求 |

## 验证检查清单

每类成果物生成后必须验证：

- [ ] 包路径符合 `{basePackage}.{serviceName}.{layer}` 规范
- [ ] Entity 继承项目基础实体类，含逻辑删除字段
- [ ] Controller 返回项目标准响应类型，API 文档注解完整
- [ ] Service 不互相注入，通过 Repository 访问数据
- [ ] Mapper XML 含逻辑删除条件
- [ ] DDL 表名加项目前缀，含逻辑删除字段
- [ ] 日期字段类型与 JSON 序列化格式符合项目规范
- [ ] 单元测试覆盖正常路径 + 至少1个异常路径

## 异常场景处理

| 场景 | 处理策略 |
|------|---------|
| 设计文档字段类型缺失 | 使用最合理的默认类型推断（如 String/VARCHAR），标注 `// TODO: 请确认字段类型` |
| 设计文档与现有代码冲突（类名/表名已存在） | 提示冲突点，提供重命名建议或扩展现有类的方案 |
| 用户要求只生成部分成果物 | 确认生成范围，跳过未指定的类型，提醒 skipped 的依赖关系 |
| 设计文档中接口未定义响应DTO | 根据接口描述推断响应结构，命名为 `{方法名}Response`，标注待确认 |
| 复杂查询无法从设计文档推断 | 生成基础 CRUD Mapper，复杂查询标注 `// TODO: 请补充复杂查询SQL` |

## 参考资源

- **代码模板**：具体代码模板（含项目专属实现）见 `specifications/cls/artifact-template.md`（通过 `project-context` 查阅）。
- **共享规范**：通过 `project-context` 查阅以下规范：
  - 项目包路径与服务划分
  - 基础实体规范与数据库约束
  - 统一响应格式与错误码规范
  - 日志与安全规范
  - API 文档注解规范
