# Morphonode Predictive Model (MPM)
The **Morphonode Predictive Model**, and the related **R** package `morphonode`, is the computational framework for ultrasound signatures detection and malignancy prediction in vulvar cancer.

&nbsp;

## Install the latest release from the source package

The latest stable release is the **morphonode version 1.0.0**. You can download this and older versions from the "Releases" panel of this website.
The zip or tar.gz package can be installed (from the home directory) in **R** with:

``` r
install.packages("~/morphonode-1.0.0.tar.gz", repos = NULL)
```
&nbsp;

## Installation from source through the R package `devtools`

The latest development version can be installed through **GitHub** from any **R** environment:

``` r
# install.packages("devtools")
devtools::install_github("Morphonodepredictivemodel/morphonode")
```

This will download the source code, build the morphonode tar.gz package, and install it locally.

&nbsp;

# Using the Morphonode Predictive Model suite

## Morphonode Predictive Model basic usage

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
The results should look like the following:
```
       Morphonode Predictive Model output
# ------------------------------------------------------------------------------------------- #
|     Prediction (Morphonode-RFC): MALIGNANT (Y = 1)                                          |
|      Estimated prediction error: 0.029     (cutoff: E < 1)                                  |
|  Risk estimate (Morphonode-RBM): 0.974                                                      |
|                      Risk level: HIGH (> 0.29)                                              |
|       Signature (Morphonode-DT): HMR (high metastatic risk)                                 |
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

- **Morphonode-RFC**. Random forest classification (RFC) and prediction error (E) estimate.
  The predicted phenotype can be either *malignant* (y = 1) or *non-malignant* (y = 0).
  As a rule of thumb, if E is above or equal to 1, the prediction should be considered as unreliable.
- **Morphonode-RBM**. Malignancy risk estimation by robust binomial modeling (RBM).
  The RBM offers a continuous estimation of y (i.e., the malignancy risk), thus the higher the accordance with the RFC, the higher the prediction
  reliability.
  This module also suggests when the risk reach *moderate* (p > 0.23) or *high* levels (p > 0.29). These cutoffs reflect the optimal risk cutpoint between
  malignant and non-malignant subjects, by maximizing F1 score and Sensitivity/Specificity, respectively.
- **Morphonode-DT**. Decision tree-based metastatic risk signature detection.
  A *high metastatic risk* (HMR) signature is characterized by a high risk of a single metastasis event, whereas
  a *metastatic signature* (MET) is typical of malignancies showing multiple metastasis events.
  Conversely, a *low metastatic risk* (LMR) signature is generally associated with non-malignant phenotypes.
  Finally, a *moderate malignancy risk* (MMR) signature is the group with highest heterogeneity and requires RFC and RBM results to be characterized.
- **Morphonode-SP**. Similarity prolfiling module. The module searches and ranks ultrasound profiles from the given (by default, the simulated) ultrasound
  features dataset. The default function is cosine similarity and 5 top-similar profiles are shown to screen.

Both the input ultrasound profile and the similar ones are reported as a list of attributes, including:

1. **Progressive number** (the smaller the number, the higher the similarity; "0" is the input profile).
2. **Similarity coefficient** (w). By default, cosine similarity is used.
3. **Short axis**: length in millimeters.
4. **Cortical thickness**: thickness in millimeters.
5. **Nodal core sign** (hilum): *absent* (0, metastatic trait), *present* (1).
6. **Perinodal hyperechogenic ring**: *absent* (0), *present* (1, metastatic trait).
7. **Cortical interruption**: *absent* (0), *present* (1, metastatic trait).
8. **Echogenicity**: *homogeneous* (0), *inhomogeneous* (1).
9. **Focal intranodal deposit**: *absent* (0), *hyperechoic* (1), *anaechoic, cystic areas* (2), *both* (3).
10. **Vascular flow localization**: *non-vascularized* (0), *central* (1), *peripheral* (2), *extranodal* (3), *combined* (4).
11. **Cortical thickening**: *not evaluable* (0), *absent* (1), *focal* (2), *concentric* (3), *eccentric* (4).
12. **Vascular flow architecture pattern**: *non-vascularized* (0), *longitudinal axis* (1), *scattered* (2), *branched* (3), *chaotic* (4).
13. **Cortical-Medullar interface distortion**: *absent* (1), *focal* (2), *diffused* (3), *medulla not visible* (4).
14. **Shape**: *elliptic* (1), *circular* (2), *irregular* (3).
15. **Grouping**: *absent* (1), *moderate* (2), *complete* (3).
16. **Color score**: ordinal variable from 1 to 5.
17. **Outcome**: *non-malignant* (0), *malignant* (1).
18. **Metastatic risk signature**: LMR (low risk), MMR (moderate risk), HMR (high risk, single metastasis), MET (metastatic, multiple metastases).

## Defining an ultrasound profile

A new profile can be initialized manually, including each ultrasound value in the same order of the previous chapter (points 3 to 16).

```r
x <- new.profile(c(10.0, 6.3, 1, 0, 0, 0, 0, -1, 2, 2, 3, -1, -1, -1))
```

As shown in the coden above, the object `x` contains -1 values corresponding to missing data:

```
> x
$ultrasound
          shortAxis            cortical               hilum  inflammatoryStroma 
               10.0                 6.3                 1.0                 0.0 
extracapsularSpread        ecostructure                 FID                 VFL 
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

```r
...
```








