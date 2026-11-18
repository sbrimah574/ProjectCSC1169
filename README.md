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

How well can we predict UCLA loneliness scores, and how much does each type of friendship contact improve predictions beyond the other variables?

We compare contact measures on their own, together, and alongside the same set of context variables:

- **Close friends, relationship status and household composition** describe the social relationships and living arrangements that contact frequency alone cannot capture.
- **Age, gender, education and employment** allow the model to account for differences in social circumstances across respondents. The JRC analysis of the same survey informed this choice.
- **Municipality type and country** describe where respondents live and let us examine whether prediction errors differ across municipality types.

We kept identifiers, survey weights and the three items used to calculate the loneliness score out of the predictors. We chose the context variables after exploring the data in Presentation 1.

### Model development and evaluation

In Notebook 03 we reserve 20% of respondents for testing. We fit missing-value handling, category encoding and scaling for the linear models inside the training pipeline. We compare mean and median baselines, linear regression, Ridge, a decision tree and a random forest with the same five cross-validation folds. We select the final model using mean absolute error (MAE). Ridge is used throughout the predictor comparisons so the model type stays consistent.

Notebook 04 generates test predictions and reports MAE, RMSE and R². We compare the contact predictor sets on the same test respondents, plot prediction errors, and use permutation importance to see which variables help predict the score. We also check errors by age and municipality type, omitting groups with fewer than 30 test respondents.

### Survey weights

Presentation 1 used unweighted results. For Presentation 2, we use the survey's EU27 weight, `w_eu27`, which combines within-country demographic adjustment with country population size. We compare weighted and unweighted rank correlations on the analysis sample, and weighted and unweighted MAE on the same test predictions. Models and cross-validation remain unweighted. The weighted correlations and MAEs are point estimates. The paired bootstrap intervals describe the unweighted comparisons.

### Findings and limitations

- The random forest had the lowest cross-validation MAE. On the test set, it was off by 1.397 loneliness points on average (RMSE 1.699, R² 0.158), compared with 1.556 points for the median baseline. With survey weights, MAE was 1.390 for the forest and 1.567 for the baseline.
- Weighting weakened both contact correlations. Face-to-face remained more strongly associated with loneliness (unweighted ρ = −0.173, weighted ρ = −0.142) than remote contact (−0.139 and −0.102). The unweighted paired bootstrap interval for the difference was −0.045 to −0.023.
- In the Ridge models, adding both contact types to the context variables lowered test MAE from 1.458 to 1.418. The weighted comparison also improved, from 1.445 to 1.418. Once face-to-face contact was included, adding remote contact made very little difference, especially after weighting.
- Shuffling the number of close friends increased test error the most, followed by age and relationship status.

Permutation importance describes how much the fitted model uses each variable. The survey was collected at one point in time, so these results cannot tell us how loneliness changes over time or whether changing contact would reduce loneliness. The test set comes from the same online survey, and we had already explored the full sample in Presentation 1. The weights adjust for measured characteristics, while the original panel recruitment and our complete-core-response filter can still affect who is represented. We report aggregate results and treat estimates for the youngest respondents cautiously because of the small test group.

### Addressing Students' Questions from Presentation 1

1. **Which factors mattered most, and how strong were the results?** Shuffling the number of close friends increased the forest's test error the most, followed by age and relationship status. Both contact correlations had unweighted p-values below .001, but the associations were weak. The importance ranking describes prediction and does not test the statistical significance of individual predictors.
2. **Did face-to-face contact predict better than remote contact? Could the pattern have changed over time?** With contact as the only predictor, the face-to-face Ridge model had slightly lower test MAE than the remote model (1.543 vs 1.552). The paired unweighted bootstrap interval for the improvement was 0.002 to 0.017 points. The survey comes from one period, so we cannot assess changes over time.
3. **Did we widen the analysis to include more variables?** We kept the focus on friendship contact and added context variables to test whether contact still improved prediction when other information was available.
4. **How did we group the predictors?** We compared contact alone, context alone and combinations of the two. Close-friend count and relationship status were included together in the context models. We did not test a separate interaction between them.
5. **Did we split the data?** Presentation 1 described the full cleaned sample. In Presentation 2, we trained and tuned models on 19,544 respondents and held out 4,887 for testing.
6. **Would the results hold outside the EU?** Contact habits and social circumstances may differ elsewhere. We would need data from outside the EU to test whether the findings transfer.
7. **Could urban and rural settings affect the findings?** Access to meeting places and travel distances could matter. We included municipality type and checked prediction errors by area. Unweighted forest MAE ranged from 1.36 to 1.42 across the three area groups. This comparison shows how errors varied, while differences in the contact–loneliness relationship by area remain an open question.
8. **What about respondents aged 16–17?** The original survey covered people aged 16 and over. The methodological report describes ethics review and participant consent. We use anonymised responses and report aggregate results. Only 32 respondents in the test set were aged 16–17, so we avoid drawing strong conclusions about this group.
9. **What interventions might follow?** The findings suggest testing ways to make in-person contact easier, such as accessible local meet-ups, while keeping remote options available. An intervention study would be needed to assess whether these changes reduce loneliness.

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
│   │   ├── presentation1/                       # Presentation 1 plots
│   │   └── presentation2/                      # Presentation 2 plots
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
3. Run `notebooks/01_data_inspection_and_cleaning.ipynb` from top to bottom. This creates the cleaned files in `data/processed/` and the audit tables in `results/tables/presentation1`.
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
