# 命名规范

> 基于《阿里巴巴 Java 开发手册》，针对本项目补充细化。

---

## 一、通用规则

| 场景 | 规则 | 正例 | 反例 |
|------|------|------|------|
| 类名 | UpperCamelCase | `UserService` | `userService` |
| 方法/变量 | lowerCamelCase | `findByName` | `FindByName` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` | `maxRetryCount` |
| 包名 | 全小写，单数 | `com.example.user` | `com.example.users` |
| 枚举 | UpperCamelCase 类名 + 全大写值 | `OrderStatus.PENDING` | - |

---

## 二、各层命名约定

| 层 | 后缀 | 示例 |
|----|------|------|
| Controller | `Controller` | `UserController` |
| Service 接口 | `Service` | `UserService` |
| Service 实现 | `ServiceImpl` | `UserServiceImpl` |
| Mapper | `Mapper` | `UserMapper` |
| 数据传输对象 | `DTO` | `UserCreateDTO` |
| 视图对象 | `VO` | `UserVO` |
| 实体类 | `Entity`（或不加后缀） | `UserEntity` / `User` |
| 查询对象 | `Query` | `UserQuery` |

---

## 三、方法命名

| 操作 | 前缀 | 示例 |
|------|------|------|
| 查询单条 | `get` / `find` | `getUserById` |
| 查询列表 | `list` / `find` | `listActiveUsers` |
| 查询数量 | `count` | `countByStatus` |
| 新增 | `create` / `save` / `add` | `createUser` |
| 修改 | `update` | `updateUserProfile` |
| 删除 | `delete` / `remove` | `deleteById` |
| 判断 | `is` / `has` / `can`（返回 boolean） | `isAdmin` |
