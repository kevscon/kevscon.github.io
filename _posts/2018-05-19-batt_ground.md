---
layout: post
title: Battleground Counties in the 2016 Election
---

![]({{ "/assets/images/batt_ground/county_map.png" | absolute_url }})

Our nation is divided. There is a bitter split among voters between our two major political parties. As time marches forward, it seems the rift between these two sides only deepens throughout the country. An election's outcome may come down to a set of particular voters in a handful of counties. Can characteristics of these voters be identified? Practically, this information could be used to shed light on which voters political campaigns should target to tip the balance in a tight election. More generally, insight may be gained into the attributes of the ever-shrinking segment of our population that is not politically entrenched.

## Summary
In this analysis, the results of the 2016 election were examined at the county level. Classification models were developed to determine what county demographic trends were indicative of the most undecided counties.

## Data
All voting counties in the US and the District of Columbia (a total of 3,142) were examined for this project. Demographic data was retrieved from the American Community Survey and election results were based on data from the New York Times. For the purposes of this analysis, a "swing" county was defined as a county with an election result with a 5% or lower margin of victory. For example, a county with 52% of the vote going to Donald Trump (R) and 47% going to Hilary Clinton (D) would be considered a swing county. Fourteen features were investigated for correlation with a swing county prediction.

As a visual overview, the fourteen features were decompressed into two for principle component analysis. There is no region in the plot which is exclusive to "swing" county observations. They tend to be somewhat clustered in the middle along with many "safe" county observations.

![]({{ "/assets/images/batt_ground/pca.png" | absolute_url }})

## Modeling
Supervised machine learning methods were employed using binary classification. Counties were classified as either "swing" (positive class) or "safe" (negative class). One of the major challenges of this project was a significant class imbalance - the vast majority (95%) of counties fell into the "safe" class. Due to this imbalance, an overall accuracy of 95% could be achieved by a model that simply classified all counties as "safe"! Obviously to tell us anything useful, this analysis requires a model based on other metrics such as precision or recall.

![]({{ "/assets/images/batt_ground/cls_rep.png" | absolute_url }})

The goal of the analysis was to gather insight into the characteristics of "swing" counties. This lead to two approaches for modeling:
1. Recall was prioritized over precision. This resulted in fewer false negatives but resulted in the trade-off of increased false positives.
2. Logistic Regression and Random Forest models were selected since they both provide insight into feature impacts on outcomes.

Initial models resulted in very low recall as a direct result of the severe class imbalance. Fortunately, methods were available to boost recall performance:
1. Balanced weights can be used as an input parameter when initializing each model. This method modifies the "weights" of the two classes in the model algorithm's cost function in order to place greater importance on the "swing" class. Balanced weights are inversely proportional to the frequency of class observations.
2. Synthetic Minority Over-sampling Technique (SMOTE) is an oversampling method that generates synthetic samples of the "swing" class. The oversampling is applied to the data before initializing each model.  

All Logistic Regression and Random Forest models were tuned using input parameters that lead to optimal model performance. The bar plot below shows the recall performance for the three different Logistic Regression and Random Forest models. Note the recall value for the initial Logistic Regression model wasn't mistakenly left out of the plot - it was zero!

![]({{ "/assets/images/batt_ground/recall.png" | absolute_url }})

The best Logistic Regression model used balanced weights and the best Random Forest model utilized SMOTE. These two models were compared with each other to determine the final model to be used for the analysis.

### Model Comparison
In the next few graphs, we've got a different "red vs blue" contest going on - the optimized Logistic Regression and Random Forest are compared for various metrics. The plot below shows the recall scores and Logistic Regression is far and away the winner here.

![]({{ "/assets/images/batt_ground/brecall.png" | absolute_url }})

As previously stated, the recall score is being prioritized in this project but other metrics were compared as well. Below is the ROC curve which compares the true positive rate to the false positive rate. Based on this plot, both models appear to have good performance since they trend toward the desirable upper left quadrant. However when comparing the area under the curve (AUC), Random Forest beats Logistic Regression.

![]({{ "/assets/images/batt_ground/roc.png" | absolute_url }})

The ROC curve and AUC value are typical classification metrics. However both axes of the plot (true positive rate and false positive rate) do not reflect class imbalances so this is not a suitable metric for this analysis.

A better metric is the Precision-Recall Curve. A tug-of-war between precision (rate based on false positives) and recall (rate based on false negatives) can be observed for both models in the plot below. Precision generally declines as more true positives are identified by each model and recall increases. As more "swing" counties are correctly identified by each model, the number of "safe" counties incorrectly identified as "swing" increases. In other words, the model becomes less selective for determining the positive case.

![]({{ "/assets/images/batt_ground/pre_rec.png" | absolute_url }})

Random Forest outperformed Logistic Regression in terms of Precision-Recall trade-off. However as overall recall was significantly better with the Logistic Regression model, it was selected for final analysis.

## Results
The heat map below shows the confusion matrix (as percentages) of the final Logistic Regression model. 72% of the "swing" counties were correctly identified using this model. Quite an improvement from the initial 0%.

![]({{ "/assets/images/batt_ground/conf_mat.png" | absolute_url }})

By default, binary classification models predict an observation to be in the positive class if the algorithm outputs a probability greater than 50% that the observation is in the positive class. If desired, this decision threshold can be lowered to increase positive class prediction. This will further increase recall and decrease precision performance. In some cases, the trade-off may be worth it. Perhaps a well-funded political campaign is in the final few weeks of an extremely tight race and is willing to commit additional resources. It may be worth knocking on doors in every county that could possibly be swayed. With this in mind, the final Logistic Regression model can be modified by lowering the decision threshold to 35% and all "swing" counties in the 2016 election are correctly identified. Below, is a heat map showing the confusion matrix of the model with the modified threshold.

![]({{ "/assets/images/batt_ground/thr_conf_mat.png" | absolute_url }})

Note the significant increase in false positives (upper right) with the lower threshold.

### Model Interpretation
With the final model selected, features correlating with a "swing" county prediction can be analyzed. The bar graph below shows the positive and negative weights which contribute to identification as a "swing" county. For this task, I took advantage of the same feature weight sorting function developed in my independent film linear regression project - [Forecasting Indie Film Performance](https://kevscon.github.io/2018/04/29/indie_films.html). "Swing" counties tend to have residents with a higher college degree rate, a lower white population and a slightly higher number of households.

![]({{ "/assets/images/batt_ground/log_feat.png" | absolute_url }})

Note that, after tuning, only three of the fourteen features impacted the model. The optimal Logistic Regression model utilized L1 regularization which has the ability to completely zero out features.

## Conclusions

 In general, the demographics identified in the model interpretation typically trend toward Democratic voters. Perhaps this analysis indicates that in the 2016 election, Donald Trump had an advantage since counties with Democratic-leaning demographics were up for grabs by either candidate.

 Below is a table of four swing counties of interest as well as the county average for the nation. These are swing counties in pivotal swing states that helped determine the outcome of the 2016 election. All four counties have above average college degree holding residents. Half of the counties have a below average white population and three of the four have an above average number of households.

![]({{ "/assets/images/batt_ground/swingers.png" | absolute_url }})

Isabella and Kent counties in Michigan are circled in the map below.

![]({{ "/assets/images/batt_ground/MI_map.png" | absolute_url }})
*Source: New York Times*

Kenosha and Racine counties in Wisconsin are circled in the map below.

![]({{ "/assets/images/batt_ground/WI_map.png" | absolute_url }})
*Source: New York Times*

As indicated by the pale red tint in the maps above, these four counties went for Trump but just barely. While the election was not close in terms of electoral votes, increased Democratic turnout in just a few counties similar to these could have tipped the balance in a handful of swing states and lead to a very different electoral outcome.

Jupyter notebooks of my data processing and classification modeling for this project can be found here:

[Battle Ground Counties Github repository](https://github.com/kevscon/battle-counties)
