# 异常处理规范

---

## 一、统一异常体系

```java
// 业务异常基类
public class BusinessException extends RuntimeException {
    private final ErrorCode errorCode;

    public BusinessException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }

    public ErrorCode getErrorCode() {
        return errorCode;
    }
}

// 错误码枚举
public enum ErrorCode {
    USER_NOT_FOUND(10001, "用户不存在"),
    USERNAME_EXISTS(10002, "用户名已存在"),
    PASSWORD_INVALID(10003, "密码错误");

    private final int code;
    private final String message;
    // constructor, getter...
}
```

> 错误码的完整规划参见 [api-conventions.md](api-conventions.md#四业务错误码规划)。

---

## 二、全局异常处理器

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // 业务异常
    @ExceptionHandler(BusinessException.class)
    public Result<Void> handleBusiness(BusinessException e) {
        log.warn("业务异常: code={}, msg={}", e.getErrorCode().getCode(), e.getMessage());
        return Result.error(e.getErrorCode());
    }

    // 参数校验异常
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result<Void> handleValidation(MethodArgumentNotValidException e) {
        String msg = e.getBindingResult().getFieldErrors().stream()
            .map(err -> err.getField() + ": " + err.getDefaultMessage())
            .collect(Collectors.joining("; "));
        return Result.error(400, msg);
    }

    // 兜底
    @ExceptionHandler(Exception.class)
    public Result<Void> handleUnexpected(Exception e) {
        log.error("系统异常", e);
        return Result.error(500, "系统繁忙，请稍后重试");
    }
}
```

---

## 三、规则

- Controller 中**禁止**写 `try-catch`，统一由全局异常处理器处理
- 自定义异常继承 `RuntimeException`（非受检异常），避免代码被 `throws` 污染
- 异常信息对用户友好，不暴露技术细节（如 SQL 异常的堆栈）
- 捕获异常时必须记录完整堆栈：`log.error("操作失败", e);`
- 禁止"吞掉异常"：`catch (Exception e) { }`（空 catch 块）
