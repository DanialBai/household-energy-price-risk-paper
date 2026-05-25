# Westminster 高风险家庭：Who Exposed? 与 Why Exposed? 综合分析总结

> 基于当前已完成的 high-risk explanation stage：5 个 Excel 结果表、`Energy_Burden_Metrics_and_High_Risk_Groups.md`、`2026-05-24_high-risk-results-synthesis.md`、`2026-05-24_daily-work-technical-flow.md` 及项目核心材料整理。

---

## 1. 总体判断

当前这轮工作完成的是 **“current high modelled energy-burden exposure” 的解释性结果层**，即回答：

1. **Who is currently exposed?** 目前哪些物业/家庭-住宅类型处于高模型能源负担风险？
2. **Why are they exposed?** 这些高风险是由哪些住房、收入、产权、供热、燃料和适应能力机制共同造成的？

需要强调的是：这还不是完整的 **future household fuel price risk** 模型。当前 HR10/HR20 不是官方 fuel poverty，也不是未来 fossil-fuel price shock 下的风险结果，而是基于当前 SAP 所需能源成本、LSOA 收入分布和基线能源负担构造出来的 **当前高负担概率分组**。

一句话概括当前贡献：

> Westminster 的高风险尾部不是由单一变量解释的，不是“低 EPC = 高风险”，也不是“低收入 = 高风险”，而是由 **需求强度、围护/能效弱点、收入敏感性、产权治理、电/气燃料暴露、以及低碳保护可及性约束** 共同构成。

这非常适合支撑论文主论点：

> Domestic net zero interventions are conditional de-risking tools, not automatic protection.

也就是说，净零技术是否能保护家庭，取决于它是否准确进入不同的风险路径，而不是取决于技术本身是否“低碳”。

---

## 2. 方法与数据质量判断

### 2.1 EPC enrichment 已经形成稳定 property-level backbone

技术流程为：

```text
Raw property master
    + EPC certificate records
    -> UPRN normalisation and EPC deduplication
    -> Left join EPC fields to property master
    -> EPC enrichment QA
    -> Derived EPC/fabric/fuel/tenure variables
    -> Current burden-risk group flags
    -> Descriptive and mechanism tables
    -> Priority-rule pathway typology
    -> Results synthesis for manuscript writing
```

关键 QA 结果：

| 项目 | 结果 | 含义 |
|---|---:|---|
| Enriched master 行数 | 124,980 | Westminster property-level 样本保持完整 |
| EPC 匹配率 | 100% | master properties 均匹配到 EPC certificate records |
| unmatched master rows | 0 | UPRN 标准化修正有效 |
| 新增/派生列 | 72 | EPC performance、成本、fabric、tenure、HR/BR 等均已落表 |
| EPC tenure 修正 | social-first classification | 避免 `rental (social)` 被误判为 private rented |

EPC tenure proxy 修正后的分布：

| Tenure proxy | Properties |
|---|---:|
| Private rented | 58,744 |
| Owner occupied | 43,686 |
| Social rented | 20,066 |
| Unknown | 2,484 |

这个 QA 很重要，因为它避免了后续结果被 merge error 或 tenure misclassification 破坏。尤其是 `EPC_TENURE_GROUP_FIXED` 的修正，使 social rented 从原本可能被误判为 private rented 的状态中分离出来。

### 2.2 当前指标边界清楚

当前结果使用两个核心连续指标：

| 指标 | 含义 | 角色 |
|---|---|---|
| `burden_prob_gt_10` | 在 LSOA 收入分布下，所需能源负担超过收入 10% 的概率 | 主分析 |
| `baseline_required_energy_burden` | SAP 模型所需能源成本 / MSOA 住房成本后可支配收入 | 稳健性指标 |

对应高风险分组：

| 组 | 定义 | 数量 | 占比 |
|---|---|---:|---:|
| HR10 | `burden_prob_gt_10` 前 10% | 12,500 | 10.00% |
| HR20 | `burden_prob_gt_10` 前 20% | 24,996 | 20.00% |
| BR10 | `baseline_required_energy_burden` 前 10% | 12,498 | 10.00% |
| BR20 | `baseline_required_energy_burden` 前 20% | 24,996 | 20.00% |
| BR > 10% | `baseline_required_energy_burden > 10%` | 737 | 0.59% |

HR10 与 BR10 重叠 7,128 个 property，占各自组的 57.0%。这说明概率化收入分布与确定性负担比例识别的是相关但不相同的高风险群体。换句话说，收入分布假设确实改变了“谁算高风险”。

---

## 3. Who exposed? 哪些物业/家庭-住宅类型最暴露？

### 3.1 最强信号：大面积 × 低 EPC 的交互

结果中最强的 descriptive signal 是 **floor area 与 EPC performance 的交互**。不是低 EPC 单独导致最高风险，而是低 EPC 在高热需求/大面积住宅中被放大。

| 分组 | Property count | HR10 share | HR20 share | Median baseline burden |
|---|---:|---:|---:|---:|
| EPC E-F-G 且面积 >120m² | 2,144 | 69.68% | 85.21% | 7.26% |
| House 且 EPC E-F-G | 1,842 | 56.08% | 71.12% | 6.25% |
| House 且 EPC D | 5,153 | 30.82% | 51.97% | 4.45% |
| Maisonette 且 EPC E-F-G | 978 | 34.66% | 58.49% | 4.78% |
| Flat 且 EPC E-F-G | 9,052 | 19.38% | 34.48% | 3.68% |

这说明：

> “低能效”本身不是足够解释变量；低能效在高需求建筑形态中才最危险。

EPC E-F-G 的大面积住宅几乎构成高风险尾部的核心：HR10 share 接近 70%，HR20 share 超过 85%。

### 3.2 HR10 的高风险 profile

HR10 与 non-HR10 的连续变量差异明显：

| 变量 | HR10 中位数/均值 | 非 HR10 中位数/均值 | 解释 |
|---|---:|---:|---|
| `TOTAL_FLOOR_AREA` | mean 114.3m²；median 87.3m² | mean 70.0m²；median 60.0m² | 高风险物业明显更大 |
| `SAP` | mean 56.0；median 58.5 | mean 67.7；median 70.0 | 高风险物业能效更差 |
| `model_energy_cost_from_sap` | mean £2,249；median £1,846 | mean £1,146；median £1,004 | 所需能源成本几乎翻倍 |
| `baseline_required_energy_burden` | mean 5.6%；median 5.11% | mean 2.49%；median 2.27% | 当前负担压力明显更高 |
| `EPC_HEATING_COST_SHARE` | mean 73.1%；median 75.3% | mean 65.9%；median 68.2% | 高风险更由供热成本驱动 |

这支持一个重要判断：

> Westminster 的 current high-burden exposure 首先是 required heat demand problem，其次才是一般性的“账单贵”问题。

### 3.3 分类变量特征

HR10 中 house、maisonette、E/F/G、top-floor flat 明显过度代表：

| 变量 | HR10 中占比 | 非 HR10 中占比 | 差异 |
|---|---:|---:|---:|
| EPC E-F-G | 25.01% | 7.78% | +17.23 pp |
| EPC D | 40.00% | 31.55% | +8.45 pp |
| EPC A/B/C | 34.99% | 60.67% | -25.68 pp |
| House | 25.78% | 6.40% | +19.39 pp |
| Maisonette | 13.05% | 4.96% | +8.09 pp |
| Flat | 61.17% | 88.64% | -27.47 pp |
| floor area >120m² | 33.79% | 9.48% | +24.31 pp |
| Top-floor flat | 29.51% | 17.76% | +11.76 pp |
| Weak fabric flag | 87.57% | 82.11% | +5.46 pp |
| Electricity exposure | 20.70% | 16.84% | +3.86 pp |

这说明 HR10 的“谁暴露”不是普通 flats，而是几类特殊组合：

- 大面积 house；
- 低 EPC maisonette；
- top-floor flat；
- 电加热低效单元；
- social rented blocks；
- 收入/剥夺驱动的 residual group。

---

## 4. Why exposed? 六条高风险路径的机制解释

当前最有论文价值的输出是 **P1–P6 pathway typology**。它把“谁暴露”转成了“为什么暴露”。路径分型是互斥的 priority-rule taxonomy，不是因果模型；它的作用是为后续 shock/intervention model 提供分组结构。

### 4.1 Pathway overview

| Pathway | Stock share | HR10 count | Pathway 内 HR10 share | 对全部 HR10 贡献 |
|---|---:|---:|---:|---:|
| P1: Inefficient large owner-occupied gas houses | 2.24% | 1,408 | 50.25% | 11.26% |
| P2: Social rented estate/block risk | 5.61% | 1,590 | 22.66% | 12.72% |
| P3: Private rented weak-fabric gas flats | 27.74% | 1,450 | 4.18% | 11.60% |
| P4: Electric-heated inefficient flats/maisonettes | 8.21% | 1,418 | 13.83% | 11.34% |
| P5: High-demand inefficient maisonettes | 1.78% | 573 | 25.81% | 4.58% |
| P6: Other high-risk residual group | 10.45% | 6,061 | 46.40% | 48.49% |

这里必须区分两种排序：

| 排序问题 | 结果 | 含义 |
|---|---|---|
| 哪些 pathway 内部风险最高？ | P1、P5、P2 | 小而强，适合精准深改/定向治理 |
| 哪些 pathway 对全体 HR10 贡献最大？ | P6、P2、P3、P4、P1 | 规模型问题，适合政策组合与空间治理 |

这正是结果的理论亮点：

> Policy targeting 不能只看 within-group risk，也不能只看 group size。

小群体可能风险强度极高，大群体可能单户风险不高但总体贡献很大。

---

## 5. 六条路径的详细解释

### 5.1 P1：低效大面积 owner-occupied gas houses

P1 只有 2.24% stock share，但 pathway 内 HR10 share 达到 50.25%，HR20 share 达到 68.17%。它的机制是：

```text
大面积 + house + gas + weak fabric + EPC D/E-F-G
    -> required heat demand 极高
    -> 当前模型能源负担概率高
```

P1 的理论意义在于挑战了 “owner-occupied = 高 adaptive capacity” 的直觉。即使业主持有产权，也可能因为 heritage constraints、planning restrictions、capital cost、施工扰动、老建筑复杂性而无法迅速去风险。

**政策含义：** P1 需要 fabric-first deep retrofit、heritage-sensitive retrofit、低息融资/补贴、规划许可协调，而不是只靠信息建议或普通能效宣传。

### 5.2 P2：social rented estate/block risk

P2 stock share 为 5.61%，贡献全部 HR10 的 12.72%，pathway 内 HR10 share 为 22.66%，HR20 share 为 41.23%。P2 的收入 profile 很突出：约 53.45% 位于 AHC income Q1_lowest，约 54.16% 位于 income deprivation Q4_high_deprivation。

核心机制不是单户是否愿意 retrofit，而是：

- social landlord；
- estate-level retrofit sequencing；
- block heating/system governance；
- funding programme；
- tenant protection。

**政策含义：** P2 应优先用于 social housing retrofit programme、estate/block-level heat decarbonisation、租户保护、施工期支持、租后账单保护评估。

### 5.3 P3：private rented weak-fabric gas flats

P3 是最大路径，占总 stock 的 27.74%，但 pathway 内 HR10 share 只有 4.18%；即便如此，它仍贡献全部 HR10 的 11.60%。这说明 P3 是典型的 **scale pathway**。

P3 的机制是：

```text
私人租赁 + flats + weak fabric + gas + 中等 EPC
    -> 单户风险未必最高
    -> 但因 stock 很大，对 HR10 贡献显著
```

P3 提醒我们：EPC A/B/C 或 D 不一定表示没有风险，因为 component-level fabric weakness、top-floor exposure、租户改造权不足、房东-租户 split incentives 可能共同制造风险。

**政策含义：** P3 不适合只做深度补贴给少数最差住宅，而要靠 MEES enforcement、landlord regulation、租户保护、批量低成本 fabric improvements、私租 sector targeting。

### 5.4 P4：electric-heated inefficient flats/maisonettes

P4 stock share 为 8.21%，贡献全部 HR10 的 11.34%；pathway 内 HR10 share 13.83%，HR20 share 24.90%。它是论文中最重要的 **electrification caution**。

它说明：

> “用电”并不自动代表去风险。

低效电加热 flats/maisonettes 在现有 GB electricity-gas pass-through 和高电价结构下，可能直接暴露于电价冲击。若没有 fabric improvement、tariff reform、flexibility access，electrification 不能被写成自动 de-risking。

**政策含义：** P4 后续干预模拟必须同时测试 fabric retrofit + clean heat + smart tariff/flexibility，而不是只把 heating fuel 从 gas 迁移到 electricity。

### 5.5 P5：high-demand inefficient maisonettes

P5 stock share 只有 1.78%，但 pathway 内 HR10 share 达到 25.81%，HR20 share 达 45.59%。它不是最大贡献者，却是风险强度较高的典型形态。

Maisonette 的机制在于：

- 介于 flat 与 house 之间；
- 可能有较多 exposed surfaces；
- 上下层热损失复杂；
- tenure / leasehold 责任边界复杂；
- retrofit 技术路径不标准。

**政策含义：** P5 需要被单独成类，而不能被简单并入 flat 或 house。后续 retrofit package 应测试 loft/roof/floor/party wall/controls 等组合，并加入 tenure/leasehold feasibility。

### 5.6 P6：Other high-risk residual group

P6 是最重要的 residual group：占 stock 10.45%，但贡献全部 HR10 的 48.49%，并且 HR20 share 为 100%。这说明 P1–P5 的住房路径只能解释约一半 HR10，剩下近一半高风险来自更复杂的收入、地理、混合住房机制。

P6 的 income profile 非常强：

| 指标 | P6 特征 |
|---|---:|
| AHC income Q1_lowest | 约 59.60% |
| Income deprivation Q4_high_deprivation | 约 68.22% |
| Flats | 约 67.30% |
| Houses | 约 24.47% |
| EPC A/B/C | 约 58.91% |
| EPC D | 约 25.73% |
| EPC E-F-G | 约 15.36% |

这说明 P6 不全是“差房子”，而是 **收入敏感性和空间剥夺把中等住房也推入高风险尾部**。

**政策含义：** P6 可能是 social tariff、standing charge reform、automated bill support、income support、targeted local assistance 最相关的组，而不一定主要靠技术 retrofit 解决。

---

## 6. “暴露—敏感性—适应能力”机制框架

当前结果可以映射到论文的概念框架：

```text
Household fuel price risk = exposure + sensitivity - adaptive capacity
```

但目前测到的是 current burden exposure / sensitivity / adaptive-capacity proxies，不是 shock-realised future risk。

### 6.1 Exposure：暴露

当前 exposure 机制包括：

| 机制 | 证据 |
|---|---|
| 高 required energy demand | HR10 的 mean model energy cost £2,249，非 HR10 为 £1,146 |
| 大面积放大低能效 | E-F-G 且 >120m² 的 HR10 share 69.68% |
| 供热成本主导 | HR10 heating cost share mean 73.1% |
| gas dependence | HR10 中 gas 79.10% |
| electricity exposure | P4 100% electricity exposure，HR10 中 electricity exposure 占 20.70% |
| weak fabric | HR10 中 weak fabric 87.57% |

这说明当前的“暴露”不是单一燃料维度，而是：

```text
热需求 × fabric × fuel × dwelling form
```

的复合暴露。

### 6.2 Sensitivity：敏感性

敏感性最强体现在 P2 与 P6：

- P2 是 low-income social rented block pathway；
- P6 是 residual income/deprivation-driven tail。

特别是 P6，将近 60% 在最低 AHC income quartile、超过 68% 在最高 income deprivation quartile。说明即使房屋条件不是最差，低剩余收入也会把家庭推入高负担概率尾部。

Westminster 作为富裕 borough 的风险，不应被写成“贫困地区燃料贫困”，而应写成：

> high-cost city 中 residual income squeeze 与 housing stock exposure 的叠加。

### 6.3 Adaptive capacity：适应能力

当前 `EPC_TENURE_GROUP_FIXED` 只是 proxy，但已经揭示了 adaptive capacity 的差异：

| Tenure / form | 风险机制 |
|---|---|
| Owner-occupied large houses | 有产权但可能受 capital cost、heritage、planning、disruption 限制 |
| Social rented blocks | 租户不能单户改造，依赖 landlord/estate programme |
| Private rented flats | split incentives，MEES enforcement，tenant power 弱 |
| Electric-heated flats/maisonettes | 需要 tariff/flexibility/smart meter/access，不只是技术替换 |
| Maisonettes/block flats | retrofit responsibility 和物理边界复杂 |

---

## 7. 稳健性与解释边界

### 7.1 HR 与 BR 的差异是优点，不是问题

HR10/HR20 是主结果，因为它们使用 LSOA income distribution 下的 `P(burden > 10%)`。BR10/BR20 是 robustness，因为它们使用 MSOA 点收入的 deterministic burden。两者重叠 57%/61.5%，说明它们识别的高风险群体既相关又不完全相同。

这可以写成方法贡献：

> 引入收入分布后，高风险识别不再只由物业成本排序决定，而是同时反映 local income tail exposure。

### 7.2 BR > 10% 很小，说明绝对 10% 阈值不足以描述 Westminster 的相对风险尾部

只有 737 个 property 的 baseline required energy burden >10%，占 0.59%。但 HR10 仍有 12,500 个 property。这不是矛盾，而是两个问题不同：

- `BR_GT_10PCT` 回答：谁已经超过经典 10% burden line？
- `HR_TOP10` 回答：在 Westminster 当前样本中，谁处于最高负担概率尾部？

因此论文中不能把 HR10 写成 “10% burden fuel poor households”。更准确写法是：

> top-decile modelled burden-risk group under current required-cost and income-distribution assumptions.

### 7.3 EPC tenure 是 proxy，需要 caveat

EPC tenure 来自证书字段，可能滞后于当前 occupancy。修正后的 `EPC_TENURE_GROUP_FIXED` 可用于分组解释，但论文中应明确写成 “tenure proxy”，不能写成 verified household tenure。

### 7.4 当前阶段不能替代 price shock

当前模块支持基线负担概率化表达，但完整风险框架中的 shocked burden、risk uplift、Household Fuel Price-at-Risk、de-risking dividend 都仍待建立。

---

## 8. 可直接进入论文 Results 的主线

### Result 1：Westminster high-risk tail is heterogeneous

不是一种 household type，也不是一种 dwelling type。HR10 由 large inefficient gas houses、social rented blocks、PRS weak-fabric flats、electric-heated inefficient flats/maisonettes、high-demand maisonettes 和 income/deprivation residual group 共同构成。

### Result 2：Demand intensity × low energy performance produces the strongest within-type risk

EPC E-F-G 且 >120m² 的 HR10 share 为 69.68%；House + E-F-G 的 HR10 share 为 56.08%。这说明最强的 current burden exposure 来自 high heat demand 与 poor fabric/efficiency 的叠加，而不是单独的 EPC rating。

### Result 3：Tenure changes the route to protection

Owner occupation、social renting、private renting 对应不同 adaptive capacity。P1 的问题是 capital/heritage/planning；P2 是 landlord/estate/block programme；P3 是 split incentives 和 PRS enforcement。tenure 不是简单 vulnerability label，而是决定 “who can act” 的治理机制。

### Result 4：Electrification is not automatic de-risking

P4 说明电加热低效 flats/maisonettes 已经处于高风险尾部的一部分。若未来 clean heat/electrification 仍受高电价、gas-linked electricity pass-through、缺乏 smart tariff/flexibility 的影响，电气化可能先减排但未必先降 household risk。

### Result 5：Nearly half of HR10 is residual and income/deprivation-linked

P6 贡献 48.49% HR10，并且强烈集中于 lowest AHC income 与 highest income deprivation。这说明“房屋技术解释”只能解释约一半当前高风险尾部；另一半需要 income sensitivity、spatial deprivation、possibly overcrowding/underheating/benefits/arrears 等后续变量进一步解释。

---

## 9. 论文表述建议

### 9.1 English version

> The current high-burden tail in Westminster is not produced by a single housing deficit. Rather, high modelled burden probability emerges through multiple pathways combining required heat demand, dwelling energy performance, fuel exposure, tenure governance, income sensitivity and constrained adaptive capacity. The strongest within-type risks are found among large inefficient dwellings, especially EPC E-F-G properties above 120m² and inefficient houses, while major contributions to the overall HR10 group also come from social rented estate/block risks, private rented weak-fabric gas flats, and electric-heated inefficient flats/maisonettes. Nearly half of HR10 properties remain in a residual pathway strongly associated with low after-housing-cost income and local income deprivation, indicating that future de-risking analysis must separate housing-driven and income-driven exposure.

### 9.2 中文版本

> Westminster 当前高负担尾部并非由单一住房缺陷产生，而是由所需供热需求、建筑能效、燃料暴露、产权治理、收入敏感性与有限适应能力共同形成。最高的组内风险集中在大面积低效住宅，尤其是 EPC E-F-G 且超过 120m² 的物业和低效 houses；但从全体 HR10 的贡献来看，social rented estate/block risk、private rented weak-fabric gas flats、电加热低效 flats/maisonettes 也构成相近规模的高风险来源。接近一半 HR10 仍位于 residual pathway，并与低住房成本后收入和高收入剥夺显著相关，这说明后续去风险建模必须区分 housing-driven 与 income-driven exposure。

---

## 10. 下一步最值得做的分析

### 10.1 先拆 P6，再做 price shock

P6 太大，贡献 48.49% HR10，不能继续作为一个黑箱 residual。建议拆成四类：

| P6 子类 | 识别逻辑 | 目的 |
|---|---|---|
| P6a income-driven | Q1_lowest AHC income + 不低 EPC/中低成本 | 判断收入政策/social tariff 作用 |
| P6b geography-driven | 高 deprivation LSOA/MSOA concentration | 判断空间靶向 |
| P6c mixed-mechanism | 低收入 + weak fabric + moderate/high cost | 判断 combined package |
| P6d unusual dwelling-profile | atypical fuel/form/size combinations | 检查数据或特例机制 |

### 10.2 建立 grouped attribution model

下一步不要立刻跳到 SHAP。建议先用 grouped logistic regression 或 regularised logistic model，以 HR10/HR20 为 outcome，分 blocks 输入：

1. dwelling demand：floor area、property type、built form；
2. fabric/performance：SAP、EPC band、weak fabric、efficiency gap；
3. fuel/heating exposure：main fuel、electricity exposure、mains gas；
4. tenure/adaptive capacity：fixed tenure、top-floor/block/maisonette proxies；
5. income sensitivity：AHC income band、income deprivation；
6. geography：LSOA/MSOA fixed effects 或 spatial summaries。

再用 random forest/SHAP 作 supplementary non-linear check。这样论文方法更稳，也能避免把机器学习解释写成因果推断。

### 10.3 先做地图，再做 intervention

在 price shock 前，建议先生成 6 张核心地图：

1. LSOA HR10 share；
2. LSOA HR20 share；
3. dominant pathway among HR10；
4. P2 concentration；
5. P4 concentration；
6. P6 residual concentration。

这些地图会直接服务论文叙事：

> Westminster 的风险不是平均分布，而是由不同机制在不同空间上集中。

### 10.4 Shock engine 应按 pathway 汇报

后续 price shock scenarios 至少应按 P1–P6 输出：

| Shock / intervention | 预期最敏感路径 |
|---|---|
| Gas shock | P1、P3、P5、部分 P2/P6 |
| Electricity shock | P4、电加热 archetypes |
| Gas-to-electricity pass-through | P4 + future electrification candidates |
| Standing charge shock | 小面积/低用量/低收入组，可能 P2/P6 |
| Social tariff / bill support | P2、P6 |
| Fabric retrofit | P1、P5、E-F-G large area groups |
| PRS regulation | P3 |
| Block / estate programme | P2 |
| Tariff / flexibility | P4 |

---

## 11. 最重要内容总结

1. **你已经完成了一个高质量的 who/why exposure explanatory layer。** 它可以支撑 Results 的第一阶段，但应明确这是 current modelled burden-risk，而不是完整 future fuel price risk。

2. **Westminster 高风险不是单一群体。** P1–P6 显示至少六条路径：大面积低效业主 gas houses、社会住房 block 风险、私租 weak-fabric gas flats、电加热低效 flats/maisonettes、高需求 maisonettes、收入/剥夺残余高风险。

3. **最大 within-pathway risk 与最大 HR10 contribution 不同。** P1、P5、P2 是强度型高风险；P6、P2、P3、P4 是贡献型高风险。政策不能只按强度或只按数量靶向。

4. **最强物理机制是 floor area × EPC performance。** EPC E-F-G 且 >120m² 的 HR10 share 达 69.68%，说明低能效在高热需求物业中被显著放大。

5. **收入敏感性不可忽视。** P6 贡献接近一半 HR10，且强烈集中在最低 AHC income 与最高 income deprivation，说明住房技术变量不能完全解释高风险尾部。

6. **产权是 adaptive capacity 机制，不只是人口标签。** Owner-occupied 也可能因资金、规划和 heritage 约束无法去风险；social rented 依赖 landlord/block programme；PRS 受 split incentives 影响。

7. **电气化不是自动去风险。** P4 证明电加热低效 flats/maisonettes 已是当前高风险的一部分；后续必须把 fabric、tariff reform、flexibility 和 electricity/gas price ratio 纳入 clean heat 评估。

8. **下一步应先拆 P6、做 attribution、出空间图，再做 price shock 和 intervention simulations。** 只有这样，第三问 “Can low-carbon interventions de-risk this exposure?” 才能真正回答 “who receives protection and who remains exposed”。

---

## 12. 建议文件用途

这个 Markdown 文件可以作为：

- Results section 中文研究备忘录；
- 后续英文 Results 初稿的结构骨架；
- 与 Cursor/Codex 协作时的高风险解释层 reference note；
- 进入 price shock engine 前的 analytical checkpoint；
- 后续地图、归因模型和 intervention simulation 的设计依据。

