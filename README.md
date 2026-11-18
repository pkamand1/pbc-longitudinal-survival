# Longitudinal biomarkers and survival in primary biliary cholangitis

This repository is a reproducible R analysis of the Mayo Clinic randomized trial of D-penicillamine in primary biliary cholangitis (PBC), a chronic autoimmune liver disease, using the trial data distributed with R's `survival` package. It brings together the main tools of longitudinal clinical data analysis: a linear mixed-effects model for a biomarker measured at irregular visits, Kaplan-Meier estimation and Cox regression for survival, a time-dependent Cox model in counting-process form that updates each patient's biomarkers at every visit, and a Fine and Gray competing-risks sensitivity analysis for liver transplantation.

The rendered report, with code, figures, and interpretation, is in [`docs/analysis.html`](docs/analysis.html).

## Questions

1. Did D-penicillamine slow the rise in serum bilirubin over time?
2. Did D-penicillamine prolong survival?
3. How strongly does bilirubin, at baseline and as it changes during follow-up, predict death?

## Data

312 randomized patients (158 D-penicillamine, 154 placebo) with 1,945 clinic visits, a median of 5 visits per patient over a median follow-up of 6.3 years; 140 deaths and 29 liver transplants. Laboratory values recorded at each visit include bilirubin, albumin, prothrombin time, alkaline phosphatase, AST, platelets, and cholesterol.

## Results in brief

| Question | Result |
| --- | --- |
| Change in log bilirubin per year, placebo arm (mixed model) | +0.179, about 20% per year |
| Difference in that slope on D-penicillamine | -0.004 per year (95% CI -0.054 to 0.045, p = 0.86) |
| Hazard ratio for death, D-penicillamine against placebo | 1.00 (95% CI 0.72 to 1.39), log-rank p = 0.99 |
| Hazard ratio per unit log bilirubin, baseline Mayo-covariate Cox model | 2.35 (95% CI 1.93 to 2.85); concordance 0.84 |
| Same, with bilirubin, albumin, prothrombin time, and edema updated at each visit | 2.96 (95% CI 2.38 to 3.68); concordance 0.91 |
| Fine and Gray model, transplant as a competing event | log bilirubin subdistribution hazard ratio 2.15 (95% CI 1.78 to 2.60) |

D-penicillamine had no effect on bilirubin progression or on survival, consistent with the trial's published conclusion. Bilirubin dominated prognosis, and a patient's most recent value predicted death considerably better than the value at entry, which is the case for modelling biomarkers as time-varying covariates in longitudinal clinical and electronic health record data.

## Methods

The mixed model is `log(bilirubin) ~ years * treatment + age + sex + (years | id)`, fitted by REML with lme4, with Satterthwaite degrees of freedom from lmerTest; correlated random intercepts and slopes give each patient an individual starting level and rate of change. Survival analyses treat transplantation as censoring, following the original analyses, with the covariates of the Mayo risk score (age, log bilirubin, log albumin, log prothrombin time, and edema). Proportional hazards are checked with scaled Schoenfeld residuals (`cox.zph`). The time-dependent model is built with `tmerge`, so that each interval between visits carries the laboratory values measured at its start. The competing-risks model uses `finegray` with transplantation as the competing event.

## Repository layout

```
analysis.Rmd                  the analysis (R Markdown)
docs/analysis.html            the rendered report
results/tables/               baseline table, mixed-model estimates, Cox and Fine-Gray hazard ratios
results/figures/              trajectories, model checks, Kaplan-Meier curves, hazard-ratio comparison
environment.yml               conda environment (R 4.4, survival, lme4, lmerTest, broom.mixed, ggplot2, rmarkdown)
```

## How to run

```
mamba env create -f environment.yml
mamba activate pbc-longitudinal-survival
Rscript -e 'rmarkdown::render("analysis.Rmd", output_file = "docs/analysis.html")'
```

The data load directly from the `survival` package (`pbc` and `pbcseq`), so nothing needs to be downloaded. Treatment assignment is taken from `pbc`, where its coding is documented, because the coding of `trt` in `pbcseq` has changed between package versions.

## References

Fleming, T. R., & Harrington, D. P. (1991). *Counting processes and survival analysis*. Wiley.

Therneau, T. M., & Grambsch, P. M. (2000). *Modeling survival data: Extending the Cox model*. Springer.

## Author

Patricia K. Kamanda. Released under the MIT License.
