# API 设计规范（示例）

## URL 规范

| 规则 | 示例 |
|------|------|
| 前缀 | `/api/v1` |
| 资源名 | 复数名词，如 `/users`、`/orders` |
| 操作 | HTTP Method 表达动作，如 GET /users/{id} |

## 接口设计示例

### 查询用户列表

```java
@Tag(name = "用户管理", description = "用户相关接口")
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    @Operation(summary = "查询用户列表", description = "支持分页和条件筛选")
    @ApiResponse(responseCode = "200", description = "查询成功")
    @GetMapping
    public Result<PageResult<UserVO>> listUsers(@Valid UserQueryDTO query) {
        // ...
    }
}
```

## DTO/VO 规范

- Request DTO：类名以 `Request` 或 `DTO` 结尾
- Response VO：类名以 `VO` 或 `Response` 结尾
- 日期格式：`yyyy-MM-dd HH:mm:ss`
- 枚举字段：使用 `String` 类型，标注可选值

## 参数校验

| 规则 | 注解 | 示例 |
|------|------|------|
| 必填 | `@NotNull` / `@NotBlank` | `@NotBlank private String name;` |
| 长度 | `@Size` | `@Size(max=50) private String title;` |
| 范围 | `@Min` / `@Max` | `@Min(1) private int pageNum;` |
