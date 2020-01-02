---
layout: post
title: Forecasting Independent Film Performance
---

![]({{ "/assets/images/ind_film/film.jpg" | absolute_url }})

For the past five years (2013-2017), the winners for Best Picture at the Academy Awards have been independent films. The cultural significance of these movies will live on long after we've forgotten the intricate plot of the latest Avengers flick. While the incentive for creating independent films may not be to have top-grossing blockbusters at the box office, sufficient revenue is needed to motivate producers to finance and ultimately create them. Exploring patterns of successful independent movies may be one method to help filmmakers connect with their intended audience.

## Summary
Based on trends in recent years, the financial performance of independent films was predicted using a linear regression model. After model development, specific features leading to good or poor performance were identified.

## Data
Film characteristics were scraped from movie-based websites. Movie feature data was scraped from Box Office Mojo and financial data was scraped from the Numbers. Scrapy scripts were employed for these tasks. For the purposes of this project, independent movies were defined as all movies NOT produced by the Big 6 distributors. Movie data was restricted to:
- Movies NOT distributed by Buena Vista, Warner Bros., Columbia, Fox, Universal or Paramount
- Movies released between 2008 and 2017

Variables collected for building the regression model:
- Numerical:
  - budget
  - runtime
  - number of days in theaters
  - international gross ticket sales
  - disc sales
- Categorical:
  - genre
  - MPAA rating
  - release month

### Data Cleaning
Data processing prior to modeling included:
- reformatting date values from string to date data types and parsing into separate year and month values
- merging Box Office Mojo and the Numbers data based on film title and release year
- reformatting dollar and count values from string to numeric data types
- adjusting values for inflation for fair comparison between different years

The inflation adjustments were completed using historical CPI (Consumer Price Index) values and a custom inflation calculator function. Each year is assigned a CPI value and the difference between two values results in the inflation that occurred between those years. The CPI values were stored in a list called cpi_values. A CPI dictionary was created with year keys. The dictionary contained factors for converting each year into 2017 dollars:
```python
# initiate blank dictionary
cpi_2017 = {}

# iterate through years 2008 through 2017
for year in range(2008, 2018):
    # assign each year a 2017-based conversion factor
    cpi_2017[year] = cpi_values[2017] / cpi_values[year]
```
Then the following function was applied to all columns containing dollar values:
```python
# define function to apply inflation factors to dataframe
def infl_calc(value, date):
    '''
    Convert to 2017 dollars
    value : value in given date
    date : date for given value
    '''
    try:
        year = date.year
        # apply CPI factor for input value
        infl_val = int(cpi_2017[int(year)] * value)
        return(float(infl_val))
    except:
        return(np.nan)
```

Budget and disc sale data retrieved from the Numbers website had many fewer observations than Box Office Mojo so many of these values were missing from the merged data frame. All missing numeric values were replaced with the mean value for each particular feature.

The target variable was the combination of international gross revenue and disc sales.

## Modeling
The distribution of both target and most features was not normal. Since this is an assumption of linear regression, both were transformed to log scale.

The initial model yielded a very poor r-squared value. Both train and test r-squared values were low, indicating an underfit model. In order to increase effectiveness, the model's complexity was increased using polynomial features which accounted for interaction. This method increased the total number of features several-fold.  The result was an overfit model with an increased train score and a test score that was still very low. Ridge regularization was used to reduce overfitting.

## Results

### Model Predictions
Model prediction performance was not spectacular. The model achieved an r-squared of about 0.33 - meaning the model can explain 33% of the variation in a film's revenue. As an example of model output, revenue predictions are provided for the past five Best Picture winners.

![]({{ "/assets/images/ind_film/mod_pred.png" | absolute_url }})

What may be the culprit here? The data does not follow a normal distribution and this is the case even after converting it to log scale. Looking at the residual plot, there is a clear revenue threshold - where few films gross below and many films bunch around. This non-normal distribution of the residual plot indicates that the data does not follow the normal distribution that linear regression assumes. This is a good indicator that the model's effectiveness is limited.

![]({{ "/assets/images/ind_film/residuals.png" | absolute_url }})

So does this mean that our independent film model is a total waste? Not exactly - otherwise I would have nothing to blog about!

### Model Interpretation
While model prediction accuracy left much to be desired, valuable insights were derived from the feature weights of the linear regression model. Sklearn (the python library used for modeling) is able to output model coefficients in the order they were input into the model. But it's much more useful to have the coefficients sorted by absolute value (largest to smallest) with an indication of whether the impact is positive or negative. The following function was developed to streamline this process. It outputs a sorted data frame with two columns - one is the absolute value of feature weights and the other indicates if it's a positive value. The output can also be limited to a specified number of top values.

```python
def feat_sort(values, labels, ret_num='all'):
    '''
    Return dataframe of sorted (by absolute value) feature weights
    values : feature weight values from analysis
    labels : names of features
    ret_num : number of top features to return, returns all by default
    '''

    df = pd.DataFrame(values, index=labels, columns=['feat_wgt'])
    # drop weights = 0
    df = df[df['feat_wgt'] != 0]
    # note which weights are positive
    df['positive'] = df['feat_wgt'] > 0
    # take absolute value of weights
    df['feat_wgt'] = df['feat_wgt'].apply(abs)
    # sort weights (largest to smallest)
    df.sort_values(by='feat_wgt', ascending=False, inplace=True)
    if ret_num == 'all':
        return(df)
    else:
        return(df.iloc[:ret_num, :])
```

Due to the use of polynomial features, the handful of initial features exploded to over 3000! For practical reporting purposes, the top 10 model coefficients and their respective feature labels are shown below.

![]({{ "/assets/images/ind_film/lr_feat.png" | absolute_url }})

The feature weights displayed above indicate which factors will favorably or negatively impact the revenue of independent films. Observations:  
- The most significant factors include budget, number of days in the theater and number of theaters that are showing the movie. These factors are intuitive. A larger budget can be used to finance a more appealing film. A film being shown for a longer amount of time and in more locations has much more opportunity to collect revenue.
    - These factors interact with themselves and other features.
    - Interestingly for unrated films, the number of days is correlated with negative performance.
- Movies in the romance genre with a PG-13 ratings and a November release perform well.
- PG-13 movies tend to do well when combined with more days in theaters.
- There is a negative correlation with performance for horror films released in July.

## Conclusions
While the model could not give a high level of accurate revenue prediction, it does provide valuable insight into factors which can enhance or diminish performance.

When inspecting the features of the top grossing independent movies in the dataset, the patterns indicated by the model can be clearly identified. The top five movies are from the Twilight and Hunger Games series. The Twilight series are PG-13 romance films released in November with above average budgets and days in theaters. The Hunger Games series are PG-13 films with a long run in theaters and hefty budgets.

<!-- movie posters -->
<div class="film_row">
  <div class="film_col">
    <img src="/assets/images/ind_film/twilight.jpg" alt="Twilight" style="width:100%">
  </div>
  <div class="film_col">
        <img src="/assets/images/ind_film/hunger_games.jpg" alt="Twilight" style="width:100%">
  </div>
</div>

Here's a table of the top film features and the average for the dataset:

| Title | Genre | MPAA Rating | Release Month | Budget ($) | No. Days |
| --- | --- | --- | --- | --- | --- |
| Indie Film Average | -- | -- | -- | 28,296,713 | 76 |
| The Twilight Saga: New Moon | Romance | PG-13 | November | 57,127,674 | 133 |
| The Hunger Games: Catching Fire | Action / Adventure | PG-13 | November | 136,787,475 | 133 |
| The Twilight Saga: Eclipse | Romance | PG-13 | June | 76,439,813 | 114 |
| The Hunger Games | Action / Adventure | PG-13 | March | 85,409,897 | 168 |
| The Twilight Saga: Breaking Dawn Part 2 | Romance | PG-13 | November | 28,296,713 | 112 |

Jupyter notebooks of my data processing and regression modeling for this project can be found here:

[Indie Film Github repository](https://github.com/kevscon/indie-films)
