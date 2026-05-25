# 英格兰 Household Fuel Price Exposure and Risk 研究框架与计划

## 0. 研究核心定位

本研究以英格兰官方 fuel poverty 作为静态 baseline，进一步构建一个对 fuel price、household energy system 和 net-zero retrofit 动态敏感的 **Household Fuel Price Exposure and Risk Framework**。

一句话概括：

> 以官方 fuel poverty 作为静态 baseline，在 property level 构建一个对 fuel price、energy system 和 net-zero retrofit 动态敏感的 household fuel price exposure and risk framework，并解释风险来源及可缓解路径。

研究的核心区别是：

- **Fuel poverty** 判断家庭是否处于燃料贫困状态；
- **Fuel price exposure / risk** 判断家庭在能源价格变化下，账单压力有多高、对价格有多敏感、哪些技术路径能够降低这种脆弱性。

---

## 1. 关键概念修正

### 1.1 Fuel poverty 中的 “low” 应理解为 low energy efficiency

英格兰官方 LILEE 指标中的两个核心条件是：

1. 低收入；
2. 低能效，而不是低能耗。

更准确地说，fuel poverty 是由 **low income + low energy efficiency / high required fuel costs** 共同决定。

官方 LILEE 逻辑为：

- 家庭居住在低能效住宅中，通常对应 FPEER band D/E/F/G；
- 在支付 required fuel costs 后，家庭的等效 after housing costs income 低于官方贫困线。

因此，本研究中使用：

```text
EPC band below C
```

作为低能效住宅的 proxy 是合理的，但需要说明它不是官方 FPEER 的完全等价物。

### 1.2 EPC < C 是 FPEER < C 的 proxy，而不是完全复制

建议论文中表述为：

> 本研究以 EPC band below C 作为 FPEER below C 的 property-level proxy，用于构建可大规模应用的 fuel poverty baseline；该 proxy 不完全复制官方 FPEER，但能捕捉主要的 building efficiency dimension。

### 1.3 £19,009 的使用方式

你提到的 £19,009 应被精确表述为：

> For the 2025 fuel poverty setting, the income poverty line is taken as £19,009, corresponding to 60% of the national median equivalised AHC income in the official 2025 fuel poverty dataset.

不要将 £19,009 写成跨年份固定贫困线，而应明确其为 2025 fuel poverty setting 下的贫困线。

---

## 2. 研究主线：从 Static Fuel Poverty 到 Dynamic Fuel Price Risk

本研究可以分成四个逻辑层次：

```text
Fuel Poverty Baseline
        ↓
Fuel Price Exposure
        ↓
Fuel Price Risk
        ↓
Net-zero Risk Mitigation
```

---

## 3. Layer 0：Fuel Poverty Baseline

### 3.1 目的

首先构建一个 property-level fuel poverty proxy，用于回答：

> 在官方 fuel poverty 逻辑下，哪些房屋或家庭会被归为 fuel poor？

### 3.2 指标定义

对 property / household proxy \(h\)：

\[
FP_h = 1(EPC_h < C) \times 1(Y_l - RFC_h < PL)
\]

其中：

- \(h\)：property 或 household proxy；
- \(l\)：property 所在 LSOA；
- \(Y_l\)：该 LSOA income scenario 下的等效 AHC disposable income；
- \(RFC_h\)：required fuel costs；
- \(PL = £19,009\)：2025 低收入贫困线；
- \(EPC_h < C\)：EPC band D–G。

### 3.3 该层的作用

这一层不是最终目标，而是 baseline。

其作用是：

- 对齐官方 fuel poverty 定义；
- 为后续 exposure / risk 指标提供比较对象；
- 识别哪些家庭被官方 fuel poverty 捕捉；
- 进一步识别哪些家庭虽然不属于 fuel poor，但仍对 fuel price highly exposed。

---

## 4. Layer 1：Fuel Price Exposure

### 4.1 研究问题

> 哪些 households / properties / areas 的 required energy bill 占 after housing costs disposable income 的比例最高？

### 4.2 指标定义

\[
Exposure_{h,t} = \frac{Bill_{h,t}}{Y_l}
\]

其中：

- \(Bill_{h,t}\)：在时间 \(t\)、价格情景和技术组合下的 required energy bill；
- \(Y_l\)：LSOA level income distribution 或代表性 income percentile。

### 4.3 Bill 的动态建模

不建议直接使用 EPC 中的 annual fuel cost 作为最终 bill，因为 EPC fuel cost 对价格年份、tariff、系统效率和低碳技术组合不够敏感。

更合适的做法是使用你的 **household energy system simulator** 重新计算：

\[
Bill_{h,t} =
p^{gas}_t Q^{gas}_{h,t}
+
p^{elec}_t Q^{elec,import}_{h,t}
+
SC^{gas}_t I^{gas}_h
+
SC^{elec}_t
-
p^{export}_t Q^{export}_{h,t}
\]

其中：

- \(p^{gas}_t\)：gas unit price；
- \(p^{elec}_t\)：electricity unit price；
- \(SC^{gas}_t\)：gas standing charge；
- \(SC^{elec}_t\)：electricity standing charge；
- \(Q^{gas}_{h,t}\)：gas demand；
- \(Q^{elec,import}_{h,t}\)：electricity import；
- \(Q^{export}_{h,t}\)：PV export；
- \(I^{gas}_h\)：是否保留 gas meter。

---

## 5. Layer 2：Fuel Price Risk

Exposure 是某一价格情景下的账单压力；risk 则进一步衡量在价格波动或收入不确定性下超过风险阈值的概率和程度。

### 5.1 Threshold risk

\[
Risk^{threshold}_{h,t,\theta} = P(Exposure_{h,t} > \theta)
\]

可选阈值：

- 10%；
- 15%；
- 20%。

其中 10% 具有较强政策解释力，因为 fuel affordability literature 中常使用 energy costs over income 的比例作为 affordability burden 指标。

### 5.2 Price sensitivity

Gas price sensitivity：

\[
\frac{\partial Exposure_h}{\partial p^{gas}} = \frac{Q^{gas}_h}{Y_l}
\]

Electricity price sensitivity：

\[
\frac{\partial Exposure_h}{\partial p^{elec}} = \frac{Q^{elec,import}_h}{Y_l}
\]

该指标可以回答：

> 一个 household 是 gas-price exposed，electricity-price exposed，还是 standing-charge exposed？

### 5.3 Extreme price risk

可进一步定义：

\[
VaR_{\alpha}(Exposure_h)
\]

\[
CVaR_{\alpha}(Exposure_h)
\]

用于衡量历史价格波动、energy crisis shock 或 geopolitical shock 下的 exposure 上尾风险。

---

## 6. Layer 3：Net-zero Risk Mitigation

### 6.1 核心问题

> 哪些 fabric retrofit 和 low-carbon technologies 在什么 household / property archetypes 中，能降低 exposure、降低 price risk，同时减少 CO₂？

### 6.2 你的两个模型的作用

#### 模型一：多模态 property 能效预测模型

用于模拟：

- wall insulation；
- roof insulation；
- window upgrade；
- floor insulation；
- fabric package improvement。

输出：

- required energy consumption；
- CO₂ emission；
- EPC / SAP proxy；
- bill change input。

#### 模型二：Household Energy System Simulator

用于模拟不同技术组合：

- gas boiler；
- heat pump；
- direct electric heater；
- PV；
- battery storage；
- EV；
- electricity tariff；
- export tariff。

输出：

- annual gas demand；
- annual electricity import；
- electricity export；
- annual energy bill；
- CO₂ emission；
- price sensitivity；
- technology-specific exposure change。

---

## 7. Income Modelling Framework

## 7.1 数据来源

收入建模使用两个主要数据源：

1. **ONS MSOA level equivalised disposable income after housing costs**
   - mean income；
   - upper confidence limit；
   - lower confidence limit。

2. **IoD25 Income Deprivation score**
   - LSOA level lower-tail deprivation information；
   - 用作低收入尾部 soft constraint。

### 7.2 建议模型：Soft-constrained Lognormal Income Distribution

对每个 LSOA \(l\)，设：

\[
Y_l \sim Lognormal(\mu_l, \sigma_l^2)
\]

则：

\[
E(Y_l)=\exp(\mu_l+\frac{1}{2}\sigma_l^2)
\]

\[
P(Y_l<PL)=
\Phi\left(\frac{\ln(PL)-\mu_l}{\sigma_l}\right)
\]

\[
Q_l(q)=\exp(\mu_l+\sigma_l\Phi^{-1}(q))
\]

其中：

- \(PL\)：贫困线，2025 setting 下为 £19,009；
- \(Q_l(q)\)：LSOA \(l\) 的第 \(q\) 分位数收入。

### 7.3 不建议每个 LSOA mean 都强制等于 MSOA mean

你当前想法是：

> 每个 LSOA 的收入均值约等于其所在 MSOA 的 income mean。

这个做法简单，但会弱化同一 MSOA 内不同 LSOA 之间的收入差异。

更推荐的约束是：

\[
\frac{\sum_{l \in m} N_l E(Y_l)}{\sum_{l \in m} N_l}
\approx
\bar{Y}_{m,s}
\]

也就是说：

> 只要求一个 MSOA 内所有 LSOA 的 household-weighted mean 加总后等于 ONS MSOA mean，而不是要求每个 LSOA 单独等于 MSOA mean。

这样可以允许：

- IoD 高的 LSOA 具有更低 mean、更高 lower-tail probability；
- IoD 低的 LSOA 具有更高 mean、更低 lower-tail probability；
- MSOA 层面总量仍被 ONS income estimate 约束。

### 7.4 IoD25 Income Deprivation score 应作为 soft tail constraint

IoD Income Deprivation score 不是严格的 household poverty rate，也不是 income below £19,009 的比例。

建议表述为：

> IoD25 Income Deprivation score is used as a calibrated lower-tail prior rather than a direct poverty rate.

可定义：

\[
\tilde{p}_l = logit^{-1}(\alpha + \beta \cdot logit(IoD_l))
\]

其中 \(\tilde{p}_l\) 是由 IoD25 Income Deprivation score 校准得到的 lower-tail probability proxy。

### 7.5 参数求解目标函数

建议使用如下 soft-constrained objective：

\[
\min_{\mu_l,\sigma_l}
\quad
w_1
\left[
\ln
\left(
\frac{\sum_{l \in m} N_l E(Y_l)}
{\sum_{l \in m} N_l}
\right)
-
\ln(\bar{Y}_{m,s})
\right]^2
+
w_2
\left[
logit(P(Y_l<PL)) - logit(\tilde{p}_l)
\right]^2
+
w_3(\sigma_l-\sigma_0)^2
+
w_4 SpatialSmoothness
\]

其中：

- \(w_1\)：MSOA mean income constraint 权重；
- \(w_2\)：IoD lower-tail constraint 权重；
- \(w_3\)：income dispersion regularisation；
- \(w_4\)：空间平滑约束；
- \(\sigma_0\)：合理的收入分布离散度 prior。

---

## 8. Income Scenarios and Cases

### 8.1 三个大场景

| Scenario | Income input | Interpretation |
|---|---|---|
| Baseline | MSOA mean income | 中性收入场景 |
| Optimistic | MSOA upper confidence limit | 乐观收入场景 |
| Conservative | MSOA lower confidence limit | 保守收入场景 |

### 8.2 每个场景下的五个 income percentile cases

每个 scenario 下，从 LSOA income distribution 中取：

- P90；
- P75；
- P50；
- P25；
- P10。

形成：

```text
3 income scenarios × 5 percentile cases = 15 income cases
```

### 8.3 对该设计的评价

该设计的优点是：

- 透明；
- 可解释；
- 避免将收入人为分配到每个 household；
- 适合作为 stress-test representative cases。

但需要避免声称：

> 这些 percentiles 代表真实 household-level income assignment。

更准确的表述是：

> These percentile cases are representative income stress-test cases for each LSOA, rather than inferred household-level income assignments.

---

## 9. 推荐增加：Probabilistic Exposure 指标

除了 15 个 deterministic income cases，建议增加一个更强的 probabilistic 指标。

### 9.1 Property-level probabilistic exposure

\[
P(Exposure_h>\theta)
=
P\left(\frac{Bill_h}{Y_l}>\theta\right)
=
P\left(Y_l<\frac{Bill_h}{\theta}\right)
=
F_l\left(\frac{Bill_h}{\theta}\right)
\]

这一步非常重要，因为它避免了 household-level income assignment。

你不需要知道某个 property 住户的真实收入，只需要知道该 property 所在 LSOA 的 income distribution，就可以计算：

> 在该 LSOA income distribution 下，该 property 的 bill burden 超过 10%、15%、20% 的概率。

### 9.2 LSOA-level risk aggregation

\[
Share^{risk}_{l,\theta}
=
\frac{1}{N_l}
\sum_{h \in l}
F_l\left(\frac{Bill_h}{\theta}\right)
\]

该指标比单纯 P10/P25/P50 cases 更有统计含义，建议作为主结果之一。

---

## 10. 第一层分析：谁最暴露？

### 10.1 核心问题

> Which households, properties and areas are most exposed to fuel prices?

### 10.2 主要输出

建议输出：

- property-level exposure distribution；
- LSOA / MSOA median exposure；
- top 10% exposure；
- share of properties above 10%、15%、20% thresholds；
- exposure gap；
- fuel poverty baseline vs high-exposure households 的重叠和差异。

### 10.3 Exposure gap

\[
ExposureGap_{h,\theta}
=
\max(0, Bill_h - \theta Y_l)
\]

该指标可以衡量超过 affordability threshold 的 burden depth。

### 10.4 Exposure typology

建议形成以下 household / property archetypes：

1. **Low-income efficient homes**  
   EPC A–C，但 income 很低；官方 fuel poverty 可能无法捕捉，但 bill burden 仍可能高。

2. **Low-income inefficient homes**  
   典型 fuel poverty group：低收入 + EPC D–G。

3. **Moderate-income high-demand homes**  
   收入不算低，但 detached / large floor area / solid wall，价格 shock 下 risk 高。

4. **Electric heating exposed homes**  
   direct electric heating 或高电力依赖家庭，electricity price risk 高。

5. **Off-gas or alternative-fuel homes**  
   可能集中在 rural / edge areas，系统替代路径不同。

6. **Transition-sensitive homes**  
   heat pump、EV、PV、battery 组合下 exposure 取决于 tariff、COP、self-consumption 和 capital cost。

---

## 11. 第二层分析：为什么暴露？

### 11.1 核心分解

\[
Exposure = \frac{Bill}{Income}
\]

因此，暴露来源可以初步分成：

- bill 高；
- income 低。

进一步，bill 可以拆成：

\[
Bill =
FabricDemand
\times
SystemEfficiency
\times
FuelPrice
+
StandingCharge
-
OnsiteGenerationBenefit
\]

### 11.2 暴露来源分类

建议将 exposure drivers 分为：

1. **Income effect**  
   低收入导致 bill burden 高。

2. **Fabric effect**  
   poor insulation、large heat loss、large floor area 导致 required demand 高。

3. **System effect**  
   heating system efficiency、fuel type、heat pump COP、direct electric heating 影响 bill。

4. **Technology effect**  
   PV、battery、EV、smart tariff 影响 electricity import/export 和 tariff exposure。

5. **Price structure effect**  
   unit rate、standing charge、gas-electricity price ratio 影响风险。

6. **Geographic / climate effect**  
   regional price、weather、rurality、off-gas constraints 影响 exposure。

### 11.3 Counterfactual decomposition

#### A. Income-normalised exposure

\[
Exposure^{income-normalised}_h
=
\frac{Bill_h}{MedianIncome_{England}}
\]

用于解释 income effect。

#### B. Fabric-normalised bill

将 property 的 wall / roof / window / floor insulation 改成 improved fabric package：

\[
Bill^{fabric+}_h
\]

\[
FabricContribution_h =
\frac{Bill_h - Bill^{fabric+}_h}{Y_l}
\]

#### C. System-normalised bill

保持 fabric 不变，替换 heating system：

\[
SystemContribution_h =
\frac{Bill_h - Bill^{system+}_h}{Y_l}
\]

#### D. Technology-normalised bill

加入 PV、battery、smart tariff、EV managed charging：

\[
TechContribution_h =
\frac{Bill^{system+}_h - Bill^{tech+}_h}{Y_l}
\]

#### E. Price shock contribution

\[
PriceRiskContribution_h =
\frac{Bill^{shock}_h - Bill^{base}_h}{Y_l}
\]

---

## 12. 第三层分析：Net-zero 路线能否缓解 exposure？

### 12.1 核心原则

必须区分：

- carbon reduction；
- bill reduction；
- fuel price risk reduction。

三者不一定总是同向。

### 12.2 Retrofit / technology scenarios

| Scenario | 内容 | 主要问题 |
|---|---|---|
| S0 Baseline | 当前 EPC + 当前系统 + 当前价格 | 当前 exposure / risk |
| S1 Fabric only | wall / roof / window / floor insulation | 是否降低 heat demand？ |
| S2 Heating system only | gas boiler upgrade 或 heat pump | 系统替换是否降低 bill？ |
| S3 Fabric + heat pump | 先降热需求再 electrify | 是否避免 heat pump bill risk？ |
| S4 PV only | rooftop PV | 是否降低 electricity import？ |
| S5 PV + battery | 提高 self-consumption | 是否降低 electricity price exposure？ |
| S6 Heat pump + PV + battery | 深度 electrification package | 是否同时降 CO₂ 和 exposure？ |
| S7 EV + smart tariff | 加入 EV load 和 flexible tariff | EV 是否增加或降低 risk？ |
| S8 Direct electric heating | 高电力依赖 | 最坏 electricity price risk benchmark |

### 12.3 每个 scenario 的输出指标

#### Annual bill change

\[
\Delta Bill = Bill^{scenario} - Bill^{baseline}
\]

#### Exposure change

\[
\Delta Exposure =
\frac{Bill^{scenario}}{Y_l}
-
\frac{Bill^{baseline}}{Y_l}
\]

#### Risk reduction

\[
\Delta P(Exposure > \theta)
\]

#### CO₂ reduction

\[
\Delta CO_2
\]

#### Risk reduction per pound

\[
RiskReductionPerPound =
\frac{\Delta P(Exposure>\theta)}{CapitalCost}
\]

#### Bill saving per tonne CO₂

\[
BillSavingPerTonneCO_2 =
\frac{\Delta Bill}{\Delta CO_2}
\]

### 12.4 Heat pump 的重要判断

Heat pump 不一定自动降低 exposure。

其结果取决于：

- heat pump COP；
- electricity-to-gas price ratio；
- fabric condition；
- tariff；
- 是否有 PV / battery；
- 是否移除 gas standing charge；
- household income。

建议识别以下类型：

- Heat-pump-ready households；
- Heat-pump-risk households；
- Fabric-first households；
- PV / battery benefit households；
- Policy-support-required households。

---

## 13. 数据计划

### 13.1 EPC / property data

数据来源：EPC recorded dataset / EPC bulk data。

主要变量：

- EPC band；
- property type；
- built form；
- floor area；
- wall type；
- wall insulation；
- roof type；
- roof insulation；
- glazing type；
- main heating system；
- main fuel；
- hot water system；
- existing PV；
- estimated energy consumption；
- estimated CO₂ emissions；
- postcode / UPRN / LSOA / MSOA mapping。

处理建议：

- 只保留 domestic properties；
- 删除 cancelled / not for issue records；
- 对同一 UPRN 或 address 选择 latest valid EPC；
- 保留 lodgement date，用于识别 certificate age；
- 用 ONSPD 或官方 lookup 匹配 LSOA / MSOA / Local Authority；
- 标记 EPC age，做 sensitivity test；
- 对 EPC coverage bias 做说明，必要时按 VOA / census dwelling stock 进行 reweighting。

### 13.2 Income data

使用 ONS MSOA equivalised disposable income after housing costs：

- mean；
- lower confidence limit；
- upper confidence limit。

构建三个大 income scenarios。

### 13.3 IoD25 Income Deprivation data

使用 LSOA level Income Deprivation score 作为 lower-tail prior。

注意：

- 不将其解释为 household poverty rate；
- 不将其解释为 income below £19,009 的真实比例；
- 仅作为 LSOA-level lower-tail deprivation signal。

### 13.4 Fuel price and tariff data

至少包括：

1. Ofgem price cap regional unit rates and standing charges；
2. DESNZ Quarterly Energy Prices；
3. Flat tariff scenarios；
4. Time-of-use tariff scenarios；
5. Export tariff scenarios；
6. EV tariff scenarios。

---

## 14. Robustness and Validation Strategy

### 14.1 Fuel poverty baseline validation

将 property-level proxy aggregation 到：

- LSOA；
- MSOA；
- Local Authority；
- region。

然后与官方 sub-regional fuel poverty estimates 进行 broad validation。

注意：

> 官方 LSOA-level fuel poverty estimates 本身也是模型估计，不应作为精确 ground truth，只适合用于 broad spatial pattern validation。

### 14.2 Income model sensitivity

建议测试：

- lognormal vs gamma / GB2；
- IoD tail weight \(w_2\) 高 / 中 / 低；
- LSOA mean fixed to MSOA mean vs MSOA aggregate constraint；
- poverty line £19,009 ± inflation adjustment；
- P10 / P25 / P50 / P75 / P90 percentile cases；
- probabilistic exposure vs deterministic percentile assignment。

### 14.3 EPC uncertainty

建议测试：

- latest EPC only vs all EPC weighted；
- EPC issued within last 10 years vs all records；
- missing UPRN records excluded vs retained by address matching；
- EPC energy consumption vs modelled energy consumption；
- property archetype reweighting to match census / VOA stock。

### 14.4 Technology uncertainty

建议测试：

- heat pump COP low / medium / high；
- PV generation low / central / high；
- battery operation strategy；
- EV mileage and charging profile；
- tariff assumptions；
- gas standing charge removed vs retained。

---

## 15. 建议论文 / 报告结构

### Chapter 1: Introduction

说明：

- fuel poverty 的政策背景；
- energy price volatility 的问题；
- official LILEE 指标的静态特征；
- 本研究提出 dynamic fuel price exposure and risk framework。

### Chapter 2: Conceptual Framework

定义：

- fuel poverty；
- affordability；
- exposure；
- vulnerability；
- risk；
- net-zero mitigation。

核心公式：

\[
Exposure = Bill / Income
\]

\[
Risk = P(Exposure>\theta)
\]

\[
Sensitivity = \partial Exposure / \partial Price
\]

### Chapter 3: Data and Income Distribution Modelling

介绍：

- EPC data；
- ONS MSOA income；
- IoD25 lower-tail constraint；
- lognormal income distribution；
- 3 × 5 income scenarios；
- probabilistic exposure method。

### Chapter 4: Baseline Fuel Poverty Proxy

构建 EPC-based LILEE proxy：

- EPC < C；
- AHC income minus required fuel costs < £19,009；
- 与官方 sub-regional estimates 做 broad validation。

### Chapter 5: Fuel Price Exposure Mapping

回答：

- 哪些 properties / areas 暴露最高；
- exposure 与 EPC、property type、floor area、heating fuel、income 的关系；
- exposure threshold maps。

### Chapter 6: Drivers of Exposure

使用 counterfactual decomposition 分析：

- income effect；
- fabric effect；
- system effect；
- fuel price effect；
- technology effect。

### Chapter 7: Dynamic Fuel Price Risk

分析：

- gas price shock；
- electricity price shock；
- combined energy crisis shock；
- standing charge shock；
- price-to-threshold；
- VaR / CVaR。

### Chapter 8: Net-zero Mitigation Scenarios

模拟：

- fabric insulation；
- heat pump；
- PV；
- battery；
- EV；
- smart tariff；
- combined packages。

输出：

- bill burden reduction；
- risk reduction；
- CO₂ reduction；
- intervention priority typology。

### Chapter 9: Discussion and Policy Implications

讨论：

- 哪些家庭被 official fuel poverty 漏掉但 exposure 很高；
- 哪些 fuel-poor homes 可以通过 fabric retrofit 降低风险；
- heat pump policy 的 income / fabric / tariff 条件；
- PV / battery 的分配公平性；
- targeted retrofit 的优先级。

---

## 16. 对当前研究想法的专家评价

### 16.1 总体评价

该研究问题非常有潜力，逻辑上成立，数据路径也基本可行。

最强的创新点不是单纯提出 bill / income ratio，而是将以下四个维度结合：

1. property-level exposure；
2. fuel price dynamic sensitivity；
3. technology counterfactual simulation；
4. net-zero transition risk and mitigation。

### 16.2 主要优点

#### 优点一：问题定义准确

你抓住了 LILEE 的局限：

- 它适合做 fuel poverty classification；
- 但不一定适合衡量 dynamic fuel price risk。

#### 优点二：数据组合有创造性

EPC、ONS income 和 IoD25 的组合能够支持高空间分辨率的 exposure framework。

#### 优点三：避免 household income pseudo-allocation 是明智的

不强行给每户分配 income，可以减少人为权重带来的伪精度。

建议进一步使用：

\[
P(Exposure>\theta)
\]

作为主指标之一。

#### 优点四：模型工具与研究问题高度匹配

你的两个已有模型正好支撑：

- 为什么暴露；
- 如何缓解暴露。

### 16.3 主要风险

#### 风险一：IoD score 被误用为 poverty rate

必须明确：

> IoD25 Income Deprivation score is not a direct household poverty rate.

#### 风险二：P10 / P25 / P50 / P75 / P90 被误读为 household-level income assignment

建议写成：

> representative income percentile stress-test cases。

#### 风险三：income 和 bill 的 equivalisation 需要一致

如果 denominator 是 equivalised AHC income，那么 numerator 的 required fuel costs 是否需要 equivalisation 必须明确。

可写成：

> The proposed metric uses dwelling-level required bill divided by an LSOA-level equivalised AHC income proxy, and this distinction is explicitly acknowledged.

#### 风险四：EPC < C 只是 proxy

不能声称完全复制 FPEER。

建议使用术语：

> EPC-based LILEE proxy。

#### 风险五：Net zero 不一定自动降低 bill risk

特别是：

- heat pump；
- direct electric heating；
- EV；
- high electricity price tariff。

这些情形可能增加 electricity exposure。

#### 风险六：资本成本不能完全忽略

建议输出两类指标：

1. **Operational exposure**：只看能源账单；
2. **Transition-adjusted exposure**：加入 annualised capital cost 或 subsidy / grant scenarios。

---

## 17. 最推荐的最终研究框架

最终研究框架：

```text
Fuel Poverty Baseline
        ↓
Fuel Price Exposure
        ↓
Fuel Price Risk
        ↓
Driver Attribution
        ↓
Net-zero Risk Mitigation
```

对应研究问题：

### RQ1. Who is exposed?

哪些 households / properties / LSOAs 的 required energy bill 占 AHC income 比例最高？

### RQ2. Who is at risk under price volatility?

哪些 households 在 gas / electricity / standing charge shock 下最容易超过 10%、15%、20% burden threshold？

### RQ3. Why are they exposed?

暴露来自：

- low income；
- poor fabric；
- inefficient heating system；
- fuel mix；
- tariff；
- lack of PV / battery / flexible technology？

### RQ4. Can net zero reduce exposure and risk?

哪些 fabric 和 low-carbon technology packages 能同时降低：

- bill burden；
- price risk；
- CO₂ emissions？

哪些 household 需要 subsidy 或 fabric-first approach？

### RQ5. Who is missed by official fuel poverty?

哪些 EPC A–C 或 non-fuel-poor households 仍然具有：

- high bill burden；
- high price sensitivity；
- high transition risk？

---

## 18. 推荐实施路径

### Step 1: Pilot area

先选择：

- 1–2 个 region；或
- 若干 local authorities；或
- urban / suburban / rural mixed sample。

目标是跑通完整流程。

### Step 2: EPC cleaning and geocoding

完成：

- EPC cleaning；
- latest certificate selection；
- UPRN / postcode matching；
- LSOA / MSOA matching；
- property archetype construction。

### Step 3: Income distribution construction

完成：

- ONS MSOA income scenario construction；
- IoD25 LSOA lower-tail calibration；
- lognormal distribution fitting；
- P10 / P25 / P50 / P75 / P90 extraction；
- probabilistic exposure function construction。

### Step 4: Baseline fuel poverty proxy

完成：

- EPC < C classification；
- income minus required fuel costs below £19,009；
- aggregation to LSOA / MSOA / LA；
- broad validation against official estimates。

### Step 5: Exposure mapping

完成：

- baseline exposure calculation；
- threshold exposure shares；
- exposure gap；
- spatial mapping；
- property archetype analysis。

### Step 6: Driver attribution

完成：

- income-normalised counterfactual；
- fabric counterfactual；
- system counterfactual；
- technology counterfactual；
- price shock counterfactual。

### Step 7: Net-zero mitigation scenarios

完成：

- fabric-only scenarios；
- heat pump scenarios；
- PV scenarios；
- battery scenarios；
- EV and smart tariff scenarios；
- combined retrofit scenarios。

### Step 8: England-wide scale-up

在 pilot 成功后扩展至英格兰全域。

---

## 19. 参考资料

以下资料建议作为后续正式写作和方法论说明的主要参考来源：

1. DESNZ fuel poverty statistics reports, including 2025 and 2026 publications.
2. DESNZ sub-regional fuel poverty statistics.
3. ONS small area model-based income estimates.
4. English Indices of Deprivation 2025 statistical release and technical documentation.
5. EPC bulk data and EPC statistical technical notes.
6. Ofgem energy price cap default tariff levels.
7. DESNZ Quarterly Energy Prices.

