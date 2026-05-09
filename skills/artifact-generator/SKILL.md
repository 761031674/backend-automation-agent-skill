---
name: artifact-generator
description: 成果物生成规范，定义从技术设计文档生成数据库DDL、Entity、Controller、Service、Mapper/Repository、DTO/VO、单元测试的完整规范、代码模板和验证检查清单。当执行代码生成、脚手架搭建、根据设计文档产出代码时使用。触发词：代码生成、生成DDL、生成实体类、生成Controller、生成Service、生成Mapper、生成repository、生成单元测试、脚手架、代码骨架
trigger: model_decision
---

> **项目上下文**：当前项目的包路径、技术栈版本、ORM框架、架构约束通过 `project-context` 查阅。
>
> **编码规范**：实体类规范、日志规范、代码风格通过 `project-context` 查阅。
>
> **调用方式**：当生成功能归属某个微服务时，先通过 `project-context` 确定当前项目和服务名，再查阅对应规范文件。
>
> **输入格式示例**：
> ```
> 服务名: cls-customer
> 功能: 用户管理模块
> 数据库表:
>   - t_user (id, username, password, status, create_time...)
> 接口:
>   - POST /api/v1/users (创建用户)
>   - GET /api/v1/users/{id} (查询用户)
> 核心流程: [时序图描述]
> ```

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

## 快速决策：何时生成什么

```
有数据库设计? → 是 → 生成 DDL + Entity + Mapper
有接口设计?   → 是 → 生成 DTO/VO + Controller
有核心流程?   → 是 → 生成 Service
用户要求全部? → 是 → 按顺序全部生成
用户要求部分? → 是 → 仅生成指定类型，提醒依赖关系
```

## 生成工作流程

> 快速参考：`数据库DDL → 迁移脚本 → Entity → DTO/VO → Mapper/Repository → Service → Controller → 单元测试`

按以下顺序逐层生成，**每层生成后必须验证通过再继续下一层**。

### 第一步：生成数据层
**输入**：数据库设计（表结构、字段、类型、索引、约束）
**输出**：
- DDL 脚本（`V{版本}__{描述}.sql`）
- Entity 类（继承项目基础实体类，含 `@TableName` 注解）
- Mapper 接口 + XML（继承 `BaseMapper<T>`）
- 如使用 Repository 模式：Repository 接口 + 实现
**检查点与验证动作**：
1. Entity 是否继承项目基础实体类（通过 `project-context` 查阅 `coding-standard.md`）→ **验证**：查看 Entity 类声明行
2. Mapper XML 是否包含逻辑删除条件（`WHERE deleted = 0`）→ **验证**：搜索 XML 中 `deleted` 关键字
3. DDL 表名是否加项目前缀（通过 `project-context` 查阅 `database-design.md`）→ **验证**：检查 `CREATE TABLE` 语句表名
4. 字段类型是否与数据库设计一致 → **验证**：逐字段比对设计文档与 Entity/DDL

> **用户确认**：展示生成的 DDL 和 Entity 摘要，询问"数据层代码是否符合预期？确认后继续生成业务层。"

### 第二步：生成业务层
**输入**：接口设计（URL/Method/请求DTO/响应VO）+ 第一步成果物
**输出**：
- Request DTO（含 `@Schema` 和 `@Valid` 注解）
- Response VO（含 `@Schema` 注解）
- Service 接口 + 实现（含 `@Transactional`、日志）
**检查点**：
1. DTO 字段是否与 Entity 字段一致（字段名、类型、校验规则）
2. Service 是否通过 Repository/Mapper 访问数据（禁止 Service 互相注入）
3. Service 是否包含必要的 `@Transactional` 和日志记录

> **用户确认**：展示 DTO/VO 字段列表和 Service 方法签名，询问"业务层代码是否符合预期？确认后继续生成表现层。"

### 第三步：生成表现层
**输入**：接口设计 + 第二步成果物
**输出**：Controller 类（含 `@Tag`、`@Operation`、`@ApiResponse` 注解）
**检查点**：
1. Controller 是否返回项目标准响应类型（如 `Result<T>`，通过 `project-context` 查阅）
2. API 文档注解是否完整（`@Tag`、`@Operation`、`@ApiResponse`）
3. 参数校验是否使用 `@Valid`
4. 接口 URL 是否符合项目规范（通过 `project-context` 查阅 `api-design.md`）

> **用户确认**：展示 Controller 接口列表（URL + Method），询问"API 接口定义是否符合预期？确认后继续生成单元测试。"

### 第四步：生成单元测试
**输入**：Service 和 Controller 逻辑
**输出**：单元测试类
**检查点**：验证单元测试覆盖正常路径 + 至少1个异常路径。

### 第五步：最终验证与交付
**输入**：所有成果物
**输出**：验证报告 + 成果物包
1. 逐文件对照「验证检查清单」验证
2. 检查成果物之间的字段一致性（数据库 ↔ Entity ↔ DTO）
3. 标注所有 `// TODO` 待确认项
**检查点**：向用户展示生成摘要，确认是否接受或需要调整。

## 生成规范

各类成果物的具体代码模板、项目专属约束和命名规范，优先通过 `project-context` 查阅。若项目未定义专属模板，使用本 Skill 提供的通用占位模板（见下方「成果物生成速查模板」）。

### 各类型通用约束摘要

| 类型 | 包路径占位 | 核心约束 |
|------|-----------|---------|
| 数据库 DDL | — | 表名前缀、索引命名、基础实体字段、字符集通过 `project-context` 查阅 |
| Entity | `{basePackage}.{serviceName}.entity` | 继承基础实体类、禁用 `@Data`、ORM 注解通过 `project-context` 查阅 |
| DTO / VO | `{basePackage}.{serviceName}.dto` / `.vo` | API 文档注解、校验注解、日期格式通过 `project-context` 查阅 |
| Mapper | `{basePackage}.{serviceName}.mapper` | 继承 `BaseMapper<T>`、防 SQL 注入、逻辑删除条件通过 `project-context` 查阅 |
| Service | `{basePackage}.{serviceName}.service` | 禁止互相注入、`@Transactional`、日志规范通过 `project-context` 查阅 |
| Controller | `{basePackage}.{serviceName}.controller` | 返回统一响应类型、参数校验、安全校验通过 `project-context` 查阅 |

### 成果物生成速查模板

> **模板说明**：以下最小模板基于常见 Java 技术栈（Spring Boot + MyBatis-Plus + Swagger v3 + Lombok）编写，仅供 `project-context` 无法获取项目专属模板时作为占位参考。实际生成时，应优先通过 `project-context` 获取项目真实技术栈和包路径进行替换。

当通过 `project-context` 无法获取具体规范时，使用以下占位模板：

**Entity 最小模板**：
```java
package {basePackage}.{serviceName}.entity;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Getter;
import lombok.Setter;
import java.time.LocalDateTime;

@Getter
@Setter
@TableName("{tablePrefix}_{tableName}")
public class {EntityName} extends BaseEntity {
    // TODO: 根据设计文档添加字段
}
```

**Service 最小模板**：
```java
package {basePackage}.{serviceName}.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Slf4j
@Service
public class {ServiceName} {
    // TODO: 根据设计文档实现业务逻辑
}
```

**Request DTO 最小模板**：
```java
package {basePackage}.{serviceName}.dto;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.NotBlank;
import lombok.Data;

@Data
@Schema(description = "{描述}")
public class {RequestDTO} {
    @NotBlank(message = "字段不能为空")
    @Schema(description = "字段描述", example = "示例值")
    private String fieldName;
    // TODO: 根据接口设计添加字段及校验注解
}
```

**Response VO 最小模板**：
```java
package {basePackage}.{serviceName}.vo;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;

@Data
@Schema(description = "{描述}")
public class {ResponseVO} {
    // TODO: 根据接口设计添加字段
}
```

**Controller 最小模板**：
```java
package {basePackage}.{serviceName}.controller;

import io.swagger.v3.oas.annotations.tags.Tag;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/{path}")
@RequiredArgsConstructor
@Tag(name = "{控制器名称}", description = "{描述}")
public class {ControllerName} {
    // TODO: 根据接口设计添加方法
}
```

**Mapper XML 最小模板**：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="{basePackage}.{serviceName}.mapper.{MapperName}">

    <!-- 基础 CRUD 由 MyBatis-Plus 提供，复杂查询在此补充 -->
    <!-- TODO: 根据设计文档补充复杂查询 -->

</mapper>
```

**DDL 最小模板**：
```sql
-- V{版本}__{描述}.sql
CREATE TABLE `{tablePrefix}_{tableName}` (
    `id` BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY COMMENT '主键ID',
    -- TODO: 根据设计文档添加字段
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='{表注释}';
```

**单元测试 最小模板**：
```java
package {basePackage}.{serviceName}.service;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class {ServiceName}Test {

    @Mock
    private {RepositoryName} {repositoryName};

    @InjectMocks
    private {ServiceName} {serviceName};

    @Test
    void should_success_when_{scenario}() {
        // TODO: 编写正常路径测试
    }

    @Test
    void should_fail_when_{scenario}() {
        // TODO: 编写异常路径测试
    }
}
```

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
- [ ] 生成范围与用户请求一致，无遗漏也未多做
- [ ] 所有 `// TODO` 项已在摘要中列出并说明原因

## 异常场景处理

| 场景 | 处理策略 |
|------|---------|
| 设计文档字段类型缺失 | 使用最合理的默认类型推断（如 String/VARCHAR），标注 `// TODO: 请确认字段类型` |
| 设计文档与现有代码冲突（类名/表名已存在） | 提示冲突点，提供重命名建议或扩展现有类的方案 |
| 用户要求只生成部分成果物 | 确认生成范围，跳过未指定的类型，提醒 skipped 的依赖关系 |
| 设计文档中接口未定义响应DTO | 根据接口描述推断响应结构，命名为 `{方法名}Response`，标注待确认 |
| 复杂查询无法从设计文档推断 | 生成基础 CRUD Mapper，复杂查询标注 `// TODO: 请补充复杂查询SQL` |
| 设计文档缺少核心流程定义 | 生成基础 CRUD 骨架，核心业务逻辑标注 `// TODO: 请补充业务规则` |
| 表名与项目命名规范冲突 | 按项目命名规范转换表名（如加前缀），在输出中提示用户确认 |
| `project-context` 无法查阅规范 | 使用占位符（如 `{basePackage}`），在代码中标注 `// TODO: 请确认项目包路径` |
| 设计文档中缺少索引设计 | 根据 WHERE 条件和 JOIN 字段推断必要索引，标注 `// TODO: 请确认索引设计` |
| 字段命名不符合项目规范 | 按项目命名规范转换（如驼峰/下划线），在注释中标注原始字段名 |
| 设计文档要求的技术栈与项目不符 | 使用项目实际技术栈替换，在注释中说明替换原因 |

## 参考资源

- **代码模板**：通用占位模板见上文「成果物生成速查模板」；项目专属实现优先通过 `project-context` 查阅。
- **共享规范**：通过 `project-context` 查阅项目包路径、基础实体规范、响应格式、日志安全、API 文档注解等规范
