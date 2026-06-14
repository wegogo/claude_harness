# 文档与 README 模板

> 适用于所有项目的 README、API 文档、变更日志等模板。

---

## 一、README 模板

以下为标准 README 结构，按需取用，不必全部填充。

```markdown
# <项目名称>

> 一句话描述项目是什么、解决什么问题。

## 功能特性

- 特性一
- 特性二
- 特性三

## 技术栈

| 类别 | 技术 | 版本 |
|------|------|------|
| 语言 | Java | 21 |
| 框架 | Spring Boot | 3.5.x |
| 数据库 | MySQL | 8.0 |

## 快速开始

### 环境要求

- JDK 21+
- Maven 3.9+ / Gradle 8.x
- MySQL 8.0+

### 安装与运行

# 1. 克隆仓库
git clone <repo-url>
cd <project-name>

# 2. 配置环境变量
cp .env.example .env
# 编辑 .env 填入数据库连接等信息

# 3. 启动
./mvnw spring-boot:run

### Docker 部署（可选）

docker build -t <project-name> .
docker run -p 8080:8080 <project-name>

## 项目结构

项目名称/
├── src/main/java/
│   └── com/example/project/
│       ├── controller/    # 控制器层
│       ├── service/       # 业务逻辑层
│       ├── repository/    # 数据访问层
│       ├── model/         # 数据模型
│       └── config/        # 配置类
├── src/main/resources/    # 配置文件
└── src/test/              # 测试代码

## 配置说明

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `server.port` | 服务端口 | 8080 |
| `spring.datasource.url` | 数据库地址 | - |

## API 文档

启动后访问 Swagger UI：`http://localhost:8080/swagger-ui.html`

或参见 [API 文档](docs/api.md)。

## 测试

# 运行全部测试
./mvnw test

# 运行指定测试类
./mvnw test -Dtest=UserServiceTest

## 贡献指南

参见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 版本历史

参见 [CHANGELOG.md](CHANGELOG.md)。

## 许可证

[MIT](LICENSE)
```

---

## 二、CHANGELOG 模板

基于 [Keep a Changelog](https://keepachangelog.com/) 格式：

```markdown
# 变更日志

本文件记录项目的所有重要变更。

格式遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)，
版本号遵循 [Semantic Versioning](https://semver.org/lang/zh-CN/)。

## [Unreleased]

### Added
- 待发布的新功能...

## [1.1.0] - 2025-06-01

### Added
- 新增用户头像上传功能
- 新增批量导入接口

### Changed
- 用户列表接口返回结构优化，支持分页

### Fixed
- 修复高并发下 Token 刷新失败的问题

### Security
- 升级 Spring Security 修复已知漏洞

## [1.0.0] - 2025-05-01

### Added
- 项目初始版本发布

[Unreleased]: https://github.com/user/repo/compare/v1.1.0...HEAD
[1.1.0]: https://github.com/user/repo/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/user/repo/releases/tag/v1.0.0
```

### 变更类型说明

| 类型 | 说明 |
|------|------|
| `Added` | 新增功能 |
| `Changed` | 对已有功能的变更 |
| `Deprecated` | 即将废弃的功能 |
| `Removed` | 已移除的功能 |
| `Fixed` | Bug 修复 |
| `Security` | 安全相关修复 |

---

## 三、CONTRIBUTING 模板

```markdown
# 贡献指南

感谢你参与本项目！请遵循以下规范。

## 开发环境搭建

1. Fork 并克隆仓库
2. 参考 [README](README.md) 完成本地环境搭建
3. 确认所有测试通过：`./mvnw test`

## 开发流程

1. 从 `develop` 创建功能分支：`feature/<issue-id>-<desc>`
2. 遵循 [Git 提交规范](git-conventions.md)
3. 遵循项目编码规范（参见 rules.md）
4. 提交 PR 到 `develop`，等待 Code Review
5. PR 通过后合并

## 编码要求

- 遵循项目 rules.md 中的编码规范
- 新增功能需附带单元测试，覆盖率不低于 80%
- 公共接口变更需更新 API 文档
- 提交前执行格式化检查：`./mvnw validate`

## PR 标题格式

与 commit message 的 subject 保持一致：

<type>(<scope>): <description>

示例：`feat(auth): 新增 OAuth2 登录`

## Review 标准

参见 [Code Review 检查清单](../backend-java/code-review-checklist.md)。
```

---

## 四、API 文档模板

```markdown
# API 文档

> 基础路径：`/api/v1`

## 用户模块

### 创建用户

POST /users

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| username | string | 是 | 用户名，3-20 字符 |
| email | string | 是 | 邮箱地址 |
| password | string | 是 | 密码，至少 8 位 |

**请求示例**

{
  "username": "john",
  "email": "john@example.com",
  "password": "********"
}

**响应示例**

成功（201 Created）：

{
  "code": 0,
  "message": "success",
  "data": {
    "id": 1,
    "username": "john",
    "email": "john@example.com",
    "createdAt": "2025-06-01T10:00:00Z"
  }
}

失败（400 Bad Request）：

{
  "code": 1001,
  "message": "用户名已存在",
  "data": null
}

**错误码**

| 错误码 | HTTP 状态 | 说明 |
|--------|----------|------|
| 1001 | 400 | 用户名已存在 |
| 1002 | 400 | 邮箱格式错误 |
| 1003 | 400 | 密码强度不足 |
```

---

## 五、文档书写通用规范

1. **语言**：中文项目用中文，技术术语保留英文（如 Service、Controller）
2. **代码块**：标注语言类型，便于高亮显示
3. **表格**：列对齐，表头清晰
4. **层级**：标题不超过 4 级（`####`），更深的内容拆分为子文档
5. **更新**：代码变更时同步更新文档，PR 中包含相关文档变更
6. **链接**：内部链接用相对路径，外部链接用完整 URL
