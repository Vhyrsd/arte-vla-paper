# ArTE-VLA 论文写作修改清单

本文档用于逐条比较、修改和验收 `root.tex`。检查基于 2026-09-15 的稿件版本；文中行号均为本次检查时的初始行号，后续应优先按章节和原句定位。

## 使用方法

- 状态统一使用：`[ ] 待处理`、`[-] 修改中`、`[x] 已完成`、`[~] 暂不修改`。
- 完成一项时，在“修改记录”中填写最终选择及对应 commit、日期或新行号。
- 涉及实验补充的项目，先确定是否能够补实验，再决定如何调整论文主张。
- 优先级含义：
  - **P0**：可能直接影响论文结论可信度或审稿判断。
  - **P1**：影响方法可理解性、实验可复现性或论证完整性。
  - **P2**：语言、术语、引用与排版层面的改进。

## 一、P0：投稿前必须解决

### P0-01 主结果采用 benchmark-wise best 超参数

- 状态：`[ ] 待处理`
- 初始位置：Experimental Setup / Closed-Loop Performance，`root.tex:345-346, 465-466, 520-537`
- 当前问题：主表分别采用 LIBERO-Object 上的 `lambda_E=1.0` 和 RoboCasa 上的 `lambda_E=0.1`，正文同时说明没有基于验证集进行模型选择。如果两个权重由最终评测结果确定，则属于在测试结果上选择配置。
- 风险：84.3% 和 67.0% 容易被审稿人视为 post-hoc best-of-three，而不是可复现的 primary result。
- 推荐处理：
  1. 首选：使用独立验证集选择 `lambda_E`，然后只在测试集评测一次；
  2. 次选：预先固定一个跨 benchmark 共用的 `lambda_E`；
  3. 若无法补实验：将主表明确命名为 “best tested configuration”，把完整 sweep 作为主要结果，并将比较定性为 exploratory。
- 需要同步修改：摘要、贡献列表、主表 caption、结果段和结论中的 “improves/outperforms”。
- 验收标准：读者能够明确知道超参数何时、根据什么数据选定；任何主结果都不存在未披露的测试集选择。
- 修改记录：

### P0-02 区分训练种子与 rollout seeds

- 状态：`[x] 已完成`
- 初始位置：Abstract，`root.tex:56-61`；Evaluation Protocol，`root.tex:473-490`；Limitations，`root.tex:742-745`；Conclusion，`root.tex:774-781`
- 当前问题：全文多次写 “three evaluation seeds”，但所有结果只来自一个 seed-1000 training checkpoint。读者容易将其误解为三次独立训练。
- 推荐措辞：
  > Using a single training checkpoint evaluated under three rollout seeds, ...
- 统一术语：使用 “training seed” 与 “rollout seed” 或 “evaluation-randomness seed”，不要笼统写 “seed”。
- 验收标准：摘要、表格和正文首次报告结果时均明确说明 `one training seed + three rollout seeds`。
- 修改记录：2026-09-15 已统一全文相关措辞。摘要和首次报告结果处明确写为每种方法使用一个 seed-1000 training checkpoint，并在三个 rollout seeds 下评测；实验协议、表格 caption、消融、讨论和结论统一使用 “rollout seed”，同时明确这些结果衡量 rollout stochasticity，而非独立训练初始化的方差。

### P0-03 重新审视 McNemar 检验及独立性假设

- 状态：`[ ] 待处理`
- 初始位置：Evaluation Protocol，`root.tex:473-485`；Results，`root.tex:526-528`；Conclusion，`root.tex:774-777`
- 当前问题：1,500 次 rollout 重复使用 demonstration-derived initial states；同一初始状态在不同 rollout seeds 下的结果可能相关。直接汇总后进行 exact McNemar test 可能高估有效样本量。
- 推荐处理：
  - 解释同一 initial state 跨 seeds 的随机性和依赖关系；
  - 增加按 task 或 initial state 聚类的 bootstrap confidence interval；
  - 报告 seed-wise 和 task-wise 结果；
  - 将 pooled McNemar `p` 值定位为辅助分析，而不是核心结论。
- 验收标准：统计单位、配对方式和独立性假设均有明确说明；主结论不依赖可能存在伪重复的极小 `p` 值。
- 修改记录：

### P0-04 明确定义 target，避免与目标容器混淆

- 状态：`[ ] 待处理`
- 初始位置：Abstract，`root.tex:46-55`；Introduction，`root.tex:95-114`；Implicit Target Representation，`root.tex:233-252`；Replay-Based Effect Supervision，`root.tex:356-391`
- 当前问题：PickPlace 指令通常同时包含被移动物体和目标容器；“language-referred target” 与 “primary target” 无法说明监督的是哪一个实体。
- 推荐定义：
  > We define the manipulation target as the object or articulated part whose physical state is expected to change, rather than the destination receptacle.
- 术语选项：若实验中始终监督被移动物体，优先考虑全文改用 “manipulated entity” 或 “manipulated object”。
- 验收标准：首次出现 target 时有无歧义的操作性定义；数据标注、模型预测和实验分析使用同一含义。
- 修改记录：

### P0-05 定义 effect，并避免不受支持的意图或因果含义

- 状态：`[ ] 待处理`
- 初始位置：Title；Abstract，`root.tex:47-65`；Introduction，`root.tex:81-103`；Problem Formulation，`root.tex:218-231`；Effect Schema，`root.tex:273-286`
- 当前问题：标签来自演示中实际发生的未来运动，不是独立定义的任务意图；“effect” 还可能被理解为 causal effect。当前的 “should change”“intended physical evolution” 容易过度解释监督信号。
- 推荐定义：
  > Here, “effect” denotes the manipulation target's observed future motion under the demonstrated behavior; it does not imply a causal effect.
- 推荐替换：
  - “how the target should change” → “how the manipulated entity changes over future horizons”
  - “intended physical evolution” → “task-relevant future motion”
  - “compact physical future” → “compact representation of future target motion”
- 可选标题：
  > ArTE-VLA: Autoregressive Target-Motion Prediction for Vision-Language-Action Models
- 验收标准：全文不把 demonstration-derived future motion 写成已验证的意图或因果效应。
- 修改记录：

### P0-06 增加完整测试集上的 effect prediction 质量

- 状态：`[ ] 待处理`
- 初始位置：Effect-Loss Ablation，`root.tex:588-621`；Qualitative Effect Inspection，`root.tex:661-724`
- 当前问题：论文声称预测结构化 target effects，但只展示一个 Butter 训练样本，没有完整 held-out distribution 上的预测误差。
- 推荐补充：
  - 各 horizon 的 translation、rotation、linear/angular velocity MAE；
  - activity precision、recall、F1 或 AUROC；
  - rescue tasks 与 regression tasks 的预测误差对比；
  - LIBERO 与 RoboCasa 的 held-out 结果；
  - 误差单位、归一化方式和聚合方式。
- 验收标准：读者可以判断 effect 是否准确、误差如何随 horizon 增长，以及性能下降任务是否伴随更差的 effect prediction。
- 修改记录：

### P0-07 收紧 intervention 的结论

- 状态：`[ ] 待处理`
- 初始位置：Abstract，`root.tex:62-66`；Effect Interventions，`root.tex:393-405`；Action Sensitivity，`root.tex:623-659`；Conclusion，`root.tex:781-785`
- 当前问题：实验只证明 effect 数值被改变后 action chunk 出现数值差异，并未证明变化方向一致、具有闭环控制价值或能够解释性能提升。
- 推荐摘要措辞：
  > Under fixed action noise, effect-value interventions alter all 32 evaluated action chunks.
- Reverse 的准确解释：它交换 effect values 与 horizon contexts 的对应关系，不是逆转 effect predictor 的 autoregressive computation。
- “prespecified 1% threshold” 应在结果前定义并给出理由；如果实际上没有预先确定，则删去 “prespecified”。
- 补充 relative RMS 的公式、分母和接近零时的处理方式。
- 验收标准：结论只声称 numerical/computational sensitivity，不把它表述为 causal utility 或 closed-loop benefit。
- 修改记录：

## 二、P1：论证完整性和复现性

### P1-01 增加能够隔离核心贡献的 controls

- 状态：`[ ] 待处理`
- 初始位置：Baseline and Ablation，`root.tex:492-505`；Limitations，`root.tex:742-751`
- 当前问题：现有实验无法区分性能变化来自显式物理值、隐藏 horizon context、新增参数，还是 autoregressive structure。
- 推荐优先级：
  1. capacity-matched extra-token baseline；
  2. hidden-context-only；
  3. effect-values-only；
  4. parallel/non-autoregressive horizon prediction；
  5. ground-truth effect conditioning，作为上界或诊断。
- 验收标准：至少能够回答“显式 effect values 是否比等容量 token 更有用”。
- 修改记录：

### P1-02 补全 RoboCasa 训练数据和标注流程

- 状态：`[ ] 待处理`
- 初始位置：Replay-Based Effect Supervision，`root.tex:356-391`；Benchmark and Data，`root.tex:413-423`
- 当前问题：LIBERO 报告了 episodes、frames、valid labels 和 active labels，RoboCasa 没有对应信息；“We apply this ... throughout”过于笼统。
- 需要补充：
  - training demonstrations、episodes 和 frames 数；
  - valid/active horizon labels 数；
  - manipulated entity 的确定方式；
  - simulator state extraction 和 replay 细节；
  - PickPlace-5 是官方 suite 还是作者选取的子集。
- 验收标准：两个 benchmark 的数据来源和 enrichment 过程达到相近的描述粒度。
- 修改记录：

### P1-03 补全 action 与 rollout 协议

- 状态：`[ ] 待处理`
- 初始位置：Problem Formulation，`root.tex:211-231`；Action Expert，`root.tex:307-321`；Evaluation Protocol，`root.tex:471-490`
- 当前缺失：
  - action dimension；
  - action chunk length `C`；
  - 每次 replanning 实际执行多少步；
  - “fixed time horizon”的具体步数或秒数；
  - success 判定来源；
  - RoboCasa “task-seeded episodes”的准确含义；
  - instructions 是固定、模板化还是随机生成。
- 验收标准：独立读者能够据此复现闭环 rollout。
- 修改记录：

### P1-04 解释 effect labels 的量纲与归一化

- 状态：`[ ] 待处理`
- 初始位置：Training Objective，`root.tex:323-354`；Effect Labels，`root.tex:356-375`
- 当前问题：位置、旋转、线速度和角速度量纲及数值尺度不同，但正文说 12 个维度等权。未说明输入损失前是否标准化。
- 推荐处理：
  - 若有标准化：给出统计量来源、按 component 还是全局标准化，以及 inference 时如何还原；
  - 若无标准化：报告各分量典型尺度并解释等权设计；
  - 说明 Smooth-`L1` 的 beta/delta 参数。
- 验收标准：损失定义在量纲和数值尺度上可复现、可解释。
- 修改记录：

### P1-05 澄清 activity label 的时间语义

- 状态：`[ ] 待处理`
- 初始位置：Effect Schema，`root.tex:275-286`；Effect Labels，`root.tex:372-375`
- 当前问题：activity 由累计位移、旋转增量以及未来时刻速度共同决定，因此它既不是单纯的未来瞬时状态，也不只是区间运动标记。
- 推荐处理：明确写为 “motion-activity label for horizon `h`”，并解释它表示区间显著变化或终点速度满足任一阈值。
- 验收标准：读者能够仅根据文字和公式复现二值标签。
- 修改记录：

### P1-06 澄清 Transformer encoder 的输入形状

- 状态：`[ ] 待处理`
- 初始位置：Implicit Target Representation，`root.tex:235-252`；Multi-Horizon Effect Chain，`root.tex:254-271`
- 当前问题：公式把 `f_z(z_t)+f_x(x_bar_t)` 表示为单个向量，却对其使用 two-layer, four-head Transformer encoder；同时只保留一个 valid entity slot。读者无法判断 attention 在哪些 token 或 slots 上计算。
- 推荐处理：给出完整 tensor shape，并说明 masked entity slots 是否实际参与注意力。如果有效序列长度始终为 1，应解释使用 Transformer block 的实现原因，或用更准确的模块名称描述。
- 验收标准：公式、实现描述和网络结构图中的维度一致。
- 修改记录：

### P1-07 准确描述 effect 的参考坐标系

- 状态：`[ ] 待处理`
- 初始位置：Effect Schema，`root.tex:279-286`；Effect Labels，`root.tex:361-370`
- 当前问题：translation 和 velocities 表示在时刻 `t` 的 end-effector frame 中，rotation increment 则由目标自身当前坐标系定义。整体称为 “target-centric effect schema”不够准确。
- 推荐处理：逐分量说明参考系，并将总称改为 “structured target-motion schema” 或类似表述。
- 验收标准：每个三维量的参考系和时间基准均无歧义。
- 修改记录：

### P1-08 统一 effect-loss ablation 的评测条件

- 状态：`[x] 已完成`
- 初始位置：Baseline and Ablation，`root.tex:492-505`；Table 2，`root.tex:595-614`
- 当前问题：`lambda_E=0.3` 的 LIBERO evaluation batch size 为 20，其他设置为 10，但正文称只改变 loss weight。
- 推荐处理：最好统一 batch size 重跑；否则说明 batch size 是否改变环境随机数映射、数值结果或同步 rollout 行为。
- 验收标准：消融表中的差异可以合理归因于 `lambda_E`，或明确披露额外变量。
- 修改记录：2026-09-15 已确认原文中 `lambda_E=0.3` 使用 evaluation batch size 20 的说明属于未修正的文本错误；实际所有 LIBERO evaluations 均使用 batch size 10。已据此修正 Table 2 caption，消融评测条件一致，无需重跑。

### P1-09 将定性分析扩展到有代表性的 held-out 样本

- 状态：`[ ] 待处理`
- 初始位置：Qualitative Effect Inspection，`root.tex:661-724`
- 当前问题：目前只展示从提升最大的 Butter 任务中选取的一个训练样本；“deterministically selected”不能消除先选择任务带来的偏差。
- 推荐补充：
  - 至少一个 rescue task、一个 regression task、一个 RoboCasa task；
  - 使用 held-out observations；
  - 在正文明确样本选择规则；
  - 单样本结论统一限定为 “in this example”。
- 验收标准：定性证据不只展示最有利的训练案例，也不从单一样本推广到整体行为。
- 修改记录：

### P1-10 优化 Related Work 的比较逻辑

- 状态：`[ ] 待处理`
- 初始位置：`root.tex:132-177`
- 当前问题：当前结构较像逐篇摘要，尚未建立稳定的比较维度；引言第一段也较依赖 survey，而不是代表性原始工作。
- 推荐比较维度：
  - intermediate prediction 的对象；
  - 表示是否显式、结构化；
  - inference 是否需要额外输入；
  - 是否直接条件化 action policy；
  - 是否能进行数值干预与诊断。
- 推荐替换：
  > These methods do not explicitly supervise and expose a compact object-motion representation that conditions the action expert.
- 验收标准：每个 related-work 小节最后都明确指出本文与该类方法的实质差异，而不是笼统称现有系统为 direct mapping。
- 修改记录：

## 三、P2：逐节语言与结构润色

### P2-01 Abstract

- 状态：`[ ] 待处理`
- 初始位置：`root.tex:45-67`
- 修改要点：
  - “language-referred target” → “instruction-conditioned manipulation target”；
  - “compact physical future” → “compact representation of future target motion”；
  - 明确单训练 checkpoint 和三个 rollout seeds；
  - 若主结果为事后最优配置，应披露 benchmark-specific setting；
  - “consistent changes” 改成可直接由数据支持的表述。
- 推荐开头：
  > Most vision-language-action models map observations and language instructions directly to actions without explicitly representing the future motion of the manipulated entity. We introduce ArTE-VLA, which predicts a compact, multi-horizon representation of this motion and uses it to condition action generation.
- 修改记录：

### P2-02 Introduction

- 状态：`[ ] 待处理`
- 初始位置：`root.tex:72-128`
- 修改要点：
  - 将 direct mapping 的困难写得更具体，减少宽泛判断；
  - “less robust correlation” → “spurious visual or task correlations”；
  - 减少同一句中 survey 的堆叠，优先引用代表性原始论文；
  - 判断 replay-based labeling pipeline 是否足够构成独立贡献；若保留，应突出可迁移性或技术难点；
  - 第三项贡献中的 “outperforms” 与主结果统计口径保持一致。
- 修改记录：

### P2-03 Method 的行文压缩与术语统一

- 状态：`[ ] 待处理`
- 初始位置：`root.tex:195-405`
- 修改要点：
  - “effect chain” 与 “effect predictor”择一作为模块名称；
  - 将 entity-slot implementation legacy 等非核心细节移到附录；
  - 避免反复说明 “simulator state is not required at inference”，在摘要、方法开头和 supervision 小节各保留一次即可；
  - 用一致方式区分 `e_t^h`、预测值 `hat e_t^h` 和监督标签 `y_{t,h}`；
  - 概率分解与实际 deterministic regression/flow-matching 实现之间增加一句解释。
- 修改记录：

### P2-04 Results 的因果措辞

- 状态：`[ ] 待处理`
- 初始位置：`root.tex:518-586`
- 当前句：
  > target effects can resolve severe failures of a direct policy
- 推荐句：
  > Target-effect conditioning yields large gains on two tasks where the baseline performs poorly, while substantial regressions on Ketchup and Milk show that the benefit is not uniform.
- 其他替换：
  - “produces 380 rescues” 可保留为配对计数，但不要据此断言机制；
  - “smaller but more consistent aggregate gain” → “smaller aggregate gain distributed across four of five tasks”；
  - 表格均值之外增加 seed-wise variance 或 confidence intervals。
- 修改记录：

### P2-05 Discussion

- 状态：`[ ] 待处理`
- 初始位置：`root.tex:726-759`
- 修改要点：
  - 当前边界说明总体较好，建议保留其克制程度；
  - 将可能机制明确标为 hypotheses，并与计划补充的诊断实验对应；
  - 若 P0/P1 实验得到补充，应删去已经解决的 limitation；
  - 避免把所有未来工作连续堆在最后一句，可按 robustness、attribution、real-world labeling 三类组织。
- 修改记录：

### P2-06 Conclusion

- 状态：`[x] 已完成`
- 初始位置：`root.tex:762-795`
- 当前问题：重复了主结果数字、McNemar 检验、loss-weight sweep、intervention 和 limitations，篇幅偏长。
- 推荐结构：
  1. 一句话概括方法；
  2. 一句话概括最主要的跨 benchmark 发现；
  3. 一句话限定证据边界；
  4. 一句话说明最重要的下一步。
- 目标：压缩约三分之一，不再逐项重复实验结果。
- 修改记录：2026-09-15 已按“方法—主要结果—证据边界—下一步”重写并压缩 Conclusion。删除了重复的 rescue/regression 计数、McNemar `p` 值、完整 loss-weight sweep 和多轮限制复述；保留了单训练 checkpoint、三个 rollout seeds、任务间异质性以及 intervention 不能证明独立闭环收益等必要限定。

### P2-07 统一术语与格式

- 状态：`[x] 已完成`
- 全文检查项：
  - 将 “PI0.5” 统一为官方形式 `pi_{0.5}`，建议定义 LaTeX macro；
  - 统一 “multi-view”“four-horizon”“13-D”“12-D”的连字符；
  - 统一 “RoboCasa PickPlace-5”“five-task PickPlace family”“RoboCasa”三个称呼；
  - benchmark 的每处引用按现有方式保留，不做删减；
  - 统一 “agent view”“workspace view”“wrist view”“eye-in-hand view”的命名；
  - 检查 “target effect”“target-effect”“physical effect”的名词与定语用法。
- 修改记录：2026-09-15 已完成术语与格式统一：新增 `\pihalf` macro，将正文和表格中的 “PI0.5” 统一为官方数学形式；统一使用 “RoboCasa PickPlace-5”，并在数据部分说明该名称指五个 atomic PickPlace tasks；统一 LIBERO 定性图中的 workspace/wrist view 命名，同时保留 RoboCasa 的 agent/wrist (eye-in-hand) view 区分；修正 target-effect 作为复合定语时的连字符。按用户要求，LIBERO 和 RoboCasa 在正文各处原有的引用均予以保留。

### P2-08 调整部分不自然或不够精确的表达

- 状态：`[ ] 待处理`
- 建议替换：

| 当前表达 | 推荐表达 |
|---|---|
| language-referred target | instruction-conditioned manipulation target |
| a compact physical future | a compact representation of future target motion |
| how the target should change | how the manipulated entity changes over future horizons |
| produces consistent changes | produces measurable changes in all evaluated action chunks |
| target effects can resolve severe failures | target-effect conditioning yields large gains on tasks where the baseline performs poorly |
| smaller but more consistent aggregate gain | smaller aggregate gain distributed across four of five tasks |
| The effect chain first combines... | The effect predictor first combines... |
| Steady-state checkpoint intervals indicate approximate... | Training on eight NVIDIA H20 GPUs takes approximately... |

- 修改记录：

## 四、参考文献与排版

### P2-09 修复 BibTeX 条目类型和字段

- 状态：`[x] 已完成`
- 初始位置：`ref.bib:35-50`
- 当前警告：
  - `libero` 是 `@inproceedings`，却使用 `journal`；应改为合适的会议条目字段；
  - `cotvla` 是 `@article`，却使用 `booktitle`；应改为 `@inproceedings` 或使用正确的出版信息。
- 验收标准：BibTeX 编译无 empty journal/booktitle 警告。
- 修改记录：2026-09-15 用户已修正两个条目：`libero` 改为与 `journal` 字段匹配的 `@article`，`cotvla` 改为与 `booktitle` 字段匹配的 `@inproceedings`。重新检查 `root.blg` 后，原有的 empty journal/booktitle 警告均已消失。

### P2-10 保护参考文献标题中的专有名称

- 状态：`[x] 已完成`
- 初始位置：`ref.bib` 全文
- 检查对象：`{OpenVLA}`、`{CoT-VLA}`、`{LIBERO}`、`{RoboCasa}`、`{VLA}`、`{AI}`、`{3D}`、模型名称及缩写。
- 原因：IEEEtran bibliography style 可能自动改变未保护字符的大小写。
- 验收标准：最终 PDF 中所有专有名称和缩写大小写正确。
- 修改记录：2026-09-15 已在 `ref.bib` 中保护并校正模型名、benchmark 名和缩写的大小写，包括 `OpenVLA`、`LIBERO`、`CoT-VLA`、`WorldVLA`、`GR-2`、`Track2Act`、`VLM`、`VLA`、`AI`、`ACoT-VLA`、`FutureVLA`、`OCRA`、`3D`、`RoboCasa`、`IEEE/CVF`、`CVPR` 和 `ICRA`。重新运行 BibTeX 和 LaTeX 后，生成的 `root.bbl` 保留了预期大小写，且无 BibTeX、undefined reference 或 overfull box 警告。

### P2-11 最终编译和投稿格式检查

- 状态：`[ ] 待处理`
- 当前情况：没有 undefined citation/reference；日志存在少量 underfull box 警告。
- 提交前检查：
  - 将 `draftnotestrue` 切换为 `draftnotesfalse`；
  - 替换匿名作者块或按匿名投稿要求保留；
  - 对照最终 ICRA 2027 官方模板复核页码、页眉、页边距和 bibliography；
  - 检查表格、figure captions 和双栏浮动位置；
  - 清理 underfull boxes，但避免为了消除低风险 warning 破坏行文。
- 验收标准：从干净环境完整编译，引用、参考文献、图表和格式均无提交级错误。
- 修改记录：

## 五、推荐执行顺序

1. 决定主结果的超参数选择方案（P0-01）。
2. 确定是否补独立训练 seeds、聚类统计和 effect metrics（P0-02、P0-03、P0-06）。
3. 固定 target/effect 的定义与全文术语（P0-04、P0-05）。
4. 补充 controls、RoboCasa 数据和 rollout 细节（P1-01 至 P1-09）。
5. 重写摘要、贡献、结果与结论，使主张和最终证据一致。
6. 做全文逐句润色及术语统一。
7. 修复 bibliography，完成最终编译与格式检查。

## 六、总体进度

- P0：1 / 7 完成
- P1：1 / 10 完成
- P2：4 / 11 完成
- 当前阶段：待确定实验与主结果呈现方案
