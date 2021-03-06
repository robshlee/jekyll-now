---
title: "Predicting Kobe with Regularization"
layout: post
date: 2017-12-27 22:44
image: /assets/images/markdown.jpg
headerImage: false
tag:
- markdown
- elements
star: false
category: blog
author: roblee
description: Experimenting with different regularization penalties.
---

# Predicting Shots per Game by Kobe Bryant

![title](../assets/images/kobe_title.jpg)

In this post I'll be demonstrating regularization penalties through linear models that predict how many shots Kobe Bryant made per game in his career. <br/><br/>
Please note I don't follow the NBA and thus don't have domain knowledge for this dataset. As a disclaimer, I believe domain knowledge is a key element in any analysis. That being said it's always been interesting to analyze data without any biases from knowing about the topic. It offers fresh perspective and the ability to truly allow the data to tell a story rather than trying to shape the story based on your preconceived notions.


```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

from sklearn.linear_model import LinearRegression, RidgeCV, LassoCV, ElasticNetCV
from sklearn.model_selection import cross_val_score, train_test_split
from sklearn.preprocessing import StandardScaler

import warnings
warnings.filterwarnings("ignore")
%matplotlib inline
```


```python
kobe = pd.read_csv('./datasets/kobe_superwide_games.csv')
kobe.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>SHOTS_MADE</th>
      <th>AWAY_GAME</th>
      <th>SEASON_OPPONENT:atl:1996-97</th>
      <th>SEASON_OPPONENT:atl:1997-98</th>
      <th>SEASON_OPPONENT:atl:1999-00</th>
      <th>SEASON_OPPONENT:atl:2000-01</th>
      <th>SEASON_OPPONENT:atl:2001-02</th>
      <th>SEASON_OPPONENT:atl:2002-03</th>
      <th>SEASON_OPPONENT:atl:2003-04</th>
      <th>SEASON_OPPONENT:atl:2004-05</th>
      <th>...</th>
      <th>ACTION_TYPE:tip_layup_shot</th>
      <th>ACTION_TYPE:tip_shot</th>
      <th>ACTION_TYPE:turnaround_bank_shot</th>
      <th>ACTION_TYPE:turnaround_fadeaway_bank_jump_shot</th>
      <th>ACTION_TYPE:turnaround_fadeaway_shot</th>
      <th>ACTION_TYPE:turnaround_finger_roll_shot</th>
      <th>ACTION_TYPE:turnaround_hook_shot</th>
      <th>ACTION_TYPE:turnaround_jump_shot</th>
      <th>SEASON_GAME_NUMBER</th>
      <th>CAREER_GAME_NUMBER</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.0</td>
      <td>0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.0</td>
      <td>1</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2.0</td>
      <td>1</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>3</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2.0</td>
      <td>1</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.0</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>5</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 645 columns</p>
</div>




```python
kobe.shape
```




    (1558, 645)



Regularization can be useful for this dataset because with the high number of features, the model can have high complexity and may be prone to be overfit. By using regularization, we can reduce the model's variance inherent with complex models.

## Preprocessing


```python
# Set target vector and feature matrix
target = kobe['SHOTS_MADE']
nc = [x for x in kobe.columns if x != 'SHOTS_MADE']
X = kobe.loc[:, nc].values

# Train test split
X_train, X_test, y_train, y_test = train_test_split(X, target, test_size=0.25, \
                                                    random_state=1337)

# Use standard scaler
ss = StandardScaler()
X_train = ss.fit_transform(X_train)
X_test = ss.transform(X_test)
```

## Modeling


```python
# Test base linear regression model
lr = LinearRegression()
lr.fit(X_train, y_train)

# Cross-validation
xfold_mean = np.mean(cross_val_score(lr, X_train, y_train, cv=5))
xfold_std = np.std(cross_val_score(lr, X_train, y_train, cv=5))

print('crossfold mean: ', xfold_mean)
print('crossfold std: ', xfold_std)
```

    crossfold mean:  -4.40627525789e+28
    crossfold std:  1.1718160823e+28


And you thought you saw bad model performmance before. Let's try this again with L2 regularization. 


```python
# RidgeCV performs best searching alphas through logarithmic space. 
ridgeCV = RidgeCV(alphas=(np.logspace(-5, 5, 100)))
ridgeCV.fit(X_train, y_train)

# Best alpha
print('Best alpha: ', ridgeCV.alpha_)

# Cross-validation
xfold_mean_ridge = np.mean(cross_val_score(ridgeCV, X_train, y_train, cv=5))
xfold_std_ridge = np.std(cross_val_score(ridgeCV, X_train, y_train, cv=5))

print('Ridge crossfold mean: ', xfold_mean_ridge)
print('Ridge crossfold std: ', xfold_std_ridge)
print('Validation accuracy: ', ridgeCV.score(X_test, y_test))
```

    Best alpha:  1519.91108295
    Ridge crossfold mean:  0.619968376475
    Ridge crossfold std:  0.0281234152548
    Validation accuracy:  0.603733966375


By running a linear model with L2 regularlization, we're able to see a huge bump in accuracy. With a feature set in the hundreds- this was expected. The regularization is helping the model be less overfit by penalizing features with higher coefficients and dampening features with very small coefficients. <br/><br/>
Let's run a Lasso regression to see how that will perform against the Ridge regression.


```python
# LassoCV performs best searching for alpha through linear space. By default, LassoCV
# will decide itself what alphas to use.
lassoCV = LassoCV()
lassoCV.fit(X_train, y_train)

# Best alpha
print('Best alpha: ', lassoCV.alpha_)

xfold_mean_lasso = np.mean(cross_val_score(lassoCV, X_train, y_train, cv=5))
xfold_std_lasso = np.std(cross_val_score(lassoCV, X_train, y_train, cv=5))

print('LassoCV crossfold mean: ', xfold_mean_lasso)
print('LassoCV crossfold std: ', xfold_std_lasso)
print('Validation accuracy: ', lassoCV.score(X_test, y_test))
```

    Best alpha:  0.10792078438
    LassoCV crossfold mean:  0.639611906999
    LassoCV crossfold std:  0.0163110524353
    Validation accuracy:  0.62856668496


The lasso regression performed slightly better than our ridge model above. Because L1 regularization allows coefficients to 'zero-out' opposed from L2 regularlization which doesn't allow coefficients to hit zero, I suspect this helped lessen the overfitting that was prone with a model with this high of a complexity. <br/><br/>
Let's take a look at how many coefficients ended up zero-ing out from the L1 regularization.


```python
zero_coef_count = np.count_nonzero(lassoCV.coef_==0)

print('Number of "zero-ed out" feature coefficients: ', zero_coef_count)
print('Count of total features: ', len(lassoCV.coef_))
print('')
print('Percent of "zero-ed out" features: {}%'.format(round((zero_coef_count / \
                                                    len(lassoCV.coef_)) * 100, 2)))
```

    Number of "zero-ed out" feature coefficients:  587
    Count of total features:  644
    
    Percent of "zero-ed out" features: 91.15%


Over 90% of the features in this dataset had a zero coefficient after the L1 regularization. That shows that most of the features had very little to no impact to the target. <br/><br/>
So then what were the most important features for predicting Kobe's points per game?


```python
df = pd.DataFrame(kobe.loc[:, nc].columns, columns=['Features'])
df['Coefficients'] = lassoCV.coef_

df.sort_values('Coefficients', ascending=False).head(10)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Features</th>
      <th>Coefficients</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>579</th>
      <td>COMBINED_SHOT_TYPE:jump_shot</td>
      <td>1.249313</td>
    </tr>
    <tr>
      <th>574</th>
      <td>SHOT_TYPE:2pt_field_goal</td>
      <td>0.807399</td>
    </tr>
    <tr>
      <th>566</th>
      <td>SHOT_ZONE_BASIC:restricted_area</td>
      <td>0.366831</td>
    </tr>
    <tr>
      <th>577</th>
      <td>COMBINED_SHOT_TYPE:dunk</td>
      <td>0.335194</td>
    </tr>
    <tr>
      <th>582</th>
      <td>SECONDS_REMAINING</td>
      <td>0.308373</td>
    </tr>
    <tr>
      <th>569</th>
      <td>SHOT_ZONE_AREA:center(c)</td>
      <td>0.172471</td>
    </tr>
    <tr>
      <th>575</th>
      <td>SHOT_TYPE:3pt_field_goal</td>
      <td>0.101126</td>
    </tr>
    <tr>
      <th>333</th>
      <td>SEASON_OPPONENT:nyk:2008-09</td>
      <td>0.098314</td>
    </tr>
    <tr>
      <th>466</th>
      <td>SEASON_OPPONENT:sea:2004-05</td>
      <td>0.085991</td>
    </tr>
    <tr>
      <th>134</th>
      <td>SEASON_OPPONENT:det:2000-01</td>
      <td>0.049452</td>
    </tr>
  </tbody>
</table>
</div>



The above dataframe shows the top 10 features that showed the highest correlation with the points made per game for Kobe. The number of jump shots and 2pt field goals had the highest positive coefficients by a good margin compared to the rest of the features. <br/><br/>
A third regularization regression is the Elastic Net. This blends both the L1 and L2 regularizations and weighs either regularization through the parameter, 'l1_ratio' (0 = all ridge, 1 = all lasso. We'll run the elastic net model to see if we can get a boost in performance.


```python
# **FYI this will take a while to run
# Try several different values for 'l1_ratio'
n = np.linspace(0.01, 1, 20)

elasticCV = ElasticNetCV(l1_ratio=n)
elasticCV.fit(X_train, y_train)

# Best alpha
print('Best alpha: ', elasticCV.alpha_)

xfold_mean_en = np.mean(cross_val_score(elasticCV, X_train, y_train, cv=5))
xfold_std_en = np.mean(cross_val_score(elasticCV, X_train, y_train, cv=5))

print('Elastic Net crossfold mean: ', xfold_mean_en)
print('Elastic Net crossfold std: ', xfold_std_en)
print('Validation accuracy: ', elasticCV.score(X_test, y_test))
```

    Best alpha:  0.10792078438
    Elastic Net crossfold mean:  0.639611906999
    Elastic Net crossfold std:  0.639611906999
    Validation accuracy:  0.62856668496



```python
# Best l1 ratio
elasticCV.l1_ratio_
```




    1.0



Elastic Net found that an L1 ratio of 1 (all Lasso) was the optimum value for this exmaple.

## Conclusions
In regards to the modeling, the base linear regression was not a usable model for this dataset. Once we applied regularization penalties, we began to build models that had 'okay' accuracy scores. Comparing the L1 vs L2 penalties yielded the below validation accuracy from the test set.<br/>

Ridge: 0.6037<br/>
Lasso: 0.6286<br/><br/>

Lasso beat out Ridge by a small margin. In addition, when we ran Elastic Net the l1 ratio strongly favored Lasso. The L1 regularlization zero-ed out nearly 92% of the features in the dataset which left 57 features to be used for the model. 

We can visually see the performance of our ridge and lasso models through scatter plots.


```python
ridge_predictions = ridgeCV.predict(X_test)
lasso_predictions = lassoCV.predict(X_test)

plt.scatter(ridge_predictions, y_test)
plt.scatter(lasso_predictions, y_test)
plt.xlabel('Predicted')
plt.ylabel('Actual')
plt.legend(('Ridge', 'Lasso'), loc='lower right')
plt.title('Ridge and Lasso')
plt.show()

plt.scatter(ridge_predictions, y_test)
plt.xlabel('Predicted')
plt.ylabel('Actual')
plt.legend('Ridge', loc='lower right')
plt.title('Ridge')
plt.show()

plt.scatter(lasso_predictions, y_test, color='red')
plt.xlabel('Predicted')
plt.ylabel('Actual')
plt.legend('Lasso', loc='lower right')
plt.title('Lasso')
```


![title](../assets/images/2017-12-27-kobe_regularizatio_23_0.png)



![title](../assets/images/2017-12-27-kobe_regularizatio_23_1.png)









![title](../assets/images/2017-12-27-kobe_regularizatio_23_3.png)


On a higher level, these two models performed very similarly. As next steps I would like to run some feature engineering starting with the non-zero features from the Lasso regression. In addition I expect running the data through a boosted tree would yield a model with better predictive power.<br/><br/>
All in all I don't think the current models would be good enough for Kobe Bryant given his high standards. So we'll need to continue this effort.

![title](../assets/images/kobe_pray.jpg)

We got you Kobe.


```python

```
