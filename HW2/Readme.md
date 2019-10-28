# Problem 1

The link can be found here : https://piazza.com/class/k05u5i0wc3w540?cid=232

# Problem 2

##### Choose and critique one of the visualization by one of your fellow classmates for HW 1 Problem 2 (distribution of income across countries and continents over time). Include a link to the original. Describe the visualization and how it is similar and/or different from yours. Is it easy to interpret? Does it effectively visualize what is being asked? Why or why not?

Author : Omair Shafi Ahmed

Link : https://github.com/Omairss/ds5500_hw1/blob/master/Homework1.ipynb

Similarity / Difference : Omair's visualisation and my visualisation try to focus on distriubtion quite differently. He has plot a heat map where as I have tried to show the differences as a distribution plot. Moreover he has showcased the plot over all countries whereas I did it across continenets.

Interpretability : The plots are little hard to interpret as they fell cluttered.

Effectivness : His plots are effective and showcase the right story. One could easily see the trend of GDP per captia increases.

# Problem 3

##### Choose and critique one of the visualization by one of your fellow classmates for HW 1 Problem 3 (relationship between income, life expectancy, and child mortality over time). Include a link to the original. Describe the visualization and how it is similar and/or different from yours. Is it easy to interpret? Does it effectively visualize what is being asked? Why or why not?

Author : Omair Shafi Ahmed

Link : https://github.com/Omairss/ds5500_hw1/blob/master/Homework1.ipynb

Similarity / Difference : We both have tried to show the correlation as density plots, however to see the overall effect of time on these variables he has plot contries on different time whereas my plots show the average of all countires.

Interpretability : The plots are interpretable and easy to read and understand.

Effectivness : One could easily find the growing trend in life expectancy and income, whereas drop in child mortality over time in his plots and hence making it very effective for the visual story we are trying to present

### Helper for Problem 4 and Problem 5

```
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error

def modelHelper(model, X, Y, Xlab, Ylab):
    model.fit(X, Y)
    scr = 'MSE : ' + str(mean*squared_error(Y, model.predict(X)))
    print('Model Coefficent ' + str(model.coef*[0]))
    print('Model Intercept ' + str(model.intercept\_))
    plt.scatter(X,Y,color='g')
    plt.plot(X,model.predict(X),color='k')
    plt.xlabel(Xlab)
    plt.ylabel(Ylab)
    plt.title(scr)
    plt.show()

gdp = pd.read_csv("../data/ddf--datapoints--gdppercapita_us_inflation_adjusted--by--geo--time.csv")
life = pd.read_csv('../data/ddf--datapoints--life_expectancy_years--by--geo--time.csv')
cm = pd.read_csv('../data/ddf--datapoints--child_mortality_0_5_year_olds_dying_per_1000_born--by--geo--time.csv')
cont_map = pd.read_csv('../data/ddf--entities--geo--country.csv')

df = pd.merge(gdp, life)
df = pd.merge(df, cm)
df = pd.merge(df,cont_map[['country', 'world_4region']], left_on = 'geo', right_on = 'country')
df.head()
```

# Problem 4

##### Choose and fit one or more models to quantify the relationship betweem income (GDP per capita) and life expectancy over time. Justify your choice of model and comment on its appropriateness. (You are not required to handle the autocorrelation of time series, but should comment on how this impacts your analysis.) Visualize the model(s) and comment on what they tell you about the relationship between income and life expectancy over time.

```
gdp_life = df.groupby('time').mean().reset_index()\
                [['time','gdppercapita_us_inflation_adjusted', 'life_expectancy_years']]

gdp_life.columns = ['time', 'gdp', 'life']
plt.plot(gdp_life['gdp'], gdp_life['life'], 'o')
plt.show()
gdp_life.corr()

```

![Image of GDP and Life](https://github.com/sarthakkothari/DS5500/tree/master/HW2/GDP_Life.png)

```
X = gdp_life['gdp'].values.reshape(-1, 1)
Y = gdp_life['life'].values
modelHelper(LinearRegression(), X, Y, 'GDP', 'Life Expectancy', 'GDP_Life.png')

```

![Image of GDP and Life Model](https://github.com/sarthakkothari/DS5500/tree/master/HW2/GDP_Life_Model.png)

Since there exist a high correlation between the GDP and the life expectancy it is easy to estimate the relationship via a linear model. As one can see the model is able to estimate the relationship quite well.

# Problem 5

##### Choose and fit one or more models to quantify the relationship betweem income (GDP per capita) and child mortality over time. Justify your choice of model and comment on its appropriateness. (You are not required to handle the autocorrelation of time series, but should comment on how this impacts your analysis.) Visualize the model(s) and comment on what they tell you about the relationship between income and child mortality over time.

```
gdp_child = df.groupby('time').mean().reset_index()\
            [['time','gdppercapita_us_inflation_adjusted', 'child_mortality_0_5_year_olds_dying_per_1000_born']]
gdp_child.columns = ['time', 'gdp', 'child']
plt.plot(gdp_child['gdp'], gdp_child['child'], 'o')
plt.savefig('GDP vs Child.png')
plt.show()
gdp_child.corr()

```

![Image of GDP and Child](https://github.com/sarthakkothari/DS5500/tree/master/HW2/GDP_Child.png)

```
X = gdp_child['gdp'].values.reshape(-1, 1)
Y = gdp_child['child'].values
modelHelper(LinearRegression(), X, Y, 'GDP', 'Child Mortality', 'GDP and Child.png')
```

![Image of GDP and Child Model](https://github.com/sarthakkothari/DS5500/tree/master/HW2/GDP_Child_Model.png)

Similar to GDP and Life expectancy, GDP and child mortality too have a high correlation. Using a linear model again delivers a good estimate of the relationship between the two variables. It is intresting to see the steady drop in the child mortality rate as overall gdp increases.
