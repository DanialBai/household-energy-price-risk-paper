# Fuel price scenario 所需数据清单与数据描述

> 版本：2026-05-27  
> 本文档描述已下载或整理到 `/mnt/data/fuel_price_work/data/` 的官方价格数据。  
> 数据用途：构建 Westminster household fuel price risk 的 official price spine、真实 shock scenarios 与 standing charge decomposition。

## 1. 文件目录

```text
fuel_price_work/
├── data/
│   ├── raw/
│   │   ├── ofgem/   # Ofgem default tariff cap historical/current workbooks
│   │   └── desnz/   # DESNZ petroleum and road-fuel official datasets
│   └── processed/   # manually structured scenario/rate CSVs from official web tables
├── docs/            # markdown documentation
└── figures/         # visual workflow diagrams
```

## 2. 已下载 raw datasets

| 文件 | 来源 | 官方机构 | 覆盖/用途 | 主要信息 | 备注 |
|---|---|---|---|---|---|
| `data/raw/ofgem/ofgem_default_tariff_cap_level_v1.8_2021-04_2021-09.xlsx` | Ofgem default tariff cap levels | Ofgem | S0 reference baseline | electricity/gas cap model; unit/standing derivation inputs | Apr-Sep 2021 reference period |
| `data/raw/ofgem/ofgem_default_tariff_cap_level_v1.9_2021-10_2022-03.xlsx` | Ofgem default tariff cap levels | Ofgem | S1 post-COVID uplift | electricity/gas cap model | Oct 2021-Mar 2022 |
| `data/raw/ofgem/ofgem_default_tarif_cap_level_v1.10_2022-04_2022-09.xlsx` | Ofgem default tariff cap levels | Ofgem | S2 severe uplift | electricity/gas cap model | Official filename contains `tarif` typo in URL/path |
| `data/raw/ofgem/ofgem_default_tariff_cap_level_v1.13_2022-10_2022-12.xlsx` | Ofgem default tariff cap levels | Ofgem | S3 Ukraine unmitigated cap exposure | electricity/gas cap model | Oct-Dec 2022 |
| `data/raw/ofgem/ofgem_annex9_levelised_cap_levels_v1.8_2026-01_2026-03.xlsx` | Ofgem Annex 9 | Ofgem | S6 comparison period | levelised cap levels and payment/region logic | Jan-Mar 2026 |
| `data/raw/ofgem/ofgem_annex9_levelised_cap_levels_v1.9_2026-04_2026-06.xlsx` | Ofgem Annex 9 | Ofgem | S6 standing-charge scenario | levelised cap levels and payment/region logic | Apr-Jun 2026 |
| `data/raw/ofgem/ofgem_default_tariff_cap_table_of_public_information_2026.xlsx` | Ofgem public information table | Ofgem | metadata / public input sources | public information for Default Tariff Cap model | current public-input reference |
| `data/raw/desnz/desnz_petroleum_prices_qep_4_1_1_to_4_1_3_2026-04.xlsx` | Monthly and annual prices of road fuels and petroleum products | DESNZ | S5 heating-oil/off-gas scenario | monthly petroleum products, road fuels, home-heating fuels | official monthly workbook |
| `data/raw/desnz/desnz_weekly_road_fuel_prices_2018_2026_2026-05-25.csv` | Weekly road fuel prices | DESNZ | S7 optional road-fuel/EV appendix | weekly petrol/diesel pence/litre and duty/VAT | CSV, 2018-2026 |

## 3. Processed / structured helper CSVs

| 文件 | 来源 | 用途 | 字段 |
|---|---|---|---|
| `data/processed/fuel_price_scenario_set.csv` | 研究设计整理 | scenario engine 控制表 | scenario ID, window, primary fuels, source priority, notes |
| `data/processed/govuk_epg_oct_dec_2022_regional_unit_rates.csv` | GOV.UK Energy Price Guarantee regional rates | S4 policy-mitigated observed 2022Q4 unit-rate scenario | payment method, region, electricity/gas unit rates ex VAT, period |
| `data/processed/ofgem_average_price_cap_rates_2026q2_q3.csv` | Ofgem unit rates and standing charges page | S6 average standing charge/unit-rate reference | period, fuel, unit rate incl VAT, daily standing charge incl VAT |
| `data/processed/downloaded_file_checksums.csv` | local generated manifest | reproducibility | file path, file size, sha256 checksum |

## 4. 官方来源说明

### 4.1 Ofgem default tariff cap levels

- URL: https://www.ofgem.gov.uk/energy-regulation/domestic-and-non-domestic/energy-pricing-rules/energy-price-cap/energy-price-cap-default-tariff-levels
- 数据类型：official guidance and Excel workbooks。
- 模型用途：electricity/gas unit rate 与 daily standing charge 的历史 price spine 主干。
- 粒度：cap period × Ofgem region × payment method × meter/fuel type。
- 重要性：这是 household-facing regulated retail pass-through 的正式来源。

### 4.2 Ofgem current unit rates and standing charges

- URL: https://www.ofgem.gov.uk/information-consumers/energy-advice-households/energy-price-cap-unit-rates-and-standing-charges
- 数据类型：consumer-facing official price cap rates。
- 模型用途：当前 / 近未来 standing-charge 与 unit-rate structure sensitivity。
- 已整理到：`ofgem_average_price_cap_rates_2026q2_q3.csv`。
- 注意：页面表格中的 consumer-facing figures 已包含 5% VAT，且 rounded to two decimals。

### 4.3 GOV.UK Energy Price Guarantee regional rates

- URL: https://www.gov.uk/government/publications/energy-price-guarantee-regional-rates/energy-price-guarantee-regional-rates
- 数据类型：transparency data。
- 覆盖：2022-10-01 至 2022-12-31。
- 字段：region, payment method, electricity single-rate p/kWh ex VAT, gas p/kWh ex VAT。
- 模型用途：S4 `ukraine_epg_observed_2022Q4`，用于和 Ofgem unmitigated cap scenario 对照。
- 注意：该表只给 unit rates；如果要做完整账单，需要另接 standing charge 或清楚注明只模拟 unit-rate policy mitigation。

### 4.4 DESNZ monthly petroleum prices

- URL: https://www.gov.uk/government/statistical-data-sets/oil-and-petroleum-products-monthly-statistics
- 数据类型：official statistical data workbook。
- 覆盖：monthly and annual petroleum product prices。
- 模型用途：heating oil/off-gas scenario。Westminster 中只应用于 EPC heating oil / off-gas subset。
- 注意：不同表可能使用 £/1000 litres、pence/litre 或 index，需要在 parser 中确认单位。

### 4.5 DESNZ weekly road fuel prices

- URL: https://www.gov.uk/government/statistics/weekly-road-fuel-prices
- 数据类型：accredited official statistics。
- 字段：ULSP petrol pump price, ULSD diesel pump price, duty rate, VAT percentage。
- 模型用途：optional road-fuel/EV appendix。
- 处理：weekly rows → calendar-month average。

## 5. 单位与 VAT 处理原则

| 数据 | 常见单位 | 建议模型单位 | VAT |
|---|---|---|---|
| Ofgem historical cap models | £/year benchmark charges; p/kWh derivable | p/kWh, p/day | workbook often ex VAT; verify and standardise |
| Ofgem current consumer page | p/kWh, p/day | p/kWh, p/day | incl VAT |
| EPG regional rates | p/kWh | p/kWh | ex VAT |
| DESNZ road fuel | p/litre | p/litre | pump price includes applicable taxes; duty/VAT columns also present |
| DESNZ heating oil | varies by table | p/litre | verify from workbook notes |

推荐全部 price spine 存两个字段：

```text
price_value
vat_status = ex_vat | inc_vat | pump_price | unknown_check_notes
```

正式 energy-burden 输出应使用 household-facing **inc VAT** price。若某些 raw input 为 ex VAT，则统一乘以 `1.05`。

## 6. 下载状态与完整性

`data/processed/downloaded_file_checksums.csv` 记录了所有 raw/processed 数据文件的 byte size 与 SHA256 checksum。后续若重新下载，可用该文件检查版本差异。

## 7. 下一步处理建议

1. 写 `scripts/build_official_price_spine.py`，读取 raw Ofgem/DESNZ 文件并输出 monthly price spine。
2. 写 `scripts/validate_price_spine.py`，检查日期覆盖、London region、payment method、VAT、单位和 missing values。
3. 将 price spine 与 Westminster property-level demand/cost dataset 连接。
4. 生成 property-event-level shock metrics。
5. 聚合到 LSOA/MSOA/pathway，并输出 standing charge decomposition。

## 8. Caveats

- EPG regional rates表是 unit-rate policy support，不是完整总账单 cap。完整账单需要配合 standing charge。
- Ofgem default tariff cap 保护的是 default tariffs 上的 household，不覆盖 fixed tariffs、business contracts、heat networks 或 heating oil users。
- Heating oil 属于 unregulated off-gas fuel exposure，不能用 Ofgem cap 替代。
- Road fuel/EV 不属于 domestic heating/electricity 主模型必需项；只在保留 transport-energy exposure 时使用。

## 9. Visual overview

### Scenario timeline

![Fuel price scenario timeline](../figures/fuel_price_scenario_timeline.png)

### Implementation workflow

![Fuel price implementation workflow](../figures/fuel_price_implementation_workflow.png)
