# Predictive Model for Honey Fermentation - User Guide

This document serves as a user guide for the predictive model developed to enhance the Specialty Coffee Association (SCA) score for the honey fermentation process. This guide provides step-by-step instructions on how to use the provided Jupyter Notebook to predict the SCA score based on various input parameters related to the fermentation and drying processes.

### Difference Between Washed Fermentation and Honey Fermentation

Washed fermentation and honey fermentation are two different methods of processing coffee beans. In washed fermentation, the coffee cherries are soaked in water to remove the outer fruit layer before the beans are dried. This method gives the coffee a cleaner and brighter taste.

Honey fermentation (Anaerobic fermentation), on the other hand, a type of honey fermentation, happens in a sealed environment without oxygen (in this case, Cafe Anima Matris choose to seal them in the barrel), which can create unique flavors due to the special way microbes work in these conditions.


---

## Table of Contents
1. [Introduction](#introduction)
3. [Setup](#setup)
4. [Data Preparation](#data-preparation)
5. [Exploratory Data Analysis (EDA)](#exploratory-data-analysis-eda)
6. [Model Development](#model-development)
7. [Model Evaluation](#model-evaluation)
8. [Usage](#usage)
---

## Introduction

This notebook aims to predict the SCA score for coffee beans undergoing the honey fermentation process. By leveraging machine learning techniques, the model uses historical data to forecast optimal conditions for fermentation and drying, thus improving coffee quality.

### Features

- **Predictive Modeling:** Uses machine learning to predict SCA scores based on fermentation and drying conditions.
  - The Gradient Boosting model was identified as the best performing model, providing highly accurate predictions with minimal error.
    
- **Data Preprocessing:** Handles data cleaning, feature engineering, and encoding of categorical variables.
  - Ensures that the data is in the optimal format for model training and evaluation.
    
- **Model Evaluation:** Provides metrics to evaluate model performance.
  - Comprehensive evaluation using Mean Squared Error (MSE) and R² scores to ensure the best model is selected.
    
- **Hyperparameter Tuning:** Utilizes grid search for optimizing model parameters.
  - Ensures the model is tuned for the best performance, resulting in higher accuracy and better predictions.
    
- **New Data Prediction:** Easily predicts SCA scores for new batches of coffee beans.
  - The saved model can be reused to predict scores for new data, providing a scalable solution for ongoing quality improvement.

### Data Sources
The data used in this project is generated and stored in a CSV file named `fake_main_data.csv`, containing detailed information about the fermentation and drying processes of coffee beans. **Specfically, the model will only use data that related with honey fermentation process.**

If you have any questions about how the fake data was generated, please refer to the Synthetic Data folder for more detailed information.

---

## Setup

### 1. Importing Libraries

### 1. Installing Necessary Libraries

Ensure you have the necessary libraries installed. This includes standard data manipulation libraries like `pandas` and `numpy`, visualization libraries such as `matplotlib` and `seaborn`, and machine learning libraries like `scikit-learn` and `xgboost`.

You can install these libraries using `pip`. Open your terminal or command prompt and run the following commands:

```sh
!pip install pandas numpy matplotlib seaborn scikit-learn xgboost joblib


import sys
assert sys.version_info >= (3, 7)
from packaging import version
import sklearn
import joblib
assert version.parse(sklearn.__version__) >= version.parse("1.0.1")
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import xgboost as xgb
from xgboost import XGBRegressor
from sklearn import tree
from sklearn.linear_model import LinearRegression, Ridge, Lasso
from sklearn.tree import DecisionTreeRegressor
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.svm import SVR
from sklearn.model_selection import train_test_split, GridSearchCV, cross_val_score
from sklearn.metrics import mean_squared_error, r2_score
from sklearn.preprocessing import StandardScaler, PolynomialFeatures
from scipy import stats
from matplotlib.ticker import MaxNLocator, PercentFormatter
```
### 2. Importing the Dataset
Load the dataset from the provided CSV file.

```python
df = pd.read_csv("/path/to/your/fake_main_data.csv")
```

## Data Preparation

### 3. Data Cleaning

Filter the dataset to include only rows related to the honey fermentation method. Drop columns that are specific to washed fermentation.

```python
df_honey = df[df['Fermentation Method'] == 'Honey']
df_honey = df_honey.drop([ 'Type of Water Used'],  axis=1)
```
### 4. Calculating Durations

Convert date columns to datetime and calculate the duration of fermentation and drying processes.

```python
df_honey['Fermentation Start'] = pd.to_datetime(df_honey['Fermentation Start'])
df_honey['Fermentation End'] = pd.to_datetime(df_honey['Fermentation End'])
df_honey['Fermentation Duration (hours)'] = (df_honey['Fermentation End'] - df_honey['Fermentation Start']).dt.total_seconds() / 3600

df_honey['Drying Start Date/Time'] = pd.to_datetime(df_honey['Drying Start Date/Time'])
df_honey['Drying End Date/Time'] = pd.to_datetime(df_honey['Drying End Date/Time'])
df_honey['Drying Fermentation Duration (hours)'] = (df_honey['Drying End Date/Time'] - df_honey['Drying Start Date/Time']).dt.total_seconds() / 3600
```

### 5. Handling Weather Conditions

Count occurrences of various weather conditions during fermentation and create new columns for each condition.

```python
weather_conditions = ["Sunny", "Cloudy", "Rainy", "Partly Cloudy", "Overcast", "Light Rain", "Heavy Rain"]

for condition in weather_conditions:
    df_honey[condition] = df_honey['Fermentation Weather'].apply(lambda x: x.count(condition) if pd.notna(x) else 0)

df_honey.drop(columns=['Fermentation Weather'], inplace=True)
```

## Exploratory Data Analysis (EDA)

### 6. Correlation Analysis

Compute and visualize the correlation matrix for numerical columns.

```python
numerical_columns = ['Average Temp', 'Average PH', 'Average Brix', 'Average Humidity', 'Fermentation Duration (hours)', 'Drying Duration (hours)', 'SCA Score', "Sunny", "Cloudy", "Rainy", "Partly Cloudy", "Overcast", "Light Rain", "Heavy Rain", 'Average Drying Temp', 'Average Drying Humidity']

# Calculate correlation matrix for numerical columns
correlation_matrix = df_honey[numerical_columns].corr()

# Remove "SCA Score" from the list of numerical columns
numerical_columns_excluding_sca = [col for col in numerical_columns if col != 'SCA Score']

# Calculate the correlation matrix for numerical columns excluding "SCA Score"
correlation_matrix = df_honey[numerical_columns_excluding_sca].corr()

# Display the correlation matrix
print("Correlation Matrix:\n", correlation_matrix)

# Visualize the correlation matrix using a heatmap
plt.figure(figsize=(10, 8))
sns.heatmap(correlation_matrix, annot=True, cmap='coolwarm', linewidths=0.5)
plt.title('Correlation Matrix Heatmap')
plt.show()
```
From the correlation analysis, it is found that variable of "Partly Cloudy" and "Cloudy" is moderately correlated. Thus, it is determined that "Partly Cloudy" should be removed before training the model.

### 7. Univariate Analysis
Plot histograms for each numerical column to understand their distributions.

```python
num_cols = len(numerical_columns)
num_rows = num_cols // 3 + (num_cols % 3 > 0)

fig, axes = plt.subplots(nrows=num_rows, ncols=3, figsize=(20, 5 * num_rows))
axes = axes.flatten()

for i, col in enumerate(numerical_columns):
    sns.histplot(df_honey[col], stat='density', ax=axes[i], kde=True, line_kws={'linewidth': 2})

plt.tight_layout()
plt.show()
```

### 8. Categorical Features
Analyze categorical features and their distributions.

```python
categoric_columns = ['Coffee Variety', 'Additives for fermentation', 'Pre-fermentation for Honey']

for col in categorical_columns:
    df_honey[col].fillna('None', inplace=True)

fig, axes = plt.subplots(nrows=1, ncols=len(categorical_columns), figsize=(20, 5))
axes = axes.flatten()

for i, col in enumerate(categorical_columns):
    sns.countplot(x=df_honey[col], ax=axes[i], palette='viridis')
    axes[i].tick_params(axis='x', rotation=45)

plt.tight_layout()
plt.show()
```

## Model Development

### 9. Encoding Categorical Variables

One-hot encode categorical variables for model input.

```python
df_honey = pd.get_dummies(df_honey, columns=categorical_columns, drop_first=True)
```
### 10. Dropping Irrelevant Columns

Drop non-essential and correlated columns for the model.

```python
df_honey.drop(['Batch ID', 'Batch Name', 'Harvest Date', 'Fermentation Method', 'Fermentation Records', 'Drying Records'], axis=1, inplace=True)

df_honey = df_honey.drop([ 'Partly Cloudy'],  axis=1)
```

### 11. Train-Test Split
Split the dataset into training and testing sets.

```python
# Define features and target
X = df_honey.drop(columns=['SCA Score'])
y = df_honey['SCA Score'] 

# Split data into training and testing sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Normalize the features
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)
```

### 12. Model Training

Train multiple regression models and evaluate their performance.

```python
# Function to evaluate the model
def evaluate_model(model):
    model.fit(X_train_scaled, y_train)
    y_pred = model.predict(X_test_scaled)
    mse = mean_squared_error(y_test, y_pred)
    r2 = r2_score(y_test, y_pred)
    return mse, r2

# Linear Regression
linear_model = LinearRegression()
linear_mse, linear_r2 = evaluate_model(linear_model)

# Ridge Regression
ridge_model = Ridge()
ridge_mse, ridge_r2 = evaluate_model(ridge_model)

# Lasso Regression
lasso_model = Lasso()
lasso_mse, lasso_r2 = evaluate_model(lasso_model)

# Polynomial Regression
poly = PolynomialFeatures(degree=2)
X_train_poly = poly.fit_transform(X_train_scaled)
X_test_poly = poly.transform(X_test_scaled)
poly_model = LinearRegression()
poly_model.fit(X_train_poly, y_train)
y_pred_poly = poly_model.predict(X_test_poly)
poly_mse = mean_squared_error(y_test, y_pred_poly)
poly_r2 = r2_score(y_test, y_pred_poly)

# Decision Tree
tree_model = DecisionTreeRegressor(random_state=42)
tree_mse, tree_r2 = evaluate_model(tree_model)

# Random Forest
rf_model = RandomForestRegressor(random_state=42)
rf_mse, rf_r2 = evaluate_model(rf_model)

# Gradient Boosting
gb_model = GradientBoostingRegressor(random_state=42)
gb_mse, gb_r2 = evaluate_model(gb_model)

# XGBoost
xgb_model = XGBRegressor(random_state=42)
xgb_mse, xgb_r2 = evaluate_model(xgb_model)

# Support Vector Regression
svr_model = SVR()
svr_mse, svr_r2 = evaluate_model(svr_model)

# Print results
results = pd.DataFrame({
    'Model': ['Linear Regression', 'Ridge Regression', 'Lasso Regression', 'Polynomial Regression',
              'Decision Tree', 'Random Forest', 'Gradient Boosting', 'XGBoost', 'SVR'],
    'MSE': [linear_mse, ridge_mse, lasso_mse, poly_mse, tree_mse, rf_mse, gb_mse, xgb_mse, svr_mse],
    'R2': [linear_r2, ridge_r2, lasso_r2, poly_r2, tree_r2, rf_r2, gb_r2, xgb_r2, svr_r2]
})

print(results)
```
|              Model               |     MSE     |     R²     |
|:--------------------------------:|:-----------:|:----------:|
|       Linear Regression          |  1.620849   |  0.735029  |
|       Ridge Regression           |  1.621028   |  0.735000  |
|       Lasso Regression           |  4.736795   |  0.225645  |
|      Polynomial Regression       |  1.619201   |  0.735298  |
|          Decision Tree           |  3.300797   |  0.460397  |
|          Random Forest           |  1.453125   |  0.762448  |
|      Gradient Boosting           |  1.402577   |  0.770711  |
|              XGBoost             |  1.595555   |  0.739164  |
|                  SVR             |  1.658731   |  0.728836  |


#### The Best Performed Model - Gradient Boosting

The Gradient Boosting model performs the best among the listed models, with the lowest Mean Squared Error (MSE) and the highest R² score. This indicates that Gradient Boosting is highly effective at capturing the underlying patterns in the data, resulting in accurate predictions with relatively minimal error. The high R² score demonstrates that the model explains a significant portion of the variance in the target variable.

## Model Evaluation

### 13. Model Tuning 

Using Hyperparameter grid search to tune the Gradient Boosting model and then find the optimal hyperparameters.

```python
# Function to evaluate the model
def evaluate_model(model, X_train, X_test, y_train, y_test):
    model.fit(X_train, y_train)
    y_pred = model.predict(X_test)
    mse = mean_squared_error(y_test, y_pred)
    r2 = r2_score(y_test, y_pred)
    return mse, r2

# Define the parameter grid
param_grid = {
    'n_estimators': [100, 200, 300],
    'learning_rate': [0.01, 0.05, 0.1],
    'max_depth': [3, 5, 7],
    'subsample': [0.7, 0.8, 0.9],
    'min_samples_split': [2, 5, 10],
    'min_samples_leaf': [1, 2, 4]
}

# Initialize the model
gb_model = GradientBoostingRegressor(random_state=42)

# Initialize GridSearchCV
grid_search = GridSearchCV(estimator=gb_model, param_grid=param_grid, cv=3, scoring='neg_mean_squared_error', verbose=2, n_jobs=-1)

# Fit the model
grid_search.fit(X_train, y_train)

# Get the best parameters
best_params = grid_search.best_params_

# Initialize the model with the best parameters
best_gb_model = GradientBoostingRegressor(**best_params, random_state=42)

# Evaluate the model
best_gb_mse, best_gb_r2 = evaluate_model(best_gb_model, X_train, X_test, y_train, y_test)

print(f"Best Hyperparameters: {best_params}")
print(f"Best Gradient Boosting MSE: {best_gb_mse}")
print(f"Best Gradient Boosting R2: {best_gb_r2}")
```
It is found that the optimal Hyperparameters are {'learning_rate': 0.05, 'max_depth': 3, 'min_samples_leaf': 1, 'min_samples_split': 2, 'n_estimators': 200, 'subsample': 0.7}

### 14. Cross Validation

Cross validation was made on the tuned model, ensuring that the final model is valid for using.

```python

# Define features and target
X = df_honey.drop(columns=['SCA Score'])
y = df_honey['SCA Score'] 

# Split data into training and testing sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Initialize the model with the best hyperparameters
best_gb_model = GradientBoostingRegressor(
    n_estimators=200,
    learning_rate=0.05,
    max_depth=3,
    subsample=0.7,
    min_samples_split=2,
    min_samples_leaf=1,
    random_state=42
)

# Perform cross-validation
cv_scores = cross_val_score(best_gb_model, X_train, y_train, cv=5, scoring='neg_mean_squared_error')

# Convert negative MSE to positive
cv_mse_scores = -cv_scores

# Calculate the mean and standard deviation of the MSE scores
mean_mse = np.mean(cv_mse_scores)
std_mse = np.std(cv_mse_scores)

# Calculate R^2 scores using cross-validation
cv_r2_scores = cross_val_score(best_gb_model, X_train, y_train, cv=5, scoring='r2')

# Calculate the mean and standard deviation of the R^2 scores
mean_r2 = np.mean(cv_r2_scores)
std_r2 = np.std(cv_r2_scores)

print(f"Cross-Validated MSE: {mean_mse} ± {std_mse}")
print(f"Cross-Validated R2: {mean_r2} ± {std_r2}")

```
### Observations:

**Results:**
- Cross-Validated MSE: 1.675422723822886 ± 0.11731219243846999
- Cross-Validated R2: 0.7154326592503628 ± 0.022838231751375723

- **Mean Squared Error (MSE):** The average MSE across the cross-validation folds is 1.6754, with a standard deviation of 0.1173. This low MSE suggests that the model makes accurate predictions with minimal error. The small standard deviation indicates consistency in the model's performance across different data splits.
  
- **R² Score:** The average R² score is 0.7154, with a standard deviation of 0.0228. This high R² score means that the model explains 71.54% of the variance in the target variable, indicating a very good fit. The low standard deviation further suggests that the model's predictive power is stable across different subsets of the data.

Overall, these results demonstrate that the Gradient Boosting model is both accurate and consistent in predicting the target variable, making it a reliable choice for this dataset.


### 15. Fit the Model and Save it

Fitting the best hyperparameters to the model and save it as the pkl format **('gb_honey_model.pkl')** for future predictions.

```python
import joblib

# Define features and target
X = df_honey.drop(columns=['SCA Score'])
y = df_honey['SCA Score'] 

# Split data into training and testing sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Initialize the Gradient Boosting model with the best hyperparameters
gb_honey_model = GradientBoostingRegressor(
    n_estimators=200,
    learning_rate=0.05,
    max_depth=3,
    subsample=0.7,
    min_samples_split=2,
    min_samples_leaf=1,
    random_state=42
)

# Train the Gradient Boosting model
gb_honey_model.fit(X_train, y_train)

# Save the fitted model using joblib
joblib.dump(gb_honey_model, 'gb_honey_model.pkl')

```

## Usage

### 16. Predicting SCA Scores with New Data Using a Trained Model
In this case, we created a new data frame that include three data points to make predictions on SCA score.
(Please note that these data points are randomly generatd for showcasing the model prediction ability.)

```python
# Simulate loading the new batch of data into a DataFrame (new_data_df)
new_data = {
    'Average Temp': [24.7, 21.3, 26.7],
    'Average PH': [4.794, 4.639, 4.32],
    'Average Brix': [15.7, 17.4, 19.7],
    'Average Humidity': [77, 69, 74],
    'Average Drying Temp': [22.4, 22.4, 20.9],
    'Average Drying Humidity': [76.591, 71.111, 73.626],
    'Barrel Open in (Hour)': [1, 0, 0],
    'Barrel Close in (Hour)': [78, 105, 110],
    'Pre-fermentation Details (Hour) for Honey': [0, 0, 0],
    'Fermentation Duration (hours)': [111, 85, 98],
    'Drying Fermentation Duration (hours)': [114, 100, 90],
    'Sunny': [0, 0, 1],
    'Cloudy': [1, 2, 2],
    'Rainy': [1, 0, 0],
    'Overcast': [0, 0, 0],
    'Light Rain': [2, 1, 1],
    'Heavy Rain': [0, 1, 0],
    'Coffee Variety_Special': [True, True, False],
    'Additives for fermentation_Molasses': [False, False, False],
    'Additives for fermentation_None': [True, True, False],
    'Additives for fermentation_Sugar': [False, False, False],
    'Additives for fermentation_Yeast': [False, False, False],
    'Pre-fermentation for Honey_Yes': [False, False, False]
}

new_data_df = pd.DataFrame(new_data)

# Load the saved model
gb_honey_model = joblib.load('gb_honey_model.pkl')

# Prepare the input data
input_features = new_data_df  # Directly use the columns from the new data

# Make predictions
predictions = gb_honey_model.predict(input_features)

# Add predictions to the DataFrame
new_data_df['Predicted SCA Score'] = predictions

# Print out the results for each new data point in the specified format
for index, row in new_data_df.iterrows():
    input_vars = row.drop('Predicted SCA Score').to_dict()
    print(f"Data Point {index + 1}:")
    print(f"Input Variables: {input_vars}")
    print(f"Predicted SCA Score: {row['Predicted SCA Score']}\n")
```

### Predicted SCA Scores for Data Points

### Predicted SCA Scores for Data Points

#### Data Point 1:
- **Input Variables:**
  - Average Temp: 24.7
  - Average PH: 4.794
  - Average Brix: 15.7
  - Average Humidity: 77
  - Average Drying Temp: 22.4
  - Average Drying Humidity: 76.591
  - Barrel Open in (Hour): 1
  - Barrel Close in (Hour): 78
  - Pre-fermentation Details (Hour) for Honey: 0
  - Fermentation Duration (hours): 111
  - Drying Fermentation Duration (hours): 114
  - Sunny: 0
  - Cloudy: 1
  - Rainy: 1
  - Overcast: 0
  - Light Rain: 2
  - Heavy Rain: 0
  - Coffee Variety_Special: True
  - Additives for fermentation_Molasses: False
  - Additives for fermentation_None: True
  - Additives for fermentation_Sugar: False
  - Additives for fermentation_Yeast: False
  - Pre-fermentation for Honey_Yes: False

- **Predicted SCA Score:** 88.1210

---

#### Data Point 2:
- **Input Variables:**
  - Average Temp: 21.3
  - Average PH: 4.639
  - Average Brix: 17.4
  - Average Humidity: 69
  - Average Drying Temp: 22.4
  - Average Drying Humidity: 71.111
  - Barrel Open in (Hour): 0
  - Barrel Close in (Hour): 105
  - Pre-fermentation Details (Hour) for Honey: 0
  - Fermentation Duration (hours): 85
  - Drying Fermentation Duration (hours): 100
  - Sunny: 0
  - Cloudy: 2
  - Rainy: 0
  - Overcast: 0
  - Light Rain: 1
  - Heavy Rain: 1
  - Coffee Variety_Special: True
  - Additives for fermentation_Molasses: False
  - Additives for fermentation_None: True
  - Additives for fermentation_Sugar: False
  - Additives for fermentation_Yeast: False
  - Pre-fermentation for Honey_Yes: False

- **Predicted SCA Score:** 84.2862

---

#### Data Point 3:
- **Input Variables:**
  - Average Temp: 26.7
  - Average PH: 4.32
  - Average Brix: 19.7
  - Average Humidity: 74
  - Average Drying Temp: 20.9
  - Average Drying Humidity: 73.626
  - Barrel Open in (Hour): 0
  - Barrel Close in (Hour): 110
  - Pre-fermentation Details (Hour) for Honey: 0
  - Fermentation Duration (hours): 98
  - Drying Fermentation Duration (hours): 90
  - Sunny: 1
  - Cloudy: 2
  - Rainy: 0
  - Overcast: 0
  - Light Rain: 1
  - Heavy Rain: 0
  - Coffee Variety_Special: False
  - Additives for fermentation_Molasses: False
  - Additives for fermentation_None: False
  - Additives for fermentation_Sugar: False
  - Additives for fermentation_Yeast: False
  - Pre-fermentation for Honey_Yes: False

- **Predicted SCA Score:** 86.2442


## Notes:
Based on the model the team has trained and saved, there is a simple and user-friendly chatbot developed for clients to deploy, ensuring sustainability and ease of use. For more details, please refer to the video in the **"Chatbots" folder** demonstrates how to use the chatbot effectively







