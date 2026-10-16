# Travel Choices of Parents with Young Children — NHTS 2017

When parents of young children leave home for a tour of trips, which of those trips include their child? This project uses **home-based tours** from the 2017 National Household Travel Survey (NHTS) trip file. It classifies each tour by how much of it a child under 10 joins, and estimates **discrete choice models** (multinomial logit with interactions, and mixed logit) to see how trip purpose, tour structure, household composition, and gender shape that outcome.

## Outcome

Each adult tour is classified by child accompaniment (children under 10):

| Alternative | Definition | Tours |
|---|---|---|
| **All trips with child** | A child under 10 is on every trip of the tour | 6,006 (13.7%) |
| **Some trips with child** | A child is on some trips, e.g. dropped off or picked up mid-tour | 2,478 (5.6%) |
| **No child** (reference) | No child under 10 on any trip | 35,470 (80.7%) |

Analytic sample: **43,954 tours** made by adults in households with children (NHTS life-cycle codes 3–6, children under 10 present). Tours are within town only.

## Data

- NHTS 2017 public-use files: `trippub.csv`, `perpub.csv`, `hhpub.csv`. Source: U.S. Department of Transportation, Federal Highway Administration, 2017 National Household Travel Survey, https://nhts.ornl.gov.
- The data are not included in this repository. The intermediate tour file is about 380 MB.

## Pipeline

All steps are R Markdown files with **Python** chunks (pandas, Biogeme). Run them in order. Each step reads and writes intermediate CSVs in `data/`.

| Step | File | What it does |
|---|---|---|
| 1 | `01_build_tours.rmd` | Assigns trip IDs. Chains each person's trips into **closed home-based tours**. Merges household and person files. Keeps households with children under 10. Classifies each tour by child accompaniment. |
| 2 | `02_data_description.Rmd` | Household and tour counts by number and age of children |
| 3 | `03_variable_setting.Rmd` | Builds model variables: primary mode and purpose of the tour, tour duration, multi-stop tour flag, income quartile, household composition, single-adult status, worker and gender flags. Drops out-of-town tours and invalid codes. |
| 4 | `04_sample_cleaning_descriptives.rmd` | Final missing-data handling and sample counts, with descriptive breakdowns by purpose, mode, income, and child presence |
| 5 | `05_mixed_logit.rmd` | **Mixed logit** with a normally distributed random coefficient on log tour time (500 draws) |
| 6 | `06_mixed_logit_by_purpose.rmd` | The mixed logit estimated separately for each tour purpose (work, chauffeuring, leisure/social, shopping/errands, other), and for all purposes with and without chauffeuring |
| 7 | `07_interaction_logit.rmd` | **Main model:** multinomial logit with gender interactions (female × single parent, female × worker, and others), plus **simulations** of predicted tour-type probabilities and marginal effects across tour durations from 10 to 720 minutes |

Estimation reports from Biogeme, one per model, are in `biogeme_output/`. Knitted reports for steps 4 and 7 are the `.html` files with matching names; download them and open them in a browser, since GitHub shows HTML as source.

## Main results

From the interaction model (`biogeme_output/interaction_logit.html`): 43,954 tours, 38 parameters, ρ² = 0.545 relative to the null model. Coefficients are relative to a tour with **no child**.

| | All trips with child | Some trips with child |
|---|---|---|
| Leisure/social tour | **+2.06\*\*\*** | −0.90\*\*\* |
| Shopping/errands tour | **+1.29\*\*\*** | −1.54\*\*\* |
| Work tour | −1.37\*\*\* | −2.07\*\*\* |
| Multi-stop tour (3+ trips) | −0.26\*\*\* | **+1.75\*\*\*** |
| Log tour time | +0.18\*\*\* | +0.44\*\*\* |
| Children under 5 in household | +0.39\*\*\* | +0.26\*\*\* |
| Children 10–17 in household | −0.14\*\*\* | −0.09\*\*\* |
| Single parent | −0.66\*\*\* | −0.48\*\*\* |
| Single parent × female | **+0.53\*\*\*** | **+0.61\*\*\*** |
| Worker | +0.45\*\*\* | 0.04 |
| Worker × female | **−0.26\*\*** | 0.02 |
| Female | 0.10 | 0.01 |
| Lowest income quartile | +0.52\*\*\* | +0.62\*\*\* |

\*\* p<0.01, \*\*\* p<0.001

- **Purpose drives whether children come along.** Children usually join leisure and shopping tours from start to finish and are rarely on work tours.
- **Partial accompaniment comes from complex tours.** A multi-stop tour sharply raises the odds that a child is on only part of the tour, the pattern of dropping a child off or picking one up while chaining other trips.
- **Gender works through the household's situation, not on its own.** The female main effect is near zero. Single parents are less likely to travel with their children, but for **single mothers** the female interaction largely offsets this. Employment raises child accompaniment for men, but the effect is smaller for **working women**.
- **Older siblings substitute for parents.** Teenagers in the household lower parent–child co-travel. The chi-square test of tour type against having an older sibling gives χ²(2) = 277, p < 0.001.

## How to run

1. Put `trippub.csv`, `perpub.csv` and `hhpub.csv` from NHTS 2017 in `data/`.
2. Use RStudio with `reticulate`, and a Python environment with `pandas`, `numpy`, `scipy`, `matplotlib`, `seaborn` and `biogeme`. `biogeme.toml` holds the Biogeme settings used for estimation.
3. Knit the files in numerical order. Biogeme writes its `.html` and `.pickle` output next to the scripts.

## Author

Dawon Oh: tour construction, variable design, model specification and estimation, and simulation.
