# claude_doc

> 集中维护日常开发中使用的 **AI 编程助手配置资产**：`CLAUDE.md`、编码规范（`rules/`）、测试规范、Code Review 清单、可复用 Skills 等。

---

## 为什么需要这个仓库？

在使用 Claude Code、Cursor、Windsurf 等 AI 编程助手时，每个项目都需要一套"规则文件"来约束 AI 的行为——代码风格、分层架构、数据库约定、安全要求……如果每个项目各写一份，很快就会：

- **重复劳动**：同样的 Git 规范、命名约定在每个项目重写一遍
- **版本不一致**：项目 A 的规范更新了，项目 B/C 还是旧的
- **无处可查**：想复用上次写好的那套规则，但找不到在哪个仓库里

这个仓库就是为了解决这些问题——**一处维护，各项目按需引用**。

---

## 组织原则

**顶层按项目/技术栈分目录，目录内按用途统一布局，跨项目通用内容抽到 `common/`。**

```
打开项目 → 进对应目录 → 把需要的东西复制到实际项目根目录
```

这样设计的好处：

| 场景 | 操作 |
|------|------|
| 新开一个 Java 后端项目 | 复制 `backend-java/` 内容，改改就能用 |
| 通用 Git 规范有变更 | 只改 `common/` 一处，各项目同步引用 |
| 找某个规范 | 路径固定，不用满仓库翻 |

---

## 目录结构

```
claude_doc/
├── common/                            # 跨项目通用
│   ├── git-conventions.md             # Git 提交规范 / 分支策略 / 分支命名
│   └── README-template.md             # README / CHANGELOG / CONTRIBUTING / API 文档模板
│
├── backend-java/                      # Java 后端（Java 21 + Spring Boot 3.5.10）
│   ├── CLAUDE.md                      # AI 编程助手项目级指令（含规则加载机制）
│   ├── rules.md                       # 编码规范索引页（指向 rules/ 下各专题）
│   ├── testing-guide.md               # 测试规范（单元测试 / 集成测试 / 覆盖率）
│   ├── code-review-checklist.md       # Code Review 检查清单（60+ 检查项）
│   ├── rules/                         # 细分规范（按主题拆分）
│   │   ├── naming.md                  # 命名规范
│   │   ├── project-structure.md       # 项目结构与分层
│   │   ├── exception-handling.md      # 异常处理
│   │   ├── api-conventions.md         # API 接口规范
│   │   ├── database.md                # 数据库规范（MySQL / MongoDB / Redis / MyBatis-Plus）
│   │   ├── logging.md                 # 日志规范
│   │   ├── comments.md                # 注释规范
│   │   ├── security.md                # 安全规范
│   │   └── dependency-management.md   # 依赖管理
│   └── skills/                        # 可复用技能（待补充）
│
├── backend-python/                    # Python 后端（待建）
├── frontend-react/                    # React 前端（待建）
│
└── README.md                          # 本文件
```

---

## 快速开始

### 1. 复制配置到你的项目

```bash
# 以 Java 后端为例
cp -r backend-java/CLAUDE.md      /path/to/your-project/
cp -r backend-java/rules/         /path/to/your-project/
cp -r backend-java/testing-guide.md  /path/to/your-project/
cp -r common/git-conventions.md   /path/to/your-project/
```

### 2. 将 CLAUDE.md 放在项目根目录

`CLAUDE.md` 是 AI 编程助手的入口文件——放在项目根目录，AI 会自动读取其中的指令。

### 3. 根据实际项目调整

复制后，根据你的项目实际情况修改：
- 数据库连接信息、表名前缀
- 包名（`com.yourcompany.yourproject`）
- 业务特定的错误码段

---

## 各文件说明

### common/

| 文件 | 内容 |
|------|------|
| `git-conventions.md` | Conventional Commits 提交格式、简化版 Git Flow 分支策略、SemVer 版本号、`.gitignore` 基础规则 |
| `README-template.md` | 4 套模板：项目 README、CHANGELOG（Keep a Changelog）、CONTRIBUTING、API 文档 |

### backend-java/

| 文件 | 内容 |
|------|------|
| `CLAUDE.md` | AI 助手的角色定位、技术栈、代码风格、分层架构、规则加载机制（什么场景读什么规则文件） |
| `rules.md` | 编码规范索引页，以表格形式链接到 `rules/` 下各专题 |
| `rules/naming.md` | 包/类/方法/变量/常量命名规范，含正反示例 |
| `rules/project-structure.md` | 分层架构（Controller → Service → Mapper），含每层代码示例 |
| `rules/exception-handling.md` | 统一异常体系（`BusinessException` + `ErrorCode`），全局异常处理器 |
| `rules/api-conventions.md` | RESTful URL 设计、HTTP 方法语义、统一返回信封、分页、错误码规划、版本管理、认证授权、Swagger 注解、限流/幂等/脱敏 |
| `rules/database.md` | MySQL 表设计规范、MyBatis-Plus 使用规范（Mapper/Entity/条件构造器/Service 继承/分页）、MongoDB 集合设计、Redis 键设计 |
| `rules/logging.md` | 日志框架配置、日志级别使用场景、敏感信息脱敏、链路追踪 |
| `rules/comments.md` | Javadoc 要求、类/方法/字段注释规范、TODO/FIXME 格式 |
| `rules/security.md` | SQL 注入/XSS/CSRF/敏感数据防护，认证鉴权约定 |
| `rules/dependency-management.md` | 版本管理策略、依赖范围约定、BOM 使用 |
| `testing-guide.md` | 测试金字塔分层、AAA 模式、Service 单元测试（Mockito）、Controller 切片测试（@WebMvcTest）、Mapper 集成测试（@MybatisPlusTest + Testcontainers）、参数化测试、JaCoCo 80% 覆盖率配置 |
| `code-review-checklist.md` | 13 个维度 60+ 检查项（功能/命名/架构/异常/DB/缓存/日志/安全/性能/测试/API/依赖），附 Reviewer 评语分级模板 |

---

## AI 工具适配

不同的 AI 编程助手读取规则的方式略有不同，但 `CLAUDE.md` 的引导逻辑是通用的：

| 工具 | 规则文件 | 加载方式 |
|------|---------|---------|
| **Claude Code** | `CLAUDE.md` | 自动读取项目根目录，根据 CLAUDE.md 中的场景引导加载对应 `rules/` 文件 |
| **Cursor** | `.cursor/rules/*.mdc` | 支持 `globs` 路径匹配自动触发（可基于本仓库 rules/ 生成适配版） |
| **Windsurf** | `.windsurfrules` | 全局生效 |
| **GitHub Copilot** | `.github/copilot-instructions.md` | 全局生效 |

---

## 技术栈速览

### backend-java

| 组件 | 版本 | 用途 |
|------|------|------|
| Java | 21 | LTS |
| Spring Boot | 3.5.10 | 应用框架 |
| MyBatis-Plus | 3.5.x | ORM 框架（默认） |
| MySQL | 8.0 | 主数据库 |
| MongoDB | 7.0 | 文档数据库 |
| Redis | 6.0 | 缓存 |
| JUnit 5 | 5.x | 测试框架 |
| Mockito | 5.x | Mock 框架 |
| Testcontainers | 1.x | 集成测试容器 |
| JaCoCo | 0.8.x | 覆盖率 |

---

## 项目状态

| 目录 | 技术栈 | 状态 |
|------|--------|------|
| `common/` | 通用 | Git 规范 ✅ / 文档模板 ✅ |
| `backend-java/` | Java 21 + Spring Boot 3.5.10 + MyBatis-Plus + MySQL/MongoDB/Redis | ✅ 已完成 |
| `backend-python/` | （规划中） | ⏳ 待建 |
| `frontend-react/` | （规划中） | ⏳ 待建 |

---

## 许可证

仅供个人 / 团队内部使用。
