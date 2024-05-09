---
layout: page
title: Predicting Car Purchase
description: Using logistic regression to predict if a car will be purchased.
img: assets/img/13.jpg
importance: 3
category: fun
---

This project focuses on predicting whether a customer will purchase a car based on certain features using logistic regression. The dataset used contains information about customers' demographics and other relevant details, including their likelihood to purchase a car. By analyzing and visualizing this data, I built a predictive model that accurately predicts whether a customer will purchase a car based on selected features.

{% raw %}
```python
# Importing all the dependencies

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline
import seaborn as sns

df_train = pd.read_csv("car_data.csv")
#Size and info of data

print(f"The size of the data is {df_train.shape}")
print(f"The data have  {df_train.shape[0]} rows and {df_train.shape[1]} columns.")
print()
print(f"The overall information of the data:")
print()
print(df_train.info())

# Initial Statistical Analysis
df_train.describe()

# We have to observe and visualize the annual salary income

df_train.boxplot(column = 'AnnualSalary')
# Checking how many null vlaues in the dataset

df_train.isnull().sum()
#Change categorical values to 0's and 1's
df_train['Gender'] = df_train['Gender'].replace('Male',0)
df_train['Gender'] = df_train['Gender'].replace('Female',1)

# x have independent Variable
x = df_train.iloc[:, np.r_[1:4]].values

# y have only dependent variable
y = df_train.iloc[:, 4].values
# Importing the necessary library for implementation

from sklearn.model_selection import train_test_split
x_train, x_test, y_train, y_test = train_test_split(x,y, test_size=0.2, random_state=0)
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score

# Standardize the features
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(x_train)
X_test_scaled = scaler.transform(x_test)

# Initialize logistic regression model
logreg = LogisticRegression()

# Fit the model on the training data
logreg.fit(X_train_scaled, y_train)

# Make predictions on the testing data
y_pred = logreg.predict(X_test_scaled)

# Calculate accuracy
accuracy = accuracy_score(y_test, y_pred)
print("Accuracy:", accuracy)
```

{% endraw %}
