# Overview of Predictive Models for Coffee Fermentation

This document provides a brief summary of two predictive models developed to enhance the Specialty Coffee Association (SCA) score for coffee beans processed through two different fermentation methods: washed fermentation and honey (anaerobic) fermentation. Each model uses historical data and machine learning techniques to forecast optimal conditions for fermentation and drying, thus improving coffee quality.

---

## Table of Contents
1. [Introduction](#introduction)
2. [Washed Fermentation Model](#washed-fermentation-model)
3. [Honey Fermentation Model](#honey-fermentation-model)
4. [Common Features](#common-features)

---

## Introduction

The two models aim to predict the SCA score for coffee beans undergoing washed fermentation and honey fermentation. By leveraging machine learning techniques, the models use historical data to forecast optimal conditions for fermentation and drying, improving the coffee's overall quality. 

Different fermentation methods can be impacted by different variables, which is why two separate models were developed. For honey fermentation, since the fermentation occurs in the barrel without water and air, factors such as the time the barrel is open and whether the beans have been pre-fermented are important. In contrast, for washed fermentation, the type of water used in the process can be crucial.

### The Washed Fermentation Model was developed before the Honey Fermentation Model. Therefore, it is recommended to review the files for the Washed Fermentation Model first to follow the logical sequence of development.

### Video Demonstration

You can watch the code demo video for the predictive model of washed fermentation by clicking the link below:

[Code Demo of Predictive Model of Washed Fermentation](https://mcgill-my.sharepoint.com/:v:/r/personal/dhevin_desilva_mail_mcgill_ca/Documents/Community%20Project/2.%20Code%20Demo%20Video%20for%20Models/Code%20Deom%20of%20Predictive%20Model%20of%20Washed%20Fermentation.mp4?csf=1&web=1&nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=ZVE8NX)

---

## Washed Fermentation Model

### NOTE: A video code demo for Washed Fermentation Model is available under the "Predictive Model of Washed Fermentation".
- The purpose of this code demo is to provide a clear and detailed understanding of how to use the model to enhance the quality of coffee beans through optimized fermentation processes. (The predictive model for Honey Fermentation follows a similar process, so only one detailed code demo for model development is provided.)

- The video thoroughly walks through the entire notebook, demonstrating the development and evaluation of the predictive model for coffee fermentation, including data preprocessing, exploratory data analysis, model training, hyperparameter tuning, and cross validations of the model.

### Features
- **Predictive Modeling:** Uses machine learning to predict SCA scores based on fermentation and drying conditions.
  - The Gradient Boosting model was identified as the best-performing model, providing highly accurate predictions with minimal error.
- **Data Preprocessing:** Handles data cleaning, feature engineering, and encoding of categorical variables.
  - Ensures that the data is in the optimal format for model training and evaluation.
- **Model Evaluation:** Provides metrics to evaluate model performance.
  - Comprehensive evaluation using Mean Squared Error (MSE) and R² scores to ensure the best model is selected.
- **Hyperparameter Tuning:** Utilizes grid search for optimizing model parameters.
  - Ensures the model is tuned for the best performance, resulting in higher accuracy and better predictions.
- **New Data Prediction:** Easily predicts SCA scores for new batches of coffee beans.
  - The saved model can be reused to predict scores for new data, providing a scalable solution for ongoing quality improvement.

### Data Sources
The data used in this project is generated and stored in a CSV file named `fake_main_data.csv` with fermentation method of **Washed**, containing detailed information about the fermentation and drying processes of coffee beans.

### Important Variables
- **Type of Water Used:** The type of water used in the washed fermentation process can significantly impact the quality and taste of the coffee.

---

## Honey Fermentation Model

### Features
- **Predictive Modeling:** Uses machine learning to predict SCA scores based on fermentation and drying conditions.
  - The Gradient Boosting model was identified as the best-performing model, providing highly accurate predictions with minimal error.
- **Data Preprocessing:** Handles data cleaning, feature engineering, and encoding of categorical variables.
  - Ensures that the data is in the optimal format for model training and evaluation.
- **Model Evaluation:** Provides metrics to evaluate model performance.
  - Comprehensive evaluation using Mean Squared Error (MSE) and R² scores to ensure the best model is selected.
- **Hyperparameter Tuning:** Utilizes grid search for optimizing model parameters.
  - Ensures the model is tuned for the best performance, resulting in higher accuracy and better predictions.
- **New Data Prediction:** Easily predicts SCA scores for new batches of coffee beans.
  - The saved model can be reused to predict scores for new data, providing a scalable solution for ongoing quality improvement.

### Data Sources
The data used in this project is generated and stored in a CSV file named `fake_main_data.csv` with fermentation process of **Honey**, containing detailed information about the fermentation and drying processes of coffee beans.

### Important Variables
- **Barrel Open Time:** The duration the barrel is open during the honey fermentation process can influence the fermentation outcome.
- **Pre-fermentation:** Whether the beans have been pre-fermented before the main fermentation process can affect the coffee quality.

---

## Common Features

Both models share the following features:
- **Comprehensive Data Preprocessing:** Ensures data is cleaned, features are engineered, and categorical variables are encoded appropriately.
- **Robust Model Evaluation:** Utilizes Mean Squared Error (MSE) and R² scores to evaluate model performance, ensuring the selection of the best model.
- **Advanced Hyperparameter Tuning:** Uses grid search to optimize model parameters for better accuracy and predictions.
- **Scalability:** Both models can predict SCA scores for new data, making them scalable solutions for continuous quality improvement.
- **User-Friendly Interface:** Simple chatbot interface for predicting SCA scores based on user inputs, enhancing usability for clients.

For detailed instructions on how to use each model, please refer to the individual folder provided.
