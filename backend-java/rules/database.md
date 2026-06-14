# 数据库规范

> 覆盖 MySQL 8、MongoDB 7.0、Redis 6.0 三个数据存储的使用规范。

---

## 一、MySQL（主数据存储）

### 1.1 命名

- 表名：`t_` 前缀 + 下划线，如 `t_user`、`t_order_detail`
- 字段名：下划线，如 `created_at`、`user_name`
- 索引名：`idx_字段名`（普通索引），`uk_字段名`（唯一索引）

### 1.2 必备字段

```sql
CREATE TABLE t_user (
    id          BIGINT       NOT NULL AUTO_INCREMENT COMMENT '主键',
    -- 业务字段...
    created_at  DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at  DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted     TINYINT(1)   NOT NULL DEFAULT 0 COMMENT '逻辑删除标记',
    PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

### 1.3 SQL 规则

- 禁止 `SELECT *`，显式列出字段名
- 禁止在代码中拼接 SQL，使用 MyBatis-Plus 条件构造器或 XML 参数化查询
- 大表查询必须走索引，`EXPLAIN` 验证执行计划
- 避免 `LIKE '%keyword%'` 前模糊，无法走索引
- `IN` 子句不超过 1000 条
- 表变更通过 Flyway 脚本管理，禁止手动 DDL

### 1.4 MyBatis-Plus 使用规范

**Mapper 层约定：**

- 所有 Mapper 继承 `BaseMapper<T>`，复用内置 CRUD 方法
- Mapper 接口命名统一为 `XxxMapper`（不用 `Repository` 后缀）
- 自定义查询优先用 `@Select` 注解；复杂 SQL 写在 XML 中
- XML 文件与 Mapper 接口同名，放在 `resources/mapper/` 下

```java
public interface UserMapper extends BaseMapper<User> {

    // 自定义查询：简单 SQL 用注解
    @Select("SELECT * FROM t_user WHERE status = #{status} AND deleted = 0")
    List<User> selectByStatus(@Param("status") Integer status);

    // 复杂查询走 XML
    List<UserVO> selectUserList(@Param("query") UserQuery query);
}
```

**Entity 约定：**

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

**条件构造器（LambdaQueryWrapper）：**

```java
// 优先使用 Lambda 形式，编译期检查字段名
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
    .eq(User::getStatus, UserStatus.ACTIVE.getCode())
    .like(StringUtils.isNotBlank(keyword), User::getUsername, keyword)
    .orderByDesc(User::getCreatedAt)
    .last("LIMIT 100");

List<User> users = userMapper.selectList(wrapper);
```

**Service 层继承约定：**

```java
// Service 接口继承 IService
public interface UserService extends IService<User> {
    UserVO getById(Long id);
}

// Service 实现继承 ServiceImpl，获得内置 CRUD
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    // 可直接调用 this.getById()、this.save()、this.page() 等内置方法
}
```

**分页：**

```java
// 使用 MyBatis-Plus 分页插件，不要手写 LIMIT
Page<User> page = new Page<>(currentPage, pageSize);
Page<User> result = userMapper.selectPage(page, wrapper);
```

**MyBatis-Plus 使用红线：**

| 禁止 | 应该 |
|------|------|
| 手写 `SELECT *` | 用 `selectList(wrapper)` 或显式指定字段 |
| 字符串拼接字段名 `wrapper.eq("username", ...)` | 用 Lambda 形式 `wrapper.eq(User::getUsername, ...)` |
| 在 Service 中直接拼 SQL | 用条件构造器或 Mapper XML |
| 忽略逻辑删除配置 | Entity 加 `@TableLogic`，全局配置 `logic-delete-field` |
| 不加分页直接全表查 | 用 `selectPage()` + 分页插件 |

---

## 二、MongoDB（文档型数据）

### 2.1 适用场景

- 日志、审计记录、埋点数据
- 非结构化/半结构化数据
- 需要灵活 Schema 的数据

### 2.2 规则

- 集合名：小写复数，如 `operationLogs`、`userActions`
- 文档必须设置 `@Indexed` 字段，避免全表扫描
- 大文档（> 16MB）拆分或使用 GridFS
- 使用 `@Document` 注解映射，字段命名同 Java 驼峰或配置转换

```java
@Document(collection = "operationLogs")
public class OperationLog {
    @Id
    private String id;
    private Long userId;
    private String action;
    private Map<String, Object> detail;
    @Indexed
    private LocalDateTime createdAt;
}
```

---

## 三、Redis（缓存与会话）

### 3.1 规则

- **必须设置过期时间**（`expire`），禁止永久缓存
- Key 命名：`项目名:模块:业务标识:唯一键`，如 `app:user:token:abc123`
- 序列化使用 JSON（`GenericJackson2JsonRedisSerializer`），不使用 JDK 序列化
- 防止缓存穿透：空值也缓存，设较短 TTL
- 防止缓存击穿：热点数据使用分布式锁或 `@Cacheable(sync = true)`
- 防止缓存雪崩：TTL 加随机偏移（如基础 TTL + 随机 60s）

```java
@Cacheable(value = "user", key = "#id", unless = "#result == null")
public UserVO getById(Long id) {
    // ...
}
```
