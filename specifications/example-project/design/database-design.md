# 数据库设计规范（示例）

## 表设计规范

| 规则 | 示例 |
|------|------|
| 表名前缀 | `{project}_` 如 `example_user` |
| 逻辑删除 | 使用 `delete_flag` 字段（0=未删除，1=已删除） |
| 主键 | 自增 bigint 或雪花 ID |
| 创建/更新时间 | `create_time`、`update_time` |

## 索引规范

- 主键自动创建聚簇索引
- 外键/关联字段创建普通索引
- 联合查询字段考虑复合索引
- 索引命名：`idx_{表名}_{字段名}`

## 实体类示例

```java
@Data
public class User extends BaseEntity {
    private Long id;
    private String username;
    private String email;
    private Integer status;
    private Integer deleteFlag;
}
```

## SQL 规范

- 禁止 `SELECT *`，必须指定字段
- 禁止在 WHERE 条件中对字段使用函数
- 大表分页使用游标或延迟关联
- 复杂查询优先使用 JOIN 而非子查询

## ORM 框架规范

- 基础 CRUD 使用框架提供的默认方法
- 复杂查询手写 XML/SQL
- 批量操作使用 `saveBatch` 或 `insertBatch`
