# Workshop C — Technology Professionals' Salaries

This project studies how experience, job characteristics, reported technologies and country conditions are related to annual salary. It combines the **Stack Overflow Developer Survey 2024** with **World Bank indicators for 2024**.

The solution is in `notebooks/WorkshopC.ipynb`. All code uses Python classes, and the explanations are written in simple English. The notebook contains its own package setup, results, discussions and a seven-minute presentation guide.

## Team

| Team member | Student ID |
|---|---|
| [Add name] | [Add ID] |
| [Add name] | [Add ID] |
| [Add name] | [Add ID] |
| [Add name] | [Add ID] |

**GitHub repository:** [Add repository URL]

## Project files

| Path | Purpose |
|---|---|
| `data/stackoverflow_2024.csv` | Complete survey response data |
| `data/*_Data.csv` | World Bank extract with the four indicators and the 2024 column |
| `data/*Metadata.csv` | World Bank documentation; keep for reference |
| `notebooks/WorkshopC.ipynb` | Complete OOP analysis and saved outputs |
| `README.md` | Project description and running instructions |

Keep your current World Bank filenames. The notebook finds the file ending in `_Data.csv`, so the longer UUID filename and a shorter name such as `8577022c_Data.csv` both work. If the folder contains multiple matching data files, set `WORLD_BANK_FILENAME` in section 1.2. No separate Python module or generated `requirements.txt` is required.

## Run the notebook

1. Put the source CSVs in the `data` folder. Extract the World Bank ZIP first.
2. Open the project in VS Code or Jupyter and select your Python environment. Python 3.10 or newer is recommended; the saved run used Python 3.12.
3. Open `notebooks/WorkshopC.ipynb`.
4. Run all cells from top to bottom. The setup cell installs missing analysis packages into the selected kernel. A first-time installation needs internet access.
5. Fill in the names, IDs and GitHub URL in the first Markdown cell, then save the notebook with its outputs.

The notebook works when Jupyter starts in the project root or the `notebooks` folder. Once packages and source CSVs are available, the analysis runs locally without downloading data. Run All reads the original inputs without changing them and does not write extra output files.

If VS Code cannot offer your environment as a notebook kernel, install `ipykernel` in that environment. The analysis uses `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, `scikit-learn`, `statsmodels` and `ipython`. The saved output includes the actual versions used.

## Data sources

### Stack Overflow Developer Survey 2024

- [Survey results and methodology](https://survey.stackoverflow.co/2024/)
- [Official survey archive](https://github.com/StackExchange/Survey/tree/main/packages/archive/2024)
- [Response CSV](https://media.githubusercontent.com/media/StackExchange/Survey/refs/heads/main/packages/archive/2024/results.csv)
- [Question schema](https://github.com/StackExchange/Survey/raw/refs/heads/main/packages/archive/2024/schema.csv)

The uploaded response file has **65,437 rows and 114 columns**. `ConvertedCompYearly` is the annual compensation converted to US dollars. We use it as the target and exclude the raw compensation field `CompTotal` from the predictors. The survey is self-reported and self-selected; it is not a representative salary census. Follow the source's data licence and attribution terms when sharing data.

For a large CSV, use your browser's **Save link as** option instead of displaying the file. Keep the input available locally; the repository can document its download location instead of storing a large copy.

### World Bank — World Development Indicators

[World Development Indicators](https://databank.worldbank.org/source/world-development-indicators)

Download countries/economies, **2024 only**, and these indicators:

| Indicator | Code | Study variable |
|---|---|---|
| GDP per capita (current US$) | `NY.GDP.PCAP.CD` | `GDPPerCapita10k` |
| Individuals using the Internet (% of population) | `IT.NET.USER.ZS` | `InternetUsersPct` |
| Fixed broadband subscriptions (per 100 people) | `IT.NET.BBND.P2` | `BroadbandPer100` |
| Unemployment, total (% of total labor force), modeled ILO estimate | `SL.UEM.TOTL.ZS` | `UnemploymentPct` |

The expected extract columns are `Country Name`, `Country Code`, `Series Name`, `Series Code`, and `2024 [YR2024]`. The data preparation converts `..` to missing values and reshapes the table to one row per country/economy and year. It uses an explicit country-name map and a validated many-to-one join. Regional and income-group totals do not match survey respondents.

GDP per capita is divided by 10,000 to make its coefficient readable. The other indicators retain their original units. These describe national context, not a respondent's own income, internet connection or employment probability. Annual 2024 indicators make this a retrospective study.

## Study population and preparation

The main analysis keeps respondents who:

1. Have a unique response ID.
2. Selected exactly `Employed, full-time`, excluding mixed employment selections.
3. Reported a positive annual salary between **USD 1,000 and USD 500,000**, inclusive.
4. Have both coding-experience measures and all four World Bank values.
5. Have total coding experience at least as large as professional coding experience.

The salary bounds are a declared scope choice, not proof that excluded reports are errors. They exclude some legitimate salaries. An additional simple-OLS sensitivity check repeats the analysis without these salary bounds and discusses the effect of extreme reports.

Optional categorical answers use `Unknown`. Technology indicators mean that a technology was reported, and separate flags identify unanswered technology questions. `Less than 1 year` becomes 0.5; `More than 50 years` becomes 51. Company sizes and related roles are grouped using visible rules in the code.

The main saved run contains **15,782 respondents from 123 countries/economies** and **33 candidate predictors before category encoding**. IDs and the constant survey year are not counted as predictors. The initial encoded design has **64 non-constant predictors**. The notebook displays the complete dictionary, category references and row-removal audit.

## Methods and OOP design

| Class | Main job |
|---|---|
| `SalaryData` | Load, join, clean and prepare the study table |
| `SplitReview` | Create the fixed split and compare the two datasets |
| `StudyEncoder` | Learn category levels on training data and create consistent numeric columns |
| `RegressionStudy` | Fit OLS models, calculate metrics and run backward selection |

The workflow follows the supplied `PROG8431-SLR-Demo-Notebook.ipynb` and `PROG8431-MLR-Demo.ipynb`:

- **80% training / 20% test**, `random_state=6174`: 12,625 training and 3,157 test respondents in the saved run.
- Summary statistics, KS tests, chi-square tests, category percentages and technology reporting percentages compare the datasets. Sparse chi-square cells are reported.
- Training plots and Spearman correlations describe relationships.
- Two simple OLS models use professional coding experience and GDP per capita separately.
- The full multiple OLS model includes all prepared, non-constant candidate columns.
- Backward selection removes the largest training p-value above 0.05, one encoded term at a time. Removing a category indicator pools it with the reference effect. The full removal history is shown.
- Training F-statistic, F-test p-value, R² and adjusted R², plus train/test RMSE and test R², evaluate the models.
- A mean-salary baseline, residual plots, a standardized Q–Q plot and negative-prediction counts help explain the remaining error.

Backward selection commits to the final multiple model **before** its test evaluation. We do not choose a new final model after inspecting the test scores. All main models use the same split. Constant columns and category levels are learned from training data only. The main study has no required numeric missing values, so numeric imputation is unnecessary.

## Main results from the saved run

| Model | Predictors | Training R² | Test R² | Training RMSE, USD | Test RMSE, USD |
|---|---:|---:|---:|---:|---:|
| Simple: experience | 1 | 0.134 | 0.118 | 62,723 | 62,104 |
| Simple: country GDP | 1 | 0.372 | 0.363 | 53,399 | 52,773 |
| Multiple: full | 64 | 0.541 | 0.514 | 45,663 | 46,091 |
| Multiple: backward, final | 42 | 0.540 | 0.513 | 45,701 | 46,128 |

The mean-salary baseline has a test RMSE of about **USD 66,128**. Country GDP is the stronger simple predictor in this sample. The final multiple model improves on both simple models and the baseline. The full model has a slightly lower test RMSE than the backward model, by about **USD 38**, which we report rather than hiding it.

Backward selection removes 22 terms with very little change in training adjusted R². The final model still has substantial error and **62 negative test predictions**. These are reported without clipping. The residual plots show unequal spread and a heavy upper tail, so OLS assumptions are not well supported. Coefficients and selected p-values are exploratory, not causal or confirmatory evidence.

The exact results above use the uploaded input files and displayed settings. Different source versions, filters or package versions can change them; the notebook calculates its own result discussions when rerun.

## Limits and interpretation

- The salary restrictions and complete-case filters define a narrower population.
- Technology use, experience and country conditions are associated with salary; this does not show that learning one tool causes a pay increase.
- USD salaries are not adjusted for local purchasing power or living costs.
- Country indicators repeat across respondents. The random holdout tests respondents in a similar country mix, not unseen countries, and ordinary OLS independence assumptions are limited.
- Related predictors can share information. Backward selection and coefficients may change across samples.
- The model is not validated for individual salary decisions or fairness across demographic groups.

## Presentation and submission

Section 12 of the notebook gives a **seven-minute presentation plan**. Focus on the question, preparation, split, two simple models, final multiple model, validation and limitations. Everyone should still understand the classes and be able to change the code.

Fill in all team details and the repository link. Run all cells, save the outputs and submit the **same `.ipynb` you will present**, without zipping it. Include the GitHub repository link and complete each member's separate feedback submission.

The assignment asks for the term-project dataset to be augmented. This solution uses the salary topic chosen for this workshop. If changing the project dataset needs instructor agreement, address that course requirement before submission.
