## Shopify Data Science Challenge

# Question:

  *On Shopify, we have exactly 100 sneaker shops, and each of these shops sells only one model of shoe. We want to do some analysis of the average order value (AOV). When we look    at orders data over a 30 day window, we naively calculate an AOV of $3145.13. Given that we know these shops are selling sneakers, a relatively affordable item, something seems    wrong with our analysis.*

  *Think about what could be going wrong with our calculation. Think about a better way to evaluate this data.*
  1. *What metric would you report for this dataset?*
  2. *What is its value?*
  3. *Solutions*

# STAGING:

There is nothing wrong with your calculation. You accurately calculated the average order value! On top of doing that, your sneaker shop subject matter expertise helped you make an important observation; something looks wrong here. Thank you for bringing us this far. Let's do some more exploration together to try to get the information you seek together.

If an AOV of $3,145.13 seems outrageous then we need to address some questions:
- Are outliers skewing our data?
- Are buyers and shops transacting in volumes far greater than we intended?
- Are we experiencing some type of fraudulent activity?
- Why are we trying to calculate AOV in the first place? Towards what end is this information being used?

Let's begin with some simple calculations and visualizations in Python and work our way towards answering question 1. *What metric would you report for this dataset?*

```import pandas as pd
import matplotlib.pyplot as plt
%matplotlib inline
import numpy as np
from scipy import stats
from scipy.stats import zscore
```
We will use pandas library to manipulate and view the data in tabular form...and also get a nice preview.
```python
df = pd.read_excel(r'2019 Winter Data Science Intern Challenge Data Set.xlsx')
df.head()
```
![image](https://user-images.githubusercontent.com/75325334/117593336-5ee7be80-b109-11eb-8cbe-1ea1dc126eb3.png)

Check to see if we have null values before proceeding with some basic calculations
```python
df.isnull().sum()
```
![image](https://user-images.githubusercontent.com/75325334/117593350-69a25380-b109-11eb-953f-6c8d8fbe942b.png)
```python
df.dtypes
```
![image](https://user-images.githubusercontent.com/75325334/117593265-4b3c5800-b109-11eb-9b30-26c2031c47e6.png)

Great! With a visualization of our data and assurance that we have no nulls or unexpected data types to work with, we can get our basic suite of information displayed and understand the data a bit better:
```mean_per_user_id = np.mean(df["order_amount"])
print(f"The mean of order amounts {mean_per_user_id:.2f}")

median_per_user_id = np.median(df["order_amount"])
print(f"The median of order amounts {median_per_user_id}")

std_dev_per_user_id = np.std(df["order_amount"])
print(f"The standard deviation of order amounts {std_dev_per_user_id}")
```
![image](https://user-images.githubusercontent.com/75325334/117593791-a0c53480-b10a-11eb-897a-d2a537b92935.png)

WOW! Look at that standard deviation! It is around 12x the mean! There are definately outliers in our dataset. Let's use a box-whisker plot to see them :
```x_labels=["Order Amount"]
fig, ax = plt.subplots()
ax.boxplot(df["order_amount"], labels=x_labels)

ax.set_ylabel('$ Amount')
#ax.set_yticks(np.arange())
ax.grid()
plt.show
```
![image](https://user-images.githubusercontent.com/75325334/117595364-a0c73380-b10e-11eb-99a7-e671ad356745.png)

Amazing! We have some extreme outliers in this set! In particular, lets make a note of the outlier just above the $700,000 AOV because this individual might tell us an important story about the way our platform is being used. It is also helpful in this case to scatter plot this data to see the order amounts over time to see if these outliers appear consistently.

```python
ax1 = df.plot.scatter(x='created_at',
                      y='order_amount')
```

![image](https://user-images.githubusercontent.com/75325334/117595499-00bdda00-b10f-11eb-9769-472a07d7b0f6.png)

The outlier orders are coming in regularly. Next, lets calculate z-scores so that we can apply the boolean results back against dataframe to filter out some of our outliers.
```oa_df = df[['order_amount']]

z_scores = stats.zscore(oa_df)
abs_z_scores = np.abs(z_scores)
filtered_entries = (abs_z_scores < 3).all(axis=1)
new_df = df[filtered_entries]
```
![image](https://user-images.githubusercontent.com/75325334/117595709-8772b700-b10f-11eb-80d7-19a60f84a40a.png)

The box whisker is telling me that working with the AOV is still dominated by extreme outliers and unless we can bin our users into sensible categories we are going to have a very hard time deriving value from our AOV calculation.
