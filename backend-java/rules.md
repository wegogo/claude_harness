# Java 后端编码规范

> 基于《阿里巴巴 Java 开发手册》并针对本项目技术栈（Java 21 + Spring Boot 3.5 + MySQL/MongoDB/Redis）补充。
> 阿里规范是底线，本系列文档在其基础上有更细化的要求。两者冲突时，以本文档为准。

本规范按主题拆分为独立文件，放在 `rules/` 目录下。点击下方链接查看各专题。

---

## 规范索引

| 专题 | 文件 | 说明 |
|------|------|------|
| 命名规范 | [naming.md](rules/naming.md) | 通用命名规则、各层后缀约定、方法命名 |
| 项目结构与分层 | [project-structure.md](rules/project-structure.md) | 目录结构、Controller/Service/Model 分层规范 |
| 异常处理 | [exception-handling.md](rules/exception-handling.md) | 统一异常体系、全局异常处理器 |
| API 接口规范 | [api-conventions.md](rules/api-conventions.md) | RESTful 设计、请求响应、错误码、版本管理、安全 |
| 数据库规范 | [database.md](rules/database.md) | MySQL / MongoDB / Redis 使用规范 |
| 日志规范 | [logging.md](rules/logging.md) | 日志级别、日志格式、敏感信息处理 |
| 注释规范 | [comments.md](rules/comments.md) | 类注释、方法注释、注释原则 |
| 安全规范 | [security.md](rules/security.md) | SQL 注入、XSS、密码存储、接口鉴权、依赖安全 |
| 依赖管理 | [dependency-management.md](rules/dependency-management.md) | 依赖引入规则、版本统一管理 |

---

## 关联文档

- [CLAUDE.md](CLAUDE.md) — AI 编程助手项目级指令
- [testing-guide.md](testing-guide.md) — 单元测试 / 集成测试规范
- [code-review-checklist.md](code-review-checklist.md) — Code Review 检查清单

---

## 参考文档

- [阿里巴巴 Java 开发手册（嵩山版）](https://github.com/alibaba/p3c)
- [Spring Boot 3.5 官方文档](https://docs.spring.io/spring-boot/docs/3.5.x/reference/html/)
- [MySQL 8.0 参考手册](https://dev.mysql.com/doc/refman/8.0/en/)
- [MongoDB 7.0 文档](https://www.mongodb.com/docs/v7.0/)
- [Redis 6.0 命令参考](https://redis.io/commands/)
