# Friendship contact and loneliness in the EU Loneliness Survey 2022

Group project for **CSC1169 Introduction to Machine Learning and Data Analytics**.

We use the EU Loneliness Survey 2022 to study how contact with friends relates to loneliness. [Presentation 1](#presentation-1--data-preparation-and-exploration) covers cleaning and exploration. [Presentation 2](#presentation-2--modelling-and-evaluation) compares predictions and checks how survey weighting affects the findings.

## Data

**Source:** European Commission, Joint Research Centre. *EU Loneliness Survey 2022*. https://doi.org/10.2905/JRC.V4VT8T8

- 25,646 respondents, 208 variables, 27 EU countries
- Cross-sectional: responses were collected during one survey period, rather than following respondents over time
- Non-probability online panel: respondents were not randomly selected from the population

Raw data are stored locally in `data/raw/`, and cleaned files are saved in `data/processed/`. The `data/` folder is listed in `.gitignore`, so untracked files in both folders are excluded from Git. Files already tracked by Git are not automatically removed by this rule.

To run the notebooks, download these two files from the source above and place them in `data/raw/` at the top of the project:

- `eu_loneliness_survey_eu27_values.csv`
- `eu_loneliness_survey_eu27_labels.csv`

### Main measures

| Measure | What it records |
|---|---|
| Face-to-face friend contact | How often the person meets friends in person: seven ordered categories, from never to daily |
| Remote friend contact | How often they contact friends by phone, internet or social media: same seven categories |
| UCLA loneliness score | Sum of three items (lacking companionship, feeling left out, feeling isolated), each scored 1–3. Totals run from 3 to 9; higher means lonelier |

The loneliness score measures how lonely people say they feel. It is not a clinical diagnosis.

## Presentation 1 — Data preparation and exploration

### Research questions

**RQ1.** Among respondents aged 16 and over in the EU Loneliness Survey 2022, is more frequent contact with friends, both face-to-face and remote (phone, internet or social media), associated with lower loneliness?

**RQ2.** Is more frequent face-to-face contact more strongly associated with lower loneliness than more frequent remote contact?

**Why it matters.** Friendships are now kept up both in person and remotely. How often people have contact is also not the same as feeling connected. Previous JRC work has already shown that contact frequency is related to loneliness, so we focus on a narrower question: whether the two forms of contact relate to loneliness differently.

**Who might care.** Researchers studying loneliness and social relationships, public-health bodies, policymakers, and community organisations and charities working to reduce loneliness and social isolation.

**What we expected.** For RQ1, both forms of contact would show negative associations with loneliness. For RQ2, face-to-face contact would show the stronger one. We would count the RQ2 expectation as **not supported** if remote contact showed an equal or stronger association.

We checked the questions against the FINER criteria (Hulley et al., 2013):

| Criterion | How our project meets it |
|---|---|
| **Feasible** | The dataset contains both contact measures, loneliness and relevant context variables. The analysis is manageable within the module's timeframe. |
| **Interesting** | We examine whether different ways of maintaining friendships relate differently to feeling lonely. |
| **Novel** | Our project builds on research about social contact and loneliness by directly comparing face-to-face and remote friendship contact within the same sample, and exploring how these associations vary across relevant social and personal circumstances. |
| **Ethical** | We use anonymised secondary data, report aggregate results and consider representation and interpretation risks. |
| **Relevant** | The findings may interest researchers and organisations working on loneliness, and the project applies the module's data-cleaning and exploratory-analysis skills. |

### What we did

#### Cleaning and preparing the sample

| Stage | Respondents kept | Removed at this stage |
|---|---|---|
| Original respondents | 25,646 | – |
| Confirmed aged 16 or over | 25,631 | 15 (3 aged 15; 12 with an invalid age code) |
| Complete answers to both contact questions and all three loneliness items | **24,431** | 1,200 |

In total, 95.3% of the original sample was kept.

Key decisions:

- **Checked the basics.** IDs were unique and there were no exact duplicate rows. We checked valid ranges and treated invalid values as missing.
- **Treated documented non-response codes as missing.** Some non-responses were stored as numbers (for example 999) rather than blank cells. We checked the labels file before recoding them, so they were not counted as real answers.
- **Built the loneliness score only when all three items were answered.** We did not fill in missing answers, because that could change the measures and the relationships we are studying. We used `min_count=3` to prevent partial totals.
- **Used the same people for both contact comparisons.** Requiring complete core answers keeps face-to-face and remote results directly comparable. The trade-off is that excluded people may differ from those we kept.
- **Kept other cases in a flagged file.** Notebook 1 saves a cleaned version of every row, with flags, so these decisions can be revisited.
- **Handled other variables separately.** Missing answers on optional context variables (such as number of close friends) only affect the comparisons that use them. They do not shrink the main sample.

#### Other variables

Loneliness may relate to several co-occurring circumstances, so we also prepared variables covering age, gender, relationship status, work status, education, municipality type, general health, long-term health conditions, close friends and family, social support, family contact, club participation and household composition. Not every variable was used in every comparison.

#### Analysis

- Descriptive statistics for age, loneliness and both contact measures
- Boxplots of loneliness across each contact category
- **Spearman's rank correlation** between each contact measure and loneliness. We used it because the contact categories are in order but not evenly spaced.
- Comparisons within groups defined by age, relationship status, close friends, social support, general health and family contact

All results are **unweighted** and describe this analysis sample, not the whole EU population.

### Initial findings

- Remote contact was more common than face-to-face: 67.2% had remote contact at least weekly, compared with 46.0% for face-to-face.
- Average loneliness was 4.99 (median 5, standard deviation 1.85) on the 3–9 scale.
- Both contact types had **weak negative associations** with loneliness. More contact went with lower loneliness, but scores overlapped heavily within every contact category.

  | Contact type | Spearman's rho |
  |---|---|
  | Face-to-face | −0.173 |
  | Remote | −0.139 |

- Face-to-face had a slightly stronger negative association, descriptively consistent with our expectation. We compared the two correlations directly in Presentation 2.
- People with more close friends reported lower loneliness (mean 6.08 with none vs 4.31 with six or more; n = 22,925). Within each close-friend group, the contact associations stayed negative but were weaker than in the full sample. This weakening did not appear within age groups. These comparisons do not establish independent effects or show that close-friend numbers explain the association.

These are associations. They do not show that contact causes lower loneliness.

### Ethics

- **Privacy:** the data are anonymised, but loneliness and health are sensitive. We report only aggregate results.
- **Representation:** an online panel may under-represent some groups, such as people with limited internet access.
- **Communication:** we avoid stigmatising language and claims that the data cannot support, such as causal claims.

### Limitations

- Cross-sectional, self-reported data
- Non-probability online sample, analysed without survey weights
- Complete cases only, so excluded respondents may differ from those analysed
- Context variables were explored one at a time, not adjusted for together
- Remote contact combines several channels, so we cannot separate phone calls from social media

## Presentation 2 — Modelling and evaluation

### Research question and predictors

How well can we predict UCLA loneliness scores, and how much does each type of friendship contact improve predictions beyond the contextual information?

We carry forward all the contextual measures prepared in Notebook 01 and explored in Notebook 02:

- **Demographics and location** — age, gender, education, employment, municipality type and country.
- **Relationships and household composition** — relationship status, cleaned close-friend and close-family counts, and the separate adult, child and other household counts.
- **Support and health** — all four support items, general health and long-term health conditions.
- **Other social contact and participation** — face-to-face and remote family contact, and the prepared social-participation item.

The JRC analysis and our earlier exploration informed the contextual domains. We include the prepared measures consistently rather than select only those with strong individual correlations. Support and loneliness describe closely related experiences. Health and loneliness were measured at the same time, so their direction of association remains uncertain.

We use the cleaned counts instead of their response flags or duplicate source fields. Identifiers, survey weights and the UCLA component items are excluded from the predictors. The four support items remain separate.

### Preparation and model selection

The sample remains 24,431 respondents with complete core responses. We impute missing numerical predictors using training medians and add missingness indicators. Missing categorical responses receive their own category. Age and counts are numerical. We one-hot encode the other predictors, including ordered contact, support and health responses, allowing category-specific relationships without assuming equal steps between codes.

We use an 80/20 split (19,544 training, 4,887 test) and compare models with the same five training folds. The pipeline learns preprocessing within each fold. Numerical predictors are scaled for the linear models. We compare median and mean baselines, linear regression, Ridge, a decision tree and a random forest. Cross-validation MAE selects the final model.

We also compare seven predictor sets using Ridge throughout: each friendship-contact type alone, both together, context alone, and context with either or both contact types. These comparisons assess the additional information supplied by contact using a consistent model type.

Model selection uses training cross-validation. The test sample has been inspected during model development, so we interpret this evaluation as exploratory.

### Evaluation and survey weights

Notebook 04 reports test MAE, RMSE and R², paired bootstrap intervals for unweighted MAE differences, prediction diagnostics, permutation importance and errors by age and municipality. The bootstrap resamples test respondents for fixed fitted models and does not capture the full uncertainty of model development. Groups below 30 test respondents are omitted from the subgroup table.

We use the EU27 weight, `w_eu27`, to compare weighted and unweighted correlations and test MAE. It combines within-country demographic adjustment with country population size. Model training and selection remain unweighted. The weighted summaries are point estimates.

### Findings

- Cross-validation selected **Ridge**. Its test MAE was **1.223**, RMSE **1.530** and R² **0.317**. The median baseline MAE was **1.556**.
- Weighted test MAE was **1.230** for the selected model and **1.567** for the median baseline.
- With the full context set, adding both friendship-contact types changed Ridge test MAE from **1.227** to **1.223**. The paired mean improvement was **0.0047** points, with a 95% bootstrap interval of **0.0018 to 0.0075**. Positive differences favour adding contact.
- Weighted Ridge MAE changed from **1.232** for context alone to **1.230** with both contact types.
- The three largest permutation-importance values belonged to **health_general, age, social_support_d**. Related predictors can share information, so this ranking describes the fitted model's reliance on variables rather than causal importance.
- The correlations use the same complete-core sample as before. Weighted correlations remained −0.142 for face-to-face contact and −0.102 for remote contact, compared with unweighted values of −0.173 and −0.139.

### Addressing Students' Questions from Presentation 1

1. **Which factors mattered most?** We now model the contextual measures together. The importance table shows which variables the selected model relied on most. Predictive importance does not establish an independent causal effect or statistical significance for an individual predictor.
2. **Did face-to-face contact predict better than remote contact? Could this change over time?** Contact-only Ridge test MAE was 1.537 for face-to-face and 1.552 for remote contact. The paired improvement interval was 0.0074 to 0.0238 points. We cannot assess changes over time from this survey period.
3. **Did we include more variables?** The models now use the full contextual domains prepared for Presentation 1, including support, health, family contact and participation.
4. **How did we group predictors?** We compare contact alone, context alone and their combinations. Close-friend count and relationship status enter the context models together. We do not fit a separate interaction between those two variables in Ridge.
5. **Did we split the data?** Presentation 1 explored the cleaned sample. Presentation 2 uses the 80/20 split, with cross-validation on the training portion.
6. **Would the results hold outside the EU?** Validation on data from other settings would be needed to assess transfer.
7. **Does urban or rural location matter?** We include municipality type and compare test errors by area. The selected model's unweighted MAE ranges from 1.203 to 1.236 across the three groups. This checks prediction performance by area, while area-specific contact relationships would require a separate analysis.
8. **What about respondents aged 16–17?** The survey covers ages 16+. Its methodological report describes ethics review and participant consent. We report aggregate results and interpret the small youngest test group (n = 32) cautiously.
9. **What interventions follow?** These associations can inform questions for intervention studies. Testing whether changes to contact or support reduce loneliness would require an appropriate intervention design.

### Limitations

The selected Ridge model produced 89 test predictions outside the 3–9 outcome range. We report its raw predictions consistently and retain this as a limitation of the linear model.

Self-report, online-panel recruitment, core-response exclusions and a single survey period limit interpretation. Survey weights adjust for measured characteristics but cannot resolve every selection issue. Support overlaps with loneliness and the health relationships may operate in either direction. The models predict concurrently measured scores. Better R² does not demonstrate that causes have been identified or that the model is suitable for individual screening.

### Outputs

The modelling tables are in `results/tables/modelling/` and figures in `results/figures/presentation2/`. Notebook 03 saves the fitted models locally as `data/processed/presentation2_models.joblib`. Notebook 04 checks the sample fingerprint before loading predictions.

## Repository structure

```
├── data/                                       # local files; ignored by Git
│   ├── raw/                                    # original survey CSV files
│   └── processed/                              # cleaned datasets from Notebook 1
├── notebooks/
│   ├── 01_data_inspection_and_cleaning.ipynb   # loads raw data, cleans it, builds the analysis sample
│   ├── 02_exploratory_data_analysis.ipynb
│   ├── 03_model_development.ipynb
│   └── 04_model_evaluation_and_interpretation.ipynb
├── results/
│   ├── figures/                                # plots from both presentations
│   │   ├── presentation1/                      # Presentation 1 plots
│   │   └── presentation2/         # Presentation 2 plots
│   └── tables/                                 # summaries and audits, including modelling/
├── slides/                                     # presentation slides; untracked PPTX files ignored
├── requirements.txt
└── README.md
```

## How to run

1. Install Python 3 and the packages in `requirements.txt`:
   ```
   pip install -r requirements.txt
   ```
2. Put the two raw CSV files in `data/raw/` (see **Data** above).
3. Run `notebooks/01_data_inspection_and_cleaning.ipynb` from top to bottom. This creates the cleaned files in `data/processed/` and the audit tables in `results/tables/`.
4. Run `notebooks/02_exploratory_data_analysis.ipynb` from top to bottom. This creates the figures and the remaining tables. The final plotting section also exports the larger-font versions to `results/figures/presentation1-slides/`.
5. Run `notebooks/03_model_development.ipynb` to fit and compare models. It saves fitted models locally in `data/processed/`.
6. Run `notebooks/04_model_evaluation_and_interpretation.ipynb` to evaluate the models and run the survey-weight checks. The modelling tables go to `results/tables/modelling/`.

Run the notebooks from the repository root or from inside `notebooks/`.

## References

- European Commission, Joint Research Centre (2022). *EU Loneliness Survey 2022* [dataset]. https://doi.org/10.2905/JRC.V4VT8T8
- Hughes, M. E., Waite, L. J., Hawkley, L. C. & Cacioppo, J. T. (2004). A short scale for measuring loneliness in large surveys. *Research on Aging*, 26(6), 655–672.
- Hulley, S. B., Cummings, S. R., Browner, W. S., Grady, D. G. & Newman, T. B. (2013). *Designing Clinical Research* (4th ed.). Lippincott Williams & Wilkins.
- Joint Research Centre. [EU Loneliness Survey methodological report](https://publications.jrc.ec.europa.eu/repository/bitstream/JRC139563/JRC139563_01.pdf), section 6.2 on weights.
- Joint Research Centre (2023). [Loneliness and social connectedness: insights from a new EU-wide survey](https://publications.jrc.ec.europa.eu/repository/bitstream/JRC133351/JRC133351_01.pdf).
