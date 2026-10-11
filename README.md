# Predicting Body Weight from Lifestyle & Demographic Factors

Multiple linear regression in R to predict body weight (kg) from eating habits, physical activity, and demographics, using the UCI *Estimation of Obesity Levels* dataset (2,087 individuals from Mexico, Peru, and Colombia).

Final project for **STAT 420: Statistical Modeling in R** (UIUC MCS), by Evan Yuan

**[Full report (PDF)](report/project-evanyy2.pdf)** | **[HTML version](https://evany50.github.io/Body-Weight-Prediction/)**
---

## Key Results

| Model | 10-fold CV RMSE (kg) | CV R² | CV MAE (kg) |
|---|---|---|---|
| **Combined (AIC selection + Height × Gender)** | **16.79** | **0.590** | 13.57 |
| Polynomial (Height², Age²) | 16.80 | 0.591 | 13.18 |
| Height × Gender interaction | 16.83 | 0.588 | 13.61 |
| Backward AIC | 17.08 | 0.577 | 13.61 |
| Full additive model (baseline) | 17.12 | 0.573 | 13.60 |
| Backward BIC | 17.16 | 0.573 | 13.70 |

**Final model:** adjusted R² = 0.592 (up from 0.577 for the full model); 10-fold CV RMSE ≈ 16.8 kg.

```r
Weight ~ Height * Gender + Age + family_history_with_overweight + FAVC + FCVC +
         CAEC + CH2O + SCC + FAF + TUE + CALC + MTRANS
```

### Key findings
- **Height × Gender interaction is the biggest single improvement**: weight rises more steeply with height for females (~1.7 kg/cm) than for males (~0.8 kg/cm), and the difference is highly significant (p < 1e-16).
- **Family history of overweight** is associated with about **+17.7 kg**, holding other factors constant.
- **Physical activity (FAF)** is negatively associated with weight (about −2.8 kg per unit), while **vegetable consumption (FCVC)** is unexpectedly positively associated (about +8.0 kg per unit).
- Stepwise AIC/BIC selection barely changed predictive performance; adding structure (interactions, polynomial terms) mattered more than removing predictors.

## Methodology
1. **Cleaning**: removed 24 duplicate rows, converted categorical variables to factors. No missing values.
2. **EDA**: distributions, scatterplots against key predictors, correlation matrix.
3. **Baseline**: full additive model (excluding `NObeyesdad`, which is derived from weight/BMI and would leak the target).
4. **Model improvement**: backward/forward/both-direction stepwise selection (AIC and BIC), polynomial terms, interaction terms, and a Box-Cox transformation.
5. **Evaluation**: 10-fold cross-validation (`caret`, `set.seed(420)`), nested F-tests, and diagnostics (residual plots, Shapiro-Wilk, Breusch-Pagan, VIF, Cook's distance).

## Limitations
- Residuals still show **heteroscedasticity** (Breusch-Pagan p < 0.001) and departures from normality in the tails, so p-values and intervals should be read with caution.
- A Box-Cox transformation (λ ≈ 0.18) was explored but not adopted, since it makes coefficients harder to interpret in kg.
- About **77% of the dataset is synthetic** (generated with SMOTE by the dataset authors), and it covers only three countries.
- Linear models only; tree-based or other non-linear methods are a natural next step.

## Reproduce
```r
install.packages(c("tidyverse", "car", "MASS", "glmnet", "caret", "lmtest", "corrplot", "rmarkdown"))
rmarkdown::render("analysis/project-evanyy2.Rmd")
```
The Rmd reads the data from `../data/ObesityDataSet_raw_and_data_sinthetic.csv`, so keep the folder structure above when knitting.

## Data
Palechor, F. M., & de la Hoz Manotas, A. (2019). *Estimation of obesity levels based on eating habits and physical condition* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5H31Z (CC BY 4.0)
