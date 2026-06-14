# Git 规范

> 适用于所有项目的 Git 提交规范、分支策略与分支命名约定。

---

## 一、分支策略

采用简化版 Git Flow，保持轻量、实用。

### 1.1 分支类型

| 分支类型 | 命名格式 | 说明 | 生命周期 |
|---------|---------|------|---------|
| 主分支 | `main` | 生产环境代码，始终可部署 | 永久 |
| 开发分支 | `develop` | 日常开发集成分支，最新开发成果 | 永久 |
| 功能分支 | `feature/<issue-id>-<short-desc>` | 新功能开发 | 合并后删除 |
| 修复分支 | `fix/<issue-id>-<short-desc>` | Bug 修复 | 合并后删除 |
| 热修分支 | `hotfix/<issue-id>-<short-desc>` | 生产紧急修复 | 合并后删除 |
| 发版分支 | `release/v<x.y.z>` | 版本发布准备 | 合并后删除 |

### 1.2 分支流转规则

```
main ─────────────────────────────────────────── ──── (tag: vx.y.z)
  ↑                                          ↑
  └─ release/v1.0.0 ─────────────────────── ┘   (发版合并)
                                         ↑
develop ── feature/DEV-101-user-login ────┘
        ── fix/DEV-105-login-bug ─────────┘
        ── ...
```

**核心规则：**

- `main` 只接受来自 `release/*` 或 `hotfix/*` 的合并（PR/MR）
- `develop` 接受来自 `feature/*` 或 `fix/*` 的合并
- 禁止直接向 `main` 或 `develop` push，必须走 PR/MR
- `hotfix/*` 分支需同时合并回 `main` 和 `develop`
- 合并后删除功能/修复分支，保持仓库整洁

---

## 二、分支命名规范

### 2.1 命名格式

```
<type>/<issue-id>-<short-description>
```

### 2.2 命名规则

| 规则 | 正例 | 反例 |
|------|------|------|
| 全小写 | `feature/dev-101-user-login` | `feature/DEV-101-UserLogin` |
| 用连字符分隔单词 | `fix/dev-105-token-expired` | `fix/dev-105_token_expired` |
| issue-id 前缀可选 | `feature/user-profile` | - |
| 描述简洁有意义 | `feature/jira-201-payment-gateway` | `feature/new-stuff` |
| 不含特殊字符 | `hotfix/v1-2-3-crash-on-startup` | `hotfix/v1.2.3 crash!` |

### 2.3 版本号规范（SemVer）

```
v<major>.<minor>.<patch>
```

- **major**: 不兼容的 API 变更
- **minor**: 向下兼容的功能新增
- **patch**: 向下兼容的 Bug 修复

示例：`v1.0.0` → `v1.1.0`（新功能）→ `v1.1.1`（修 Bug）

---

## 三、Commit 提交规范

### 3.1 Commit Message 格式

采用 Angular/Conventional Commits 风格：

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 3.2 Type 类型

| Type | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(auth): 新增 OAuth2 登录` |
| `fix` | Bug 修复 | `fix(order): 修复订单金额计算错误` |
| `docs` | 文档变更 | `docs: 更新 README 安装步骤` |
| `style` | 代码格式（不影响逻辑） | `style: 统一缩进为 4 空格` |
| `refactor` | 重构（非新增功能也非修复） | `refactor(user): 拆分 UserService` |
| `perf` | 性能优化 | `perf(query): 为用户表添加索引` |
| `test` | 测试相关 | `test(auth): 补充登录单元测试` |
| `chore` | 构建/工具/依赖变更 | `chore: 升级 Spring Boot 到 3.5.10` |
| `ci` | CI/CD 配置 | `ci: 配置 GitHub Actions 自动构建` |
| `revert` | 回滚提交 | `revert: feat(auth) 新增 OAuth2 登录` |

### 3.3 Subject 书写规则

| 规则 | 正例 | 反例 |
|------|------|------|
| 中文或英文均可，全项目统一 | `feat(auth): 新增 OAuth2 登录` | 混用 |
| 不超过 50 字 | 简明扼要 | 冗长描述 |
| 不加句号 | `新增 OAuth2 登录` | `新增 OAuth2 登录。` |
| 使用祈使语气（英文） | `add oauth2 login` | `added oauth2 login` |
| 首字母不大写（英文） | `add login page` | `Add login page` |

### 3.4 Body 书写规则（可选）

- 解释 **为什么** 做这个变更，而不是 **做了什么**（代码本身已说明）
- 每行不超过 72 字
- 可使用列表 `-` 列举要点

示例：

```
fix(order): 修复订单金额计算错误

订单优惠折扣计算时，未考虑满减与折扣叠加的场景，
导致部分订单最终金额为负数。

- 修复 DiscountCalculator 中叠加计算逻辑
- 增加 BusinessRule 校验，金额不可为负
```

### 3.5 Footer（可选）

用于关联 Issue、标记 BREAKING CHANGE：

```
feat(api): 重构用户接口返回结构

BREAKING CHANGE: /api/users 返回字段从 snake_case 改为 camelCase

Closes #123, #124
```

### 3.6 完整示例

```
feat(auth): 新增 JWT Token 刷新机制

原方案中 access_token 过期后用户需重新登录，体验较差。
新增 refresh_token 机制，access_token 过期时自动刷新。

- 新增 TokenRefreshFilter
- /auth/refresh 接口签发新 token
- refresh_token 有效期 7 天，支持续签

Closes DEV-101
```

---

## 四、操作规范

### 4.1 日常开发流程

```bash
# 1. 从 develop 拉取最新代码
git checkout develop
git pull origin develop

# 2. 创建功能分支
git checkout -b feature/dev-101-user-login

# 3. 开发并提交（建议频繁提交，保持小步快跑）
git add <files>
git commit -m "feat(auth): 新增登录接口"

# 4. 开发完成后同步 develop 最新代码
git fetch origin
git rebase origin/develop

# 5. 推送并创建 PR
git push origin feature/dev-101-user-login
# → 在平台上创建 PR 到 develop
```

### 4.2 提交粒度

- **一次提交做一件事**：一个提交只包含一个逻辑变更
- **禁止混合提交**：不把功能开发和代码格式化混在一个 commit
- **禁止提交不可编译的代码**：每个 commit 都应能独立编译通过
- **建议频繁提交**：小步快跑，便于 Review 和回滚

### 4.3 禁止事项

- 禁止 `git push --force` 到共享分支（`main`/`develop`）
- 禁止提交敏感信息（密码、密钥、Token）到仓库
- 禁止提交 IDE 配置文件（`.idea/`、`.vscode/` 除非团队约定）
- 禁止提交大型二进制文件，使用 Git LFS 或外部存储
- 禁止 `--no-verify` 跳过 hooks（除非有明确理由并告知团队）

---

## 五、.gitignore 基础规则

确保项目 `.gitignore` 至少包含：

```gitignore
# IDE
.idea/
*.iml
.vscode/
*.swp

# OS
.DS_Store
Thumbs.db

# Build
target/
build/
dist/
out/

# Logs
*.log

# Env / Secrets
.env
.env.local
*.pem
*.key
