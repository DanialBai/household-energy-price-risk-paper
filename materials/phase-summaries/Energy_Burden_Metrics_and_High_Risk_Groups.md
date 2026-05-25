# 能源负担指标与高风险分组：概念对照手册

本文档汇总项目中 **能源负担（energy burden）** 相关连续指标与 **高风险分组（high-risk group flags）** 的定义、区别、嵌套关系及分析用途。内容对应 `scripts/build_lsoa_income_distributions.py`、`scripts/epc_enrichment_and_high_risk_analysis.py` 及 `materials/LSOA_income_distribution_methodology.md`。

**核心原则（勿混淆）：**

| 概念层次 | 问的问题 |
|----------|----------|
| **Fuel poverty**（如 LILEE） | 当前是否已陷入官方定义的能源困难 |
| **Energy burden**（本文档） | 所需能源成本占可支配收入的比例或超阈概率 |
| **Household fuel price risk** | 未来燃料价格冲击下的暴露与适应能力（**尚未实现**） |

下文所有指标均为 **当前价格、当前 SAP 模型成本** 下的快照，**不包含** 未来价格冲击情景。

---

## 1. 共同基础

### 1.1 所需能源成本（共同分子）

所有负担指标共用：

```text
model_energy_cost_from_sap
```

由 SAP 能效评级与 `TOTAL_FLOOR_AREA` 反推（SAP Energy Cost Factor），表示 **required（所需）** 能源支出，而非实际账单或实际消费量。详见 `materials/LSOA_income_distribution_methodology.md` §4.4。

### 1.2 10% 负担阈值

英国经典燃料贫困文献中的 **10% affordability** 思想贯穿本项目：当所需能源成本占可支配收入超过 10% 时，负担压力显著。两个连续指标对该阈值的运用方式不同（见 §2）。

---

## 2. 连续指标：负担概率 vs 基线负担比例

### 2.1 `baseline_required_energy_burden`（基线所需能源负担）

**定义（确定性点估计）：**

```text
baseline_required_energy_burden = model_energy_cost_from_sap / Disposable annual income after housing costs
```

| 维度 | 说明 |
|------|------|
| **数学形式** | 单值比例（如 0.08 = 8%，0.15 = 15%） |
| **收入数据** | MSOA 点估计：`Disposable annual income after housing costs`（同 MSOA 内物业相同） |
| **不确定性** | 不建模 LSOA 内收入分布 |
| **缺失处理** | 收入缺失或 ≤ 0 → `NaN` |
| **分析角色** | **稳健性（robustness）** 指标 |
| **脚本位置** | `scripts/epc_enrichment_and_high_risk_analysis.py` |

**含义：** 在现行价格与 SAP 模型成本下，该物业所需能源支出占其所在 MSOA **税后、扣除住房成本后** 可支配收入的比例。

---

### 2.2 `burden_prob_gt_10`（负担超过 10% 的概率）

**定义（概率型）：**

```text
burden_prob_gt_10 = P( Y_l < model_energy_cost_from_sap / 0.10 )
                  = P( model_energy_cost_from_sap / Y_l > 10% )
```

其中 `Y_l` 为物业所在 LSOA 的对数正态收入 `LogNormal(μ_l, σ_l)`；`μ_l, σ_l` 由 MSOA 的 `λ` 校准，使 LSOA 模型燃料贫困率 `modeled_lsoa_poverty_rate` 贴近官方 `Proportion of households fuel poor`。

| 维度 | 说明 |
|------|------|
| **数学形式** | 0–1 概率（如 0.35 = 35% 概率超 10% 阈值） |
| **收入数据** | LSOA 对数正态分布（经 MSOA `λ` 与官方 LSOA 燃料贫困率对齐） |
| **不确定性** | 显式建模 LSOA 内收入异质性 |
| **分析角色** | **主分析** 指标；用于 LSOA 贫困率拟合与 `property_poverty_score` |
| **脚本位置** | `scripts/build_lsoa_income_distributions.py` |

**含义：** 在 LSOA 收入分布下，随机抽取一名住户，其所需能源负担超过收入 10% 的概率。

**关联列：**

- `property_poverty_score`：SAP 等级为 A/B/C 时取 0，否则等于 `burden_prob_gt_10`（极高能效物业在当前成本模型下构造上难以超 10% 阈值）。
- `modeled_lsoa_poverty_rate`：LSOA 内 `property_poverty_score` 的均值。

---

### 2.3 两连续指标对比

| 维度 | `baseline_required_energy_burden` | `burden_prob_gt_10` |
|------|-------------------------------------|---------------------|
| 形式 | 比例 \(C/Y\) | 概率 \(P(C/Y > 10\%)\) |
| 收入 | MSOA 点收入 | LSOA 对数正态 \(Y_l\) |
| 与官方燃料贫困 | 不校准 | 通过 LSOA 模型率校准 |
| 与 fuel price risk | 当前快照 | 当前快照 |

\(C\) = `model_energy_cost_from_sap`，\(Y\) = 可支配收入。

**为何会 diverge（同一物业两数不一致）：**

1. MSOA 平均收入 vs LSOA 收入分布尾部 → 概率高但点估计比例可能偏低。
2. 点估计未超 10%（如比例 0.09），但分布下仍有较高超阈概率（如 0.40）。
3. 后续高风险分组基于不同排序，住户集合不必重合。

**一句话：**

- **`baseline_required_energy_burden`**：这栋楼所需能源费占 MSOA 平均可支配收入多少？（确定、直观）
- **`burden_prob_gt_10`**：在 LSOA 收入分布下，随机住户负担超 10% 的概率多大？（分布化、可校准官方贫困率）

---

## 3. 高风险分组标记（0/1）

五个变量均为 **二元标记**，用于描述统计、机制路径分型（`scripts/high_risk_pathway_typology.py`）及 HR/BR 交叉稳健性检验。它们 **不是** 官方燃料贫困状态，也 **不** 反映未来价格风险。

**命名规律：**

| 前缀 | 含义 |
|------|------|
| **HR** | High-risk，基于 `burden_prob_gt_10`（**主分析**） |
| **BR** | Baseline burden robustness，基于 `baseline_required_energy_burden`（**稳健性**） |
| **TOP10 / TOP20** | 全样本有效值上的 **相对排名**（前 10% / 前 20%） |
| **GT_10PCT** | **绝对阈值**（负担比例 > 10%） |

**计算逻辑**（`add_risk_groups()` in `epc_enrichment_and_high_risk_analysis.py`）：

```text
# HR：基于 burden_prob_gt_10
t10 = quantile(burden_prob_gt_10, 0.90)
t20 = quantile(burden_prob_gt_10, 0.80)
HR_TOP10_BURDEN_PROB = 1  if burden_prob_gt_10 >= t10
HR_TOP20_BURDEN_PROB = 1  if burden_prob_gt_10 >= t20

# BR：基于 baseline_required_energy_burden
t10_br = quantile(baseline_required_energy_burden, 0.90)
t20_br = quantile(baseline_required_energy_burden, 0.80)
BR_TOP10_BASELINE_BURDEN     = 1  if baseline_required_energy_burden >= t10_br
BR_TOP20_BASELINE_BURDEN     = 1  if baseline_required_energy_burden >= t20_br
BR_GT_10PCT_BASELINE_BURDEN  = 1  if baseline_required_energy_burden > 0.10
```

分位数仅在 **非缺失** 记录上计算；底层指标缺失的物业，比较结果为 0（未标为高风险），故 TOP10/TOP20 实际占比可能略低于名义 10%/20%。

---

### 3.1 对照表

| 变量 | 底层指标 | 入选条件 | 预期规模* | 分析角色 |
|------|----------|----------|-----------|----------|
| `HR_TOP10_BURDEN_PROB` | `burden_prob_gt_10` | ≥ 全样本有效值 P90 | ≈ 10% | **主分析**（最紧） |
| `HR_TOP20_BURDEN_PROB` | 同上 | ≥ P80 | ≈ 20% | 主分析（更宽） |
| `BR_TOP10_BASELINE_BURDEN` | `baseline_required_energy_burden` | ≥ P90 | ≈ 10% | 稳健性（相对排名） |
| `BR_TOP20_BASELINE_BURDEN` | 同上 | ≥ P80 | ≈ 20% | 稳健性（更宽） |
| `BR_GT_10PCT_BASELINE_BURDEN` | 同上 | **> 0.10**（严格大于） | **不固定** | 稳健性（绝对 10% 线） |

\*名义占比；缺失处理见上文。

---

### 3.2 结构关系

```text
                    ┌─ HR_TOP10  (burden_prob ≥ P90)  ≈10%  【主分析·紧】
burden_prob_gt_10 ──┤
                    └─ HR_TOP20  (burden_prob ≥ P80)  ≈20%  【主分析·宽】

                              ┌─ BR_TOP10  (burden ratio ≥ P90)  ≈10%
baseline_required_energy_burden ─┼─ BR_TOP20  (burden ratio ≥ P80)  ≈20%
                              └─ BR_GT_10PCT (burden ratio > 10%)  规模不定
```

**组内嵌套（一定成立）：**

| 关系 | 说明 |
|------|------|
| `HR_TOP10` ⊆ `HR_TOP20` | P90 ≥ P80 |
| `BR_TOP10` ⊆ `BR_TOP20` | 同上 |
| `BR_GT_10PCT` ⊆ `BR_TOP10` | **不一定** — 刚超 10% 可能仍低于全城 P90 |
| `HR_TOP10` ⊆ `BR_TOP10` | **不一定** — 排序依据不同 |

**相对排名 vs 绝对阈值：**

- **TOP10 / TOP20**：样本量稳定在约 10%/20%，适合“谁最糟”排序与 archetype 对全样本 HR10 的贡献度。
- **`BR_GT_10PCT`**：人数随分布变化；可与政策 10% 叙事对话，但与 `burden_prob_gt_10` 的 10% 含义**不可直接等同**。

示例：若全城负担普遍偏低，可能几乎无人 `BR_GT_10PCT=1`，但仍有 10% 进入 `BR_TOP10`；若负担普遍偏高，`BR_GT_10PCT` 占比可能远超 10%。

---

### 3.3 HR 与 BR 的配对稳健性检验

管线 `build_overlap_table()` 计算以下重叠：

| HR 组 | BR 组 |
|-------|-------|
| `HR_TOP10_BURDEN_PROB` | `BR_TOP10_BASELINE_BURDEN` |
| `HR_TOP20_BURDEN_PROB` | `BR_TOP20_BASELINE_BURDEN` |
| `HR_TOP10_BURDEN_PROB` | `BR_GT_10PCT_BASELINE_BURDEN` |
| `HR_TOP20_BURDEN_PROB` | `BR_GT_10PCT_BASELINE_BURDEN` |

重叠高 → 概率排名与点估计排名一致；重叠低 → 收入分布假设显著改变“谁算高风险”。

---

## 4. 与论文三大概念及官方指标的边界

```text
model_energy_cost_from_sap
         │
         ├─→ burden_prob_gt_10 ──→ HR_TOP10 / HR_TOP20
         │      (LSOA 收入分布 + P(负担>10%))
         │
         └─→ baseline_required_energy_burden ──→ BR_TOP10 / BR_TOP20 / BR_GT_10PCT
                (MSOA 点收入比例)
```

| 概念 | 与本文档指标的关系 |
|------|-------------------|
| **Fuel poverty（LILEE 等）** | 不等价；不看官方“低收入 + 低能效”双条件 |
| **Energy burden** | `baseline_required_energy_burden` 为直接比例；`burden_prob_gt_10` 为超 10% 负担的概率化表达 |
| **Household fuel price risk** | 尚未建模；需价格冲击情景引擎，**不能**用上述指标替代 |

**文档与代码中的明确警示：**

- HR/BR 均为 **exploratory high-risk groups**，非官方燃料贫困认定。
- `baseline_required_energy_burden` 为 energy burden **稳健性** 指标，非 LILEE。
- 当前负担概率 **≠** 未来 household fuel price risk。

---

## 5. 分析选用速查

### 5.1 连续指标

| 研究问题 | 更合适指标 |
|----------|------------|
| 与官方 LSOA 燃料贫困率对齐、谁更可能超 10% | `burden_prob_gt_10` |
| 简单可解释的“账单压力占收入比例” | `baseline_required_energy_burden` |
| 检验收入分布假设是否改变排序 | 两指标及 HR vs BR 分组对比 |
| 未来气/电价上涨后的暴露 | 待 shock 模块；不可用上述指标 |

### 5.2 高风险分组

| 研究问题 | 更合适标记 |
|----------|------------|
| 论文主结果：当前模型下谁最可能高负担 | `HR_TOP10_BURDEN_PROB` |
| 更大样本、机制路径更稳 | `HR_TOP20_BURDEN_PROB` |
| 不依赖 LSOA 收入分布、MSOA 点收入 | `BR_TOP10` / `BR_TOP20` |
| 与 10% 政策阈值叙事对话 | `BR_GT_10PCT_BASELINE_BURDEN` |
| 检验 HR 结论稳健性 | HR vs BR 重叠表 + BR 三组互比 |
| 路径分型、archetype 贡献（现行管线） | 以 **HR10** 为主（见 `high_risk_pathway_typology.py`） |

---

## 6. 相关脚本与延伸阅读

| 资源 | 内容 |
|------|------|
| `scripts/build_lsoa_income_distributions.py` | `burden_prob_gt_10`、`property_poverty_score`、LSOA 校准 |
| `scripts/epc_enrichment_and_high_risk_analysis.py` | `baseline_required_energy_burden`、五个 HR/BR 标记、重叠表 |
| `scripts/high_risk_pathway_typology.py` | 以 HR10/HR20 为主的路径机制分型 |
| `materials/LSOA_income_distribution_methodology.md` | 收入分布与负担概率公式详述 |
| `materials/DP2.md` | fuel poverty vs price risk vs energy burden 概念辨析 |
| `AGENTS.md` | 项目核心概念与建模阶段状态 |

---

*文档版本：v1（汇总自负担指标与高风险分组概念对照讨论）。*
