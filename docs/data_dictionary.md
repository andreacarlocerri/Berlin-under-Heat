# Data dictionary

This page describes the cleaned tables in `data/clean/` and the table behind the Tableau dashboard in `tableau/`. All tables have one row per day and are joined on `date`.

## `data/clean/daily_summer.csv`

The main analysis table, built in notebook 02. It covers 25 May to 31 August, 2018–2026 (891 days). The 63 late-May days are helper rows for the 7-day history in notebook 06; the analysis uses the 828 June–August days, of which 822 have mission data.

| Column | Type | Meaning |
|---|---|---|
| `date` | date | Calendar day |
| `tmax` | float | Daily maximum temperature in °C at DWD station 00433, Berlin-Tempelhof |
| `quality` | int | DWD quality level. 9 = historical archive (2018–2025); 3 or 1 = recent archive (2026, preliminary) |
| `ems_missions` | int | Emergency medical (EMS) missions created that day, whole city (Berlin Fire Brigade). Empty for the 6 days without usable data |
| `cpr_alert_missions` | int | EMS missions dispatched with a CPR alert (suspected need for resuscitation at the time of the call, not a confirmed cardiac arrest) |
| `warning_level` | int | DWD heat warning for Berlin: 0 = none, 1 = strong heat stress, 3 = extreme heat stress |
| `heat_warning` | 0/1 | 1 if any DWD heat warning was issued for Berlin that day |
| `hot_day` | 0/1 | 1 if `tmax` ≥ 30°C (DWD's definition of a hot day) |
| `year`, `month` | int | Calendar year and month |
| `weekday` | text | Day name, Monday to Sunday |
| `weekday_num` | int | 0 = Monday … 6 = Sunday |
| `in_summer` | 0/1 | 1 for June–August, 0 for the late-May helper days |

## Other cleaned tables

| File | Rows | Content |
|---|---|---|
| `missions_summer_clean.csv` | 822 | `date`, `ems_missions`, `cpr_alert_missions` for June–August 2018–2026 |
| `weather_clean.csv` | 891 | `date`, `tmax`, `quality` for 25 May – 31 August 2018–2026 |
| `heat_warnings_clean.csv` | 97 | `date`, `warning_level`, only for days with a warning |

## `tableau/tableau_daily.csv`

The table behind the Tableau dashboard, built in notebook 05. One row per summer day with mission data (822 rows). It adds these columns to the ones above:

| Column | Meaning |
|---|---|
| `ems_index` | EMS demand index: that day's missions ÷ that summer's average × 100. 100 = an average day of that summer |
| `temp_band` | 2°C band of `tmax`: <20, 20–22, …, 32–34, 34+. Lower bound included, upper bound excluded |
| `temp_band_order` | 1–9, sort order for `temp_band` |
| `day_label` | "Hot day (≥30°C)" or "Normal day" |
| `warning_label` | "No warning", "Strong heat stress" or "Extreme heat stress" |
| `day_type` | "Hot + warning", "Hot, no warning", "Warning, not hot" or "Normal day" |

## Excluded and missing days

- **7 June 2018** is excluded as a suspected reporting anomaly: EMS missions fell by about 47% compared with the neighbouring days. The raw value stays in `data/raw/`.
- **19 Jun, 21 Jun, 13 Jul, 23 Jul and 24 Aug 2026** are missing from the Fire Brigade source data and are not filled in.

## Other Tableau data files

`tableau/` also contains the other tables exported by notebooks 05–07 (yearly summary, response times, full-year data, forecasts, model scores and key numbers). The dashboard only uses `tableau_daily`; the packaged workbook carries its own copy of that data.
