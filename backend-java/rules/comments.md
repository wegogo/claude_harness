# 注释规范

---

## 一、类注释

```java
/**
 * 用户服务。负责用户的增删改查及权限管理。
 *
 * @author zhangsan
 * @since 1.0.0
 */
@Service
public class UserService { }
```

---

## 二、方法注释

公共方法使用 Javadoc：

```java
/**
 * 根据用户名查询用户。
 *
 * @param username 用户名，不可为空
 * @return 用户信息，不存在返回 Optional.empty()
 * @throws BusinessException 当 username 为空时抛出
 */
public Optional<User> findByUsername(String username) { }
```

---

## 三、注释原则

- 代码应自解释，好的命名优于注释
- 注释解释 **为什么**，不是 **是什么**
- 变更代码时同步更新注释
- 删除被注释掉的代码块，用 Git 管理历史
- 禁止 `TODO` 无对应 Issue 编号
