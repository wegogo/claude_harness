# API 接口规范

> 适用于 Java 21 + Spring Boot 3.5 项目的 RESTful API 设计与开发。
> 所有对外暴露的 HTTP 接口均须遵循本规范。

---

## 一、基础约定

### 1.1 URL 根路径

```
/api/v1/<模块>/<资源>
```

| 组成 | 规则 | 示例 |
|------|------|------|
| `/api` | 固定前缀，标识 API 入口 | `/api` |
| `/v1` | 版本号，从 v1 开始 | `/api/v1`、`/api/v2` |
| `<模块>` | 业务模块名，小写单数 | `/api/v1/user`、`/api/v1/order` |
| `<资源>` | 资源名，小写复数名词 | `/api/v1/users`、`/api/v1/orders` |

### 1.2 URL 设计原则

| 原则 | 正例 | 反例 |
|------|------|------|
| 用名词，不用动词 | `GET /users` | `GET /getUsers` |
| 用复数表示资源集合 | `/users` | `/user` |
| 用层级表达从属关系 | `/users/{id}/orders` | `/getUserOrders?userId=` |
| 层级不超过 3 层 | `/users/{id}/orders/{orderId}` | `/users/{id}/orders/{orderId}/items/{itemId}/tags` |
| 全小写，连字符分词 | `/user-profiles` | `/userProfiles`、`/user_profiles` |
| 路径参数用 ID，不用名称 | `/users/{id}` | `/users/zhangsan` |

### 1.3 HTTP 方法语义

| 方法 | 语义 | 安全 | 幂等 | 示例 |
|------|------|------|------|------|
| `GET` | 查询资源 | 是 | 是 | `GET /users/{id}` |
| `POST` | 创建资源 | 否 | 否 | `POST /users` |
| `PUT` | 全量更新资源 | 否 | 是 | `PUT /users/{id}` |
| `PATCH` | 部分更新资源 | 否 | 是 | `PATCH /users/{id}` |
| `DELETE` | 删除资源 | 否 | 是 | `DELETE /users/{id}` |

> **安全**：不改变服务端状态。**幂等**：多次请求结果一致。

### 1.4 资源操作映射

```
GET    /api/v1/users          # 查询用户列表（分页）
GET    /api/v1/users/{id}     # 查询单个用户
POST   /api/v1/users          # 创建用户
PUT    /api/v1/users/{id}     # 全量更新用户
PATCH  /api/v1/users/{id}     # 部分更新用户（如修改状态）
DELETE /api/v1/users/{id}     # 删除用户
```

**非 CRUD 操作的处理：**

当操作无法用标准 CRUD 表达时，使用子动作路径：

```
POST /api/v1/users/{id}/lock       # 锁定用户
POST /api/v1/users/{id}/unlock     # 解锁用户
POST /api/v1/orders/{id}/cancel    # 取消订单
POST /api/v1/orders/{id}/pay       # 支付订单
```

规则：`POST /<资源集合>/{id}/<动作>`，动作为小写动词。

---

## 二、请求规范

### 2.1 请求头

| 头部 | 是否必须 | 说明 |
|------|---------|------|
| `Content-Type` | POST/PUT/PATCH 必须 | 固定 `application/json; charset=UTF-8` |
| `Authorization` | 视接口而定 | `Bearer <token>` |
| `Accept-Language` | 可选 | 国际化语言，如 `zh-CN`、`en-US` |
| `X-Request-Id` | 可选 | 请求追踪 ID，未传则服务端生成 |

### 2.2 查询参数

**分页参数：**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `page` | int | 1 | 页码，从 1 开始 |
| `size` | int | 20 | 每页条数，最大 100 |
| `sort` | string | - | 排序，格式 `字段,方向`，如 `createdAt,desc` |

```
GET /api/v1/users?page=1&size=20&sort=createdAt,desc
```

**过滤参数：**

直接用字段名作为查询参数，不加额外前缀：

```
GET /api/v1/users?status=ACTIVE&role=ADMIN&createdAtAfter=2025-01-01
```

**禁止用法：**

```
# 错误：把过滤条件塞进一个参数
GET /api/v1/users?filter=status:ACTIVE,role:ADMIN
```

### 2.3 请求体

- 统一使用 JSON 格式
- 字段名使用 **camelCase**（与 Java 变量名一致）
- 时间字段使用 ISO 8601 格式：`2025-06-14T10:30:00Z`
- 布尔值用 `true` / `false`，不用 `0` / `1`
- 空值传 `null`，不省略字段（保持结构一致性）

```json
{
  "username": "zhangsan",
  "email": "zhangsan@example.com",
  "isActive": true,
  "birthday": "1990-01-15",
  "roles": ["USER", "ADMIN"]
}
```

### 2.4 参数校验

使用 Jakarta Validation 注解在 DTO 层校验：

```java
public record UserCreateDTO(
    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 20, message = "用户名长度 3-20 字符")
    String username,

    @NotBlank(message = "邮箱不能为空")
    @Email(message = "邮箱格式错误")
    String email,

    @NotBlank(message = "密码不能为空")
    @Size(min = 8, max = 64, message = "密码长度 8-64 字符")
    @Pattern(regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d).+$",
             message = "密码需包含大小写字母和数字")
    String password,

    @NotNull(message = "角色不能为空")
    @Size(min = 1, message = "至少分配一个角色")
    List<@NotBlank String> roles
) {}
```

Controller 中加 `@Valid` 触发校验：

```java
@PostMapping
public Result<UserVO> create(@Valid @RequestBody UserCreateDTO dto) {
    return Result.success(userService.create(dto));
}
```

---

## 三、响应规范

### 3.1 统一响应结构

所有接口返回统一信封格式：

```java
public record Result<T>(
    int code,        // 业务状态码，0 表示成功，非 0 表示业务错误
    String message,  // 提示信息
    T data,          // 业务数据，失败时为 null
    String traceId   // 链路追踪 ID（可选，便于排查问题）
) {
    public static <T> Result<T> success(T data) {
        return new Result<>(0, "success", data, null);
    }

    public static <T> Result<T> error(ErrorCode errorCode) {
        return new Result<>(errorCode.getCode(), errorCode.getMessage(), null, null);
    }

    public static <T> Result<T> error(int code, String message) {
        return new Result<>(code, message, null, null);
    }
}
```

**成功响应：**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "id": 1,
    "username": "zhangsan"
  }
}
```

**失败响应：**

```json
{
  "code": 10001,
  "message": "用户不存在",
  "data": null
}
```

### 3.2 分页响应

```java
public record PageResult<T>(
    List<T> list,       // 当前页数据
    long total,         // 总记录数
    long page,          // 当前页码
    long size,          // 每页条数
    long totalPages     // 总页数
) {
    public static <T> PageResult<T> of(Page<T> page) {
        return new PageResult<>(
            page.getRecords(),
            page.getTotal(),
            page.getCurrent(),
            page.getSize(),
            page.getPages()
        );
    }
}
```

**分页响应示例：**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [
      {"id": 1, "username": "zhangsan"},
      {"id": 2, "username": "lisi"}
    ],
    "total": 156,
    "page": 1,
    "size": 20,
    "totalPages": 8
  }
}
```

### 3.3 HTTP 状态码使用

本项目 API 业务状态码放在响应体 `code` 字段中，HTTP 状态码用于表达协议层状态：

| HTTP 状态码 | 场景 | 说明 |
|------------|------|------|
| `200 OK` | 请求成功 | GET / POST / PUT / PATCH 成功 |
| `201 Created` | 资源创建成功 | POST 创建资源（可选，也可统一返回 200） |
| `204 No Content` | 无返回体 | DELETE 成功 |
| `400 Bad Request` | 参数错误 | 参数缺失、格式错误、校验失败 |
| `401 Unauthorized` | 未认证 | 未携带或携带了无效的 Token |
| `403 Forbidden` | 无权限 | 认证通过但无访问权限 |
| `404 Not Found` | 资源不存在 | 请求的 URL 或资源不存在 |
| `409 Conflict` | 冲突 | 唯一约束冲突、状态冲突 |
| `429 Too Many Requests` | 限流 | 请求频率超限 |
| `500 Internal Server Error` | 服务端错误 | 未预期的异常 |

> **约定**：本项目业务错误（如"用户不存在"）统一返回 HTTP 200 + 业务错误码，
> 只有协议层错误（参数格式错、认证失败等）才使用对应的 HTTP 错误码。

### 3.4 业务错误码规划

错误码为 5 位整数，按模块分段：

| 范围 | 模块 | 示例 |
|------|------|------|
| `10000 - 19999` | 用户模块 | `10001` 用户不存在 |
| `20000 - 29999` | 订单模块 | `20001` 订单不存在 |
| `30000 - 39999` | 支付模块 | `30001` 支付超时 |
| `40000 - 49999` | 商品模块 | `40001` 商品下架 |
| `90000 - 99999` | 系统通用 | `90001` 系统繁忙 |

```java
public enum ErrorCode {
    // 用户模块 10000-19999
    USER_NOT_FOUND(10001, "用户不存在"),
    USERNAME_EXISTS(10002, "用户名已存在"),
    PASSWORD_INVALID(10003, "密码错误"),
    USER_DISABLED(10004, "用户已被禁用"),

    // 订单模块 20000-29999
    ORDER_NOT_FOUND(20001, "订单不存在"),
    ORDER_ALREADY_PAID(20002, "订单已支付"),
    ORDER_CANNOT_CANCEL(20003, "订单当前状态不可取消"),

    // 系统通用 90000-99999
    SYSTEM_BUSY(90001, "系统繁忙，请稍后重试"),
    RATE_LIMITED(90002, "请求过于频繁，请稍后再试");

    private final int code;
    private final String message;
    // constructor, getter...
}
```

**错误码规则：**
- 全局唯一，不可复用
- 已废弃的错误码不回收，标记为 `@Deprecated`
- 新增模块按段分配区间，在本文档登记

---

## 四、接口版本管理

### 4.1 版本策略

采用 **URL 路径版本**，简单直观：

```
/api/v1/users    # 第一版
/api/v2/users    # 第二版（不兼容变更时）
```

### 4.2 兼容性规则

| 变更类型 | 是否需要升版本 | 说明 |
|---------|--------------|------|
| 新增字段（响应） | 否 | 向后兼容 |
| 新增可选字段（请求） | 否 | 向后兼容 |
| 删除字段 | **是** | 破坏性变更 |
| 修改字段类型 | **是** | 破坏性变更 |
| 修改字段语义 | **是** | 破坏性变更 |
| 新增接口 | 否 | 向后兼容 |

### 4.3 废弃流程

1. 旧版本接口标记 `@Deprecated`，Swagger 文档标注废弃说明
2. 响应头添加 `Sunset` 和 `Deprecation` 提示
3. 通知所有调用方迁移
4. 观察 2 个版本周期后下线

```java
@Deprecated
@GetMapping("/v1/users/{id}")
@Operation(deprecated = true, description = "使用 v2 接口，v1 将于 2025-12-31 下线")
public Result<UserVO> getByIdV1(@PathVariable Long id) { }
```

---

## 五、数据模型约定

### 5.1 DTO / VO 职责划分

| 类型 | 全称 | 方向 | 职责 |
|------|------|------|------|
| `DTO` | Data Transfer Object | 请求入参 | 接收前端请求，携带校验注解 |
| `VO` | View Object | 响应出参 | 返回给前端的数据视图 |
| `Query` | Query Object | 请求入参 | 查询条件封装（非 POST body 场景） |

```
请求 → DTO → Service → Entity → VO → 响应
```

**规则：**
- Entity 禁止直接返回给前端
- 禁止用同一个类同时做请求和响应（除非极简单的场景）
- DTO 与 VO 之间的转换放在 Converter 类中

```java
public class UserConverter {
    public static User toEntity(UserCreateDTO dto) {
        return new User(dto.username(), dto.email(), encodePassword(dto.password()));
    }

    public static UserVO toVO(User user) {
        return new UserVO(user.getId(), user.getUsername(), user.getEmail());
    }
}
```

### 5.2 字段命名

- JSON 字段统一 **camelCase**：`userName`、`createdAt`
- 时间字段统一后缀：
  - `xxxAt`：时间点（`createdAt`、`updatedAt`、`expiredAt`）
  - `xxxDate`：日期（`birthday`、`hireDate`）
- 布尔字段避免 `is` 前缀（JSON 序列化兼容性差），用形容词或过去分词：
  - `active`（不用 `isActive`）
  - `enabled`、`verified`、`deleted`

### 5.3 时间格式

| 类型 | 格式 | 示例 |
|------|------|------|
| 日期 | `yyyy-MM-dd` | `2025-06-14` |
| 日期时间 | ISO 8601 | `2025-06-14T10:30:00Z` |
| 时间戳 | 毫秒级 Unix 时间戳 | `1718350200000` |

> 项目内统一选一种日期时间格式，推荐 ISO 8601。

---

## 六、认证与授权

### 6.1 认证方式

使用 Bearer Token（JWT）：

```
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

### 6.2 接口分类

| 类型 | 说明 | 认证 |
|------|------|------|
| 公开接口 | 登录、注册、健康检查 | 不需要 |
| 认证接口 | 绝大多数业务接口 | 需要 Token |
| 内部接口 | 服务间调用 | IP 白名单 / 内部 Token |

白名单显式配置：

```java
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/v1/auth/login", "/api/v1/auth/register").permitAll()
            .requestMatchers("/actuator/**").permitAll()
            .anyRequest().authenticated()
        );
        return http.build();
    }
}
```

### 6.3 数据权限

接口不仅要验证身份，还要验证**是否有权操作该数据**：

```java
@GetMapping("/{id}")
public Result<UserVO> getById(@PathVariable Long id, @AuthenticationPrincipal Principal principal) {
    // 当前用户只能查自己的信息（除非是管理员）
    if (!principal.getId().equals(id) && !principal.hasRole("ADMIN")) {
        throw new BusinessException(ErrorCode.FORBIDDEN);
    }
    return Result.success(userService.getById(id));
}
```

---

## 七、接口文档（Swagger / OpenAPI）

### 7.1 注解要求

所有接口必须有 Swagger 注解：

```java
@RestController
@RequestMapping("/api/v1/users")
@Tag(name = "用户管理", description = "用户的增删改查及权限管理")
@RequiredArgsConstructor
public class UserController {

    @Operation(summary = "查询用户详情", description = "根据用户 ID 查询用户信息")
    @Parameters({
        @Parameter(name = "id", description = "用户 ID", required = true, example = "1")
    })
    @GetMapping("/{id}")
    public Result<UserVO> getById(@PathVariable Long id) { }

    @Operation(summary = "创建用户")
    @PostMapping
    public Result<UserVO> create(@Valid @RequestBody UserCreateDTO dto) { }
}
```

```java
@Schema(description = "用户创建请求")
public record UserCreateDTO(
    @Schema(description = "用户名", example = "zhangsan", requiredMode = REQUIRED)
    @NotBlank String username,

    @Schema(description = "邮箱", example = "zhangsan@test.com", requiredMode = REQUIRED)
    @NotBlank @Email String email
) {}
```

### 7.2 文档访问

```
# Swagger UI
http://localhost:8080/swagger-ui.html

# OpenAPI JSON
http://localhost:8080/v3/api-docs
```

---

## 八、接口安全

### 8.1 限流

核心接口配置限流策略：

| 接口类型 | 限流策略 |
|---------|---------|
| 登录 | 同 IP 每分钟 5 次 |
| 短信验证码 | 同手机号每小时 5 次，每天 10 次 |
| 查询接口 | 同用户每秒 10 次 |
| 写操作 | 同用户每秒 5 次 |

```java
@RateLimiter(name = "loginRateLimiter")
@PostMapping("/auth/login")
public Result<TokenVO> login(@Valid @RequestBody LoginDTO dto) { }
```

### 8.2 幂等性

写操作（支付、下单）需保证幂等，客户端传递幂等 Key：

```
POST /api/v1/orders
Header: Idempotency-Key: <uuid>
```

服务端缓存 2 小时，相同 Key 返回首次结果：

```java
@Idempotent(expireSeconds = 7200, headerName = "Idempotency-Key")
@PostMapping("/orders")
public Result<OrderVO> createOrder(@Valid @RequestBody OrderCreateDTO dto) { }
```

### 8.3 敏感数据脱敏

响应中对敏感字段脱敏：

```java
public record UserVO(
    Long id,
    String username,

    @JsonSerialize(using = PhoneMaskSerializer.class)
    String phone,          // 138****8888

    @JsonSerialize(using = IdCardMaskSerializer.class)
    String idCard,         // 110101********1234

    String email
) {}
```

---

## 九、接口清单模板

新增模块时，先定义接口清单，再写代码：

| 接口 | 方法 | 路径 | 认证 | 描述 |
|------|------|------|------|------|
| 用户列表 | GET | `/api/v1/users` | 是 | 分页查询用户列表 |
| 用户详情 | GET | `/api/v1/users/{id}` | 是 | 查询单个用户 |
| 创建用户 | POST | `/api/v1/users` | 是 | 创建新用户 |
| 更新用户 | PUT | `/api/v1/users/{id}` | 是 | 全量更新 |
| 修改状态 | PATCH | `/api/v1/users/{id}/status` | 是 | 修改用户状态 |
| 删除用户 | DELETE | `/api/v1/users/{id}` | 是 | 删除用户 |

---

## 十、附：完整接口示例

### 10.1 Controller

```java
@RestController
@RequestMapping("/api/v1/users")
@Tag(name = "用户管理", description = "用户的增删改查及权限管理")
@RequiredArgsConstructor
@Validated
public class UserController {

    private final UserService userService;

    @Operation(summary = "分页查询用户列表")
    @GetMapping
    public Result<PageResult<UserVO>> list(
            @RequestParam(defaultValue = "1") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(required = false) String status,
            @RequestParam(required = false) String keyword) {
        UserQuery query = new UserQuery(page, size, status, keyword);
        return Result.success(userService.list(query));
    }

    @Operation(summary = "查询用户详情")
    @GetMapping("/{id}")
    public Result<UserVO> getById(@PathVariable Long id) {
        return Result.success(userService.getById(id));
    }

    @Operation(summary = "创建用户")
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Result<UserVO> create(@Valid @RequestBody UserCreateDTO dto) {
        return Result.success(userService.create(dto));
    }

    @Operation(summary = "全量更新用户")
    @PutMapping("/{id}")
    public Result<UserVO> update(@PathVariable Long id,
                                  @Valid @RequestBody UserUpdateDTO dto) {
        return Result.success(userService.update(id, dto));
    }

    @Operation(summary = "修改用户状态")
    @PatchMapping("/{id}/status")
    public Result<Void> updateStatus(@PathVariable Long id,
                                      @Valid @RequestBody UserStatusDTO dto) {
        userService.updateStatus(id, dto);
        return Result.success(null);
    }

    @Operation(summary = "删除用户")
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

### 10.2 请求与响应

**创建用户请求：**

```
POST /api/v1/users
Content-Type: application/json

{
  "username": "zhangsan",
  "email": "zhangsan@example.com",
  "password": "Secret123",
  "roles": ["USER"]
}
```

**创建用户响应（201）：**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "id": 1,
    "username": "zhangsan",
    "email": "zhangsan@example.com",
    "active": true,
    "createdAt": "2025-06-14T10:30:00Z"
  }
}
```

**参数校验失败（400）：**

```json
{
  "code": 400,
  "message": "username: 用户名不能为空; email: 邮箱格式错误",
  "data": null
}
```
