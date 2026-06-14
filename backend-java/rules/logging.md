# 日志规范

---

## 一、日志声明

```java
private static final Logger log = LoggerFactory.getLogger(UserService.class);
// 或使用 Lombok: @Slf4j
```

---

## 二、日志级别

| 日志级别 | 使用场景 |
|---------|---------|
| ERROR | 影响业务功能的异常、系统级错误 |
| WARN | 可恢复的异常、业务降级、需关注但不致命 |
| INFO | 关键业务操作、系统启动/关闭、配置变更 |
| DEBUG | 开发调试信息，生产关闭 |

---

## 三、规则

- 使用占位符 `{}`，不要用字符串拼接：

```java
// 正确
log.info("用户登录: userId={}, ip={}", userId, ip);

// 错误
log.info("用户登录: userId=" + userId + ", ip=" + ip);
```

- 捕获异常时记录完整堆栈：`log.error("操作失败", e);`
- 敏感信息（密码、Token、身份证、银行卡）禁止打印到日志
- 生产环境禁止使用 `System.out.println()`
- 避免在循环中打高频日志（DEBUG 级除外）
