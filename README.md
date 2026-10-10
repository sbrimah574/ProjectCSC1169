# Friendship contact and loneliness in the EU Loneliness Survey 2022

Group project for **CSC1169 Introduction to Machine Learning and Data Analytics**.

This repository holds the code and outputs for our project. It follows the work from raw survey data through cleaning and exploratory analysis. The modelling stage (Session 12) will be added later.

## Research questions

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

## Repository structure

```
├── data/                                       # local files; ignored by Git
│   ├── raw/                                    # original survey CSV files
│   └── processed/                              # cleaned datasets from Notebook 1
├── notebooks/
│   ├── 01_data_inspection_and_cleaning.ipynb   # loads raw data, cleans it, builds the analysis sample
│   └── 02_exploratory_data_analysis.ipynb      # descriptive statistics, plots and correlations
├── results/
│   ├── figures/                                # all plots from Notebook 2
│   │   └── presentation/                       # cleaned-up versions used on the slides
│   └── tables/                                 # summary tables and audit files (CSV)
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
4. Run `notebooks/02_exploratory_data_analysis.ipynb` from top to bottom. This creates the figures and the remaining tables. The final plotting section also exports the larger-font versions to `results/figures/presentation/`.

Both notebooks work whether you open them from the project folder or from inside `notebooks/`. Re-running them overwrites outputs that have the same name.

## What we did

### Cleaning and preparing the sample

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

### Other variables

Loneliness may relate to several co-occurring circumstances, so we also prepared variables covering age, gender, relationship status, work status, education, municipality type, general health, long-term health conditions, close friends and family, social support, family contact, club participation and household composition. Not every variable was used in every comparison.

### Analysis

- Descriptive statistics for age, loneliness and both contact measures
- Boxplots of loneliness across each contact category
- **Spearman's rank correlation** between each contact measure and loneliness. We used it because the contact categories are in order but not evenly spaced.
- Comparisons within groups defined by age, relationship status, close friends, social support, general health and family contact

All results are **unweighted** and describe this analysis sample, not the whole EU population.

## Initial findings

- Remote contact was more common than face-to-face: 67.2% had remote contact at least weekly, compared with 46.0% for face-to-face.
- Average loneliness was 4.99 (median 5, standard deviation 1.85) on the 3–9 scale.
- Both contact types had **weak negative associations** with loneliness. More contact went with lower loneliness, but scores overlapped heavily within every contact category.

  | Contact type | Spearman's rho |
  |---|---|
  | Face-to-face | −0.173 |
  | Remote | −0.139 |

- Face-to-face had a slightly stronger negative association, descriptively consistent with our expectation. We have **not** yet tested whether this difference is statistically significant.
- People with more close friends reported lower loneliness (mean 6.08 with none vs 4.31 with six or more; n = 22,925). Within each close-friend group, the contact associations stayed negative but were weaker than in the full sample. This weakening did not appear within age groups. These comparisons do not establish independent effects or show that close-friend numbers explain the association.

These are associations. They do not show that contact causes lower loneliness.

## Ethics

- **Privacy:** the data are anonymised, but loneliness and health are sensitive. We report only aggregate results.
- **Representation:** an online panel may under-represent some groups, such as people with limited internet access.
- **Communication:** we avoid stigmatising language and claims that the data cannot support, such as causal claims.

## Limitations

- Cross-sectional, self-reported data
- Non-probability online sample, analysed without survey weights
- Complete cases only, so excluded respondents may differ from those analysed
- Context variables were explored one at a time, not adjusted for together
- Remote contact combines several channels, so we cannot separate phone calls from social media

## Next steps (Session 12)

- Check the survey-weight definitions and determine appropriate weighting for the next analysis
- Build a model of loneliness that includes contact alongside a justified set of context variables
- For prediction: fit any preprocessing on training data only, and compare held-out performance against a simple baseline
- Be transparent that the full dataset has already been explored when reporting model evaluation

## References

- European Commission, Joint Research Centre (2022). *EU Loneliness Survey 2022* [dataset]. https://doi.org/10.2905/JRC.V4VT8T8
- Hughes, M. E., Waite, L. J., Hawkley, L. C. & Cacioppo, J. T. (2004). A short scale for measuring loneliness in large surveys. *Research on Aging*, 26(6), 655–672.
- Hulley, S. B., Cummings, S. R., Browner, W. S., Grady, D. G. & Newman, T. B. (2013). *Designing Clinical Research* (4th ed.). Lippincott Williams & Wilkins.

