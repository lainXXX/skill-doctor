# skill-doctor

[![CI](https://github.com/lainXXX/skill-doctor/actions/workflows/ci.yml/badge.svg)](https://github.com/lainXXX/skill-doctor/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

一个用于 Claude Code 的 skill，评估单个 skill 的触发边界、结构、安全风险、质量、结果证据和写作工艺，并给出带行号证据的体检报告与修复优先级。

A Claude Code skill for evidence-based reviews of skill design, trigger boundaries, safety, and maintainability.

默认只读评估，不自动修改被评估文件；用户明确要求保存报告时才落盘。当前处于早期开发阶段，规则变化见 [更新记录](CHANGELOG.md)。

## 安装

需要支持本地 skills 的 Claude Code。skill 本体是 Markdown 指令与参考协议，无需安装 npm 或 Python 运行时依赖。Git 用于克隆或更新仓库。

macOS / Linux：

```bash
git clone https://github.com/lainXXX/skill-doctor.git ~/.claude/skills/skill-doctor
```

Windows PowerShell：

```powershell
git clone https://github.com/lainXXX/skill-doctor.git "$env:USERPROFILE\.claude\skills\skill-doctor"
```

保持安装目录名为 `skill-doctor`，并保留整个 `references/` 目录。如果目标目录已存在，请先检查其内容，避免覆盖自己的修改。

## 使用

在 Claude Code 会话中指定单个 skill 目录或 `SKILL.md` 路径，例如：

```text
用 skill-doctor 体检 ~/.claude/skills/my-skill，只评估，不修改文件。
```

Windows 示例：

```text
用 skill-doctor 体检 C:/path/to/my-skill/SKILL.md，重点检查触发边界和维护性。
```

其他常见请求：

```text
这个 skill 为什么触发不灵？请按 skill-doctor 的协议检查。
请审查这个 skill 的设计，列出带行号证据的问题和 P0/P1/P2 修复建议。
将刚才的体检报告保存到我指定的文件。
```

创建或修改 skill、解释 skill 基础概念，以及多个 skill 的对比选型，不属于本 skill 的职责。

## 评估内容

| 阶段 | 内容 |
|------|------|
| Gate | frontmatter、名称、危险模式和目标 skill 引用完整性 |
| Reference integrity | 检查 skill-doctor 自身参考协议是否齐全 |
| Trigger | 正例、难负例和歧义查询的模拟触发实验 |
| Structure / Anti-pattern | 步骤与失败分支，以及 AP1–AP15 反模式 |
| Quality | 有效性、判断力、Agent 适配性和可操作性，共 12 条标准 |
| Result Assurance | 声明与证据是否匹配，以及尚未验证的范围 |
| Craft | C1–C5 写作工艺，独立于健康与优秀度评级 |

具体规则以 [SKILL.md](SKILL.md) 及其引用的协议为准。

## 报告结果

报告包含 Summary、Gate Result、触发实验、优秀标准、结果保证、写作工艺、Findings 和 Priority Fixes。

- **健康等级 🟢 / 🟡 / 🔴**：设计与使用风险；Gate 失败标记为 BLOCKED。
- **优秀度 ⭐ / ⭐⭐ / ⭐⭐⭐**：质量与证据共同决定，纯 E0/E1 静态评估最多两星。
- **结果可信度 ⚪ / 🟡 / 🟢**：区分未验证、静态通过与受控用例内的运行验证。
- **P0 / P1 / P2**：修复优先级，每项应带证据、置信度和修复方向。

## 证据与深度模式

默认使用 E0/E1 静态评估，不执行被评估 skill 的命令。可以在对话中显式请求更深层的审查：

| 模式 | 作用与边界 |
|------|------------|
| E2 / `--deep` | 解析、schema、依赖存在性等干跑检查，不执行目标工作流 |
| E3 / `--evidence` | 只读审查已有日志、输出或测试证据；缺失时标注未提供 |
| E3-MD | 审查纯 Markdown skill 的真实使用产物与独立复核证据 |
| E4 / `--sandbox` | 仅在宿主提供隔离环境且用户明确授权后实跑；不支持时降级 E1 |

这些标记是会话中的模式约定，本仓库不提供同名命令行程序或内置沙箱引擎。

触发实验是静态模拟，不能证明 Claude Code 的真实路由效果。评级依赖评估模型、上下文和证据质量；缺少运行证据时应说明限制，不能把 CI 通过当成评估效果验证。完整协议见 [触发实验](references/trigger-protocol.md)、[结果保证](references/result-assurance.md) 和 [纯 Markdown 证据路径](references/stars.md)。

## 项目结构

```text
SKILL.md                         skill 入口、执行顺序和报告契约
references/anti-patterns.md       AP1–AP15 反模式
references/craft.md               C1–C5 写作工艺
references/quality-rubric.md      12 条优秀标准
references/result-assurance.md   结果证据与深度模式
references/stars.md              纯 Markdown 与元工具的优秀度证据
references/test-templates.md     三层查询集生成协议
references/trigger-protocol.md   模拟触发实验协议
.github/                        文档完整性 CI、Issue 和 PR 模板
```

## 参与贡献

欢迎提交脱敏的问题反馈、真实评估案例、协议改进和文档修正。请先阅读 [贡献指南](CONTRIBUTING.md)；漏洞报告见 [安全政策](SECURITY.md)。维护者的上传与更新步骤见 [发布指南](PUBLISHING.md)。

CI 仅检查 skill frontmatter、参考文件和文档链接，不调用模型、不运行被评估 skill，也不自动给项目评级。

## 许可证

本项目的 skill、参考协议与文档采用 [MIT 许可证](LICENSE)。第三方 skill、用户提交的输入材料和评估产物中的第三方内容保留各自的权利与许可。

本项目是独立社区项目，与 Anthropic 无官方隶属或背书关系。
