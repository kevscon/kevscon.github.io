---
layout: post
title: Forecasting Independent Film Performance
---

![]({{ "/assets/images/ind_film/film.jpg" | absolute_url }})

The past five years (2013-2017) the winners for Best Picture at the Academy Awards were all independent films. The cultural significance of these movies will live on long after we've forgotten the intricate plot of the latest Avengers flick. While the incentive for creating independent films may not be to have top-grossing blockbusters at the box office, sufficient revenue is needed to motivate producers to finance them. One method filmmakers could use to effectively connect with their intended audiences is to determine what patterns exist in successful independent movies.

## Summary
Based on trends in recent years, the financial performance of independent films was predicted using a linear regression model. After model development, specific features leading to good or poor performance were identified.

### Data
Film characteristics were scraped from movie-based websites. Movie feature data was scraped from Box Office Mojo and financial data was scraped from the Numbers. Scrapy scripts were employed for these tasks. For the purposes of this project, independent movies were defined as all movies NOT produced by the Big 6 distributors. Movie data was restricted to:
- Movies NOT distributed by Buena Vista, Warner Bros., Columbia, Fox, Universal or Paramount
- Movies released between 2008 and 2017

Features input into regression model:
- Numerical:
  - budget
  - runtime
  - number of days in theaters
- Categorical:
  - genre
  - MPAA rating
  - release month

The target variable was the combination of international gross revenue combined with disc sales.

### Modeling
The distribution of both target and most features was not normal. Both were transformed to log scale.

The initial model yielded a very poor r-squared value. Both train and test r-squared values were low, indicating an underfit model.

In order to increase effectiveness, the model's complexity was increased using polynomial features. This created an overfit model with a higher train score but very low test score. Ridge regularization was used to reduce overfitting.

### Results
Model performance was not spectacular. The model achieved an r-squared of 0.3. Example revenue predictions are provided for the past five Best Picture winners.

![]({{ "/assets/images/ind_film/mod_pred.png" | absolute_url }})

### Conclusions
While model prediction accuracy was low, valuable insights were derived from the feature weights of the linear regression model. The values of the model coefficients were retrieved and the top 10 are shown below.

![]({{ "/assets/images/ind_film/lr_feat.png" | absolute_url }})

The feature weights displayed above indicate which factors will favorably or negatively impact the revenue of independent films. Observations:  
- The most significant factors include budget, number of days in the theater and number of theaters that are showing the movie. These factors are intuitive. A larger budget can be used to finance a more appealing film. A film being shown for a longer amount of time and in more locations has much more opportunity to collect revenue.
    - These factors interact with themselves and other features.
    - Interestingly for unrated films, the number of days is correlated with negative performance.
- Movies in the romance genre with a PG-13 ratings and a November release perform well.
- PG-13 movies tend to do well when combined with more days in theaters.
- There is a negative correlation with performance for horror films released in July.

Looking at the top grossing independent movies for the dataset, we see these patterns. The top five movies are from the Twighlight and Hunger Games series. The Twighlight series are PG-13 romance films released in November with above average budgets and theater release. The Hunger Games series are PG-13 films with wide release and hefty budgets.
