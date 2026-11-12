# Berlin Under Heat

**Does summer heat put more pressure on Berlin's emergency medical services?**

This project combines Berlin Fire Brigade open data with German Weather Service (DWD) temperatures and heat warnings for nine summers (June–August, 2018–2026, 822 days). Final project of the Ironhack Data Analytics bootcamp, November 2026.

**Presentation:** [Google Slides](https://docs.google.com/presentation/d/1N-xmSMpJOtyMYb7xWUeDEqQ-kdMrranp8ifUUBq_TRY/edit)

**Project files:** [Film](Berlin_Under_Heat_65s.mp4) · [Packaged Tableau workbook](tableau/Berlin-under-Heat_forecast.twbx) · [Data dictionary](docs/data_dictionary.md)

## Research questions

1. Do hot days (daily maximum of at least 30°C) have more emergency medical (EMS) missions than normal summer days?
2. How does demand change across temperature ranges?
3. Do hot days without an official DWD heat warning still show higher demand?
4. Can the next day's number of EMS missions be predicted, and does temperature improve that prediction?
5. What do hot days look like in practice: how many more missions, and do ambulances take longer to arrive?

## Methodology

1. **Collect.** Three open datasets are downloaded with Python `requests` (notebook 01): daily EMS missions from the Berlin Fire Brigade, daily maximum temperature at Berlin-Tempelhof from DWD, and DWD's historical heat warnings.
2. **Clean and join.** Each dataset is validated and the three are joined by date for June–August 2018–2026 (notebook 02). One suspected reporting anomaly (7 June 2018) is excluded and five 2026 days are missing at the source, which leaves 822 summer days.
3. **Define.** A *hot day* is a day with a maximum of at least 30°C, DWD's own definition (155 days). Because demand grows from year to year, each day is compared with its own summer through the *EMS demand index*: missions ÷ that summer's average × 100.
4. **Explore.** Distributions, correlations, weekday patterns and 2°C temperature bands (notebook 03).
5. **Test.** A one-sided Welch t-test compares hot days with other summer days (H0: hot days are not busier, α = 0.05). A sign test checks consistency across the nine summers, ANOVA and Kruskal-Wallis compare temperature bands, and a chi-square test checks that hot days are spread evenly across weekdays. An OLS regression then compares only days of the same year, month and weekday, with standard errors that allow for heatwave streaks (HAC), and is re-run without 2026 (notebook 04).
6. **Extend.** The same model in absolute missions, critical-EMS response times (split at the 2025 change in the critical category), a full-year view from January 2018 to November 2026, and a public-holiday check (notebook 07).
7. **Predict.** A linear regression, a random forest and gradient boosting are compared with a naive forecast (the previous 7 days' average). Models are selected with leave-one-summer-out cross-validation on 2018–2025 and tested once on summer 2026, using mean absolute error (MAE) (notebook 06).
8. **Visualise.** Notebook 05 exports the daily table to Tableau. The dashboard *Live Demo* shows how many hot days had no heat warning and how busy those days were. Clicking a temperature bar or a summer filters the demand chart.

## Key findings

- **Hot days were associated with about 90 more EMS missions per day** after adjusting for year, month and weekday (95% CI 74–105).
- **Indexed demand was higher on hot days.** EMS demand is about 7% higher on hot days: +7.1 index points (95% CI 5.9–8.2) after adjusting for year, month and weekday.
- **The pattern is consistent.** Hot days had higher demand in all nine summers, and the result barely changes when 2026 and its record heatwave are excluded.
- **Response times are longer.** The average daily median response time for critical EMS missions is about 17 seconds higher on hot days (95% CI 14–20), and it was higher in all nine summers.
- **Demand rises gradually.** Each additional °C is associated with about +0.8 index points. The temperature bands show no clear break at 30°C in this sample.
- **Hot days without warnings were busier.** 72 of 155 observed hot days had no DWD warning. Their mean EMS index was 5.1 points higher than on cooler days without a warning (unadjusted). The adjusted warning estimate was +1.3 index points (95% CI −0.8 to +3.4). For EMS planning, this suggests watching the temperature forecast alongside official warnings, and testing a 30°C trigger with archived forecasts.
- **Full-year context.** Mean EMS demand was higher in warmer temperature bands in the descriptive full-year comparison.
- **A forecast experiment works.** Given the measured temperature of the day, a linear regression predicts next-day demand with an average error of 42.8 missions in cross-validation, against 56.9 for a naive forecast. On the unseen summer of 2026 the error is 54.8 against 82.0. With a simulated temperature error of about 2°C it rises to about 58. The model has not been tested with real weather forecasts.

All results are associations from observational data, not proof of cause and effect.

## Data

| Source | Content | Licence and citation |
|---|---|---|
| [Berliner Feuerwehr Open Data](https://github.com/Berliner-Feuerwehr/BF-Open-Data) | Daily mission counts and response times for the whole city | CC BY 4.0. "BFw Mission Data Daily, Berliner Feuerwehr, Berlin 2026" |
| [DWD Climate Data Center](https://opendata.dwd.de/climate_environment/CDC/observations_germany/climate/daily/kl/) | Daily maximum temperature, station 00433 Berlin-Tempelhof | Source: Deutscher Wetterdienst |
| [DWD historical heat alerts](https://opendata.dwd.de/climate_environment/health/historical_alerts/heat_warnings/) | Official heat warnings for Berlin | DWD: Historische Hitzewarnungen in Deutschland, Version v001, 2021 |

Data was downloaded on 5 November 2026. The Fire Brigade file and the recent DWD file are updated daily, so a later download may differ slightly.

**EMS demand index:** a day's missions divided by that summer's average, times 100. It makes days comparable across summers, because demand grows from year to year.

## Project structure

```
data/raw/        downloaded source files
data/clean/      cleaned tables, including daily_summer.csv (one row per day)
notebooks/       analysis, in order (see below)
figures/         charts saved by the notebooks
docs/            additional documentation (data dictionary)
tableau/         dashboard workbook (open Berlin-under-Heat_forecast.twbx) and its data files
```

| Notebook | Content |
|---|---|
| `01_data_collection` | Downloads the raw files |
| `02_cleaning` | Cleans and validates the three datasets and joins them by date |
| `03_eda` | Exploratory analysis: distributions, correlations, temperature bands |
| `04_hypothesis_testing` | Statistical tests and adjusted regression models |
| `05_tableau_export` | Prepares the tables for the Tableau dashboard |
| `06_predictive_modelling` | Forecast experiment for next-day EMS missions with scikit-learn |
| `07_extensions` | Absolute numbers, response times, full-year context and a holiday check |

## How to run

Run the notebooks in order from the `notebooks` folder. Notebook 01 can be skipped, because the raw files are included in the repository.

Use Python 3.14.7, the version used for the reproducibility check. From the repository root:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
cd notebooks
python -m notebook
```

Start at notebook 02 to reproduce the bundled snapshot. Notebook 01 refreshes the raw inputs from live sources.

## Tools

Python (pandas, SciPy, statsmodels, scikit-learn, matplotlib, seaborn) and Tableau.

## Limitations

- One weather station (Tempelhof) represents the whole city.
- Daily maximum temperature does not capture all heat exposure, such as warm nights.
- Five summer days are missing in the 2026 mission data, and one suspected reporting anomaly (7 June 2018) is excluded.
- 2026 weather data is preliminary.
- The forecast experiment uses measured temperature in place of a weather forecast, so real-world errors would be larger.
- Response times cover critical missions only, and the way missions are classified as critical appears to change in 2025.

## Key learnings

- **Check definitions before comparing years.** The number of critical EMS missions dropped by about 40% from 2025. That reflects a change in the category, not fewer emergencies, so response times are analysed separately before and after it.
- **Scope down first.** SQL and a cost estimate were dropped from the plan so the three core questions could be answered properly; response times, the full-year view and the forecast were added afterwards.
- **Simple can win.** On the unseen summer of 2026, a plain linear regression beat random forest and gradient boosting, because demand was higher than anything the tree models had seen in training.

## Author

Andrea Carlo Cerri
