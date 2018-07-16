---
layout: post
title: Battleground Counties in the 2016 Election
---

<img src="/assets/images/batt_ground/county_map.png">

Our nation is divided. We have a bitter split among voters between our two major political parties. As time marches forward, it seems the rift between these two sides only deepens throughout the country. But many election outcomes may come down to just a set of particular voters in a handful of counties. Can characteristics of these voters be identified? Obviously this information could be used to shed light on which voters political campaigns should target to tip the balance in a tight election. More generally, insight may be gained into the attributes of the ever-shrinking segment of our population that is not politically entrenched.

## Summary
This project analyzed the results of the 2016 election at the county level and compared them to demographics to determine if any trends were apparent for the most undecided counties.

### Data
A total of 3,142 counties were examined for the analysis and demographic data was retrieved from the American Community Survey. Thirteen features were investigated for correlation with a close 2016 presidential election between Donald Trump and Hilary Clinton. Election result data was based on data from the New York Times. For the purposes of this project, a close election was defined as an election result within 5% (e.g., 52%-47%).

### Method
Supervised machine learning methods were employed using binary classification. Counties were classified as either "close" (positive class) or "not-close" (negative class). One of the major challenges of this project was the significant class imbalance since the vast majority (95%) of counties were in the "not-close" class. A classification model that prioritized achieving a high level of accuracy could get 95% accuracy by simply classifying all counties as "not-close". This causes a tug-of-war between precision (rate based on false positives) and recall (rate based on false negatives). As seen in the Precision-Recall Curve below, precision values drop sharply as recall values increase for the five classification models initially selected.

<img src="/assets/images/batt_ground/prcurve.png">

The priority of this project is to gather insight into the characteristics of "close" counties. This lead to using two approaches for modeling:
1. Logistic Regression and Random Forest models were selected since they both allow retrieval of feature importances.
2. Recall was prioritized over precision. This prioritization would result in fewer false negatives but more false positives.

Logistic Regression and Random Forest models were then tuned using gridsearch. Several methods for boosting model recall values were employed. The recall values for the two models for the initial tuned models, an adjusted threshold, balanced weights and SMOTE are shown below.

<img src="/assets/images/batt_ground/recall_vals.png">

The best performer was the Logistic Regression model using balanced weights which adjusted weights inversely proportional to actual class frequencies. This increased the representation of the "close" election class for analysis.

### Results
Using the weighted Logistic Regression model, the threshold can be adjusted to acquire the desired level of recall/precision. If the default threshold is used, about 70% recall is achieved for true positives. If capturing all true positives is desired (100% recall), the threshold can be reduced to 20% (all counties with a probability above 20% of being "close" are classified as positive). This has the result of significantly increasing false positive rate. Three example thresholds for the model are shown below:

<img src="/assets/images/batt_ground/t_50.png">

<img src="/assets/images/batt_ground/t_30.png">

<img src="/assets/images/batt_ground/t_20.png">

Finally, the feature coefficients were compared to determine which demographic attributes had influence on whether a county tended to be "close". A plot of the coefficients is shown below.

<img src="/assets/images/batt_ground/feat.png">

The top three attributes are populations with college degrees, a higher number of households and a lower per capita income.

### Conclusions
This project demonstrates the ability of thresholding in classification to target desired recall. Depending on project need and resources different threshold levels can be useful. As applies to an election, a well-financed campaign may be willing to commit resources to a large number of counties if they are certain to contain all up-for-grabs areas. A more limited campaign may need to strike more of a balance between true and false positives.

The model identified high college rates, high number of households and lower per capita incomes as demographic characteristics of a "close" county. Intuitively, the higher college rate and lower income features seem paradoxical. One example of a county where this may be the case is a college town that resides in a location without much other economic activity.
