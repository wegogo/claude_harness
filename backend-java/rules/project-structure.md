# 项目结构与分层规范

---

## 一、项目结构

```
src/main/java/com/example/project/
├── controller/         # 控制器层：接收请求、参数校验、调用 Service、返回结果
├── service/            # 业务逻辑层
│   ├── impl/          # Service 实现
│   └── XxxService.java # Service 接口
├── mapper/             # 数据访问层（MyBatis-Plus Mapper）
├── model/
│   ├── entity/        # 数据库实体
│   ├── dto/           # 请求传输对象
│   ├── vo/            # 响应视图对象
│   └── query/         # 查询条件对象
├── config/             # 配置类
├── exception/          # 自定义异常 + 全局异常处理器
├── common/             # 公共工具、常量、枚举
│   ├── enums/
│   ├── constant/
│   └── util/
├── interceptor/        # 拦截器
└── Application.java    # 启动类
```

**分层依赖方向**：Controller → Service → Mapper（MyBatis-Plus），禁止跨层调用或反向依赖。

---

## 二、Controller 层

```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping("/{id}")
    public Result<UserVO> getById(@PathVariable Long id) {
        return Result.success(userService.getById(id));
    }

    @PostMapping
    public Result<UserVO> create(@Valid @RequestBody UserCreateDTO dto) {
        return Result.success(userService.create(dto));
    }
}
```

**规则：**
- Controller 只做：参数接收 → 校验 → 调 Service → 返回结果
- **禁止**在 Controller 中写业务逻辑
- **禁止**在 Controller 中直接操作数据库
- 使用 `@Valid` 进行参数校验，校验规则定义在 DTO 中
- 统一返回 `Result<T>` 结构

---

## 三、Service 层

> 本项目使用 MyBatis-Plus，Service 接口继承 `IService<T>`，实现类继承 `ServiceImpl<Mapper, T>`。

```java
public interface UserService extends IService<User> {
    UserVO getById(Long id);
    UserVO create(UserCreateDTO dto);
}

@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {

    @Override
    @Transactional(readOnly = true)
    public UserVO getById(Long id) {
        // this.getById() 来自 ServiceImpl 内置方法
        User user = super.getById(id);
        if (user == null) {
            throw new BusinessException(ErrorCode.USER_NOT_FOUND);
        }
        return UserConverter.toVO(user);
    }

    @Override
    @Transactional
    public UserVO create(UserCreateDTO dto) {
        // 1. 校验
        long count = this.lambdaQuery()
            .eq(User::getUsername, dto.getUsername())
            .count();
        if (count > 0) {
            throw new BusinessException(ErrorCode.USERNAME_EXISTS);
        }
        // 2. 转换 + 持久化
        User user = UserConverter.toEntity(dto);
        this.save(user);
        // 3. 返回
        return UserConverter.toVO(user);
    }
}
```

**规则：**
- Service 接口继承 `IService<T>`，实现类继承 `ServiceImpl<Mapper, T>`，获得内置 CRUD
- `@Transactional` 加在实现类方法上，不加在接口上
- 查询方法加 `@Transactional(readOnly = true)`
- 业务校验放在 Service 层，不依赖 Controller 层的 `@Valid`
- 查询结果为 null 时显式抛业务异常，不返回 null

---

## 四、Model 层

### Entity

> 本项目使用 MyBatis-Plus，Entity 用 `@TableName`、`@TableId`、`@TableField` 注解。

```java
@TableName("t_user")
public class User {

    @TableId(type = IdType.AUTO)
    private Long id;

    private String username;

    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createdAt;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updatedAt;

    @TableLogic
    private Integer deleted;
}
```

### DTO（带校验）

```java
public record UserCreateDTO(
    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 20, message = "用户名长度 3-20 字符")
    String username,

    @NotBlank(message = "邮箱不能为空")
    @Email(message = "邮箱格式错误")
    String email,

    @NotBlank(message = "密码不能为空")
    @Size(min = 8, message = "密码至少 8 位")
    String password
) {}
```

> 推荐使用 Java 14+ 的 `record` 定义 DTO/VO/Query 等纯数据类。
