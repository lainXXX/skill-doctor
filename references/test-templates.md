# 查询集生成协议(test-templates)

被 skill-doctor 阶段 A(Trigger)加载,配合 references/trigger-protocol.md 执行模拟触发实验。
**核心约束**:查询集禁止从被评估 skill 的 description 原文关键词反推(自证循环,命中率必然虚高);从"它声称能做什么"的一句话能力定位生成。

## 1. 能力定位

- 读 description,用一句话归纳"这个 skill 声称能做什么"(不看正文细节、不抄触发词)
- 例:多 AI 咨询 skill → "把问题并行发给多个网页 AI,回收后提炼成决策文档"

## 2. 三层查询集(controller 构建,judge 不可见标签)

- **正例 must-hit(≥12)**:以能力定位为锚,写用户自然表达(口语/缩写/上下文变体),不得依赖 description 里的精确词
- **难负例 near-miss(≥12)**:语义邻近但职责边界外的任务,最考验区分力。例:评估 skill 的难负例 = "帮我写一个新 skill 的 SKILL.md"(创建≠评估)
- **歧义例 ambiguous(≥6)**:意图不清,应触发 clarify/none
- 数量不足时如实标注样本量,置信度按 trigger-protocol.md §5 降档

## 3. gold 标签来源

- skill 自带 test-cases 优先;否则以能力定位为 silver label 并标注来源
- **作者声明不清本身即 finding**:该 skill 说不清自己该响应什么,是 P1

## 4. 盲标与防自证

- 预期标签对 judge 不可见;judge 只看到冻结信号 + 候选池 + 查询
- 正例不得含 description 原文关键词(尤其不得照抄触发词表)
- 负例不得选"明显无关"的(测不出区分力);要"像"
- 信息不足:description 缺 what/when → 输出"触发质量:信息不足,无法评估",不硬生成查询集
