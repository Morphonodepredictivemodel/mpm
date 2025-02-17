# Morphonode Predictive Model (MPM)
The **Morphonode Predictive Model**, and the related **R** package `morphonode`, is the computational framework for ultrasound signatures detection and malignancy prediction in vulvar cancer. The MPM suite has two main purposes: predicting the (risk of) malignancy and the risk of single/multiple metastases, given an ultrasound profile. It provides a non-invasive diagnostic tool, highly accurate and easy to use, also in absence of experienced human examiners.

&nbsp;

## Instal the latest development version using devtools

The latest development version can be installed through **GitHub** from any **R** environment:

``` r
# install.packages("devtools")
devtools::install_github("Morphonodepredictivemodel/morphonode")
```

This will download the MPM source code, build the morphonode tar.gz package, and install it locally.

&nbsp;

## Install the latest release from the morphonode source package

The latest stable release is the **morphonode version 1.0.0**. You can download this and older versions from the **Releases** panel of this website, as a zip or tar.gz package. You can install morphonode from any directory (e.g., the home directory "~") in **R** with:

``` r
install.packages("~/morphonode-1.0.0.tar.gz", repos = NULL)
```
&nbsp;

# Overview

1. **Using the Morphonode Predictive Model suite**
   - 1.1. Morphonode Predictive Model basic usage
   - 1.2. Defining an ultrasound profile
      - 1.2.a. Interactive input
   - 1.3. MPM output object

2. **Additional functionalities**  
   - 2.1. Ultrasound data simulation
   - 2.2. RFC building and validation
   - 2.3. Building a robust binomial model (RBM)
   - 2.4. Generating bootstrap confidence intervals for predictive performance indices

&nbsp;

# 1. Using the Morphonode Predictive Model suite

## 1.1. Morphonode Predictive Model basic usage

The whole MPM suite can be launched in two very simple analysis steps:

- Step 0: &nbsp; Library loading.
``` r
library(morphonode)
```
- Step 1: &nbsp; Defining the ultrasound profile (in this example, we will use a simulated malignant profile).
``` r
x <- new.profile(us.simulate(y = 1))
```
- Step 2: &nbsp; Launching the suite!
``` r
mpm <- us.predict(x)
```
The output message should look like the following:
```
       Morphonode Predictive Model output
# ------------------------------------------------------------------------------------------- #
|     Prediction (Morphonode-RFC): MALIGNANT (Y = 1)                                          |
|  Risk estimate (Morphonode-RBM): 0.974                                                      |
|                      Risk level: HIGH (> 0.29)                                              |
|       Signature (Morphonode-DT): HMR (high metastatic risk)                                 |
|      Estimated prediction error: 0.029     (cutoff: E < 1)                                  |
|                                                                                             |
|                       DIAGNOSIS: MALIGNANT                                                  |
|                                                                                             |
|--- Input profile ---------------------------------------------------------------------------|
|                                                                                             |
| [0] w = 1.000 | 10.00 |  7.84 | 1 | 0 | 0 | 1 | 0 | 4 | 2 | 2 | 1 | 1 | 1 | 3 | Y = 1 | HMR |
|                                                                                             |
|--- Top-similar profiles (w: cosine similarity) ---------------------------------------------|
|                                                                                             |
| [1] w = 0.996 | 11.50 |  8.80 | 1 | 0 | 0 | 1 | 0 | 4 | 2 | 1 | 1 | 1 | 1 | 3 | Y = 1 | HMR |
| [2] w = 0.988 | 11.00 |  8.80 | 1 | 0 | 0 | 0 | 0 | 4 | 2 | 2 | 3 | 2 | 1 | 3 | Y = 1 | HMR |
| [3] w = 0.988 |  8.00 |  6.40 | 1 | 0 | 0 | 0 | 0 | 4 | 2 | 1 | 1 | 2 | 1 | 3 | Y = 1 | HMR |
| [4] w = 0.987 |  8.10 |  5.90 | 1 | 0 | 0 | 1 | 0 | 4 | 1 | 3 | 1 | 1 | 1 | 3 | Y = 1 | HMR |
| [5] w = 0.986 | 10.30 |  8.80 | 1 | 0 | 0 | 1 | 1 | 4 | 4 | 2 | 1 | 1 | 1 | 2 | Y = 1 | HMR |
# ------------------------------------------------------------------------------------------- #
```
&nbsp;

The MPM suite is composed by 4 modules:

- **Morphonode-RFC**. Random forest classification (RFC).
  The predicted phenotype can be either ***malignant*** (y = 1) or ***non-malignant*** (y = 0).
- **Morphonode-RBM**. Malignancy risk estimation by robust binomial modeling (RBM).
  The RBM offers a continuous estimation of y (i.e., the malignancy risk), thus the higher the accordance with the RFC, the higher the prediction
  reliability.
  This module also suggests when the risk reach ***moderate*** (p > 0.23) or ***high*** levels (p > 0.29). These cutoffs reflect the optimal risk cutpoint
  between malignant and non-malignant subjects, by maximizing F1 score and Sensitivity/Specificity, respectively.
- **Morphonode-DT**. Decision tree-based metastatic risk signature detection.
  A ***high metastatic risk*** (HMR) signature is characterized by a high risk of a **single metastasis event**, whereas
  a ***metastatic signature*** (MET) is typical of malignancies showing **multiple metastasis events**.
  Conversely, a ***low metastatic risk*** (LMR) signature is generally associated with non-malignant phenotypes.
  Finally, a ***moderate metastatic risk*** (MMR) signature is the group with highest heterogeneity and requires RFC and RBM results to be characterized.
- **Morphonode-SP**. Similarity prolfiling module. The module searches and ranks ultrasound profiles from the given (by default, the simulated) ultrasound
  features dataset. The default function is **cosine similarity** and **5 top-similar profiles** are shown to screen.

A final DIAGNOSIS field shows the decision that the practitioner should take based on the predictions above. If at least 2/3 of the predictive modules (RFC, RBM, and DT) agree on a diagnosis (e.g., RFC: MALIGNANT, RBM: p > 0.29, DT: HMR signature), that will be the final diagnosis. If they disagree and/or yield an intermediate prediction (e.g., RFC: MALIGNANT, RBM: 0.23 ≤ p ≤ 0.29, DT: MMR signature), the diagnosis will be "UNDEFINED".
In addition, the prediction error (E) provides an estimate of the overall prediction unreliability, by combining the uncertainty estimates deriving from the three predictive modules. As a rule of thumb, if E is above or equal to 1, the prediction should be considered as unreliable.

Both the input ultrasound profile and the similar ones are reported as a list of attributes, including:

1. **Progressive number** (the smaller the number, the higher the similarity; "0" is the input profile).
2. **Similarity coefficient** (w). By default, cosine similarity is used.
3. **Short axis**: length in millimeters.
4. **Cortical thickness**: thickness in millimeters.
5. **Nodal core sign** (hilum): *absent* (0, **metastatic trait**), *present* (1).
6. **Perinodal hyperechogenic ring** (inflammatory stroma): *absent* (0), *present* (1, **metastatic trait**).
7. **Cortical interruption** (extracapsular spread): *absent* (0), *present* (1, **metastatic trait**).
8. **Echogenicity** (echostructure): *homogeneous* (0), *inhomogeneous* (1).
9. **Focal intranodal deposit**: *absent* (0), *hyperechoic* (1), *anaechoic, cystic areas* (2), *both* (3).
10. **Vascular flow localization**: *non-vascularized* (0), *central* (1), *peripheral* (2), *extranodal* (3), *combined* (4).
11. **Cortical thickening**: *not evaluable* (0), *absent* (1), *focal* (2), *concentric* (3), *eccentric* (4).
12. **Vascular flow architecture pattern**: *non-vascularized* (0), *longitudinal axis* (1), *scattered* (2), *branched* (3), *chaotic* (4).
13. **Cortical-Medullar interface distortion**: *absent* (1), *focal* (2), *diffused* (3), *medulla not visible* (4).
14. **Shape**: *elliptic* (1), *circular* (2), *irregular* (3).
15. **Grouping**: *absent* (1), *moderate* (2), *complete* (3).
16. **Color score**: ordinal variable from 1 to 5.
17. **Phenotype** (y): *non-malignant* (0), *malignant* (1).
18. **Metastatic risk signature**: LMR (low risk), MMR (moderate risk), HMR (high risk, single metastasis), MET (metastatic, multiple metastases).

Ulrasound profiles showing one or more values marked as "**metastatic trait**" determine the **MET signature** (very high risk of multiple metastases).

## 1.2. Defining an ultrasound profile

A new profile can be initialized manually, including each ultrasound value in the same order shown in the previous chapter (points 3 to 16).

```r
x <- new.profile(c(10.0, 6.3, 1, 0, 0, 0, 0, -1, 2, 2, 3, -1, -1, -1))
```

As shown above, the object `x` may contain -1 values, corresponding to missing data:

```
> x
$ultrasound
          shortAxis            cortical               hilum  inflammatoryStroma 
               10.0                 6.3                 1.0                 0.0 
extracapsularSpread       echostructure                 FID                 VFL 
                0.0                 0.0                 0.0                -1.0 
 corticalThickening     vascularPattern                CMID               shape 
                2.0                 2.0                 3.0                -1.0 
           grouping          colorScore 
               -1.0                -1.0 

$missing
[1] 8 12 13 14
```

The ultrasound profile `x` is a list of two objects: an ultrasound features vector and a vector of indices indentifying missing values.
The MPM launcher gets rid of missing values by imputing them on-the-fly:

```
> mpm <- us.predict(x)
Imputation task is: Classification 
iteration 1 using rpartC in progress...done!
Difference after iteration 1 is 0 
iteration 2 using rpartC in progress...done!
Difference after iteration 2 is 0 

       Morphonode Predictive Model output
# ------------------------------------------------------------------------------------------- #
|     Prediction (Morphonode-RFC): MALIGNANT (Y = 1)                                          |
|  Risk estimate (Morphonode-RBM): 0.916                                                      |
|                      Risk level: HIGH (> 0.29)                                              |
|       Signature (Morphonode-DT): HMR (high metastatic risk)                                 |
|      Estimated prediction error: 0.033     (cutoff: E < 1)                                  |
|                                                                                             |
|                       DIAGNOSIS: MALIGNANT                                                  |
|                                                                                             |
|--- Input profile ---------------------------------------------------------------------------|
|                                                                                             |
| [0] w = 1.000 | 10.00 |  6.30 | 1 | 0 | 0 | 0 | 0 | 1 | 2 | 2 | 3 | 1 | 1 | 1 | Y = 1 | HMR |
|                                                                                             |
|--- Top-similar profiles (w: cosine similarity) ---------------------------------------------|
|                                                                                             |
| [1] w = 0.993 |  9.20 |  6.40 | 1 | 0 | 0 | 0 | 0 | 2 | 2 | 2 | 2 | 1 | 1 | 1 | Y = 1 | HMR |
| [2] w = 0.992 |  7.00 |  3.69 | 1 | 0 | 0 | 0 | 0 | 1 | 1 | 1 | 2 | 1 | 1 | 1 | Y = 0 | MMR |
| [3] w = 0.989 | 11.50 |  6.40 | 1 | 0 | 0 | 0 | 0 | 1 | 1 | 1 | 3 | 1 | 1 | 2 | Y = 1 | MMR |
| [4] w = 0.985 | 10.89 |  7.84 | 1 | 0 | 1 | 0 | 1 | 2 | 3 | 3 | 4 | 2 | 2 | 2 | Y = 1 | MET |
| [5] w = 0.985 | 11.90 |  6.30 | 1 | 0 | 0 | 0 | 1 | 1 | 2 | 3 | 2 | 1 | 2 | 2 | Y = 1 | HMR |
# ------------------------------------------------------------------------------------------- #
```

While missing values are generally not a problem, the imputation is currently disabled (so missing values are not allowed) for **short axis** and **cortical thickness**. This is due to the extremely high importance of these two features for a reliable prediction. Although possible, imputation is strongly discouraged for the metastatic markers **nodal core sign**, **perinodal hyperechogenic ring**, and **cortical interruption**. These values are fundamental to detect multiple metastases (MET signature) profiles, and hardly imputable from the other features in the input profile. As a general warning, each imputed value comes with an imputation error. Although RFCs, and the whole modular structure of the MPM, are robust to missing data, an eccess of missing values might sensibly increase prediction entropy.

### Interactive input

If the `new.profile()` function is launched without arguments, an interactive session is prompted. While short axis and cortical thickness take only numerical values and cannot be missing, the other features are categorical and allow missing values. Pressing *enter* without typing any value will introduce a missing. Each feature comes with a brief description of the allowed values. If an invalid value is given, an error will be raised.

## 1.3. MPM output object

Besides the displayed output, the MPM output (a list of 5 objects) shows some more details about the decisional process:

```
> mpm
$prediction
$prediction$y.hat
[1] 1

$prediction$decisions
[1] 1 1 1 1 1
Levels: 0 1

$prediction$oob.err
  RFC1.OOB   RFC2.OOB   RFC3.OOB   RFC4.OOB   RFC5.OOB 
0.05533597 0.04986877 0.04749340 0.05710491 0.05526316 

$E
0.03281627

$p
0.9161561 

$signature
[1] "HMR"

$profiles
      ID shortAxis cortical hilum inflammatoryStroma extracapsularSpread
1561 722      9.20 6.400000     1                  0                   0
2610 766      7.00 3.692477     1                  0                   0
181  314     11.50 6.400000     1                  0                   0
820  894     10.89 7.841718     1                  0                   1
1131 434     11.90 6.300000     1                  0                   0
     ecostructure FID VFL corticalThickening vascularPattern CMID shape
1561            0   0   2                  2               2    2     1
2610            0   0   1                  1               1    2     1
181             0   0   1                  1               1    3     1
820             0   1   2                  3               3    4     2
1131            0   1   1                  2               3    2     1
     grouping colorScore y signature        E         R        D
1561        1          1 1       HMR 0.016928 0.9927380 1.284523
2610        1          1 0       MMR 0.009800 0.9921939 4.218907
181         1          2 1       MMR 0.014112 0.9888209 2.063977
820         2          2 1       MET 0.006272 0.9854236 2.677498
1131        2          2 1       HMR 0.004232 0.9853030 2.147091
```

The `prediction` object contains the overall classification (`y.hat`), the decision taken by the 5 RFCs in the ensemble (`decisions`), and the out-of-bag error of each RFC (`oob.err`). Objects `E`, `p`, and `signature` contain the estimated RFC prediction error, the RBM malignancy risk estimate, and the computed metastatic risk signature (MRS), respectively. Additionally, the `profiles` object shows the detailed characteristics of the k top-similar profiles, including: ultrasound features, observed phenotype (y), associated MRS (signature), prediction error estimate (E) of the default RFC ensemble, similarity with the input profile (R), and euclidean distance to the input profile (D).

Inspecting this output may add further insights to understand the results. For instance, one of the similar profiles (the second) has a *non-malignant* phenotype, while all the others are *malignant* (in agreement with the predictions). Looking at D, we can see that the second profile is indeed more distant than the others, although they look close according to cosine similarity.

# 2. Additional functionalities

In addition to the prediction suite, the **morphonode** package offers a number of supplementary tools, including:

- ultrasound data **simulation**,
- **builders** for random forest and logistic classifiers,
- functions for **bootstrap-based** confidence intervals and standard error estimation.

## 2.1. Ultrasound data simulation

Data simulation can be convenient for validation purposes or to evaluate the properties of specific ultasound profiles and their impact on patients' phenotype and metastatization risk. The **morphonode** package offers the `us.simulate` function to generate a number of different ultrasound feature vectors. Launched without arguments, `us.simulate()` will create a generic feature vector that, concatenated with the `new.profile` function, allows to create a new simulated profile. It is often convenient restrict the simulation (or part of the simulated dataset) to a specific range of values. Argument `y` restricts the simulation to either malignant (y = 1) or non-malignant (y = 0) phenotypes, while argument `signature` restricts it to one of the MRSs (i.e., LMR, MMR, HMR, MET). Here is a list of examples:

```r
# Simulate a malignant and a non-malignant ultrasound profile
x1 <- new.profile(us.simulate(y = 1))
x0 <- new.profile(us.simulate(y = 0))

# Simulate a LMR, an MMR, an HMR, and a MET profile
x.lmr <- new.profile(us.simulate(signature = "LMR"))
x.mmr <- new.profile(us.simulate(signature = "MMR"))
x.hmr <- new.profile(us.simulate(signature = "HMR"))
x.met <- new.profile(us.simulate(signature = "MET"))

# Since simulation is a stochastic process, based on observed ultrasound features and phenotypes
# frequencies, it is possible to observe a low frequency of unexpected phenotypes 
# (e.g., malignant LMRs or non-malignant METs). Let's test it ...

y.lmr <- us.predict(x.lmr)
y.mmr <- us.predict(x.mmr)
y.hmr <- us.predict(x.hmr)
y.met <- us.predict(x.met)
```

In the vast majority of cases, HMR and MET signatures lead to a malignant phenotype (and a high malignancy risk), whereas LMR will be non-malignant. For a large simulated dataset, low-frequency profiles may arise at a relevant frequency and could be studied to further investigate their ultrasound properties. Argument `reps` allows the creaton of a (reps, m) matrix, where m = 14 is the number of ultrasound features:

```r
lmr0 <- us.simulate(reps = 300, signature = "LMR")
mmr0 <- us.simulate(reps = 200, signature = "MMR", y = 0)
mmr1 <- us.simulate(reps = 100, signature = "MMR", y = 1)
hmr1 <- us.simulate(reps = 200, signature = "HMR")
met1 <- us.simulate(reps = 200, signature = "MET")

simdata <- data.frame(rbind(lmr0, mmr0, mmr1, hmr1, met1))
head(simdata)
dim(simdata)
```

In the example above, we simulated a dataset of 1000 subjects (300 LMR, 300 MMR, 200 HMR, and 200 MET). As shown, the user might enforce a specific phenotype in combination with the MMR signature. This is allowed for MMR only, given its heterogeneous nature. Conversely, for LMR, HMR, and MET signatures, the phenotype is locked to 0, 1, and 1, respecively. For these three signatures, unexpected phenotypes (1, 0, and 0, respectively) might occur at a low rate (roughly 3-4% for LMR, and 5-10% for HMR and MET). Naturally, simulated datasets can be created based on phenotype frequencies:

```r
y0 <- us.simulate(reps = 300, y = 0)
y1 <- us.simulate(reps = 200, y = 1)

simdata <- data.frame(rbind(y0, y1))
head(simdata)
dim(simdata)
```

The default **morphonode** simulated dataset (object `mpm.us`) is a data.frame of 948 rows (simulated ultrasound profiles) and 18 columns, including: a progressive number used as unique profile identifier (ID), 14 ultrasound features used for RFC and RBM building, the expected phenotype used as ground truth (y = 0: non-malignant, 1: malignant), a metastatic risk signature associated to each simulated ultrasound profile (signature), and subject-level prediction error calculated as Brier score (E). This dataset was generated as a 4-fold expansion (n = 948, 440 malignant and 508 non-malignant) of the original ultrasound feature dataset of 237 groin samples (75 malignant and 162 non-malignant) from Fragomeni et al. (2022), using the **morphonode** simulation utility.

## 2.2. RFC building and validation

The **morphonode** package offers a number of functions to build and validate new RFC-based classifiers on-the-fly. These include: `buildPredictor`, `vpart`, and `brier`. The `buildPredictor` function creates an object of class `randomForest` (Liaw et al. 2002), including its performances, if a validation set is given:

```r
# Firstly, let's create an ultrasound feature dataset of N subjects (e.g., N = 500).
# In absence of a real dataset, this can be done by either simulating it ...

y0 <- us.simulate(reps = 300, y = 0)
y1 <- us.simulate(reps = 200, y = 1)
x <- data.frame(rbind(y0, y1))

# ... or by random sampling N subjects from the mpm.us object:

x <- mosaic::sample(mpm.us, 500, replace = FALSE, prob = NULL)
x <- x[, 2:16]     # This will include ultrasound features and phenotypes only

# Secondly, we prepare the ultrasound features for classification (i.e., by cheching the right
# data types), and partition the input dataset in 75% training and 25% validation, using the 
# vpart function:

x <- check.rfcdata(x)
x <- vpart(x, p = 0.75)

# Finally, we define the model and build the RFC using the x$training.set and x$validation.set, 
# generating 10000 bootstrapped trees and using 3 random variables per tree branching 
# (this may take a while):

model <- formula("y ~ .")

rfc <- buildPredictor(model, x$training.set, vset = x$validation.set, n = 10000, m = 3)
```

The output message should look like this:

```
Call:
 randomForest(formula = model, data = data, ntree = n, mtry = m,
              keep.forest = TRUE, proximity = TRUE, importance = TRUE)
              
               Type of random forest: classification
                     Number of trees: 10000
No. of variables tried at each split: 3

         OOB estimate of  error rate: 5.6%
        
Confusion matrix:
    0   1 class.error
0 192  10  0.04950495
1  11 162  0.06358382
```

The `rfc$RFC` variable is a `randomForest` object, corresponding to the classifier we have just built. In addition, the `rfc$performance` object  includes the performance indices generated by the application of the `rfc$RFC` classifier over the `x$validation.set`.
If one is not interested in validating the classifier, the `vpart` command can be skipped and the `rfc` object will coincide with the `randomForest` classifier (the `performance` list is not generated). The RFC also contains the object `rfc$RFC$importance`. This is a matrix reporting Mean Decrease Accuracy and Mean Decrease Gini impurity. The former evaluates the contribution of each feature in the predictive accuracy, while the latter evaluates their contribution to classification entropy. These indices can be used to assess the importance of each feature and rank them accordingly.

The out-of-bag error estimates and validation performances evaluate the classifier at dataset level. The **morphonode** package provides the `brier` function to evaluate the prediction error at subject level. Considering the training dataset used above (`x`):

```r
# Extract the phenotype vector
y <- x$training.set$y

# Extract the ultrasound features
data <- x$training.set[, 1:14]

# Compute the Brier scores for each subject
E <- brier(data, y)

# As a rule of thumb, if E is greater or equal to 1, the prediction should be 
# considered as unreliable.
quantile(E)
err <- length(E[E >= 1])
err
err/length(E)
```

Currently, the **morphonode** ensemble classifier is stored in the object `mpm.rfc`. It contains: (i) the 5 `randomForest` objects of the RFC ensemble (`mpm.rfc$rfc`), as well as their training (`mpm.rfc$training`) and validation (`mpm.rfc$validation`) sets, (ii) RFC validation performances (`mpm.rfc$performance`), and (iii) ultrasound feature ranking based on the average of minmax-normalized MDA and MDG values (`mpm.rfc$ranking`).

## 2.3. Building a robust binomial model (RBM)

One of the modules of the MPM suite is the RBM, for the estimation of malignancy risk. Two main limitations could affect the performances of a regression model, including the logistic one: (i) deviation from normality constraints, and (ii) high dimensionality (#variables >> #subjects). Both these aspects impacted the development of a risk estimation model in the MPM suite. The former was (at least partially) addressed by estimating bootstrap standard errors (SE), through the **morphonode** function `boot.se`. The latter, issued by the low frequency of certain values (levels) of categorical ultrasound features, was addressed by using a dichotomized version of each regressor. Let us create the building blocks of the RBM:

```r
# The first step is to obtain a dichotomous version of the MPM ultrasound feature dataset
x <- dichotomize(mpm.us, asFactor = TRUE)

# Secondly, we define the model to be fitted
model <- formula(paste0(c("y ~ shortAxis + cortical + hilum + ",
                               "inflammatoryStroma + extracapsularSpread + ",
                               "ecostructure + FID + VFL + corticalThickening + ",
                               "vascularPattern + CMID + shape + grouping + ",
                               "colorScore"), collapse = ""))

# Finally, we fit both a maximum likelihood and a bootstrapped version of the model
# (bootstrap may take a while)

fit <- glm(model, data = x, family = "binomial")

n.reps <- 2000
boot <- mosaic::do(n.reps) * coef(glm(model, data = mosaic::resample(x), family = "binomial"))

# The boot.se function will compute robust SE, related 95% confidence intervals, and P-values
SE <- boot.se(fit, boot)
SE
```

The **morphonode** object `mpm.rbm` collects these results in three objects: `mpm.rbm$model` (the fitted model), `mpm.rbm$fit` (MLE model fitting results), and `mpm.rbm$coef` (bootstrap estimates, 95% confidence intervals, and P-values).
It is also possible to assess RBM performances by using risk estimates and the `performance` function:

```r
# Compute malignancy risks in batch
p <- predict(mpm.rbm$fit, dichotomize(mpm.us[, 2:15], asFactor = TRUE), type = "response")

# Dichotomize risk values and compare them against the observed phenotypes 
# (p = 0.29 is the high-risk cutoff)
y.hat <- ifelse(p > 0.29, 1, 0)
P <- performance(obs = mpm.us$y, pred = y.hat)
P
```

## 2.4. Generating bootstrap confidence intervals for predictive performance indices

In some cases (e.g., heterogeneous, heteroschedastic, or non-gaussian data), it could be convenient to compute robust confidence intervals (CI) also for predictive performance indices, independenttly from the method used to do the prediction. The **morphonode** function `p.boot` enables the computation of point estimate and bootstrap CI, starting from observed and predicted values, for: F1 score (`f1`), `accuracy`, `sensitivity`, `specificity`, positive predictive value (`ppv` or `precision`), negative predictive value (`npv`), positive likelihood ratio (`plr`), negative likelihood ratio (`nlr`), false positive rate (`fpr`), false negative rate (`fnr`), false negative cost (`fnc`, fn/(tp + tn)). By default, `p.boot` computes the bias-corrected and accelerated (BCa) bootstrap CI (DiCiccio and Efron, 1996) for the F1 score, with 5000 sampling iterations. This number of iterations should ensure algorithm convergence (a smaller number could be insufficient). Let's see some examples:

```r
# Let's evaluate the RFC1 (first RFC of the ensemble) performances
y.hat <- predict(mpm.rfc$rfc$RFC1, mpm.rfc$validation$V1)

# Actual ultrasound phenotype values, taken from the first validation set
y <- mpm.rfc$validation$V1$y

# The p.boot input is a data.frame with two columns: predicted and observed values
Y <- data.frame(y.hat, y)

# F1 score (default performance index) bootstrap confidence intervals
F1 <- p.boot(Y)
F1$boot$t0                # F1 score observed value
F1$ci$bca[4:5]            # F1 score bca confidence interval

# Accuracy bootstrap confidence intervals
A <- p.boot(Y, formula = "accuracy")
A$boot$t0                 # Accuracy observed value
A$ci$bca[4:5]             # Accuracy bca confidence interval 

# You may try other RFCs and indices ...
```

&nbsp;

# Reference

Fragomeni SM, Moro F, Palluzzi F, Mascilini F, Rufini V, Collarino A, Inzani F, Giacò L, Scambia G, Testa AC, Garganese G. **Evaluating the Risk of Inguinal Lymph Node Metastases before Surgery Using the Morphonode Predictive Model: A Prospective Diagnostic Study in Vulvar Cancer Patients**. Cancers 2023, 15(4), 1121; https://doi.org/10.3390/cancers15041121
