# 编码规范（示例）

## 实体类规范

- 必须继承项目基础实体类 `BaseEntity`
- 禁用 `@Data`，使用 `@Getter` + `@Setter`
- 日期字段使用 `LocalDateTime`
- 枚举使用 `String` 存储，配合 `@EnumValue`

## 依赖注入

- 使用 `@Resource` 或构造函数注入
- 禁止 Service 互相注入
- Service 通过 Repository/Mapper 访问数据

## 日志规范

- 使用 `SLF4J` + `Logback`
- 禁止 `System.out.println`
- 日志级别：DEBUG（开发）、INFO（正常）、WARN（异常）、ERROR（错误）

## Controller 规范

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    @Resource
    private UserService userService;

    @GetMapping("/{id}")
    public Result<UserVO> getById(@PathVariable Long id) {
        return Result.success(userService.getById(id));
    }
}
```

## 异常处理

- 全局异常拦截器统一处理
- 业务异常使用自定义 `BusinessException`
- 异常信息不包含敏感数据
