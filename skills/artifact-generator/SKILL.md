---
name: artifact-generator
description: 成果物代码模板库与生成约束规范，提供数据库DDL、Entity、Controller、Service、Mapper/Repository、DTO/VO、单元测试的最小代码模板、类型约束表和验证检查清单。由代码生成工程师等 Agent 调用获取模板素材，本 Skill 不定义生成流程。触发词：代码模板、生成规范、成果物模板、代码约束
trigger: model_decision
---

# 成果物代码模板库

> **定位声明**：本 Skill 是**模板库 + 约束规范**，不定义生成工作流程。
> 生成流程的控制权属于调用方 Agent（如 `code-generator`）。

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

## 快速决策

```
有数据库设计? → 是 → 生成 DDL + Entity + Mapper
有接口设计?   → 是 → 生成 DTO/VO + Controller
有核心流程?   → 是 → 生成 Service
用户要求全部? → 是 → 按顺序全部生成
用户要求部分? → 是 → 仅生成指定类型，提醒依赖关系
```

## 生成顺序

```
数据库DDL → 迁移脚本 → Entity → DTO/VO → Mapper/Repository → Service → Controller → 单元测试
```

## 项目配置

项目专属配置（包路径、表名前缀、基础实体类、日志框架等）通过读取 `.qoder/projects/{project-name}/config.md` 获取。如配置文件不存在，使用通用默认值。

## 通用约束摘要

| 类型 | 核心约束 |
|------|---------|
| 数据库 DDL | 表名前缀、索引命名、基础实体字段、字符集通过 config.md 获取 |
| Entity | 继承基础实体类、ORM 注解通过 config.md 获取 |
| DTO / VO | API 文档注解、校验注解、日期格式通过 config.md 获取 |
| Mapper | 继承 BaseMapper、防 SQL 注入、逻辑删除条件 |
| Service | 禁止互相注入、事务注解、日志规范通过 config.md 获取 |
| Controller | 返回统一响应类型、参数校验、API 文档注解 |

## 代码模板

### Entity 最小模板
```java
package {basePackage}.{serviceName}.entity;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
@TableName("{tablePrefix}_{tableName}")
public class {EntityName} extends BaseEntity {
    @TableId(type = IdType.AUTO)
    private Long id;
    // 根据设计文档添加字段
}
```

### Service 最小模板
```java
package {basePackage}.{serviceName}.service;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class {ServiceName} {
    // 根据设计文档实现业务逻辑
}
```

### Request DTO 最小模板
```java
package {basePackage}.{serviceName}.dto;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.NotBlank;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
@Schema(description = "{描述}")
public class {RequestDTO} {
    @NotBlank(message = "字段不能为空")
    @Schema(description = "字段描述", example = "示例值")
    private String fieldName;
}
```

### Response VO 最小模板
```java
package {basePackage}.{serviceName}.vo;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
@Schema(description = "{描述}")
public class {ResponseVO} {
    // 根据接口设计添加字段
}
```

### Controller 最小模板
```java
package {basePackage}.{serviceName}.controller;

import io.swagger.v3.oas.annotations.tags.Tag;
import io.swagger.v3.oas.annotations.Operation;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/{path}")
@RequiredArgsConstructor
@Tag(name = "{控制器名称}", description = "{描述}")
public class {ControllerName} {
    // 根据接口设计添加方法
}
```

### Mapper XML 最小模板
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="{basePackage}.{serviceName}.mapper.{MapperName}">
    <!-- 基础 CRUD 由 ORM 框架提供，复杂查询在此补充 -->
</mapper>
```

### DDL 最小模板
```sql
CREATE TABLE `{tablePrefix}_{tableName}` (
    `id` BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY COMMENT '主键ID',
    -- 根据设计文档添加字段
    `delete_flag` VARCHAR(1) NOT NULL DEFAULT 'N' COMMENT '逻辑删除',
    `create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `update_time` DATETIME COMMENT '更新时间'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='{表注释}';
```

### 单元测试最小模板
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
        // 正常路径测试
    }

    @Test
    void should_fail_when_{scenario}() {
        // 异常路径测试
    }
}
```

## 验证检查清单

- [ ] 包路径符合 `{basePackage}.{serviceName}.{layer}` 规范
- [ ] Entity 继承项目基础实体类，含逻辑删除字段
- [ ] Controller 返回项目标准响应类型，API 文档注解完整
- [ ] Service 不互相注入，通过 Repository 访问数据
- [ ] Mapper XML 含逻辑删除条件
- [ ] DDL 表名加项目前缀，含逻辑删除字段
- [ ] 日期字段类型与 JSON 序列化格式符合项目规范
- [ ] 单元测试覆盖正常路径 + 至少1个异常路径
- [ ] 生成范围与用户请求一致
- [ ] 所有 `// TODO` 项已在摘要中列出

## 异常场景处理

| 场景 | 处理策略 |
|------|---------|
| 设计文档字段类型缺失 | 使用最合理的默认类型推断，标注 `// TODO: 请确认字段类型` |
| 设计文档与现有代码冲突 | 提示冲突点，提供重命名建议 |
| 用户要求只生成部分成果物 | 确认生成范围，提醒依赖关系 |
| 设计文档缺少核心流程 | 生成基础 CRUD 骨架，标注 `// TODO: 请补充业务规则` |
| config.md 无法读取 | 使用通用默认值，在代码中标注 `// TODO: 请确认项目配置` |
