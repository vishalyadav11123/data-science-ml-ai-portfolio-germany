# DAX Measures and Calculated Columns

Project: **Germany Economy Snapshot Dashboard**  
Dataset table: **economy_data**  
Tool: **Power BI Online**

This file documents the calculated columns and measures used in the Power BI dashboard.

---

## 1. Calculated Columns

Calculated columns were created to make the World Bank dataset easier to display in visuals, slicers, cards, and tables.

---

### `country_icon`

Purpose: Adds a small country/region icon for the indicator table.

```DAX
country_icon =
SWITCH(
    economy_data[country],
    "Germany", "🇩🇪",
    "European Union", "🇪🇺",
    "OECD members", "🌐",
    "World", "🌐",
    "🌐"
)
```

---

### `indicator_label`

Purpose: Shortens long World Bank indicator names for chart legends, slicers, cards, and tables.

```DAX
indicator_label =
SWITCH(
    economy_data[indicator_name],
    "GDP (current US$)", "GDP",
    "GDP growth (annual %)", "GDP Growth",
    "Inflation, consumer prices (annual %)", "Inflation",
    "Unemployment, total (% of total labor force) (modeled ILO estimate)", "Unemployment",
    "Foreign direct investment, net inflows (% of GDP)", "FDI",
    "Exports of goods and services (% of GDP)", "Exports",
    "Imports of goods and services (% of GDP)", "Imports",
    "GDP per capita (current US$)", "GDP/capita",
    "Population, total", "Population",
    economy_data[indicator_name]
)
```

---

### `unit`

Purpose: Adds a unit column for the latest indicator values table.

```DAX
unit =
SWITCH(
    TRUE(),
    economy_data[indicator_name] = "GDP (current US$)", "US$",
    economy_data[indicator_name] = "GDP per capita (current US$)", "US$",
    economy_data[indicator_name] = "Population, total", "People",
    CONTAINSSTRING(economy_data[indicator_name], "%"), "%",
    ""
)
```

---

### `source`

Purpose: Adds the source name for the final dashboard table.

```DAX
source =
"World Bank"
```

---

### `display_value`

Purpose: Creates clean table values such as `3.60T`, `44.72K`, and percentage values.

This column is used only for display tables, not for charts or numeric calculations.

```DAX
display_value =
SWITCH(
    TRUE(),

    economy_data[indicator_name] = "GDP (current US$)",
        FORMAT(economy_data[value] / 1000000000000, "0.00") & "T",

    economy_data[indicator_name] = "GDP per capita (current US$)",
        FORMAT(economy_data[value] / 1000, "0.00") & "K",

    economy_data[indicator_name] = "Population, total",
        FORMAT(economy_data[value] / 1000000, "0.00") & "M",

    CONTAINSSTRING(economy_data[indicator_name], "%"),
        FORMAT(economy_data[value], "0.0"),

    FORMAT(economy_data[value], "0.0")
)
```

---

### `table_sort`

Purpose: Controls the row order in the indicator values table.

```DAX
table_sort =
SWITCH(
    TRUE(),

    economy_data[country] = "Germany" &&
    economy_data[indicator_name] = "Exports of goods and services (% of GDP)", 1,

    economy_data[country] = "Germany" &&
    economy_data[indicator_name] = "Foreign direct investment, net inflows (% of GDP)", 2,

    economy_data[country] = "Germany" &&
    economy_data[indicator_name] = "GDP (current US$)", 3,

    economy_data[country] = "Germany" &&
    economy_data[indicator_name] = "GDP growth (annual %)", 4,

    economy_data[country] = "Germany" &&
    economy_data[indicator_name] = "GDP per capita (current US$)", 5,

    economy_data[country] = "Germany" &&
    economy_data[indicator_name] = "Imports of goods and services (% of GDP)", 6,

    economy_data[country] = "Germany" &&
    economy_data[indicator_name] = "Inflation, consumer prices (annual %)", 7,

    economy_data[country] = "Germany" &&
    economy_data[indicator_name] = "Unemployment, total (% of total labor force) (modeled ILO estimate)", 8,

    99
)
```

---

### `show_latest_table`

Purpose: Filters the latest indicator values table to keep only selected dashboard rows.

```DAX
show_latest_table =
IF(
    economy_data[table_sort] < 99,
    1,
    0
)
```

---

## 2. Measures

Measures were used for KPI cards. Percentage indicators from World Bank are already stored as percentage-point values, for example `2.2` means `2.2%`. Therefore, the measure divides by 100 before formatting as a percentage in Power BI.

---

### `GDP Growth %`

Purpose: Shows GDP growth as a proper percentage in the KPI card.

```DAX
GDP Growth % =
DIVIDE(
    CALCULATE(
        AVERAGE(economy_data[value]),
        REMOVEFILTERS(economy_data[indicator_name]),
        economy_data[indicator_name] = "GDP growth (annual %)"
    ),
    100
)
```

Recommended formatting in Power BI:

```text
Format = Percentage
Decimal places = 1
```

---

### `Inflation %`

Purpose: Shows inflation as a proper percentage in the KPI card.

```DAX
Inflation % =
DIVIDE(
    CALCULATE(
        AVERAGE(economy_data[value]),
        REMOVEFILTERS(economy_data[indicator_name]),
        economy_data[indicator_name] = "Inflation, consumer prices (annual %)"
    ),
    100
)
```

Recommended formatting in Power BI:

```text
Format = Percentage
Decimal places = 1
```

---

### `Unemployment %`

Purpose: Shows unemployment as a proper percentage in the KPI card.

```DAX
Unemployment % =
DIVIDE(
    CALCULATE(
        AVERAGE(economy_data[value]),
        REMOVEFILTERS(economy_data[indicator_name]),
        economy_data[indicator_name] = "Unemployment, total (% of total labor force) (modeled ILO estimate)"
    ),
    100
)
```

Recommended formatting in Power BI:

```text
Format = Percentage
Decimal places = 1
```

---

### `Foreign Direct Investment %`

Purpose: Shows FDI net inflows as a proper percentage in the KPI card.

```DAX
Foreign Direct Investment % =
DIVIDE(
    CALCULATE(
        AVERAGE(economy_data[value]),
        REMOVEFILTERS(economy_data[indicator_name]),
        economy_data[indicator_name] = "Foreign direct investment, net inflows (% of GDP)"
    ),
    100
)
```

Recommended formatting in Power BI:

```text
Format = Percentage
Decimal places = 1
```

---

## 3. Notes

- The original `value` column was kept numeric and used for charts.
- The `display_value` column was used only for the final table because it contains formatted text such as `3.60T` and `44.72K`.
- The `indicator_label` column was used to make chart legends cleaner.
- The `source` column was added so the dashboard clearly states that the data comes from the World Bank.
- The measures use `REMOVEFILTERS(economy_data[indicator_name])` so KPI cards do not break when an indicator slicer is used.
