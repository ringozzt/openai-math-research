# openai/math 调研：722 篇 AI 数学手稿是怎么来的

> OpenAI 把内部未发布模型的数学评估副产品整个端了出来：722 篇手稿、372 个结果家族，横跨 17 个数学分支，部分带 Lean 机器可验证证明，Apache-2.0。这是"AI 做研究级数学"第一次以**可审计**的形态大规模公开。

- 原仓库：[openai/math](https://github.com/openai/math)
- 调研日期：2026-10-07（仓库创建于 2026-10-06，调研时 3550 star / 314 fork）
- 方法：仓库 README、overview.pdf（41 页学科总览）、CONTENTS.md（9170 行手稿地图）、lean/ 形式化目录实测读取 + 8-10 月新闻时间线交叉验证

---

## 1. 硬数据

| 项 | 数 |
|---|---|
| 创建时间 | 2026-10-06（调研次日即 3550 star） |
| 体量 | 722 篇手稿，372 个结果家族，57,739 个文件 |
| 语言 | Lean（形式化部分）+ LaTeX/PDF（手稿） |
| 许可 | Apache-2.0（**整个仓库**，含证明） |
| 评估规模 | 约 4000 个问题投喂内部未发布模型 |
| 单结果成本 | 平均 3 小时 ChatGPT Pro 级别 thinking compute |
| 推理摘要 | 10 个家族的 abridged reasoning traces（PDF） |
| 验证状态 | **分阶段**：部分有 Lean 形式化，部分没有；官方原话"Some of the unformalized results could have issues" |

---

## 2. 这是什么：评估副产品转正

OpenAI 的原话逻辑很直白：做模型研发时拿开放研究问题做评估，旧的数学评测被刷爆（saturated）之后扩大了评估规模，模型在约 4000 个问题上产出了够分量的结果——于是把"评估副产品"整理成 722 篇手稿公开发布。

关键细节：
- 同一流程、同一未发布内部模型产出**绝大多数**结果（固定 procedure）。
- 例外两处单独点名：黎曼 zeta 零点自由区域的工作，以及 CM 阿贝尔簇 Hodge 猜想的证明——其中 Re(s) > 11/12 零点自由区域的写作为人类编辑过可读性。
- 按"结果家族"聚合：一个家族 = 主结果 + 配套论证/推论/ alternative 证明。372 个家族按数学分支分类。
- 版本史公开保留：修正作为新版本发布，旧版本可访问；每篇手稿目录自带 BibTeX。

## 3. 产出管线：4000 → 722

```
4000 个开放问题
  → 内部未发布模型（平均每结果 3h Pro 级 thinking）
    → 按显著性聚合为 372 个结果家族 / 722 篇手稿（preprints/，PDF+源码+引用）
      → 部分结果 Lean 形式化（lean/，对照 formalization.yaml）
        → ComparatorChallenges：JSON（题目）+ .lean（形式证明）对，
           用 leanprover/comparator 做机器核验
```

- `preprints/`：每篇手稿的 PDF、源码、手稿级引用与构建说明。
- `lean/`：Lean 形式化库（官方建议一次只编译小部分；整库编译可能撞上 Linux `vm.max_map_count` 上限，文档给了 mmap workaround——侧面说明形式化库体量之大）。
- `reasoning_traces/`：10 个家族的删减版推理摘要——模型"怎么想"的部分脱敏公开。
- `CONTENTS.md`：9170 行的手稿地图，每篇有摘要。
- `overview.pdf`：41 页学科总览。

## 4. 验证机制：Lean 是信任锚

这是整个发布最硬核的设计：**数学证明第一次有了"CI"**。

- Lean 4 + Mathlib（社区维护，200 万+ 行，35 万+ 已验证定理）：形式化代码能对着 Mathlib 编译通过，证明即成立——不需要信任 OpenAI 的话。
- `ComparatorChallenges/`：题目（JSON）与形式证明（.lean）成对存放，`lake env comparator ComparatorChallenges/xxx.json` 一键核验。这是 [leanprover/comparator](https://github.com/leanprover/comparator) 的标准挑战格式。
- 但注意官方的诚实声明：**不是所有手稿都有形式化**，未形式化的"could have issues"，修了会发新版本。信任是分级的：Lean formalized > 人类可读手稿 > 纯 claim。

## 5. 学科版图：17 个分支

按 overview.pdf，372 个家族分布在：数论、代数与复几何、实分析与复分析、凸与度量几何、理论计算机科学、动力系统与遍历论、组合数学、代数、概率与统计力学、数理逻辑、群论、数学物理、算子代数、拓扑、泛函分析、微分几何、偏微分方程。

## 6. 头部成果点名（选 10 个感受下密度）

- **003 准黎曼猜想**：所有 Dirichlet L 函数（含 ζ）在 Re(s) > 7/8 无零点；另有一个 Re(s) > 11/12 的独立证明。
- **032 CM 阿贝尔簇的有理 Hodge 猜想**：任意维数余维数成立，经由 Milne 定理顺带给出有限域上阿贝尔簇的 Tate 猜想。
- **002 完整 BSD 公式**：Selmer corank 0/1 的情形下 BSD 首项公式全成立（含 Tate–Shafarevich 群有限性）。
- **004 ℚ 上的 Hilbert 第十问题**：否定解决——无算法判定整系数多项式有无有理零点。
- **005 Catalan 常数无理性**；**017 π 的无理性指数恰为 2**（顺带证 Flint–Hills 级数收敛）。
- **087 Mahler 猜想**：对称与一般情形在所有维数解决。
- **074 Kakeya**：三维 maximal 猜想 + 四维 Hausdorff 维数猜想。
- **376 强制 Navier–Stokes 流中的通用计算**。
- **073 Falconer 距离猜想**：所有维数 d ≥ 2 解决。
- **102 半定阈值处的 NP-hardness**（理论 CS，推理摘要亦有收录）。

## 7. 开哪一层：开成果，闭模型

用"开哪一层"的框架看，这次发布开得很清楚：

- **开的是成果层**：722 篇手稿 + Lean 形式化证明，Apache-2.0，随便用。
- **闭的是模型层和算力层**：产出它们的"内部未发布模型"不在仓库里，3h Pro compute/结果的算力也不在。
- **Lean 证明是 portability seam**：因为机器可验证，成果脱离了"我信 OpenAI"的前提——任何人、任何模型都能独立复核。这是它和普通"论文预印本 dump"的本质区别。

## 8. 时间线：从 10 道题到 722 篇

- **2026-08-01**：OpenAI 发布 Astra（下一代模型家族）首秀——内部版 Astra 解出 10 个十年以上未解决的数学问题，附 Lean 证明，据报道 token 成本约 $2,000。Fields 奖得主 Gowers 表示会毫不犹豫地把其中一篇推荐给顶刊。
- **2026-08**：约 40 位数学家闭门会（Bryna Kra 等），讨论"模型解题比领域消化还快怎么办"；数学家们要求给论文而不是 blog 式宣发。
- **2026-09**：Navier–Stokes 署名争议（Buckmaster/Alpöge 独立工作 vs OpenAI）；群论结果的 Kun/Thom credit 纠纷（后修订致谢）。
- **2026-06-02**（前置）：Leiden Declaration——要求 AI 使用透明披露、独立验证、人类对正确性负责。
- **2026-10-06**：本仓库发布；同步成立数学与 AI 顾问组（Gowers、Hairer、Witten 等 9 人，无报酬、可公开批评、**无权干预研究节奏**）。

## 9. 给 agent 视角的三句话

1. **Autoformalization 是"AI 输出可被机器审计"的范式**：自然语言论证 → Lean 4 机器证明，审计成本从"请三个专家看半年"降到"跑一遍编译器"。这是 agent 可信度的通用解法，不只属于数学。
2. **Comparator = proof 的 CI**：题目与证明成对、可一键复验的仓库格式，是"可复现研究"在形式化时代的形状。
3. **评估饱和驱动发布**：旧评测刷爆 → 扩大评估到开放问题 → 副产品大到值得单独发布。能力曲线的外溢效应，下次可能发生在任何"评测先行"的领域。

---

## 引用

- 仓库 README（产出流程、722/372、4000 问题、3h compute、Apache-2.0）：[openai/math](https://github.com/openai/math)
- 学科总览与家族摘要：仓库内 `overview.pdf`
- 手稿地图：仓库内 `CONTENTS.md`
- 形式化目录：仓库内 `lean/formalization.yaml`、`lean/ComparatorChallenges/README.md`
- Astra 十题首秀与 $2,000 成本、Gowers 评价：[AI, Claudius](https://aiclaudius.com/article/openai-astra-ten-open-math-problems-lean-proofs-aug-2026)
- 数学家闭门会（Bryna Kra）：[AIDailyPost](https://aidailypost.com/news/openai-forms-advisory-group-amid-concerns)
- Navier–Stokes 署名争议：[The CyberSec Guru](https://thecybersecguru.com/news/openai-navier-stokes-proof/)、[NewsArchyUK](https://www.newsarchyuk.com/business/openai-forms-math-advisory-group/)
- Credit 纠纷与 Leiden Declaration：[The Daily Star](https://www.thedailystar.net/news/technology/news/openai-maths-advances-raise-questions-over-credit-and-access-4272681)
- 顾问组构成与无权说明：[NewsArchyUK](https://www.newsarchyuk.com/business/openai-forms-math-advisory-group/)
