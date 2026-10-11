---
name: skill-doctor
description: >
  评估 Claude Code skill 质量与优秀度。当用户要求给已有 skill 做体检(检查/审查/诊断其设计：
  触发效果、结构、安全风险、维护性)，或询问"skill 质量 / skill 体检 /
  这个 skill 怎么样 / skill 是否有效 / skill 触发测试 / 触发不灵"时使用。
  输入是单个 SKILL.md(或 skill 目录),输出问题清单+健康等级+修复优先级建议。
  不自动修改任何文件。明确排除:创建、修改、编写 skill,解释 skill 代码或基础概念,
  教学性询问(如"什么是 skill""如何使用 skill"),以及 skill 对比/选型建议。
---

# skill-doctor

对单个 Claude Code skill 做质量体检(lint + health check)。**只评估,不修改任何文件。**

## 输入契约

- 输入:单个 skill 目录或 SKILL.md 路径
- 输出:问题清单(带行号证据)+ 🟢🟡🔴 健康等级 + ⭐⭐⭐ 优秀度 + 结果可信度 ⚪🟡🟢 + P0/P1/P2 修复优先级
- 不自动修改文件;用户明确说"保存报告"才落盘

## 扫描顺序(写死)

1. **Gate**(前置硬检查)
2. **Reference integrity**(引用完整性:校验本 skill 自身 references 文件存在,缺失 → 报"缺少参考材料,结果仅部分有效")
3. **Trigger**(触发质量)
4. **Structure**(结构质量)
5. **Anti-pattern**(反模式扫描)
6. **Quality**(质量判断,12 条优秀标准清单)
7. **Result Assurance**(结果保证,证据等级 E0-E4)
8. **Summary**(汇总输出)

顺序不可调换。任一 Gate 级失败 → 标记 BLOCKED,**不降级、不转成建议**,只继续收集证据。

## 阶段 ① Gate 前置硬检查

只做**确定性检查**,禁止 LLM 语义猜测。任一不过 → 🔴 不合格(BLOCKED):

1. frontmatter 合法:name + description 存在,YAML 解析无失败
2. name 与目录名一致
3. 确定性危险模式(内联判定;AP1-AP3 提供误报风险/豁免细节):`curl|bash`、`rm -rf /`、上传 .env、读取 `~/.ssh` 等;命中 → 查 security_exemptions 白名单
4. 引用完整性:被评估 skill 的 SKILL.md 引用的文件确实存在

### security_exemptions 白名单

被评估 skill 的 frontmatter 可声明 `security_exemptions: [pattern]`,声明后该 pattern 命中**不 gate、降为提示**。**白名单只降误报,不绕过其它规则**。未声明的外部调用才是 smell;工具调用本身不是。

### 路径解析(写死)

- 相对路径以 SKILL.md 所在目录为基准,支持 `references/`、`scripts/`、`examples/` 等子目录
- 出现 `../` 或绝对路径 → 报**"路径非法"**,标记 BLOCKED

## 阶段 ② 快速模式(默认,纯静态)

按序评估七个子维度,标签固定枚举:`triggers / structure / antipatterns / brevity / quality / result / craft`。

### A. Trigger 触发质量(模拟触发实验)

- 加载 references/test-templates.md 生成三层查询集(正/难负/歧义)+ references/trigger-protocol.md 跑候选池多选一实验;按协议输出 must-hit 覆盖 / near-miss 误触 / 歧义合理 指标、风险阈值与条件化置信度,并带诚实标注

### B. Structure 结构质量

- 步骤清晰有序号,每步有输入/输出
- 显式编码失败分支
- 单一职责(scope drift:宣称做 X 却含 Y/Z 流程;正式判定走 AP7,此处仅轻量提示,避免重复报告)
- 指令冲突(always vs never、自动 vs 确认)

### C. Anti-pattern 反模式扫描

- 加载 references/anti-patterns.md,全部 15 条逐条机械判定(触发条件 → 豁免清单,任一满足即豁免);AP1-3 未豁免命中同入 Gate Result 与 P0

### D. Brevity 简洁性

- 行数分档:SKILL.md <200 行 🟢 / 200-500 行 🟡 / >500 行 🔴(示例不计入)

### E. Quality 质量判断(12 条优秀标准清单)

- 加载 references/quality-rubric.md 按 4 维 × 3 条 = 12 条逐条核对;AP1-AP10 命中即标准 FAIL 的证据(见 rubric 映射表,不另设评分中心);判定分级、校准锚点与整体判断(3 问,负面强制降 🟡)按 rubric

### F. Result Assurance 结果保证(默认 E0/E1 静态审计)

- 加载 references/result-assurance.md 按 R1-R8 核对(默认 E0/E1,不执行);被评估 skill 为**纯 Markdown(无脚本)或元工具类**时,追加加载 references/stars.md(E3-MD / 设计卓越 证据路径)
- 客观破坏项命中 → 拉低健康(result-assurance.md §1);⭐⭐⭐ 需 E2/E3 证据,完整评级规则见 §4

### G. Craft 写作工艺(可预测性)

- 加载 references/craft.md 按 C1-C5 逐条核对;AP11-AP15 命中即 C 标准证据(见 craft.md 映射表);C1-C5 均 Should → **不进健康/星级公式**,只作报告维度与 findings
- 整体:输出 `C1-C5 → N 通过 / M FAIL`;勾完强制答"若只改一行写作,哪里最提升可预测性",答不出 → Craft 判"表面通过"

## 阶段 ③ 深度模式(显式开启;E2 干跑探针 / E3 证据审查 / E4 沙箱实跑)

- 升级 L4 证据等级,加载 references/result-assurance.md §2-§5
- **E2 干跑探针**(--deep,低风险):bash -n / schema 校验 / 依赖存在性 / 步骤 DAG 断链;不执行 skill 命令;置信度 Medium-High
- **E3 证据审查**(--evidence,只读):审用户/CI 提供的执行证据包(日志/实际输出/失败记录),含 skill 自带 test-cases / 示例 / fixtures 只读验证;**无则报告"not provided",禁止发明测试用例**;High(受证据质量限制)
- **E3-MD 真实使用产物证据**(纯 Markdown,无脚本):加载 references/stars.md 审真实使用产物(报告/代码/文档/审查意见/对话记录),按 9 字段证据包 + 定量达标线核验;无则报告"not provided"
- **E4 沙箱实跑**(--sandbox,显式授权):隔离执行 + 故障注入(超时/断网/缺输入/权限拒绝);前置输出沙箱授权声明,网络默认阻断/文件系统只读/超时 30s;R9/R10 在此启用;只说"沙箱用例内通过"
- 无法提供隔离环境 → 返回"环境不支持深度验证",降级 E1,不阻塞

## 证据与置信度规则

1. 每条结论引 SKILL.md 具体行号或实际输出;找不到行号 → 结论降档
2. 仅对**争议结论**附反证据,不逐条
3. 评估方式标注(自评 / 交叉验证)
4. 证据不足:No evidence → no strong claim;允许"Possible issue",禁止"Invalid";引用缺失 → 输出"引用缺失,无法评估",不得推断

## 输出模板(写死,防格式漂移)

```markdown
# Skill Doctor Report
## Summary        ← 健康 🟢🟡🔴 + 优秀 ⭐⭐⭐ + 结果可信度 ⚪🟡🟢 + 一句话结论
## Gate Result    ← 通过 / 未通过(列失败项)
## 触发实验        ← 覆盖率/误触发率/歧义合理率 + 逐查询证据链 + 置信度
## 优秀标准        ← 12 条 PASS/不足/FAIL/⚠️ + 整体判断(3 问)
## 结果保证        ← R1-R8 判定 + evidence_level/confidence/verified_by/not_verified + Result Assurance 三态
## 写作工艺        ← C1-C5 判定 + AP11-AP15 命中 + 可预测性整体
## Findings       ← 每条:证据(行号)+ 严重度 + 置信度 + 修复方向(含可复制改写示例)
## Priority Fixes ← P0/P1/P2 排序
```

### 健康等级 + 优秀度(双结论,写死,按序判定,取最严)

**健康 Health**(能不能用):
- Gate 失败 / P0 AP / 触发覆盖率过低(模拟实验,根本触发不了)/ 有效性 或 可操作性 任一 FAIL / L4 客观破坏项(缺脚本、语法错误、失败仍 exit 0、未声明危险副作用)→ 🔴
- 任一 P1 finding / 判断力 或 Agent 适配性 FAIL / 整体判断负面 / ⚠️ 无法评估项 ≥4 / L4 客观破坏项(示例不可解析、步骤断链、依赖缺失声明)→ 🟡
- 其余 → 🟢(**缺执行证据 ≠ 降级**,只降执行可信度)

**优秀度 Excellence**(优不优秀):在 Health 🟢 基础上,Must 全过 + Should 达标 ≥6 + 触发实验达标(覆盖/误触达风险阈值)→ ⭐⭐⭐(**需 E2/E3 执行证据;纯 Markdown 无脚本可用 E3-MD 真实使用产物证据,见 references/stars.md**);Must 全过 + Should ≥4 → ⭐⭐;有 Must FAIL 或 Should <4 → ⭐。**纯 E0/E1 无执行证据,最多 ⭐⭐;元工具类纯 Markdown 可暂获 ⭐⭐+「设计卓越」标识(见 stars.md),与实证 ⭐⭐⭐ 明确区分。Health 🟢 且 ⭐ 低 = 健康但平庸**,如实并列输出,不因平庸改色。

### Priority Fixes 排序规则

P0 在前;同优先级按 **Gate → 结构 → 反模式 → 质量 → 简洁性**。

## Reference 导航(随阶段加载)

- 各阶段在小节"加载 references/X.md"即打开时机;引用为判定单源,阶段体不再复述细节
- references/stars.md(优秀度证据路径 E3-MD / 设计卓越)仅**纯 Markdown / 元工具**分支加载,脚本 skill 评估不加载
- 文件缺失 → 报"缺少参考材料,结果仅部分有效",不静默继续

## 自我体检

用本流程跑自己,不允许豁免;报告标注"自我评估,存在确认偏误风险,建议独立评估者盲评一次"。
