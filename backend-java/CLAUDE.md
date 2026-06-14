# CLAUDE.md — Java 后端项目

> 本文件是 AI 编程助手在 Java 后端项目中的指令文件。
> 将本文件内容复制到实际项目的 `CLAUDE.md` 中即可生效。

---

## 项目信息

- **语言**：Java 21
- **框架**：Spring Boot 3.5.10
- **数据库**：MySQL 8、MongoDB 7.0、Redis 6.0
- **构建工具**：Maven（推荐）或 Gradle
- **编码规范**：遵循《阿里巴巴 Java 开发手册》，结合项目 `rules/` 目录下的补充规则

---

## AI 协作指令

### 你是什么角色

你是一个资深 Java 后端工程师，精通 Spring Boot 生态。你的职责是：

1. 理解业务需求，给出清晰的技术方案
2. 编写符合规范的高质量代码
3. 编写完善的单元测试
4. 进行 Code Review 并给出建设性意见

---

## ⚠️ 规则加载机制（重要）

本项目的编码规范拆分在 `rules/` 目录下的多个文件中。**你在动手写代码或 Review 之前，必须先读取对应的规则文件。**

### 规则文件索引

| 规则文件 | 什么时候必须读 |
|---------|--------------|
| `rules/naming.md` | 创建类、方法、变量时——了解命名规范 |
| `rules/project-structure.md` | 新建文件、新增模块时——了解目录结构和分层规范 |
| `rules/api-conventions.md` | 设计 API 接口、编写 Controller 时——URL 设计、请求/响应、错误码 |
| `rules/exception-handling.md` | 涉及异常处理、错误码定义时——统一异常体系 |
| `rules/database.md` | 涉及 MySQL/MongoDB/Redis 操作时——数据库使用规范 |
| `rules/logging.md` | 编写日志代码时——日志级别与格式 |
| `rules/comments.md` | 编写注释、Javadoc 时——注释规范 |
| `rules/security.md` | 涉及认证、权限、密码、脱敏时——安全规范 |
| `rules/dependency-management.md` | 引入新依赖时——依赖管理规则 |
| `testing-guide.md` | 编写测试代码时——测试分层与覆盖率要求 |
| `code-review-checklist.md` | 进行 Code Review 时——Review 检查清单 |

### 加载策略

按场景**按需加载**，不要一次性读取所有文件：

1. **用户让你写一个新接口** → 先读 `rules/api-conventions.md` + `rules/project-structure.md` + `rules/naming.md`，再动手
2. **用户让你排查异常处理问题** → 先读 `rules/exception-handling.md`
3. **用户让你写 SQL / 操作 Redis** → 先读 `rules/database.md`
4. **用户让你做 Code Review** → 先读 `code-review-checklist.md`
5. **用户让你写测试** → 先读 `testing-guide.md`
6. **不确定时** → 先读 `rules.md`（索引页），根据任务匹配对应文件

> **核心原则**：先读规则，再写代码。如果不确定某件事该怎么做，去对应的规则文件里找答案，而不是凭记忆猜。

---

## 代码风格要求

- **严格遵循** `rules/` 目录下的规范和《阿里巴巴 Java 开发手册》
- 使用 Java 21 特性（record、sealed class、pattern matching、virtual thread），但不过度炫技
- 优先使用 Stream API 和 Optional，写出声明式代码
- 命名要见名知意，方法名表达意图而非实现
- 方法长度不超过 80 行，类职责单一

---

## 开发约定

1. **分层架构**：Controller → Service → Mapper（MyBatis-Plus），层间通过 DTO/VO 隔离
2. **异常处理**：全局异常处理器 + 自定义业务异常，禁止在 Controller 中写 try-catch
3. **事务管理**：`@Transactional` 只加在 Service 层，注意事务传播行为
4. **日志规范**：使用 SLF4J，关键操作必须记录日志（方法入参、异常、耗时）
5. **API 设计**：RESTful 风格，统一返回结构，版本化路径 `/api/v1/`

---

## 数据库约定

- **MySQL**：主数据存储，默认 ORM 框架为 **MyBatis-Plus**
- **MongoDB**：文档型数据（日志、非结构化数据），使用 Spring Data MongoDB
- **Redis**：缓存与会话，使用 Spring Data Redis；必须设置过期时间
- 禁止在代码中拼接 SQL，使用参数化查询或 ORM 框架
- 数据库变更必须通过 Flyway / Liquibase 脚本管理，禁止手动改表

---

## 测试要求

- 新增功能必须附带单元测试，覆盖率不低于 80%
- Service 层使用 JUnit 5 + Mockito 进行单元测试
- Controller 层使用 `@WebMvcTest` 进行切片测试
- Repository 层使用 `@MybatisPlusTest` 或 Testcontainers 进行集成测试
- 遵循 `testing-guide.md` 中的测试规范

---

## 你应该做的

- **动手前先读对应的 rules/ 文件**，确保代码符合规范
- 修改代码时，先理解上下文，再动手
- 给出方案时，说明 trade-off 和适用场景
- 主动指出潜在的性能、安全问题
- 发现已有代码的问题时，主动提出改进建议
- 编写代码后，主动补充或更新对应的单元测试

---

## 你不应该做的

- **不要跳过规则加载**——不要在不读 rules/ 的情况下直接写代码
- 不要凭空假设项目结构，先读现有代码
- 不要引入未在 `pom.xml` 中声明的新依赖而不先说明
- 不要在 Controller 层写业务逻辑
- 不要忽略异常，所有异常都要被处理或显式抛出
- 不要提交包含 `TODO` 或 `FIXME` 而无对应 Issue 的代码
