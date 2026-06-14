# 依赖管理规范

---

## 一、引入规则

1. 新增依赖前评估必要性，避免引入冗余库
2. 统一在 `pom.xml` 的 `<dependencyManagement>` 中管理版本号
3. 禁止使用 SNAPSHOT 版本（开发测试除外）
4. 定期升级依赖，关注安全漏洞（CVE）
5. 新增依赖在 PR 中说明用途

---

## 二、版本管理

- 所有第三方依赖版本集中在 `<dependencyManagement>` 声明
- 子模块 `<dependency>` 不指定版本号，统一继承
- Spring Boot 版本升级时，同步检查第三方依赖兼容性
