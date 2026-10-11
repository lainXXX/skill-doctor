# 发布指南

仓库地址：[lainXXX/skill-doctor](https://github.com/lainXXX/skill-doctor)。这是 Markdown skill 项目，GitHub 上传无需发布 npm 或 Python 包。

## 提交范围

提交 `SKILL.md`、完整 `references/`、开源文档及 `.github/` 配置。不要提交本地 agent 设置、凭证、缓存、私人评估对象、真实会话记录或未经脱敏的报告。

`.gitignore` 已排除常见本地文件及根目录下的 `reports/`、`evidence/`、`private/` 等私人数据目录。经过授权和脱敏的公开案例应使用明确的独立路径，并在 PR 中说明来源。

## 更新流程

在仓库根目录执行，并把占位参数替换为本次明确需要提交的文件：

```bash
git status --short
git add -- <本次需要提交的文件>
git diff --cached --stat
git diff --cached
git commit -m "docs: describe the change"
git push
```

先完成资料与验证，再创建提交，最后 push。上传后在 Actions 页面检查 CI；结构检查通过不代表真实触发效果或评级准确性通过。

## 正式版本

发布正式版本前，把 [更新记录](CHANGELOG.md) 的 `Unreleased` 整理为版本号与发布日期，再创建对应 Git 标签和 GitHub Release。评级规则发生变化时，说明旧报告与新规则的兼容关系。
