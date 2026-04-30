# 项目全景配置（示例）

> 本文件为示例，展示项目专属规范的标准结构。实际使用时请替换为真实项目信息。

## 项目基本信息

| 属性 | 示例值 |
|------|--------|
| 项目名称 | `{project-name}` |
| 技术栈 | Java、Spring Boot、MySQL、Redis |
| 构建工具 | Maven / Gradle |
| 包路径 | `com.example.{project-name}` |

## 微服务列表

| 服务名 | 职责 | 端口示例 |
|--------|------|---------|
| `{service-a}` | 用户/认证服务 | 8081 |
| `{service-b}` | 核心业务服务 | 8082 |
| `{service-c}` | 基础设施服务 | 8083 |

## 统一响应格式

```java
public class Result<T> {
    private int code;
    private String message;
    private T data;
    // getter/setter
}
```

## 错误码规范

| 错误码 | 含义 | 使用场景 |
|--------|------|---------|
| 200 | 成功 | 业务处理成功 |
| 400 | 参数错误 | 请求参数校验失败 |
| 401 | 未授权 | 鉴权失败 |
| 500 | 系统错误 | 内部异常 |

## 架构约束

- 服务间通信：同步调用（HTTP/RPC）或异步消息（MQ）
- 数据库：MySQL，表名加项目前缀
- 缓存：Redis
- ORM框架：MyBatis-Plus / JPA（通过本规范定义）
