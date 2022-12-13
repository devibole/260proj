260 Project
================
Devika Godbole
2022-12-13

# Predicting Dental Care Utilization

## Introduction

One in every five children experience special healthcare needs in the
United States (Williams 2021). Special healthcare needs are defined as
“physical, intellectual, and developmental disabilities, as well as
long-standing medical conditions, such as asthma, diabetes, a blood
disorder, or muscular dystrophy” (CDC 2019).

Previous studies reported that children with special healthcare needs
(CSHCN) experience a higher burden of oral health problems compared to
those who are not CSHCN (Lebrun-Harris 2021). CSHCN are at a higher risk
for dental trauma injuries due to increased fall risk and dental
anomalies such as supernumerary teeth (Devinsky 2020). Medications
prescribed to treat conditions among CSHCN, such as seizure disorders,
lead to oral conditions including xerostomia and gingival hyperplasia
(Devinsky 2020). Sociobehaviorally, CSHCN may be more prone to a
cariogenic diet due to a higher consumption of sugary foods, further
raising the risks of dental caries (Devinsky 2020).

The high caries risk among CSHCN may also be attributed to challenges
with home care, specifically oral hygiene maintenance. For example, a
survey on pre-school aged CSHCN found that four out of five CSHCN
parents reported difficulties with brushing their children’s teeth
(Huebner 2015). Limitations in dental services available to CSHCN is
also a major barrier to consider. In the United States, there is a
national shortage of dentists and healthcare institutions equipped to
provide dental services for CSHCN (Lebrun-Harris 2021). Reasons why
CSHCN families struggle to find dental care include dentists refusing to
accept Medicaid or treat patients who are CSHCN (Kagihara 2011). Within
dental education, the current training for dental professionals in
caring for CSHCN is highly limited–major inconsistencies exist across
dental education institutions in both pre-doctoral and residency
training (Inclan 2019) (Baker 2017).

While dental care utilization among children has been trending upwards
in the past decade in the U.S., the COVID-19 pandemic poses an
additional challenge to oral health access, potentially especially for
CSHCN. We consider two modeling methods to determine important features
for predicting a lack of dental care utilization and assess whether
special health care needs children are particularly affected.

## Methods

The study used 1) a more biostatistical logistic regression utilizing
domain knowledge to select covariates as well as 2) less user defined
random forest using all available features to predict dental care
utilization during the COVID-19 pandemic using data from the 2020
National Survey of Children’s Health (N=41,380). Variables in the source
included children’s demographic background and health information. The
data was split into 80% training and 20% testing set.

The National Survey of Children’s Health (NSCH) is an annual,
cross-sectional survey conducted by the U.S. Census Bureau and funded by
the Health Resources and Services Administration’s Maternal and Child
Health Bureau. This publicly-available survey dataset provided
state-level data on the health and well-being of children aged 0 to 17.
The 2020 survey was administered from June 2020 to January 2021 and is
accordingly considered “during pandemic” for this purposes of this
study. Both children and their households were randomly sampled and
screened. Caregivers of eligible children were surveyed via the phone,
internet, or mail in English or Spanish.

Modeling was completed using R/R Studio Software, version 1.3.1093.
Children aged less than 1 were excluded from the analysis and any
columns with missingness were removed. The total study population after
exclusions was N = 31229 for this data set.

The study implemented two models – a logistic regression and a random
forest model. The logistic regression used glm net with cv set to lambda
min as the regularization method. Both models consisted of “was dental
care utilized during the past 12 months year” as the binary response
variable along with typical sociodemographic covariates (age, sex,
race/ethnicity, federal poverty level, state of residence, region of
residence, health insurance coverage, and primary caregiver’s marital
status, and SHCN). The random forest algorithm had a larger selection of
available features however. Confusion matrixes were created for the test
set predictions for each model and the metrics were compared.

## Analysis

``` r
library(haven)
library(knitr)
library(kableExtra)
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following object is masked from 'package:kableExtra':
    ## 
    ##     group_rows

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(car)
```

    ## Loading required package: carData

    ## 
    ## Attaching package: 'car'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     recode

``` r
library(jtools)
library(ggplot2)
library(ggpubr)
```

    ## Registered S3 methods overwritten by 'broom':
    ##   method            from  
    ##   tidy.glht         jtools
    ##   tidy.summary.glht jtools

``` r
library(caret)
```

    ## Loading required package: lattice

``` r
library(randomForest)
```

    ## randomForest 4.7-1.1

    ## Type rfNews() to see new features/changes/bug fixes.

    ## 
    ## Attaching package: 'randomForest'

    ## The following object is masked from 'package:ggplot2':
    ## 
    ##     margin

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     combine

``` r
library(gbm)
```

    ## Loaded gbm 2.1.8.1

``` r
library(e1071)
library(gtsummary)
library(glmnet)
```

    ## Loading required package: Matrix

    ## Loaded glmnet 4.1-4

``` r
library(vip)
```

    ## 
    ## Attaching package: 'vip'

    ## The following object is masked from 'package:utils':
    ## 
    ##     vi

``` r
library(pander)
library(ConfusionTableR)
library(table1)
```

    ## 
    ## Attaching package: 'table1'

    ## The following objects are masked from 'package:base':
    ## 
    ##     units, units<-

``` r
nsch2020 <- read_sas("~/Downloads/nsch_2020_topical_SAS/nsch_2020_topical.sas7bdat",NULL)
```

``` r
x_2020 <- nsch2020[, which(colMeans(is.na(nsch2020)) == 0)]
x_2020$logit_Q1 <- ifelse(nsch2020$K4Q30_R == 1 | nsch2020$K4Q30_R == 2, 1, 0)
```

``` r
x_2020$SC_CSHCN <- ifelse(nsch2020$SC_CSHCN==1, 1, 0)
x_2020$age <- nsch2020$SC_AGE_YEARS
x_2020$fpl <- nsch2020$FPL_I1
x_2020$sex <- ifelse(nsch2020$SC_SEX == 1, 1, 0)

x_2020$insurance <- ifelse(nsch2020$CURRCOV == 2, 4, 
                 ifelse(nsch2020$K12Q12 == 1, 1, 
                 ifelse(nsch2020$K12Q03 == 1 | nsch2020$K12Q04 == 1, 2, 
                 ifelse(nsch2020$K11Q03R == 1 | nsch2020$HCCOVOTH == 1 | nsch2020$TRICARE == 1 | nsch2020$CURRCOV == 1, 3, 0))))

x_2020$race <- ifelse(!nsch2020$SC_HISPANIC_R == 1, nsch2020$SC_RACE_R, 6)
x_2020$married <- ifelse(nsch2020$A1_MARITAL == 1 |nsch2020$A1_MARITAL == 2, 1, ifelse(nsch2020$A1_MARITAL == 3 |nsch2020$A1_MARITAL == 4, 2, 3))

x_2020$regionori <- nsch2020$FIPSST

x_2020$region <- ifelse(x_2020$regionori %in% c(9, 23, 25, 33, 44, 50, 34, 36, 42), 1, ifelse(x_2020$regionori %in% c(18, 17, 26, 39, 55, 19, 31, 20, 38, 27, 46, 29), 2, ifelse(x_2020$regionori %in% c(10, 11, 12, 13, 24, 37, 45, 51, 54, 1, 21, 28, 47, 5, 22, 40, 48), 3, ifelse(x_2020$regionori %in% c(4, 8, 16, 35, 30, 49, 32, 56, 2, 6, 15, 41, 53), 4, 0))))

x_2020$regionori <- NULL
x_2020$region_ori <- NULL
x_2020$nsch2020.SC_CSHCN <- NULL

x_2020$HHID <- nsch2020$HHID
x_2020$STRATUM <- nsch2020$STRATUM
x_2020$FIPSST <- nsch2020$FIPSST
x_2020$FWC <- nsch2020$FWC
x_2020$lang <- nsch2020$HHLANGUAGE

###additional variables for specifiying SHCN 
x_2020$autism <- nsch2020$K2Q35A
x_2020$downsyn <- nsch2020$K2Q35A
x_2020$add <- nsch2020$K2Q31A

x_2020$employ <- ifelse(nsch2020$A1_EMPLOYED == 1 | nsch2020$A1_EMPLOYED == 2, 1, 0)
x_2020$married <- ifelse(nsch2020$A1_MARITAL == 1 |nsch2020$A1_MARITAL == 2, 1, ifelse(nsch2020$A1_MARITAL == 3 |nsch2020$A1_MARITAL == 4, 2, 3))
x_2020$mental <- nsch2020$A1_MENTHEALTH
x_2020$physical <- nsch2020$A1_PHYSHEALTH
x_2020$edu <- nsch2020$HIGRADE

x_2020$toothaches <- nsch2020$TOOTHACHES
x_2020$cavities <- nsch2020$CAVITIES
x_2020$allergies <- nsch2020$ALLERGIES
x_2020$docroom <- nsch2020$DOCROOM
x_2020$hcability <- nsch2020$HCABILITY

x_2020 <- x_2020[x_2020$age > 1,]
```

``` r
Q1_2020 <- x_2020 %>% filter(!is.na(SC_CSHCN) ) %>%
  filter(!is.na(logit_Q1)) %>% 
  mutate(cshcn= ifelse(SC_CSHCN==1, "CSHCN" , "Non-CSHCN")) %>% 
  mutate(Q = ifelse(logit_Q1 ==1, "Yes", "No"))  
```

``` r
PP2 <- Q1_2020 %>% ggplot(aes(x = as.factor(cshcn), fill= as.factor(Q))) +
  geom_bar(position = position_stack(), aes(group = as.factor(Q))) +
  geom_text(stat = "count", aes(label = after_stat(count)), position = position_stack(vjust = 0.5), size = 2.5) +
  xlab("2020 During Pandemic")+ 
  ggtitle("Dental Care Utilization in the Past Year")+
  ylab("") 

PP2
```

![](12-13project_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
PP1 <- Q1_2020 %>% filter(logit_Q1 == 1) %>% ggplot(aes(x=race, color = "red")) +
  geom_bar() + 
  geom_text(stat = "count", aes(label = after_stat(count)), position = position_stack(vjust = 0.5), size = 2.5) +
  xlab("Ethnicity Response")+ 
  ggtitle("Yes Dental Care Utilization in the Past Year by Race")+
  ylab("") 
P1<- PP1 + guides(fill=guide_legend(title="Answer"))

PP2 <- Q1_2020 %>% filter(logit_Q1 == 0) %>% ggplot(aes(x=race, color = "lightblue")) +
  geom_bar() + 
  geom_text(stat = "count", aes(label = after_stat(count)), position = position_stack(vjust = 0.5), size = 2.5) +
  xlab("Ethnicity Response")+ 
  ggtitle("No Dental Care Utilization in the Past Year by Race")+
  ylab("") 
P2<- PP2 + guides(fill=guide_legend(title="Answer"))

arrange1<- ggarrange(PP1, PP2,  common.legend = TRUE, legend = "bottom") 
annotate_figure(arrange1,
               top = text_grob("Dental Care Utilization in the Past Year", color = "orange", size = 13),
               left = text_grob("Count", rot = 90)) 
```

![](12-13project_files/figure-gfm/unnamed-chunk-6-2.png)<!-- -->

``` r
set.seed(1212)
nschAnalysis <- x_2020
nschAnalysis$logit_Q1 <- as.factor(nschAnalysis$logit_Q1)
nschAnalysis$sex <- as.factor(nschAnalysis$sex)
nschAnalysis$insurance <- as.factor(nschAnalysis$insurance)
nschAnalysis$race <- as.factor(nschAnalysis$race)
nschAnalysis$married <- as.factor(nschAnalysis$married)
nschAnalysis$region <- as.factor(nschAnalysis$region)
nschAnalysis$lang <- as.factor(nschAnalysis$lang)

nschAnalysis$edu <- as.factor(nschAnalysis$edu)
nschAnalysis$mental <- as.factor(nschAnalysis$mental)
nschAnalysis$physical <- as.factor(nschAnalysis$physical)
nschAnalysis$employ<-as.factor(nschAnalysis$employ)
nschAnalysis$lang <- as.factor(nschAnalysis$lang)

nschAnalysis <- nschAnalysis[complete.cases(nschAnalysis),]
dim(nschAnalysis)
```

    ## [1] 31229    65

``` r
table1(~ SC_CSHCN + age + fpl + sex + insurance + race + married + region + lang + employ + edu  | logit_Q1, data=nschAnalysis)
```

    ##                                                          0                 1
    ## 1                                                 (N=3936)         (N=27293)
    ## 2                               SC_CSHCN                                    
    ## 3                              Mean (SD)     0.226 (0.418)     0.280 (0.449)
    ## 4                      Median [Min, Max]       0 [0, 1.00]       0 [0, 1.00]
    ## 5       Age of Selected Child - In Years                                    
    ## 6                              Mean (SD)       6.52 (5.11)       10.3 (4.55)
    ## 7                      Median [Min, Max] 4.00 [2.00, 17.0] 11.0 [2.00, 17.0]
    ## 8  Family Poverty Ratio, First Implicate                                    
    ## 9                              Mean (SD)         260 (127)         300 (120)
    ## 10                     Median [Min, Max]   267 [50.0, 400]   370 [50.0, 400]
    ## 11                                   sex                                    
    ## 12                                     0      1809 (46.0%)     13377 (49.0%)
    ## 13                                     1      2127 (54.0%)     13916 (51.0%)
    ## 14                             insurance                                    
    ## 15                                     1      1257 (31.9%)      5950 (21.8%)
    ## 16                                     2      2290 (58.2%)     19720 (72.3%)
    ## 17                                     3        139 (3.5%)        934 (3.4%)
    ## 18                                     4        250 (6.4%)        689 (2.5%)
    ## 19                                  race                                    
    ## 20                                     1      2457 (62.4%)     18986 (69.6%)
    ## 21                                     2        303 (7.7%)       1598 (5.9%)
    ## 22                                     3         35 (0.9%)        152 (0.6%)
    ## 23                                     4        252 (6.4%)       1278 (4.7%)
    ## 24                                     5         15 (0.4%)         58 (0.2%)
    ## 25                                     6       573 (14.6%)      3256 (11.9%)
    ## 26                                     7        301 (7.6%)       1965 (7.2%)
    ## 27                               married                                    
    ## 28                                     1      3245 (82.4%)     22606 (82.8%)
    ## 29                                     2       547 (13.9%)      3801 (13.9%)
    ## 30                                     3        144 (3.7%)        886 (3.2%)
    ## 31                                region                                    
    ## 32                                     0       527 (13.4%)      3846 (14.1%)
    ## 33                                     1       537 (13.6%)      4168 (15.3%)
    ## 34                                     2       963 (24.5%)      6415 (23.5%)
    ## 35                                     3      1155 (29.3%)      7154 (26.2%)
    ## 36                                     4       754 (19.2%)      5710 (20.9%)
    ## 37                                  lang                                    
    ## 38                                     1      3549 (90.2%)     25813 (94.6%)
    ## 39                                     2        183 (4.6%)        764 (2.8%)
    ## 40                                     3        204 (5.2%)        716 (2.6%)
    ## 41                                employ                                    
    ## 42                                     0      1016 (25.8%)      5354 (19.6%)
    ## 43                                     1      2920 (74.2%)     21939 (80.4%)
    ## 44                                   edu                                    
    ## 45                                     1        114 (2.9%)        469 (1.7%)
    ## 46                                     2       672 (17.1%)      2833 (10.4%)
    ## 47                                     3      3150 (80.0%)     23991 (87.9%)
    ##              Overall
    ## 1          (N=31229)
    ## 2                   
    ## 3      0.273 (0.445)
    ## 4        0 [0, 1.00]
    ## 5                   
    ## 6        9.85 (4.79)
    ## 7  10.0 [2.00, 17.0]
    ## 8                   
    ## 9          295 (121)
    ## 10   354 [50.0, 400]
    ## 11                  
    ## 12     15186 (48.6%)
    ## 13     16043 (51.4%)
    ## 14                  
    ## 15      7207 (23.1%)
    ## 16     22010 (70.5%)
    ## 17       1073 (3.4%)
    ## 18        939 (3.0%)
    ## 19                  
    ## 20     21443 (68.7%)
    ## 21       1901 (6.1%)
    ## 22        187 (0.6%)
    ## 23       1530 (4.9%)
    ## 24         73 (0.2%)
    ## 25      3829 (12.3%)
    ## 26       2266 (7.3%)
    ## 27                  
    ## 28     25851 (82.8%)
    ## 29      4348 (13.9%)
    ## 30       1030 (3.3%)
    ## 31                  
    ## 32      4373 (14.0%)
    ## 33      4705 (15.1%)
    ## 34      7378 (23.6%)
    ## 35      8309 (26.6%)
    ## 36      6464 (20.7%)
    ## 37                  
    ## 38     29362 (94.0%)
    ## 39        947 (3.0%)
    ## 40        920 (2.9%)
    ## 41                  
    ## 42      6370 (20.4%)
    ## 43     24859 (79.6%)
    ## 44                  
    ## 45        583 (1.9%)
    ## 46      3505 (11.2%)
    ## 47     27141 (86.9%)

## Results

``` r
intrain <- caret::createDataPartition(y = nschAnalysis$logit_Q1, p= 0.6, list = FALSE)
training <- nschAnalysis[intrain,]
testing <- nschAnalysis[-intrain,]
```

First, we conduct the logistic regression using glmnet. Glmnet is a
package that can fit our logistic regression using cross validation,
which is a resampling method that uses different portions of the data to
test and train a model on different iterations. via penalized maximum
likelihood. The regularization path is computed for the lasso or elastic
net penalty at a grid of values (on the log scale) for the
regularization parameter lambda. In our analysis, we set lambda =
lambda.min which is the value that gives minimum mean cross-validated
error.

``` r
##LOGISTIC REGRESSION
x <- model.matrix(logit_Q1~ SC_CSHCN + age + fpl + sex + insurance + race + married + region + lang + employ + edu, training)
# Convert the outcome (class) to a numerical variable
y <- ifelse(training$logit_Q1 == "1", 1, 0)
cv.lasso <- cv.glmnet(x, y, alpha = 1, family = "binomial")
plot(cv.lasso)
```

![](12-13project_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

``` r
# Fit the final model on the training data
model <- glmnet(x, y, alpha = 1, family = "binomial", lambda = cv.lasso$lambda.min)
# Display regression coefficients
c <- data.frame(coef.name = dimnames(coef(model))[[1]], coef.value = matrix(coef(model)))
c[-2,]%>% kable()
```

<table>
<thead>
<tr>
<th style="text-align:left;">
</th>
<th style="text-align:left;">
coef.name
</th>
<th style="text-align:right;">
coef.value
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:left;">
(Intercept)
</td>
<td style="text-align:right;">
-0.1658922
</td>
</tr>
<tr>
<td style="text-align:left;">
3
</td>
<td style="text-align:left;">
SC_CSHCN
</td>
<td style="text-align:right;">
-0.0036404
</td>
</tr>
<tr>
<td style="text-align:left;">
4
</td>
<td style="text-align:left;">
age
</td>
<td style="text-align:right;">
0.1835110
</td>
</tr>
<tr>
<td style="text-align:left;">
5
</td>
<td style="text-align:left;">
fpl
</td>
<td style="text-align:right;">
0.0016219
</td>
</tr>
<tr>
<td style="text-align:left;">
6
</td>
<td style="text-align:left;">
sex1
</td>
<td style="text-align:right;">
-0.1213523
</td>
</tr>
<tr>
<td style="text-align:left;">
7
</td>
<td style="text-align:left;">
insurance2
</td>
<td style="text-align:right;">
0.1917064
</td>
</tr>
<tr>
<td style="text-align:left;">
8
</td>
<td style="text-align:left;">
insurance3
</td>
<td style="text-align:right;">
0.0000000
</td>
</tr>
<tr>
<td style="text-align:left;">
9
</td>
<td style="text-align:left;">
insurance4
</td>
<td style="text-align:right;">
-0.8340855
</td>
</tr>
<tr>
<td style="text-align:left;">
10
</td>
<td style="text-align:left;">
race2
</td>
<td style="text-align:right;">
-0.1191744
</td>
</tr>
<tr>
<td style="text-align:left;">
11
</td>
<td style="text-align:left;">
race3
</td>
<td style="text-align:right;">
-0.4427671
</td>
</tr>
<tr>
<td style="text-align:left;">
12
</td>
<td style="text-align:left;">
race4
</td>
<td style="text-align:right;">
-0.2397384
</td>
</tr>
<tr>
<td style="text-align:left;">
13
</td>
<td style="text-align:left;">
race5
</td>
<td style="text-align:right;">
0.0000000
</td>
</tr>
<tr>
<td style="text-align:left;">
14
</td>
<td style="text-align:left;">
race6
</td>
<td style="text-align:right;">
-0.0770175
</td>
</tr>
<tr>
<td style="text-align:left;">
15
</td>
<td style="text-align:left;">
race7
</td>
<td style="text-align:right;">
0.0000000
</td>
</tr>
<tr>
<td style="text-align:left;">
16
</td>
<td style="text-align:left;">
married2
</td>
<td style="text-align:right;">
0.0000000
</td>
</tr>
<tr>
<td style="text-align:left;">
17
</td>
<td style="text-align:left;">
married3
</td>
<td style="text-align:right;">
-0.0571841
</td>
</tr>
<tr>
<td style="text-align:left;">
18
</td>
<td style="text-align:left;">
region1
</td>
<td style="text-align:right;">
0.0000000
</td>
</tr>
<tr>
<td style="text-align:left;">
19
</td>
<td style="text-align:left;">
region2
</td>
<td style="text-align:right;">
-0.0395036
</td>
</tr>
<tr>
<td style="text-align:left;">
20
</td>
<td style="text-align:left;">
region3
</td>
<td style="text-align:right;">
-0.0570257
</td>
</tr>
<tr>
<td style="text-align:left;">
21
</td>
<td style="text-align:left;">
region4
</td>
<td style="text-align:right;">
0.1367816
</td>
</tr>
<tr>
<td style="text-align:left;">
22
</td>
<td style="text-align:left;">
lang2
</td>
<td style="text-align:right;">
0.0043683
</td>
</tr>
<tr>
<td style="text-align:left;">
23
</td>
<td style="text-align:left;">
lang3
</td>
<td style="text-align:right;">
-0.2435566
</td>
</tr>
<tr>
<td style="text-align:left;">
24
</td>
<td style="text-align:left;">
employ1
</td>
<td style="text-align:right;">
0.0071198
</td>
</tr>
<tr>
<td style="text-align:left;">
25
</td>
<td style="text-align:left;">
edu2
</td>
<td style="text-align:right;">
-0.2656263
</td>
</tr>
<tr>
<td style="text-align:left;">
26
</td>
<td style="text-align:left;">
edu3
</td>
<td style="text-align:right;">
0.1890289
</td>
</tr>
</tbody>
</table>

``` r
vip(model, num_features=10, geom="point")
```

![](12-13project_files/figure-gfm/unnamed-chunk-10-2.png)<!-- -->

``` r
# Make predictions on the test data
x.test <- model.matrix(logit_Q1 ~ SC_CSHCN + age + fpl + sex + insurance + race + married + region + lang + employ + edu, testing)
probabilities <- model %>% predict(newx = x.test)
predicted.classes <- as.factor(ifelse(probabilities > 0.5, 1, 0))
predicted <- as.data.frame(cbind(class_preds=as.factor(predicted.classes), Class = as.factor(testing$logit_Q1)))
predicted$class_preds <- as.factor(predicted$class_preds)
predicted$Class <- as.factor(predicted$Class)
ConfusionTableR::binary_visualiseR(train_labels = predicted$class_preds,
                                   truth_labels= predicted$Class,
                                   class_label1 = "Not utilized", 
                                   class_label2 = "Utilized",
                                   quadrant_col1 = "#28ACB4", 
                                   quadrant_col2 = "#4397D2", 
                                   custom_title = "Dental Care Utilization Confusion Matrix", 
                                   text_col= "black")
```

![](12-13project_files/figure-gfm/unnamed-chunk-10-3.png)<!-- -->

``` r
# prop.table(table(nschAnalysis$logit_Q1))
# colnames(nschAnalysis)
```

Here, we see that the sensitivity of this model is incredibly low. The
accuracy is quite high and the specificity is quite high as well. We
will compare this to our random forest.

Now, we build our random forest, including all potential covariates
except for a few that were coded repetiviely, as well as ID type
variables. We build a confusion matrix and visualize this matrix. The
function also presents the sensitivity, accuracy, specificity of our
model. We also plot the top 20 important features based on Gini index.

``` r
##RANDOM FOREST 
model_rf <- randomForest(logit_Q1 ~ . -SC_AGE_YEARS - HHID - FWC - STRATUM - FIPSST, data = training)
pred <- predict(model_rf, testing)
predicted <- as.data.frame(cbind(class_preds=as.factor(pred), Class = as.factor(testing$logit_Q1)))
predicted$class_preds <- as.factor(predicted$class_preds)
predicted$Class <- as.factor(predicted$Class)
ConfusionTableR::binary_visualiseR(train_labels = predicted$class_preds,
                                   truth_labels= predicted$Class,
                                   class_label1 = "Not utilized", 
                                   class_label2 = "Utilized",
                                   quadrant_col1 = "#28ACB4", 
                                   quadrant_col2 = "#4397D2", 
                                   custom_title = "Dental Care Utilization Confusion Matrix", 
                                   text_col= "black")
```

![](12-13project_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

``` r
feat_imp_df <- importance(model_rf) %>% 
    data.frame() %>% 
    mutate(feature = row.names(.)) %>% filter(rank(desc(MeanDecreaseGini))<=20)

  # plot dataframe
  ggplot(feat_imp_df, aes(x = reorder(feature, MeanDecreaseGini), 
                         y = MeanDecreaseGini)) +
    geom_bar(stat='identity') +
    coord_flip() +
    theme_classic() +
    labs(
      x     = "Feature",
      y     = "Importance",
      title = "Feature Importance: <Model>"
    )
```

![](12-13project_files/figure-gfm/unnamed-chunk-11-2.png)<!-- -->

Here we see that the sensitivity of this model has improved slightly but
it is still quite low. The accuracy has gone up to 88% which is quite
high. We will discuss this further in the conclusion section.

Now, we take a small sample of our data and visualize a random forest
using the 20 most important features jsut to verify that our algorithm
is working sensibly. First, we use a function to draw a dendrogram. Then
we take a sample of 100 rows randomly and then plot the dendrogram.

``` r
set.seed(1212)

  to.dendrogram <- function(dfrep,rownum=1,height.increment=0.1){

  if(dfrep[rownum,'status'] == -1){
    rval <- list()

    attr(rval,"members") <- 1
    attr(rval,"height") <- 0.0
    attr(rval,"label") <- dfrep[rownum,'prediction']
    attr(rval,"leaf") <- TRUE

  }else{##note the change "to.dendrogram" and not "to.dendogram"
    left <- to.dendrogram(dfrep,dfrep[rownum,'left daughter'],height.increment)
    right <- to.dendrogram(dfrep,dfrep[rownum,'right daughter'],height.increment)
    rval <- list(left,right)

    attr(rval,"members") <- attr(left,"members") + attr(right,"members")
    attr(rval,"height") <- max(attr(left,"height"),attr(right,"height")) + height.increment
    attr(rval,"leaf") <- FALSE
    attr(rval,"edgetext") <- dfrep[rownum,'split var']
    #To add Split Point in Dendrogram
    #attr(rval,"edgetext") <- paste(dfrep[rownum,'split var'],"\n<",round(dfrep[rownum,'split point'], digits = 2),"=>", sep = " ")
  }

  class(rval) <- "dendrogram"

  return(rval)
  }
  
sample <- training[sample(nrow(training), 190), ]
model_rf2 <- randomForest(logit_Q1 ~ age + region + FPL_I2 + A1_GRADE + FORMTYPE + FPL_I5 + FPL_I6 + FPL_I3 + FPL_I4 + FPL_I6 + mental + fpl + physical + TOTAGE_0_5 + race + HHCOUNT + insurance + docroom + TENURE + hcability, data = sample)
tree <- getTree(model_rf2,1,labelVar=TRUE)
d <- to.dendrogram(tree)
plot(d,center=TRUE,leaflab='none',edgePar=list(t.cex=1,p.col=NA,p.lty=0))
```

![](12-13project_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

It appears that our random forest and overall data feeding mechanism is
successful.

## Conclusion

When we compare the accuracy for the logistic regression to that of the
random forest we see an accuracy of 0.8492 versus 0.8623. These are
quite similar and do not indicate a significantly better performance of
either classification algorithm. The sensitivity of both methods is
significantly higher than the specificity (which are quite low for
both). In this context, this is not great because we may be interested
in using this to identify children who are at high risk of
underutilizing dental care. Clinicians, third part care apps, insurance
companies may be able to reach out to individuals and encourage them to
seek care more preventatively. Neither model suggested that child’s
special health care needs status was one of the most important factors
in whether they utilized dental care in the past year. The two most
important factors based on both variable importance and gini information
were age and federal poverty level. As age increases, the odds of dental
care not being utilized decreases. According to this data, as fpl, which
is the household income as a percentile of the federal poverty level,
increases, the odds of dental care not being utilized also increase.
This is a surprising result, but since we did conduct a proper
regression analysis we should not be too concerned. Our models were
built and should be interpreted from the prediction perspective.
Overall, I would say this analysis was somewhat successful. I think the
incredibly low sensitivity would prevent any actual healthcare entity
from being able to benefit from this analysis. However, we do see that
even during the COVID-19 pandemic, there is pretty decent utilization of
dental care among adolescents represented by this sample. With more
time, I would likely look into machine learning methods particularly
geared towards rarer outcomes. I would also try to impute the missing
data from the original dataset since we lost about 200+ variables due to
missingness. Full access to the available features may provide better
sensitivity for this model.

## References

- Child and Adolescent Health Measurement Initiative. “The 2019-2020
  National Survey of Children’s Health (NSCH) Combined Dataset FAST
  FACTS.” Data Resource Center for Child and Adolescent Health supported
  by the U.S. Department of Health and Human Services, Health Resources
  and Services Administration (HRSA), Maternal and Child Health Bureau
  (MCHB), 2021.
  <https://www.childhealthdata.org/docs/default-source/nsch-docs/2019-2020-nsch-fast-facts-cahmi.pdf?sfvrsn=8fc75f17_2>.

- Centers for Disease Control and Prevention. “Oral Health Surveillance
  Report: Trends in Dental Caries and Sealants, Tooth Retention, and
  Edentulism, United States, 1999–2004 to 2011–2016.” Atlanta, GA:
  Centers for Disease Control and Prevention, US Dept of Health and
  Human Services, 2019.
  <https://www.cdc.gov/oralhealth/pdfs_and_other_files/Oral-Health-Surveillance-Report-2019-Web-h.pdf>.

- Devinsky, Orrin, Danielle Boyce, Miriam Robbins, and Mariel Pressler.
  “Dental Health in Persons with Disability.” Epilepsy & Behavior 110
  (September 2020): 107174.
  <https://doi.org/10.1016/j.yebeh.2020.107174>. Fulda, Kimberly G.,
  Katandria L. Johnson, Kristen Hahn, and Kristine Lykens. “Do Unmet
  Needs Differ Geographically for Children with Special Health Care
  Needs?” Maternal and Child Health Journal 17, no. 3 (April 2013):
  505–11. <https://doi.org/10.1007/s10995-012-1029-4>.

- Huebner, Colleen E., Donald L. Chi, Erin Masterson, and Peter Milgrom.
  “Preventive Dental Health Care Experiences of Preschool-Age Children
  with Special Health Care Needs: PREVENTIVE CARE AMONG CHILDREN WITH
  SPECIAL NEEDS.” Special Care in Dentistry 35, no. 2 (March 2015):
  68–77. <https://doi.org/10.1111/scd.12084>.

- Inclan, Meagan L., and Beau D. Meyer. “Pre‐doctoral Special Healthcare
  Needs Education: Lost in a Crowded Curriculum.” Journal of Dental
  Education 84, no. 9 (September 2020): 1011–15.
  <https://doi.org/10.1002/jdd.12134>.

- Kagihara, Lynette E., Colleen E. Huebner, Wendy E. Mouradian, Peter
  Milgrom, and Betsy A. Anderson. “Parents’ Perspectives on a Dental
  Home for Children with Special Health Care Needs.” Special Care in
  Dentistry 31, no. 5 (September 2011): 170–77.
  <https://doi.org/10.1111/j.1754-4505.2011.00204.x>.

- Lebrun-Harris, Lydie A., María Teresa Canto, Pamella Vodicka, Marie Y.
  Mann, and Sara B. Kinsman. “Oral Health Among Children and Youth With
  Special Health Care Needs.” Pediatrics 148, no. 2 (August 2021):
  e2020025700. <https://doi.org/10.1542/peds.2020-025700>.

- Williams, Elizabeth, and MaryBeth Musumeci. “Children with Special
  Health Care Needs: Coverage, Affordability, and HCBS Access.” KFF,
  n.d.
  <https://www.kff.org/medicaid/issue-brief/children-with-special-health-care-needs-coverage-affordability-and-hcbs-access/>.

# Appendix

``` r
knitr::opts_chunk$set(echo = TRUE)
library(haven)
library(knitr)
library(kableExtra)
library(dplyr)
library(car)
library(jtools)
library(ggplot2)
library(ggpubr)
library(caret)
library(randomForest)
library(gbm)
library(e1071)
library(gtsummary)
library(glmnet)
library(vip)
library(pander)
library(ConfusionTableR)
library(table1)
nsch2020 <- read_sas("~/Downloads/nsch_2020_topical_SAS/nsch_2020_topical.sas7bdat",NULL)
x_2020 <- nsch2020[, which(colMeans(is.na(nsch2020)) == 0)]
x_2020$logit_Q1 <- ifelse(nsch2020$K4Q30_R == 1 | nsch2020$K4Q30_R == 2, 1, 0)
x_2020$SC_CSHCN <- ifelse(nsch2020$SC_CSHCN==1, 1, 0)
x_2020$age <- nsch2020$SC_AGE_YEARS
x_2020$fpl <- nsch2020$FPL_I1
x_2020$sex <- ifelse(nsch2020$SC_SEX == 1, 1, 0)

x_2020$insurance <- ifelse(nsch2020$CURRCOV == 2, 4, 
                 ifelse(nsch2020$K12Q12 == 1, 1, 
                 ifelse(nsch2020$K12Q03 == 1 | nsch2020$K12Q04 == 1, 2, 
                 ifelse(nsch2020$K11Q03R == 1 | nsch2020$HCCOVOTH == 1 | nsch2020$TRICARE == 1 | nsch2020$CURRCOV == 1, 3, 0))))

x_2020$race <- ifelse(!nsch2020$SC_HISPANIC_R == 1, nsch2020$SC_RACE_R, 6)
x_2020$married <- ifelse(nsch2020$A1_MARITAL == 1 |nsch2020$A1_MARITAL == 2, 1, ifelse(nsch2020$A1_MARITAL == 3 |nsch2020$A1_MARITAL == 4, 2, 3))

x_2020$regionori <- nsch2020$FIPSST

x_2020$region <- ifelse(x_2020$regionori %in% c(9, 23, 25, 33, 44, 50, 34, 36, 42), 1, ifelse(x_2020$regionori %in% c(18, 17, 26, 39, 55, 19, 31, 20, 38, 27, 46, 29), 2, ifelse(x_2020$regionori %in% c(10, 11, 12, 13, 24, 37, 45, 51, 54, 1, 21, 28, 47, 5, 22, 40, 48), 3, ifelse(x_2020$regionori %in% c(4, 8, 16, 35, 30, 49, 32, 56, 2, 6, 15, 41, 53), 4, 0))))

x_2020$regionori <- NULL
x_2020$region_ori <- NULL
x_2020$nsch2020.SC_CSHCN <- NULL

x_2020$HHID <- nsch2020$HHID
x_2020$STRATUM <- nsch2020$STRATUM
x_2020$FIPSST <- nsch2020$FIPSST
x_2020$FWC <- nsch2020$FWC
x_2020$lang <- nsch2020$HHLANGUAGE

###additional variables for specifiying SHCN 
x_2020$autism <- nsch2020$K2Q35A
x_2020$downsyn <- nsch2020$K2Q35A
x_2020$add <- nsch2020$K2Q31A

x_2020$employ <- ifelse(nsch2020$A1_EMPLOYED == 1 | nsch2020$A1_EMPLOYED == 2, 1, 0)
x_2020$married <- ifelse(nsch2020$A1_MARITAL == 1 |nsch2020$A1_MARITAL == 2, 1, ifelse(nsch2020$A1_MARITAL == 3 |nsch2020$A1_MARITAL == 4, 2, 3))
x_2020$mental <- nsch2020$A1_MENTHEALTH
x_2020$physical <- nsch2020$A1_PHYSHEALTH
x_2020$edu <- nsch2020$HIGRADE

x_2020$toothaches <- nsch2020$TOOTHACHES
x_2020$cavities <- nsch2020$CAVITIES
x_2020$allergies <- nsch2020$ALLERGIES
x_2020$docroom <- nsch2020$DOCROOM
x_2020$hcability <- nsch2020$HCABILITY

x_2020 <- x_2020[x_2020$age > 1,]
Q1_2020 <- x_2020 %>% filter(!is.na(SC_CSHCN) ) %>%
  filter(!is.na(logit_Q1)) %>% 
  mutate(cshcn= ifelse(SC_CSHCN==1, "CSHCN" , "Non-CSHCN")) %>% 
  mutate(Q = ifelse(logit_Q1 ==1, "Yes", "No"))  
PP2 <- Q1_2020 %>% ggplot(aes(x = as.factor(cshcn), fill= as.factor(Q))) +
  geom_bar(position = position_stack(), aes(group = as.factor(Q))) +
  geom_text(stat = "count", aes(label = after_stat(count)), position = position_stack(vjust = 0.5), size = 2.5) +
  xlab("2020 During Pandemic")+ 
  ggtitle("Dental Care Utilization in the Past Year")+
  ylab("") 

PP2

PP1 <- Q1_2020 %>% filter(logit_Q1 == 1) %>% ggplot(aes(x=race, color = "red")) +
  geom_bar() + 
  geom_text(stat = "count", aes(label = after_stat(count)), position = position_stack(vjust = 0.5), size = 2.5) +
  xlab("Ethnicity Response")+ 
  ggtitle("Yes Dental Care Utilization in the Past Year by Race")+
  ylab("") 
P1<- PP1 + guides(fill=guide_legend(title="Answer"))

PP2 <- Q1_2020 %>% filter(logit_Q1 == 0) %>% ggplot(aes(x=race, color = "lightblue")) +
  geom_bar() + 
  geom_text(stat = "count", aes(label = after_stat(count)), position = position_stack(vjust = 0.5), size = 2.5) +
  xlab("Ethnicity Response")+ 
  ggtitle("No Dental Care Utilization in the Past Year by Race")+
  ylab("") 
P2<- PP2 + guides(fill=guide_legend(title="Answer"))

arrange1<- ggarrange(PP1, PP2,  common.legend = TRUE, legend = "bottom") 
annotate_figure(arrange1,
               top = text_grob("Dental Care Utilization in the Past Year", color = "orange", size = 13),
               left = text_grob("Count", rot = 90)) 

set.seed(1212)
nschAnalysis <- x_2020
nschAnalysis$logit_Q1 <- as.factor(nschAnalysis$logit_Q1)
nschAnalysis$sex <- as.factor(nschAnalysis$sex)
nschAnalysis$insurance <- as.factor(nschAnalysis$insurance)
nschAnalysis$race <- as.factor(nschAnalysis$race)
nschAnalysis$married <- as.factor(nschAnalysis$married)
nschAnalysis$region <- as.factor(nschAnalysis$region)
nschAnalysis$lang <- as.factor(nschAnalysis$lang)

nschAnalysis$edu <- as.factor(nschAnalysis$edu)
nschAnalysis$mental <- as.factor(nschAnalysis$mental)
nschAnalysis$physical <- as.factor(nschAnalysis$physical)
nschAnalysis$employ<-as.factor(nschAnalysis$employ)
nschAnalysis$lang <- as.factor(nschAnalysis$lang)

nschAnalysis <- nschAnalysis[complete.cases(nschAnalysis),]
dim(nschAnalysis)
table1(~ SC_CSHCN + age + fpl + sex + insurance + race + married + region + lang + employ + edu  | logit_Q1, data=nschAnalysis)
intrain <- caret::createDataPartition(y = nschAnalysis$logit_Q1, p= 0.6, list = FALSE)
training <- nschAnalysis[intrain,]
testing <- nschAnalysis[-intrain,]
##LOGISTIC REGRESSION
x <- model.matrix(logit_Q1~ SC_CSHCN + age + fpl + sex + insurance + race + married + region + lang + employ + edu, training)
# Convert the outcome (class) to a numerical variable
y <- ifelse(training$logit_Q1 == "1", 1, 0)
cv.lasso <- cv.glmnet(x, y, alpha = 1, family = "binomial")
plot(cv.lasso)

# Fit the final model on the training data
model <- glmnet(x, y, alpha = 1, family = "binomial", lambda = cv.lasso$lambda.min)
# Display regression coefficients
c <- data.frame(coef.name = dimnames(coef(model))[[1]], coef.value = matrix(coef(model)))
c[-2,]%>% kable()
vip(model, num_features=10, geom="point")

# Make predictions on the test data
x.test <- model.matrix(logit_Q1 ~ SC_CSHCN + age + fpl + sex + insurance + race + married + region + lang + employ + edu, testing)
probabilities <- model %>% predict(newx = x.test)
predicted.classes <- as.factor(ifelse(probabilities > 0.5, 1, 0))
predicted <- as.data.frame(cbind(class_preds=as.factor(predicted.classes), Class = as.factor(testing$logit_Q1)))
predicted$class_preds <- as.factor(predicted$class_preds)
predicted$Class <- as.factor(predicted$Class)
ConfusionTableR::binary_visualiseR(train_labels = predicted$class_preds,
                                   truth_labels= predicted$Class,
                                   class_label1 = "Not utilized", 
                                   class_label2 = "Utilized",
                                   quadrant_col1 = "#28ACB4", 
                                   quadrant_col2 = "#4397D2", 
                                   custom_title = "Dental Care Utilization Confusion Matrix", 
                                   text_col= "black")


# prop.table(table(nschAnalysis$logit_Q1))
# colnames(nschAnalysis)
##RANDOM FOREST 
model_rf <- randomForest(logit_Q1 ~ . -SC_AGE_YEARS - HHID - FWC - STRATUM - FIPSST, data = training)
pred <- predict(model_rf, testing)
predicted <- as.data.frame(cbind(class_preds=as.factor(pred), Class = as.factor(testing$logit_Q1)))
predicted$class_preds <- as.factor(predicted$class_preds)
predicted$Class <- as.factor(predicted$Class)
ConfusionTableR::binary_visualiseR(train_labels = predicted$class_preds,
                                   truth_labels= predicted$Class,
                                   class_label1 = "Not utilized", 
                                   class_label2 = "Utilized",
                                   quadrant_col1 = "#28ACB4", 
                                   quadrant_col2 = "#4397D2", 
                                   custom_title = "Dental Care Utilization Confusion Matrix", 
                                   text_col= "black")

feat_imp_df <- importance(model_rf) %>% 
    data.frame() %>% 
    mutate(feature = row.names(.)) %>% filter(rank(desc(MeanDecreaseGini))<=20)

  # plot dataframe
  ggplot(feat_imp_df, aes(x = reorder(feature, MeanDecreaseGini), 
                         y = MeanDecreaseGini)) +
    geom_bar(stat='identity') +
    coord_flip() +
    theme_classic() +
    labs(
      x     = "Feature",
      y     = "Importance",
      title = "Feature Importance: <Model>"
    )
set.seed(1212)

  to.dendrogram <- function(dfrep,rownum=1,height.increment=0.1){

  if(dfrep[rownum,'status'] == -1){
    rval <- list()

    attr(rval,"members") <- 1
    attr(rval,"height") <- 0.0
    attr(rval,"label") <- dfrep[rownum,'prediction']
    attr(rval,"leaf") <- TRUE

  }else{##note the change "to.dendrogram" and not "to.dendogram"
    left <- to.dendrogram(dfrep,dfrep[rownum,'left daughter'],height.increment)
    right <- to.dendrogram(dfrep,dfrep[rownum,'right daughter'],height.increment)
    rval <- list(left,right)

    attr(rval,"members") <- attr(left,"members") + attr(right,"members")
    attr(rval,"height") <- max(attr(left,"height"),attr(right,"height")) + height.increment
    attr(rval,"leaf") <- FALSE
    attr(rval,"edgetext") <- dfrep[rownum,'split var']
    #To add Split Point in Dendrogram
    #attr(rval,"edgetext") <- paste(dfrep[rownum,'split var'],"\n<",round(dfrep[rownum,'split point'], digits = 2),"=>", sep = " ")
  }

  class(rval) <- "dendrogram"

  return(rval)
  }
  
sample <- training[sample(nrow(training), 190), ]
model_rf2 <- randomForest(logit_Q1 ~ age + region + FPL_I2 + A1_GRADE + FORMTYPE + FPL_I5 + FPL_I6 + FPL_I3 + FPL_I4 + FPL_I6 + mental + fpl + physical + TOTAGE_0_5 + race + HHCOUNT + insurance + docroom + TENURE + hcability, data = sample)
tree <- getTree(model_rf2,1,labelVar=TRUE)
d <- to.dendrogram(tree)
plot(d,center=TRUE,leaflab='none',edgePar=list(t.cex=1,p.col=NA,p.lty=0))
```
