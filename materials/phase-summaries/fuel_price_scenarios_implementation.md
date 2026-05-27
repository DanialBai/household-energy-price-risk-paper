# Fuel price change 真实情景与实现逻辑

> 项目：UK household fuel price risk / Westminster household-dwelling modelling  
> 目标：把真实官方价格变化转化为可复现的 fuel price shock scenario engine，并与现有 property-level required energy burden 模型衔接。  
> 版本：2026-05-27

## 1. 方法依据

`method_documentation.pdf` 的核心贡献是把 household energy-shock exposure 拆成四个机制层：

```text
Risk = external shock × structural exposure × retail/network pass-through × coping capacity
```

对应到 Westminster 项目中：

- **External shock**：不是把某个地缘政治事件当作直接处理变量，而是用该事件窗口内 electricity、gas、heating oil、petrol/diesel 的真实价格偏离 reference price 的幅度来度量。
- **Structural exposure**：由 dwelling demand 和 fuel mix 决定。Westminster 当前可用 EPC/SAP 字段应继续作为 property-level 分母/需求基础，例如 `MAIN_HEATING_FUEL`、`MAINS_GAS_FLAG`、`TOTAL_FLOOR_AREA`、SAP-derived required cost 等。
- **Retail/network pass-through**：Ofgem default tariff cap 的 unit rate 与 standing charge 是 electricity/gas price pass-through 的官方主干；standing charge 必须与 unit rate 拆开建模。
- **Coping capacity**：应继续使用 after-housing-cost residual income、income deprivation、tenure proxy、payment method、low-carbon technology access 等变量表示。

## 2. 第一版真实价格情景集

| ID | 情景名称 | 时间窗口 | 主要燃料 | 研究含义 | 官方数据输入 |
|---|---|---:|---|---|---|
| S0 | `baseline_2021_ref` | 2021-04-01 至 2021-09-30 | electricity, gas, oil, road fuel | pre-shock reference period | Ofgem v1.8; DESNZ petroleum/road fuel |
| S1 | `post_covid_cap_uplift_2021Q4_2022Q1` | 2021-10-01 至 2022-03-31 | electricity, gas | post-COVID wholesale uplift | Ofgem v1.9 |
| S2 | `pre_ukraine_severe_cap_2022H1` | 2022-04-01 至 2022-09-30 | electricity, gas | severe cap uplift | Ofgem v1.10 |
| S3 | `ukraine_unmitigated_cap_2022Q4` | 2022-10-01 至 2022-12-31 | electricity, gas | extreme unmitigated cap / market exposure | Ofgem v1.13 |
| S4 | `ukraine_epg_observed_2022Q4` | 2022-10-01 至 2022-12-31 | electricity, gas | policy-mitigated observed unit-rate scenario | GOV.UK Energy Price Guarantee regional rates |
| S5 | `heating_oil_spike_2022Q2` | 2022-03-01 至 2022-06-30 | heating oil | off-gas / unregulated fuel exposure | DESNZ petroleum product monthly prices |
| S6 | `standing_charge_reallocation_2026Q2_to_Q3` | 2026-04-01 至 2026-09-30 | electricity, gas | standing charge / fixed-cost exposure | Ofgem current unit rates and Annex 9 |
| S7 | `road_fuel_transport_shock_2022H1` | 2022-02-01 至 2022-07-31 | petrol, diesel | optional EV / transport appendix | DESNZ weekly road fuel prices |

推荐主模型先使用 **S0-S6**。S7 只有在论文保留 EV / transport fuel exposure 时进入 appendix。对 Westminster domestic fuel risk 而言，S3/S4 与 S6 最关键，S5 是理论上重要但样本量可能较小的 off-gas robustness scenario。

## 3. Price spine 实现逻辑

### 3.1 数据结构

建议将所有官方价格整理成一个 monthly price spine：

```text
price_spine_monthly
    month
    ofgem_region
    payment_method
    meter_type
    electricity_unit_p_per_kwh
    electricity_standing_p_per_day
    gas_unit_p_per_kwh
    gas_standing_p_per_day
    oil_p_per_litre
    petrol_p_per_litre
    diesel_p_per_litre
    source_period
    price_basis
```

其中：

- Westminster 默认为 `ofgem_region = London`。
- payment method 至少保留 `direct_debit`、`standard_credit`、`prepayment` 三组。
- electricity/gas 保留 unit rate 与 standing charge，不要只建模 unit rate。
- Ofgem historical workbooks 中早期数据多为 ex VAT，需要统一 VAT 处理；consumer-facing current Ofgem tables 多为 incl VAT。
- EPG 2022Q4 regional rates 是 ex VAT，需要在模型中根据 `vat_multiplier = 1.05` 转为 incl VAT，或保持全模型 ex VAT。

### 3.2 Ofgem tariff cap parsing

方法文档给出的基本解析逻辑如下：

```text
standing_charge_p_per_day = nil_consumption_charge_gbp_per_year * 100 / 365
unit_rate_p_per_kwh = (benchmark_consumption_charge - nil_consumption_charge) * 100 / benchmark_consumption_kwh
```

通常 benchmark annual consumption 可按 Ofgem cap model 中的 TDCV / benchmark assumptions 读取；若先做快速原型，可用方法文档中的默认值：single-register electricity 3100 kWh/year，gas 12000 kWh/year。正式模型应从 workbook/period metadata 中读取。

### 3.3 DESNZ fuel parsing

```text
road_fuel_weekly -> calendar-month mean
heating_oil_monthly -> direct monthly input, convert if needed
```

- Road fuel CSV 已直接给出 pence/litre。
- Petroleum product workbook 通常包括 home heating fuels / petroleum products 月度表。需要定位 heating oil / burning oil / kerosene price 行或 sheet，并统一为 pence/litre。

### 3.4 Reference price construction

以 2021-04 至 2021-09 作为 pre-shock reference window：

```text
reference_price(region, carrier) = mean(monthly_price(region, carrier) over Apr-Sep 2021)
```

这允许后续所有事件窗口都用同一个 pre-shock baseline 比较。

## 4. Property-level costing 逻辑

对于每个 property `d`、scenario `s`、month `m`：

### 4.1 Electricity

```text
electricity_cost_actual = (
    electricity_import_kwh * electricity_unit_p_per_kwh
    + days_in_month * electricity_standing_p_per_day
    - electricity_export_kwh * export_tariff_p_per_kwh
) / 100
```

### 4.2 Gas

```text
gas_cost_actual = (
    gas_kwh * gas_unit_p_per_kwh
    + days_in_month * gas_standing_p_per_day
) / 100
```

### 4.3 Heating oil / road fuel

```text
oil_cost_actual = oil_litres * oil_p_per_litre / 100
petrol_cost_actual = petrol_litres * petrol_p_per_litre / 100
diesel_cost_actual = diesel_litres * diesel_p_per_litre / 100
```

## 5. Shock metrics

### 5.1 Monthly shock increment

```text
shock_increment_monthly = total_cost_actual_monthly - total_cost_reference_monthly
```

注意这里 quantities 固定，价格变化是唯一变化项，因此该指标隔离的是 **price effect**，不是天气、行为或需求变化。

### 5.2 Event aggregation

```text
shock_increment_event = sum(shock_increment_monthly over event window)
```

### 5.3 Exposure index

```text
exposure_index = shock_increment_event / reference_cost_event
```

### 5.4 Risk uplift / burden uplift

接入 Westminster 现有收入引擎后：

```text
baseline_required_energy_burden = reference_required_cost / residual_income_after_housing_costs
shocked_required_energy_burden = actual_required_cost_under_scenario / residual_income_after_housing_costs
risk_uplift = shocked_required_energy_burden - baseline_required_energy_burden
```

也可以扩展到 6%、10%、15%、20% thresholds：

```text
threshold_crossing_prob_t = P(shocked_required_energy_cost / Y_l > t)
```

其中 `Y_l` 是现有 LSOA lognormal income distribution。

## 6. Standing charge 单独拆分

S6 的重点是 fixed-cost exposure。应在 cost table 中单独保留：

```text
electricity_unit_cost_component
electricity_standing_charge_component
gas_unit_cost_component
gas_standing_charge_component
```

然后输出：

```text
standing_charge_share = (electricity_standing + gas_standing) / total_energy_cost
standing_charge_shock = standing_charge_actual - standing_charge_reference
unit_rate_shock = unit_cost_actual - unit_cost_reference
```

这对低用量家庭、小户型、低收入 household 尤其重要，因为 fixed charges 可能在总账单中占比较高。

## 7. Westminster 主模型设置

| 参数 | 建议值 / 处理 |
|---|---|
| Ofgem region | London |
| Payment methods | Direct Debit, standard credit, prepayment 三组都跑；主文可用 Direct Debit，稳健性用其余两组 |
| VAT | 全部统一成 incl VAT 或 ex VAT；推荐 incl VAT，因 household bill burden 更直观 |
| Domestic scope | 主模型 electricity/gas/oil；road fuel/EV 作为 appendix |
| Heating oil | 只应用于 EPC oil/off-gas heating exposure |
| EPG | 与 unmitigated Ofgem 2022Q4 分开，形成 policy mitigation 对照 |
| Income | 接入现有 MSOA AHC income 和 LSOA lognormal income distribution |
| Output scale | property-level → LSOA/MSOA/tenure/EPC/heating-system/pathway summaries |

## 8. 推荐 Cursor 实现任务

```text
1. 新建 scripts/build_official_price_spine.py
2. 输入 data/raw/ofgem/*.xlsx, data/raw/desnz/*.csv/xlsx, data/processed/*scenario*.csv
3. 输出 outputs/price_spine/official_price_spine_monthly.csv
4. 新建 scripts/run_fuel_price_shock_scenarios.py
5. 读取 property-level demand/cost fields and price spine
6. 输出 property-level event cost/shock/burden metrics
7. 输出 LSOA/MSOA/pathway aggregation tables
```

### 8.1 必须 QA

- Ofgem workbook period count 与 scenario period 是否一致。
- electricity/gas unit rate 与 standing charge 是否拆分。
- VAT status 是否统一。
- Westminster/London region 是否正确匹配。
- 每个 scenario 的 month count 是否正确。
- DESNZ weekly road fuel 是否按 calendar month 平均。
- heating oil 单位是否是 pence/litre，而不是 pounds/1000 litres 或 pence/1000 litres。
- raw property row count 不变。
- standing charge cost 不随 kWh 变化。
- EPG scenario 与 unmitigated Ofgem scenario 分开保存。

## 9. 关键产出文件建议

```text
outputs/price_spine/official_price_spine_monthly.csv
outputs/price_spine/price_spine_qa_summary.csv
outputs/fuel_price_scenarios/property_event_results.csv
outputs/fuel_price_scenarios/lsoa_event_summary.csv
outputs/fuel_price_scenarios/pathway_event_summary.csv
outputs/fuel_price_scenarios/standing_charge_decomposition.csv
```

## 10. Source URLs

- Ofgem default tariff cap levels: https://www.ofgem.gov.uk/energy-regulation/domestic-and-non-domestic/energy-pricing-rules/energy-price-cap/energy-price-cap-default-tariff-levels
- Ofgem unit rates and standing charges: https://www.ofgem.gov.uk/information-consumers/energy-advice-households/energy-price-cap-unit-rates-and-standing-charges
- Ofgem price cap and standing charges explained: https://www.ofgem.gov.uk/information-consumers/energy-advice-households/energy-price-cap-and-standing-charges-explained
- GOV.UK Energy Price Guarantee regional rates: https://www.gov.uk/government/publications/energy-price-guarantee-regional-rates/energy-price-guarantee-regional-rates
- DESNZ monthly petroleum prices: https://www.gov.uk/government/statistical-data-sets/oil-and-petroleum-products-monthly-statistics
- DESNZ weekly road fuel prices: https://www.gov.uk/government/statistics/weekly-road-fuel-prices

## 11. Visual overview

### Scenario timeline

![Fuel price scenario timeline](../figures/fuel_price_scenario_timeline.png)

### Implementation workflow

![Fuel price implementation workflow](../figures/fuel_price_implementation_workflow.png)
