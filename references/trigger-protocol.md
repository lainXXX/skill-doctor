# 触发实验协议(trigger-protocol)

被 skill-doctor 阶段 A(Trigger)加载,配合 references/test-templates.md 生成查询集后执行。
**定位**:高保真静态代理,不是真实触发。结果 = "在该模拟环境下 description 是否容易被正确选中"。

## 1. 冻结触发信号

- 被评估 skill 的 **description + name**(hash 后冻结,实验全程不可修改)
- judge **不可见**正文与预期标签(泄漏正文会污染:"明明能解决问题→应该触发")
- 超长 description 按真实路由截断模拟(仅保留可见前缀,提示 judge 超长会被截断)

## 2. 候选池多选一

- 判定为**多选一**,不是二分类:候选 = 被评估 skill + 2-3 个干扰 skill + **none** + **clarify**
- 干扰 skill 从已装 skills 只读其 description:1 个语义相近 + 1 个无关
- 问"选哪个最合适",而非"是否触发"
- judge 输出:`所选候选 + 置信度(0-1)+ 一句话理由`

## 3. 查询集分层(生成见 test-templates.md)

- 正例 must-hit:应触发(选目标 skill)
- 难负例 near-miss:语义邻近但边界外,应选 none 或干扰项
- 歧义例 ambiguous:意图不清,应选 clarify
- 总样本 ≥30 条才有统计意义,否则置信度降档

## 4. 风险画像调阈值

- 先做风险画像:写文件?执行命令?网络?密钥?
- 只读/低风险:must-hit recall ≥80%,near-miss 误触 ≤15%
- 修改文件/中风险:recall ≥85%,误触 ≤10%
- 破坏性/高风险:recall ≥90%,误触 ≤5%
- 阈值仅作报告参考,标注"单模型模拟,仅供参考"

## 5. 条件化置信度

| 条件 | 置信度 |
|------|--------|
| 仅静态 description 检查 | Low |
| 小样本(<15)或无难负例 | Medium |
| ≥30 条含难负例 + 多次采样稳定 | Medium-High |
| 加真实日志校准(增强路径) | High |

## 6. 评估卫生

- 干扰 skill 只读其 description,不读取正文
- 报告标注评估方式(自评/交叉验证)与运行模型
- 整体标注"单模型高保真代理,多模型交叉验证为增强路径"
