---
layout: post
title: Predicting Bridge Performance with NBI Data
---

![]({{ "/assets/images/bridge_ratings/bridge.jpg" | absolute_url }})
*Source: hdqwalls.com*

Much of the transportation infrastructure in the United States is now exceeding its intended design life. At the same time, the budgets of many state infrastructure agencies are under constant strain. It is imperative that limited resources target the most critical assets. One method to achieve this aim is to develop a quantitative model to predict which bridges will require intervention in the near future.

Using publicly available data gleaned from the National Bridge Inventory (NBI), a classification model was developed to predict bridges that will fall into poor condition over a fixed amount of time. This analysis also explored which specific bridge characteristics lead to better or worse performance during that time period.


## Data

The NBI contains data for hundreds of thousands of bridges but this analysis deals with a specific subset of that data. For the purposes of creating the model, data was filtered based on the following:

  - Time: A 10-year window was examined. Bridge characteristics (NBI items) were retrieved from the year 2007 and each bridge's evaluation metric (Sufficiency Rating) was retrieved from 2017.
  - Location: As different states may have different bridge maintenance practices, bridges were restricted to a single state - Virginia.
  - Structures: Bridges only - culverts were excluded.
  - Initial Condition: The focus of this analysis was to create a prediction algorithm which can identify structures likely to fall into poor condition during the time window. So structures that were in good condition as well as those already in poor condition in the year 2007 were excluded. The initial condition was based on the 2007 Sufficiency Rating.
  - Limited Intervention: This analysis focused on trends of bridges that did not receive significant improvements during the time period considered. Reconstructed bridges and bridges with ratings that increased from the 2007 baseline year were excluded.

These restrictions resulted in a dataset of 1,639 bridges. For each structure, a total of 52 items were gathered from the 2007 NBI database and used as input for model prediction.


## Modeling

The 2017 Sufficiency Rating was used as the bridge performance metric. Federal Highway Administration (FHWA) considers bridges with a Sufficiency Rating less than 50 to be in poor condition. This was the threshold used for classification. The model predicts whether a given structure will fall into one of two classes - "poor" or "not poor" - during a period of ten years.

Supervised machine learning methods were employed using binary classification. Why bother with the effort of creating a model? It certainly would be much simpler to compare the average values of a handful of NBI items for each of the two classes. While this example may provide some general insight, a machine learning model will provide a much more thorough analysis. Three major advantages of this approach:

  - There is no need to manually select input items based on qualitative intuition.
  - Models are able to identify "hidden" patterns and express the cumulative affect of all variables on the final prediction output.
  - The model methodology provides confidence that the analysis is generalizable and can be applied to observations outside of the provided dataset.


### Categorical Data
Half of the NBI items used for analysis were categorical variables (as opposed to numerical). Each of these items consisted of a code representing one of several particular conditions. For use in modeling, these variables were encoded so that a "1" was input if a condition applied to a bridge observation and a "0" was input otherwise.

For example, a bridge's median is coded under NBI Item 33 as follows:

**NBI Item 33 - Bridge Median**

| NBI Code | Description |
| --- | --- |
| 0 | no median |
| 1 | open median |
| 2 | closed median without barrier |
| 3 | closed median with barrier |

For hypothetical Bridge 01 with an open median barrier and Bridge 02 with a closed median barrier, the encoded values would be:

**Encoded Bridge Median Feature**

| Structure | Code 0 | Code 1 | Code 2 | Code 3 |
| --- | --- | --- | --- | --- |
| Bridge 01 | 0 | 1 | 0 | 0 |
| Bridge 02 | 0 | 0 | 0 | 1 |

This process lead to 26 categorical items being transformed into 165 variables for use in the model. Combined with the numerical items, a total of 191 variables were used for model input.

### Train/Test Split
The model was developed using 80% of the 1,639 observations (train data) - with the other 20% set aside for evaluation of the model's performance (test data).

### Logistic Regression Model
There are several classification models in the machine learning world. For this analysis, logistic regression was selected. The major advantage of using this type of model is its interpret-ability. It provides quantitative insight into how each variable affects the model output - in terms of both magnitude and direction. A detailed overview of logistic regression can be found [here](https://towardsdatascience.com/logistic-regression-detailed-overview-46c4da4303bc).

### Class Imbalance
One challenge with the data for this analysis was class imbalance - 77% of the bridges fell into the "not poor" class. This tends to cause a model to over-predict "not poor". To account for this, the "weights" of the two classes in the model's algorithm were modified to place greater importance on the "poor" class.


## Results

Once the model's algorithm has been developed based on the train data, the test data can be used to evaluate model performance. NBI data for each bridge in the test set is input and the model outputs class predictions. Each prediction is whether or not the bridge will be in the "poor" class after a period of 10 years.

The model's overall accuracy was 81%. However, a more useful evaluation metric for a classification model is the confusion matrix. The confusion matrix provides a more complete picture of a model's performance. Based on a model's results, it consists of True Negatives, True Positives, False Positives and False Negatives. The heat map template below contains a description for each location in the matrix. A high-performing model will be shown to have darker shaded upper left and lower right squares while having lighter shaded upper right and lower left squares.

![]({{ "/assets/images/bridge_ratings/ex_cm.png" | absolute_url }})
*Descriptions of each of the four cases in the confusion matrix. Each box shows the class for the model prediction and what the true class actually is.*

The heat map below displays the confusion matrix for model predictions on the test data.

![]({{ "/assets/images/bridge_ratings/conf_mat.png" | absolute_url }})
*Rates based on test results for each of the four cases in the confusion matrix.*

The True Positive rate is important because it indicates specifically how often the "poor" class is correctly identified. Due to the class imbalance previously mentioned, relatively high overall accuracy could be achieved by simply predicting a class of "not poor" 100% of the time. A True Positive rate of 85%, as shown in the heat map above, indicates that this is not the case with this model. If this value is not sufficiently high, interpretation of the model would be unreliable.

### Model Interpretation

The top ten NBI items correlating with bridge performance are displayed in the bar chart below. Positive (green) values indicate items that tend to lead to better bridge performance over the ten year period. Negative (red) values indicate items that are associated with a bridge falling into the "poor" class. The bars are shown as a percentage of influence over the model algorithm.

![]({{ "/assets/images/bridge_ratings/feat_wgt.png" | absolute_url }})
*Top ten most influential NBI items on the classification model.*

Three of the top ten NBI items are pretty intuitive (and uninteresting) - a higher initial Sufficiency Rating, a Superstructure Condition Rating of 8 and a Superstructure Condition Rating of 7 all lead to better performance. More useful takeaways:

- Bridges with wooden/timber decks are associated with poorer performance. This item has an influence of 5% within the model prediction.
- Items that correlate with increased bridge performance are:
  - concrete superstructure
  - latex wearing surface
  - highway route underneath the structure
- Other items that correlate with decreased bridge performance are:
  - undetermined historical significance
  - route carried services minor urban artery
  - posted for load restrictions


Jupyter notebooks of my data processing and classification modeling code for this project can be found here:

[Predicting Bridge Performance with NBI Data Github repository](https://github.com/kevscon/bridge-ratings)
