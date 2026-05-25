# Household Fuel Price-at-Risk：方法论文献综述与 Westminster 实证设计

> 研究目标：构建一个 **Household Fuel Price-at-Risk（Household FPaR）** 指标，用于衡量 Westminster households 在不同 **fuel price shock scenarios** 下的 household energy burden / fuel poverty risk，以及 retrofit、heat pumps、PV、battery、smart tariffs 等 low-carbon interventions 带来的 **de-risking dividend**。  
> 引用格式：正文中的 `[R#]` 对应文末“参考资料与文献链接 / DOI”。

---

## 1. 核心结论

现有文献给出的最重要启示是：**Household Fuel Price-at-Risk 不应替代官方 fuel poverty 指标，而应作为其情景化、风险化扩展**。在英格兰，官方燃料贫困统计采用 **LILEE（Low Income Low Energy Efficiency）**：家庭同时满足低收入和低能效住房条件，即住房 FPEER 评级为 D 或以下，且扣除住房成本和模型化能源成本后的收入低于贫困线，才被认定为 fuel poor [R1][R2]。这意味着官方框架本质上是 **needs-based、model-based、efficiency-conditioned**，而不是简单的实际账单占收入比例。

与此相对，energy burden 文献常用 **energy expenditure / income** 比率，例如 6%、10% 或更高阈值来判断 affordability stress [R5][R6]。因此，Westminster 研究最稳健的设计是一个 **dual-framework model**：

1. 保留与 LILEE / fuel poverty 可比的 needs-based 指标；
2. 额外构建情景化的 **Fuel Price-at-Risk** 指标，用于衡量价格冲击下 household energy burden 的尾部风险；
3. 将 retrofit 和 low-carbon interventions 的价值定义为减少这种尾部风险的 **de-risking dividend**。

数据架构上，Westminster 特别适合采用混合模型：以 **London Building Stock Model v2（LBSM2）** 作为住房存量骨架，结合 EPC、Census、ONS/Nomis small-area data、Ofgem price cap/tariff data，以及 NEED/SERL 等实际消费数据进行校准 [R16][R17][R18][R19][R20][R21][R25][R27]。这比单独依赖 EPC 更可靠，因为 EPC / SAP / BREDEM 模型化需求和实际 metered consumption 并不等价 [R26][R27][R28]。

---

## 2. 现有研究如何建模 household energy burden 和 fuel poverty？

### 2.1 传统 10% affordability rule

英国早期 fuel poverty 定义通常采用 **required fuel costs > 10% of household income** 的规则。该方法直观，但对能源价格波动高度敏感，也可能把高收入、高能耗家庭误判为 fuel poor，同时无法很好反映低收入家庭的燃料贫困深度 [R3][R4]。

### 2.2 LIHC 与 fuel poverty gap

Hills Review 提出 **Low Income High Costs（LIHC）** 框架，将 fuel poverty 定义为低收入与高所需能源成本的交集，并强调应报告 **fuel poverty gap**，即家庭达到 fuel poverty threshold 所需补足的货币缺口 [R3]。该逻辑对 Household FPaR 很重要，因为 FPaR 不仅应报告“是否超阈值”，还应报告“超出多少”和“尾部风险多深”。

### 2.3 LILEE：英格兰官方现行框架

英格兰现行 LILEE 指标将 fuel poverty 建模为三个模块：

- **income after housing costs**；
- **modelled required energy costs**；
- **FPEER / SAP-based energy efficiency**。

家庭必须居住在 FPEER D–G 住房中，且扣除模型化能源成本后收入低于贫困线，才被认定为 fuel poor [R1][R2]。因此，在 Westminster case study 中，建议把 LILEE 作为官方可比基准，并在其外层增加 price shock risk assessment。

### 2.4 Energy burden ratio 与 Net Energy Return

国际 energy burden 文献通常定义：

\[
EB_i = \frac{C_i}{Y_i}
\]

其中 \(C_i\) 是家庭年度能源成本，\(Y_i\) 是家庭收入或扣除住房成本后的可支配收入。美国研究常用 6% 作为高负担阈值，10% 作为严重负担或 poverty-style 阈值 [R5][R6]。

Scheier and Kittner 提出基于 **Net Energy Return（NER）** 的替代测度，以缓解低收入或接近零收入家庭导致 burden ratio 爆炸的问题 [R6]。对应的 `emburden` 工具包也把 NER 方法用于 household energy burden 分析 [R7]。Westminster 若做 small-area aggregation 或 synthetic archetype aggregation，应避免让极低收入分母主导结果，建议同时报告 ratio、absolute gap 和 NER-style 指标。

### 2.5 Energy poverty premium

近期英国研究将 Understanding Society 与 NEED 数据连接，研究 low-income households 是否支付更高单位能源价格。结果表明，贫困家庭可能因 payment method、tariff、prepayment meter 等原因，每单位 gas / electricity 支付更高成本 [R8]。因此，Household FPaR 不应只模拟 energy demand，也应模拟 **pricing regime exposure**，包括：

- payment method；
- prepayment / direct debit；
- meter type；
- standing charge exposure；
- tariff eligibility；
- smart tariff access。

---

## 3. Spatial mapping、LSOA/MSOA analysis、EPC、Census、building stock models 与 synthetic archetypes

### 3.1 官方 small-area fuel poverty estimates

DESNZ 发布英格兰 sub-regional fuel poverty statistics，可到 LSOA 层面 [R9]。但官方也提示：LSOA 估计值存在不稳定性，且 fuel poverty gap 并未在 sub-regional level 完整建模。因此，LSOA 适合做 hotspot mapping 和 targeting layer，不宜过度解释精确排名。

### 3.2 Spatial energy justice 与 fuel poverty geography

Robinson、Bouzarovski、Lindley 等比较了 10% 指标与 LIHC 指标在英格兰的空间分布，显示指标选择会改变 fuel poverty geography：LIHC 往往使城市地区的燃料贫困更突出 [R11]。Bouzarovski 和 Simcock 的 spatializing energy justice 文献进一步指出，能源贫困不仅是收入问题，也是空间不平等、住房条件、能源基础设施和社会识别不足共同构成的 justice issue [R12]。

Bridgen and Robinson 使用 sequence analysis 和 clustering 研究 2010–2019 年英格兰地方政府层面的 fuel poverty persistence，表明能源剥夺具有时间延续性，且 energy efficiency obligations 的空间 targeting 不一定与长期高风险地区匹配 [R13]。

### 3.3 Multidimensional energy vulnerability mapping

Energy Deprivation Classification / Energy Deprivation Segmentation（EDS）将 GB small areas 分为 6 个 supergroups 和 14 个 groups，使用五类维度：energy efficiency、energy access、energy demand and supply、housing conditions、financial vulnerability [R14][R15]。这为 Westminster 提供了一个可复用的 multidimensional vulnerability framework，但也提示 composite index 的局限：解释性会弱于单一 burden / gap 指标。

### 3.4 London Building Stock Model v2

Westminster 的关键优势是可以使用 **LBSM2**。GLA / data.gov.uk 描述 LBSM2 为 London domestic building stock 的 property-level snapshot，包含 EPC-derived variables、heating systems、insulation、fabric variables，并对没有 EPC 的住房进行模型化补全 [R16][R17][R18]。UCL 对 LBSM 的说明也强调其通过 EPC 匹配、virtual EPC、benchmark gas/electricity use 等方法补充缺失数据 [R19]。

建议 Westminster 模型采用：

- **property-level 或 UPRN-level stock model** 作为内部建模层；
- 结果汇总到 **LSOA** 用于 hotspot maps；
- 再汇总到 **MSOA** 用于稳健比较与政策沟通。

### 3.5 EPC 与 ONS energy efficiency of housing

官方 EPC open data 可提供 England and Wales property certificates [R20]。ONS/Nomis 的 energy efficiency of housing 数据把最近十年的 EPC 记录汇总到 LSOA/MSOA/local authority 层面 [R21]。但 EPC 覆盖并不完整，且更新依赖交易、出租或改造触发，因此 EPC 不能被视为完整住房普查。

### 3.6 Building stock models 与 synthetic archetypes

CCC 2025 residential archetypes dataset 使用 EPC、housing surveys 和 ONS 数据构建超过 11,000 个 residential archetypes，并区分 delivered energy 与 useful energy [R22]。社区尺度案例中，Orkney 研究使用 GIS + EPC 创建 102 个 house archetypes，并用 EnergyPlus 模拟 retrofit impacts [R23]。机器学习研究也表明，使用 clustering / k-prototypes / XGBoost / SHAP 能识别不同 household vulnerability profiles [R24]。

Westminster 不宜只用“房屋类型 × EPC band”的简单交叉分类。更合适的是构建 **100–300 个 Westminster-specific synthetic household archetypes**，维度包括：

- dwelling type；
- dwelling age；
- floor area；
- EPC / FPEER band；
- heating fuel / heating system；
- tenure；
- household composition；
- age / health vulnerability；
- occupancy；
- income band；
- payment / tariff marker。

---

## 4. Required energy use、modelled energy demand 与 actual energy consumption 的区分

### 4.1 Required energy use

在 fuel poverty 语境中，required energy use 指家庭为了达到规定的 adequate warmth / domestic energy services 所需的能源量，而不是实际消费量。英国官方 fuel poverty modelling 通常使用 BREDEM / SAP / FPEER 相关模型估算 required energy use，包括：

- space heating；
- water heating；
- lights and appliances；
- cooking [R1][R2]。

该定义是 normative 的：它问的是“为了达到合理生活标准，应该需要多少能源”，而不是“家庭实际用了多少能源”。

### 4.2 Modelled energy demand / modelled energy costs

Modelled demand 是用住房物理属性、占用假设、气候假设、供暖制度和设备效率计算出的能源需求。Modelled energy costs 则进一步将 modelled demand 与 tariff / unit rate / standing charge 相乘得到年度成本 [R2]。

在英格兰官方 fuel poverty model 中，家庭并不使用真实 supplier tariff，而是使用 region、fuel、payment method 等平均价格序列进行估算 [R2]。这保证了可比性，但无法捕捉特殊 tariff、smart tariff optimisation、预付费惩罚或个体 supplier contract。

### 4.3 Actual energy consumption

Actual consumption 指 meter readings / billing data。NEED framework 将 gas/electricity meter readings 与 dwelling attributes、EPC、VOA property data、HEED、ECO、FIT 等连接，用于估计实际消费与节能措施效果 [R25]。它适合用于 calibration 和 validation，但不应直接替代 required energy use，因为低收入家庭可能通过少开暖气来“压低”实际消费。

### 4.4 Self-rationing、prebound 与 performance gap

DESNZ 对 theoretical vs actual energy use 的比较显示，理论能源开支平均高于实际 NEED consumption，且差距在 fuel-poor households 和 prepayment users 中更明显 [R26]。Few et al. 对 1,374 个英国 gas-heated households 的 EPC-modelled 与 smart-meter measured primary energy use 比较发现，EPC 会系统性高估部分低能效住房的能源使用 [R27]。Galvin and Sunikka-Blank 的 prebound effect 研究也指出，低效老旧住房实际消耗常低于理论模型，可能反映自我节制和燃料贫困 [R28]。

因此，Westminster 模型应同时报告：

1. **needs-based burden**：以 required / modelled energy 为基础；
2. **realised bill burden**：用 actual consumption calibration 调整；
3. **under-consumption risk**：实际消费低于 required need 的差距。

---

## 5. Fuel price shock scenarios 的设计

### 5.1 英国已有 precedent：Scottish fuel poverty scenario modelling

Scottish Government 已发布基于 Ofgem price cap 的 fuel poverty scenario modelling。方法是将 Ofgem 公布的 unit rates 与 standing charges 应用于每个家庭的 modelled consumption，估计不同 price cap 时点下的 fuel poverty 影响 [R29]。Ofgem price cap 文件也按 region、payment method、meter type 等区分 [R30]。

这为 Westminster 提供了直接可复用的场景框架：**保持 consumption model 固定，冲击 price engine**。

### 5.2 分离 gas shock 与 electricity pass-through

英国电力市场中，gas price changes 对 wholesale electricity prices 有强影响，但 retail electricity pass-through 受 price cap、regulation、tariff period 和 supplier hedging 影响 [R31][R32]。因此不应简单设置“所有能源价格 +50%”。更合理的做法是：

- gas unit rate 直接冲击；
- electricity unit rate 通过 pass-through sensitivity 冲击；
- standing charge 单独建模；
- policy support / tariff reform 单独作为 mitigation scenario。

### 5.3 建议场景结构

#### Baseline

- 使用当前 London / Southern region 或 Westminster 对应的 Ofgem price cap unit rates 与 standing charges；
- 区分 direct debit、standard credit、prepayment；
- 区分 electricity meter type。

#### Commodity shock layer

- Gas unit rate: +25%、+50%、+100%；
- Electricity unit rate: +10%、+25%、+50%；
- Standing charge: base、+10%、+25%，或保持不变作为 first-order model。

#### Gas-to-electricity pass-through layer

设 \( \lambda \in \{0, 0.5, 1.0\} \) 表示 gas shock 向 electricity commodity component 的传导程度：

\[
p^{elec}_{s} = p^{elec}_{0} \times (1 + \lambda \cdot g_s)
\]

其中 \(g_s\) 是 gas shock rate。建议只冲击 electricity commodity component，而不是 whole electricity bill，因为 network costs、policy costs 和 standing charge 不应被等比例放大 [R30][R31][R32]。

#### Tariff inequity layer

Committee on Fuel Poverty 指出 standing charge、unit rate、payment method 和创新 tariff 都是 equity 关键渠道 [R33]。因此应加入：

- standing-charge redistribution；
- unit-rate discount；
- prepayment penalty removal；
- Warm Home Discount / lump-sum support；
- social tariff；
- smart tariff access sensitivity。

---

## 6. Retrofit、heat pumps、PV、battery、smart tariffs 的影响评估

### 6.1 Retrofit / fabric-first

DESNZ National Buildings Model 和 fuel poverty strategy technical annex 可用于理解 retrofit measures 如何改变 SAP/FPEER 和 modelled energy consumption [R34]。Fuel-poverty-constrained retrofit optimization 文献显示，如果 retrofit optimisation 不纳入 fuel poverty constraints，可能高估 decarbonisation potential；高成本措施如 heat pumps、triple glazing、solid wall insulation 在某些价格结构下可能增加 fuel poverty risk [R35]。

建议 Westminster 将 retrofit packages 分为：

- no-regret / low-risk measures：draught-proofing、loft insulation、cavity wall insulation、heating controls；
- medium-cost measures：double glazing、floor insulation、partial solid wall insulation；
- high-cost measures：solid wall insulation、triple glazing、deep retrofit；
- combined fabric + heating system packages。

### 6.2 Heat pumps

Heat pumps 会降低 delivered energy demand，但也会把 heating load 从 gas 转向 electricity。因此其 affordability effect 取决于：

- seasonal COP / SPF；
- gas-electricity price ratio；
- standing charge；
- heat-pump-specific tariff；
- insulation level；
- household load profile。

Centre for Net Zero 使用 Octopus 数据的准实验研究显示，heat pump 与 time-of-use tariff 应作为组合情景建模，而不是单独建模 [R36]。

### 6.3 PV、battery 与 smart tariffs

PV 可降低 grid electricity import，battery 的收益则高度依赖 tariff spread 和 load shifting ability。UK case-study 文献指出，storage 的经济性在 energy crisis 之后更依赖 peak/off-peak differential，且不同技术组合需要不同 tariff design [R37]。Warm Homes Plan 也将 heat pump + PV + battery + TOU tariff 作为家庭节能潜力的重要组合之一 [R38]。

因此，battery 不应作为通用 add-on，而应仅在以下条件下建模：

- 有可观 PV generation；
- 有 smart meter；
- 有 TOU tariff；
- household load profile 可转移；
- peak/off-peak spread 足够大。

---

## 7. “Household Fuel Price-at-Risk” 指标设计

### 7.1 文献中是否已有类似指标？

本次检索没有发现 household energy affordability 文献中已有被广泛使用的 **Price-at-Risk** 或 **Energy Burden-at-Risk** 标准指标。最接近的概念包括：

- fuel poverty gap [R3]；
- energy burden threshold crossing [R5][R6]；
- energy poverty premium [R8]；
- Net Energy Return / NER-style transformation [R6][R7]；
- house-price-at-risk / value-at-risk 类尾部风险框架 [R39]；
- resilience dividend / triple dividend of resilience [R40]。

因此，Household FPaR 可以被定位为一个 **methodological contribution**：它把 fuel poverty gap、energy burden thresholds 和 price-shock scenario analysis 合并成一个 household-level tail-risk 指标。

### 7.2 基础变量

对 household / archetype \(i\)、price scenario \(s\)、intervention \(k\)：

- \(E^{gas}_{i,k}\)：年度 required gas demand；
- \(E^{elec}_{i,k}\)：年度 required electricity demand；
- \(p^{gas}_s\)：scenario gas unit rate；
- \(p^{elec}_s\)：scenario electricity unit rate；
- \(SC^{gas}_s\)、\(SC^{elec}_s\)：standing charges；
- \(Y_i^{res}\)：income after housing costs；
- \(H_i\)：housing cost；
- \(A_i\)：household equivalisation factor；
- \(FPEER_{i,k}\)：intervention 后 FPEER；
- \(PV_{i,k}\)、\(Battery_{i,k}\)、\(TOU_{i,k}\)：technology / tariff variables。

年度 required energy cost：

\[
C_{i,s,k} =
E^{gas}_{i,k} p^{gas}_s +
E^{elec,net}_{i,s,k} p^{elec}_s +
SC^{gas}_s + SC^{elec}_s - Support_{i,s}
\]

energy burden：

\[
EB_{i,s,k} = \frac{C_{i,s,k}}{Y_i^{res}}
\]

### 7.3 Household Fuel Price-at-Risk

定义 household energy burden 在情景集合 \(S\) 下的 \(q\)-quantile：

\[
FPaR_i(q,k) = Q_q\left( EB_{i,s,k} \right), \quad s \in S
\]

其中 \(q\) 可设为 0.90、0.95 或 0.99。

### 7.4 Threshold-Crossing Risk

\[
TCR_i(\tau,k) =
P\left( EB_{i,s,k} > \tau \right)
\]

其中 \(\tau\) 可设为 6%、10%、15%，也可设 Westminster-specific affordability threshold。

### 7.5 Affordability Gap-at-Risk

\[
AGaR_i(\tau,q,k)
=
Q_q\left[
\max(0, C_{i,s,k} - \tau Y_i^{res})
\right]
\]

该指标用 £ 表示尾部 affordability gap，比单纯百分比更适合政策预算和 intervention targeting。

### 7.6 LILEE-consistent risk

\[
LILEE_{i,s,k}=1
\]

当且仅当：

1. \(FPEER_{i,k} \leq D\)；
2. 扣除 housing costs 和 modelled energy costs 后，income below poverty line。

这允许你在 price shock scenarios 下报告“多少家庭会从非 fuel poor 变成 LILEE-risk household”。

### 7.7 De-risking dividend

对 intervention \(k\)，定义：

\[
DD_i^k(q) = FPaR_i(q, baseline) - FPaR_i(q,k)
\]

也可定义为：

\[
DD^{TCR}_{i,k} = TCR_i(\tau, baseline) - TCR_i(\tau,k)
\]

\[
DD^{AGaR}_{i,k} = AGaR_i(\tau,q,baseline) - AGaR_i(\tau,q,k)
\]

Area-level de-risking dividend 可用 household weights 汇总到 LSOA/MSOA：

\[
DD_{a,k} = \sum_{i \in a} w_i DD_{i,k}
\]

---

## 8. 可复用方法框架

### 8.1 总体模型结构

```text
Building stock layer
    ↓
Household / archetype assignment
    ↓
Required energy demand model
    ↓
Tariff and price engine
    ↓
Fuel price shock scenario engine
    ↓
Intervention engine
    ↓
Fuel poverty / burden / risk metrics
    ↓
LSOA / MSOA maps and distributional outputs
```

### 8.2 变量清单

| 模块 | 变量 | 用途 | 可能数据源 |
|---|---|---|---|
| Geography | UPRN, LSOA, MSOA, ward | 空间汇总与 mapping | LBSM2, ONS boundaries |
| Dwelling | dwelling type, age, floor area, wall type, roof type | modelled demand | LBSM2, EPC |
| Energy efficiency | EPC band, SAP, FPEER, insulation | LILEE / retrofit | EPC, LBSM2, DESNZ |
| Heating system | gas boiler, electric heating, heat pump | fuel-switching | LBSM2, EPC, Census |
| Household | household size, children, older adults, disability | occupancy / vulnerability | Census, EHS, synthetic population |
| Tenure | owner-occupied, private rent, social rent | intervention feasibility | Census TS054 |
| Income | income after housing costs, equivalised income | burden denominator | EHS, ONS income estimates, synthetic allocation |
| Consumption | required demand, metered demand | burden / calibration | BREDEM/SAP, NEED, SERL |
| Prices | unit rates, standing charges, payment method | cost engine | Ofgem |
| Tariff | prepayment, direct debit, TOU tariff | price exposure | Ofgem, supplier datasets |
| Technology | retrofit, HP, PV, battery | intervention scenarios | LBSM2, EPC, technical assumptions |
| Policy | WHD, social tariff, rebates | mitigation scenarios | DESNZ, Ofgem |

### 8.3 主要输出

- baseline required energy burden map；
- gas +50% / gas +100% burden map；
- electricity pass-through sensitivity map；
- FPaR90 / FPaR95 map；
- threshold-crossing probability map；
- affordability gap-at-risk map；
- LILEE-consistent risk map；
- de-risking dividend map by intervention；
- distribution by tenure / dwelling type / EPC band / heating fuel / income band。

---

## 9. Westminster case study：建议 empirical design

### 9.1 Spatial scale

建议采用：

- **内部建模层**：UPRN/property-level 或 LBSM2 record-level；
- **主地图输出层**：LSOA；
- **稳健性与政策汇总层**：MSOA；
- **补充边界**：ward / conservation area / social housing estates（如数据可得）。

理由：Westminster 住房结构高度异质，property-level modelling 更能捕捉 flats、historic stock、private rented sector、electric heating 等差异；但 LSOA 估计不稳定，因此 MSOA 更适合正式比较和政策摘要 [R9][R16][R17][R21]。

### 9.2 Household archetypes

建议创建 **100–300 个 Westminster-specific archetypes**，通过 k-prototypes 或 hierarchical clustering 生成。最小 archetype schema：

```text
dwelling type
x dwelling age
x EPC/FPEER band
x floor area band
x heating fuel/system
x tenure
x household composition
x income band
x vulnerability marker
x tariff/payment marker
```

重点 archetypes 可包括：

1. 私租、小面积、低 EPC、electric heating flats；
2. 社会租赁、gas boiler、低收入家庭；
3. older owner-occupiers in inefficient pre-1930 dwellings；
4. high-income but high-demand large dwellings；
5. student / shared / overcrowded households；
6. households with children or older adults；
7. heat-pump technically suitable but electricity-price exposed homes；
8. PV-suitable owner-occupied dwellings；
9. prepayment / high standing-charge exposure households；
10. conservation-area retrofit-constrained properties。

### 9.3 Shock scenarios

| 场景 | Gas unit rate | Electricity unit rate | Standing charge | 用途 |
|---|---:|---:|---:|---|
| S0 baseline | 0% | 0% | baseline | 当前负担 |
| S1 moderate gas shock | +25% | pass-through 0/50/100% | baseline | 温和冲击 |
| S2 severe gas shock | +50% | pass-through 0/50/100% | baseline | 主压力测试 |
| S3 extreme gas shock | +100% | pass-through 0/50/100% | baseline | tail risk |
| S4 electricity shock | 0% | +25%/+50% | baseline | 电采暖/HP风险 |
| S5 standing charge shock | 0% | 0% | +10%/+25% | 小用户/低用能家庭风险 |
| S6 social tariff | S2/S3 | reduced rate | adjusted | 政策缓释 |
| S7 prepayment penalty removal | S2/S3 | S2/S3 | adjusted | tariff equity |

### 9.4 Intervention scenarios

| 包 | 内容 | 关键假设 | 预期效果 |
|---|---|---|---|
| I0 baseline | 无干预 | 当前 stock | 当前风险 |
| I1 fabric-light | draught-proofing, loft top-up, controls | 低成本、广覆盖 | 降低 heating demand |
| I2 fabric-medium | cavity wall, glazing, floor insulation | 技术可行性过滤 | 降低 burden 和 FPaR |
| I3 deep retrofit | solid wall, high-performance glazing | conservation / cost constraints | 最大 demand reduction |
| I4 heat pump only | ASHP, COP/SPF assumptions | gas-to-electricity shift | 减碳但可能增加电价暴露 |
| I5 heat pump + TOU | ASHP + smart tariff | TOU access / load profile | 降低 HP bill risk |
| I6 PV only | rooftop PV | roof suitability | 降低电力进口 |
| I7 PV + battery | PV + storage | tariff spread required | 降低 peak exposure |
| I8 integrated package | fabric + HP + PV + battery + TOU | 技术与资本约束 | 最大 de-risking dividend |
| I9 policy support | WHD / social tariff / standing charge reform | eligibility | 降低 threshold crossing |

### 9.5 Output maps

建议至少生成以下地图：

1. baseline required energy burden；
2. baseline LILEE-consistent risk；
3. gas +50% burden；
4. gas +100% burden；
5. electricity pass-through sensitivity；
6. FPaR90；
7. FPaR95；
8. threshold-crossing probability at 6%、10%、15%；
9. affordability gap-at-risk (£/year)；
10. de-risking dividend from fabric retrofit；
11. de-risking dividend from heat pump + TOU；
12. de-risking dividend from integrated package；
13. policy support residual-risk map；
14. uncertainty / EPC-missing sensitivity map。

---

## 10. 潜在局限

### 10.1 EPC coverage 与 stale certificates

EPC 不是完整住房普查，且很多 retrofit 不会触发新 EPC。LBSM2 对缺失值进行模型化补全，但会引入 imputation uncertainty [R17][R18][R21]。

### 10.2 Modelled demand 与 actual consumption gap

如果过度用实际消费校准，会把 self-rationing 固化为“正常消费”，从而低估 fuel poverty risk [R26][R27][R28]。

### 10.3 Household income synthetic allocation uncertainty

小区域收入、住房成本和 household composition 的匹配通常需要 synthetic population 或 survey reweighting。Westminster 内部高收入与低收入家庭空间混杂，income imputation uncertainty 需要 sensitivity analysis。

### 10.4 Technology access inequality

Heat pumps、PV、battery 和 smart tariffs 的可及性取决于 tenure、屋顶条件、资本成本、smart meter、landlord consent、heritage constraints 和 household flexibility。技术情景不应假设所有家庭都能等比例获得低碳技术收益 [R33][R36][R37]。

### 10.5 指标新颖性

Household FPaR 不是现有官方指标，因此论文需要明确其定义、与 LILEE / 10% burden / fuel poverty gap 的关系、验证方法和政策解释边界 [R1][R3][R6][R39][R40]。

---

## 11. 推荐论文结构

1. Introduction：energy price volatility、fuel poverty、Westminster spatial inequality；
2. Literature review：fuel poverty metrics、energy burden、spatial energy justice、building stock models、retrofit impacts；
3. Data：LBSM2、EPC、Census、Ofgem、NEED/SERL calibration；
4. Methods：synthetic archetypes、required energy model、price shock engine、intervention engine；
5. Metrics：FPaR、TCR、AGaR、de-risking dividend；
6. Results：baseline maps、shock maps、intervention maps；
7. Discussion：policy targeting、fuel poverty vs decarbonisation trade-offs；
8. Limitations：data uncertainty、self-rationing、tariff access、heritage constraints；
9. Conclusion：Household FPaR as a scenario-based spatial energy justice metric。

---

## 参考资料与文献链接 / DOI

### A. Fuel poverty / energy burden metrics

- [R1] ONS. *How fuel poverty is measured in the UK*. Link: <https://www.ons.gov.uk/peoplepopulationandcommunity/housing/articles/howfuelpovertyismeasuredintheuk/march2023>
- [R2] DESNZ. *Methodology Handbook: 2026 Fuel Poverty Statistics Publication*. Link: <https://assets.publishing.service.gov.uk/media/69c3b33b380a2a73a7cf9e43/Methodology_Handbook__2026_Fuel_Poverty_Statistics_Publication_.pdf>
- [R3] Hills, J. *Getting the measure of fuel poverty: Final Report of the Fuel Poverty Review*. Link: <https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/48298/4663-fuel-poverty-final-report-summary.pdf>
- [R4] Moore, R. (2012). *Definitions of fuel poverty: Implications for policy*. **Energy Policy**. DOI: <https://doi.org/10.1016/j.enpol.2012.01.057>
- [R5] ACEEE. *Lifting the High Energy Burden in America’s Largest Cities*. Link: <https://www.aceee.org/sites/default/files/energy-affordability.pdf>
- [R6] Scheier, E., & Kittner, N. (2022). *A measurement strategy to address disparities across household energy burdens*. **Nature Communications**. DOI: <https://doi.org/10.1038/s41467-021-27673-y>
- [R7] `emburden`: *Energy Burden Analysis Using Net Energy Return Methodology*. Link: <https://www.maths.bristol.ac.uk/R/web/packages/emburden/vignettes/jss-emburden.html>
- [R8] Rasanga, F., Harrison, T., & Calabrese, R. (2024). *Measuring the energy poverty premium in Great Britain and identifying its main drivers based on longitudinal household survey data*. **Energy Economics**. DOI: <https://doi.org/10.1016/j.eneco.2024.107726>

### B. Spatial fuel poverty, energy vulnerability and energy justice

- [R9] DESNZ. *Sub-regional fuel poverty in England, 2025 report: 2023 data*. Link: <https://www.gov.uk/government/statistics/sub-regional-fuel-poverty-report-2025-2023-data/sub-regional-fuel-poverty-in-england-2025-report-2023-data>
- [R10] Data Mill North. *Fuel poverty by LSOA, England*. Link: <https://datamillnorth.org/dataset/2j70l/fuel-poverty-by-lsoa-england>
- [R11] Robinson, C., Bouzarovski, S., & Lindley, S. (2018). *Getting the measure of fuel poverty: The geography of fuel poverty indicators in England*. **Energy Research & Social Science**. DOI: <https://doi.org/10.1016/j.erss.2017.09.035>
- [R12] Bouzarovski, S., & Simcock, N. (2017). *Spatializing energy justice*. **Energy Policy**. DOI: <https://doi.org/10.1016/j.enpol.2017.03.064>
- [R13] Bridgen, P., & Robinson, C. (2023). *A decade of fuel poverty in England: A spatio-temporal analysis of needs-based targeting of domestic energy efficiency obligations*. **Energy Research & Social Science**. DOI: <https://doi.org/10.1016/j.erss.2023.103139>
- [R14] Geographic Data Service. *Energy Deprivation Classification*. Link: <https://data.geods.ac.uk/dataset/energy-deprivation-classification>
- [R15] Chen, M., Robinson, C., & Singleton, A. (2025). *Mapping multidimensional energy deprivation: Socio-spatial inequalities and policy implications in Great Britain*. **Computers, Environment and Urban Systems**. DOI: <https://doi.org/10.1016/j.compenvurbsys.2025.102324>

### C. Building stock, EPC, Census and archetype data

- [R16] Greater London Authority. *London Building Stock Model*. Link: <https://www.london.gov.uk/programmes-and-strategies/environment-and-climate-change/energy/energy-buildings/london-building-stock-model>
- [R17] data.gov.uk. *London Building Stock Model 2 (LBSM 2.1)*. Link: <https://www.data.gov.uk/dataset/e03aa07a-ee8a-4b1a-a04c-cc6e23133340/london-building-stock-model-2-lbsm-21>
- [R18] GOV.UK. *Algorithmic Transparency Record: Greater London Authority London Building Stock Model 2*. Link: <https://www.gov.uk/algorithmic-transparency-records/greater-london-authority-london-building-stock-model-2>
- [R19] UCL. *London Building Stock Model*. Link: <https://www.ucl.ac.uk/bartlett/research-projects/2020/mar/london-building-stock-model>
- [R20] MHCLG / Open Data Communities. *Energy Performance of Buildings Data: England and Wales*. Link: <https://epc.opendatacommunities.org/>
- [R21] ONS / Nomis. *Energy efficiency of housing, England and Wales, MSOA/LSOA*. Links: <https://www.nomisweb.co.uk/datasets/eeh> and <https://www.ons.gov.uk/peoplepopulationandcommunity/housing/datasets/energyefficiencyofhousingenglandandwalesmiddlelayersuperoutputarea>
- [R22] Climate Change Committee / Kamma. *Residential Archetypes Dataset*. Link: <https://www.theccc.org.uk/wp-content/uploads/2025/02/Residential-Archetypes-Dataset-Kamma-1.pdf>
- [R23] Vatougiou et al. *A GIS-based domestic building stock model for energy simulations and energy vulnerability assessment at a community scale*. Link: <https://publications.ibpsa.org/conference/paper/?id=usim2020_A1_1_Vatougiou>
- [R24] Fuel poverty machine-learning / clustering studies:  
  - *Energy poverty prediction in the United Kingdom: A machine learning approach*. DOI: <https://doi.org/10.1016/j.enpol.2023.113909>  
  - *From data to dignity: Understanding and predicting fuel poverty in the United Kingdom with machine learning*. DOI: <https://doi.org/10.1016/j.erss.2025.104410>
- [R41] ONS Census 2021 datasets:  
  - TS003 age by single year: <https://www.ons.gov.uk/datasets/TS003/editions/2021/versions/2>  
  - TS046 central heating: <https://www.ons.gov.uk/datasets/TS046/editions/2021>  
  - TS054 tenure: <https://www.ons.gov.uk/datasets/TS054/editions/2021>

### D. Required energy, modelled demand and actual consumption

- [R25] DESNZ. *Domestic National Energy Efficiency Data-Framework (NEED): Methodology*. Link: <https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/996297/Domestic_NEED_Methodology__1_.pdf>
- [R26] DESNZ. *Comparison of theoretical energy consumption with actual usage*. Link: <https://assets.publishing.service.gov.uk/media/5c9b4b1c40f0b633fc95f79c/Comparison_of_theoretical_energy_consumption_with_actual_usage.pdf>
- [R27] Few, J. et al. (2023). *The over-prediction of energy use by EPCs in Great Britain: A comparison of EPC-modelled and metered primary energy use intensity*. **Energy and Buildings**. DOI: <https://doi.org/10.1016/j.enbuild.2023.113024>
- [R28] Galvin, R., & Sunikka-Blank, M. *Prebound effect / theoretical vs actual heating energy use*. Link: <https://www.repository.cam.ac.uk/items/c15a4d71-e577-4e73-b4c4-34a53b383ee4>

### E. Price shocks, tariffs and policy support

- [R29] Scottish Government. *Fuel poverty scenario modelling based on Ofgem energy price caps: methodology*. Link: <https://www.gov.scot/publications/fuel-poverty-scenario-modelling-based-on-ofgem-energy-price-caps-up-to-april-to-june-2026/pages/methodology/>
- [R30] Ofgem. *Energy price cap*. Link: <https://www.ofgem.gov.uk/energy-regulation/domestic-and-non-domestic/energy-pricing-rules/energy-price-cap>
- [R31] UKERC. *The Price of Power: wholesale market price formation, policy costs and domestic electricity bills in Britain*. Link: <https://ukerc.ac.uk/publications/the-price-of-power-wholesale-market-price-formation-policy-costs-and-domestic-electricity-bills-in-britain/>
- [R32] Cambridge EPRG / JSTOR. *Pass-through from gas prices to electricity prices in Great Britain*. Link: <https://www.jstor.org/stable/resrep30333>
- [R33] Committee on Fuel Poverty. *Options for improving energy bill equity for fuel poor households*. Link: <https://assets.publishing.service.gov.uk/media/686c093e2cfe301b5fb67830/options-improving-energy-bill-equity-fuel-poor-households-report.pdf>

### F. Retrofit and low-carbon interventions

- [R34] DESNZ. *Fuel Poverty Strategy Technical Annex / National Buildings Model evidence*. Link: <https://assets.publishing.service.gov.uk/media/696f6dab5b6060ca6736a097/fuel-poverty-strategy-technical-annex.pdf>
- [R35] Rossi, V. A., Howard, B., & Wright, J. (2026). *Fuel-poverty-constrained retrofit optimization: A socio-technical approach to decarbonising the UK building stock*. **Energy and Buildings**. DOI: <https://doi.org/10.1016/j.enbuild.2025.116628>
- [R36] Centre for Net Zero. *Decarbonising heat: the impact of heat pumps and a time-of-use heat pump tariff on energy demand*. Link: <https://www.centrefornetzero.org/papers/decarbonising-heat-the-impact-of-heat-pumps-and-a-time-of-use-heat-pump-tariff-on-energy-demand>
- [R37] *Holistic analysis of consumer energy decarbonisation options and tariff effects*. **Applied Energy**. DOI: <https://doi.org/10.1016/j.apenergy.2023.122165>
- [R38] UK Government. *Warm Homes Plan*. Link: <https://www.gov.uk/government/publications/warm-homes-plan/warm-homes-plan-html>

### G. Price-at-Risk / resilience dividend analogues

- [R39] Bundesbank. *House price-at-risk / tail risk technical paper*. Link: <https://www.bundesbank.de/resource/blob/925506/50f9623fb90077b68b5ab7a6e386a79e/472B63F073F071307366337C94F8C870/2024-08-technical-paper-data.pdf>
- [R40] World Resources Institute. *The triple dividend of building climate resilience*. Link: <https://www.wri.org/research/triple-dividend-building-climate-resilience-taking-stock-moving-forward>
