---
layout: post
title: Battleground Counties in the 2016 Election
---

![]({{ "/assets/images/batt_ground/county_map.png" | absolute_url }})

Our nation is divided. There is a bitter split among voters between our two major political parties. As time marches forward, it seems the rift between these two sides only deepens throughout the country. An election's outcome may come down to a set of particular voters in a handful of counties. Can characteristics of these voters be identified? Practically, this information could be used to shed light on which voters political campaigns should target to tip the balance in a tight election. More generally, insight may be gained into the attributes of the ever-shrinking segment of our population that is not politically entrenched.

## Summary
In this analysis, the results of the 2016 election were examined at the county level. County demographic data was used to determine if any trends were apparent for the most undecided counties.

### Data
A total of 3,142 counties were examined for the analysis and demographic data was retrieved from the American Community Survey. For the purposes of this analysis, a "swing" county was defined as a county with an election result with a 5% or lower margin of victory. For example, a county with 52% of the vote going to Donald Trump (R) and 47% going to Hilary Clinton (D) would be considered a swing county. Thirteen features were investigated for correlation with a swing county. Election results were based on data from the New York Times.

### Modeling
Supervised machine learning methods were employed using binary classification. Counties were classified as either "swing" (positive class) or "safe" (negative class). One of the major challenges of this project was a significant class imbalance - the vast majority (95%) of counties fell into the "safe" class. Due to this imbalance, an overall accuracy of 95% could be achieved by a model that simply classified all counties as "safe"! Obviously to tell us anything useful, this analysis requires a model based on other metrics such as precision or recall.

![]({{ "/assets/images/batt_ground/cls_rep.png" | absolute_url }})

The goal of the analysis was to gather insight into the characteristics of "swing" counties. This lead to using two approaches for modeling:
1. Recall was prioritized over precision. This yielded fewer false negatives but resulted in the trade-off of increased false positives.
2. Logistic Regression and Random Forest models were selected since they both provide insight into feature impacts on outcomes.

Initial models resulted in 0% recall as a direct result of the severe class imbalance! Fortunately, methods are available to boost recall performance:
1. Balanced weights can be used as an input parameter when initializing each model. This method modifies the "weights" of the two classes in the model algorithm's cost function in order to place greater importance on the "swing" class. Balanced weights are inversely proportional to the frequency of class observations.
2. Synthetic Minority Over-sampling Technique (SMOTE) is an oversampling method that generates synthetic samples of the "swing" class. The oversampling is applied to the data before initializing each model.  

All Logistic Regression and Random Forest models were tuned using input parameters that lead to optimal model performance.

### Results

The bar plot below shows the recall performance for the three different Logistic Regression and Random Forest models. Note the recall values for the initial models weren't mistakenly left out of the plot - they were both zero!

![]({{ "/assets/images/batt_ground/recall.png" | absolute_url }})

The best Logistic Regression model used balanced weights and the best Random Forest model utilized SMOTE. These two models were compared with each other to determine the final model to be used for the analysis. A typical metric of classification is the ROC Curve. However in cases of severe imbalance, the Precision-Recall Curve is a better metric of model performance. A tug-of-war between precision (rate based on false positives) and recall (rate based on false negatives) can be observed in the plot below. Even with optimized models, precision is low even with low recall but generally continues to decline as more true positives are identified by the model. In other words, as more "swing" counties are correctly identified by each model, the number of "safe" counties incorrectly identified as "swing" increases. The model is less selective for determining the positive case.

![]({{ "/assets/images/batt_ground/pre_rec.png" | absolute_url }})

Random Forest outperformed Logistic Regression in terms of Precision-Recall trade-off. However, overall recall was significantly better with the Logistic Regression model as shown below and this model was selected for final analysis.

![]({{ "/assets/images/batt_ground/brecall.png" | absolute_url }})

The heat map below shows the confusion matrix of the final model. 78% of the "swing" counties were correctly identified using this model. Quite an improvement from the initial 0%.

![]({{ "/assets/images/batt_ground/conf_mat.png" | absolute_url }})

By default, binary classification models predict an observation to be in the positive class if the algorithm outputs a probability greater than 50% that the observation is in the positive class. If desired, this decision threshold can be lowered to increase positive class prediction. This will further increase recall and decrease precision performance. In some cases, the trade-off may be worth it. Perhaps a well-funded political campaign is in the final few weeks of an extremely tight race and is willing to commit additional resources. It may be worth knocking on doors in every county that could possibly be swayed. With this in mind, the final Logistic Regression model can be modified by lowering the decision threshold to 39% and all "swing" counties in the 2016 election are correctly identified. Below, is a heat map showing the confusion matrix of the model with the modified threshold.

![]({{ "/assets/images/batt_ground/thr_conf_mat.png" | absolute_url }})

### Conclusions

With the final model selected, features leading to a "swing" county prediction can be analyzed. The bar graph below shows the positive and negative weights which contribute to identification as a "swing" county. Note that, after tuning, only four of the thirteen features impacted the model. As indicated below, "swing" counties tend to have residents with a higher college degree rate, a lower white population, slightly lower income and slightly lower median age. In general, these demographics trend toward Democratic voters. Perhaps this indicates that in the 2016 election, Donald Trump had an advantage since counties with these particular demographics were in play.

![]({{ "/assets/images/batt_ground/lr_feat.png" | absolute_url }})
