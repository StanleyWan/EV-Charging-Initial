# Overview

This is the initial capstone project submitted to fulfill the requirements of the UC Berkeley Professional Certificate in Machine Learning and Artificial Intelligence.

The research problem is: “Can we predict whether an EV charging session represents a Casual Driver, Commuter, or Long-Distance Traveler based on charging and driving behaviors, and how can these insights support better charging infrastructure planning and personalized EV services?”

As an initial report, the emphasis is on the technical process of building and evaluating predictive models. Broader business insights and non-technical discussions will be expanded in the Final Report.

Here is the link of the Jupyter Notebook: [ev_charging_V1.ipynb](https://github.com/StanleyWan/EV-Charging-Initial/blob/main/ev_charging_V1.ipynb ) with visualizations and probability distributions. It is developed under Google's Colab.

## 1. Data Understanding
The dataset used in this project is [Global_EV_Charging_Behavior_2024.csv](https://github.com/StanleyWan/EV-Charging-Initial/blob/main/data/Global_EV_Charging_Behavior_2024.csv) downloaded from Kaggle.  It contains 800 rows and 16 Columns, providing a comprehesive and realistic view of EV charging usage and trends across different regions worldwide. 

The Dataset was chosen to replace the previoius proposed dataset [electric-vehicle-charging-patterns](https://www.kaggle.com/datasets/valakhorasani/electric-vehicle-charging-patterns). After a deeper analysis, the earlier dataset was found to be less realistic, making Global_EV_Charging_Behavior2024 a more suitable choice for the capstone project.

### Key Characteristics of the Dataset  
- **Size:** ~800 sessions × 16 attributes  
- **Coverage:** Global scope, reflecting diverse EV usage and charging environments  
- **Granularity:** Session-level data (each row corresponds to one charging session)  

### Important Columns Used  
- **Charging Start Time** – exact start timestamp, used to derive *Time of Day* and *Day Type (Weekday/Weekend)*.  
- **Battery Capacity (kWh)** – vehicle’s full battery size, a baseline for calculating State of Charge (SOC) changes.  
- **Energy Delivered (kWh)** – amount of energy provided in the session; used for labeling user type but excluded as a model feature to avoid leakage.  
- **Charging Cost ($)** – session cost, strongly influenced by vendor packages and incentives; retained as a key predictor of behavior.  
- **Payment Method** – categorical variable (Subscription, Card, App) that acts as a proxy for billing structure and package plans.  
- **Charging Station Type** – Level 1 / Level 2 / DC Fast; reflects charging speed and user patience.  
- **Charging Session Outcome** – whether a session was completed, failed, or aborted; noted but excluded since the effect is already reflected in energy delivered.  

### Label Creation (User Type)  
The dataset does not directly include user labels. To classify sessions, we derived **UserType** based on the percentage change in battery SOC during a charging session:  
- `< 20%` → **Casual Driver**  
- `20–60%` → **Commuter**  
- `> 60%` → **Long-Distance Traveler**  

### Observations from Early Analysis  
- **There are no missing data**
- **Cost is not proportional to kWh delivered**, due to discounts, subscriptions, and free-charging incentives. This makes cost a proxy for **package plan influence** rather than a direct measure of consumption.  
  The following is the graph to demonstrate the correlationship between the cost and Energy Delivered by Chargin Station Type.  It indicates that even they are correlated, the variance is large. It proves that pricing plans/packages distort the cost relationship.
  <p align="center">
  <img src="https://raw.githubusercontent.com/StanleyWan/EV-Charging-Initial/main/images/cost%20vs%20energy.png" width="800"/><br>
  <em>Figure: Correlationship between Cost and Energy Delivered by Charging Station Type</em>
</p>  

- **Payment Method** further reflects these incentives, as Subscription users behave differently from Card payers.

## 2. Data Processing and Engineering Features
  We have engineering 4 new columns:
  
  **a. TimeOfDay**:  binned into Morning, Office Hours, Evening, Night, Deep Night.   
  **b. Daytype**: Weekday vs. Weekend.
  **c. UserType**:  Derived from SOC % change
  **d. Cost_per_kWh:** Charging cost divided by energy delivered.

## 3. Modeling
  ### Baseline Model  
- **Algorithm:** Logistic Regression (OvR, default hyperparameters).  
- **Preprocessing:**  
  - Numeric → median imputation + standard scaling  
  - Categorical → most frequent imputation + one-hot encoding  
- **Train/Test split:** stratified 80/20

**Performance**

| Class          | Precision | Recall | F1-Score | Support |
|----------------|-----------|--------|----------|---------|
| Casual         | 0.87      | 0.59   | 0.70     | 22      |
| Commuter       | 0.82      | 0.82   | 0.82     | 66      |
| Long-Distance  | 0.87      | 0.96   | 0.91     | 72      |
| **Accuracy**   |           |        | **0.85** | 160     |
| **Macro Avg**  | 0.85      | 0.79   | 0.81     | 160     |
| **Weighted Avg** | 0.85    | 0.85   | 0.85     | 160     |

**Summary → Accuracy: 0.850 | Macro-F1: 0.812**  
- Model is strong for Long-Distance (96%) and Commuter (82%)  
- Casual Drivers are harder to classify (64%), often misclassified as Commuters.

## 4. Evaluation  

The following is Confusion Matrix resulted from the Logistic Regression Model  

  <p align="center">
  <img src="https://raw.githubusercontent.com/StanleyWan/EV-Charging-Initial/main/images/initial_confusion_logreg_costs.png" width="800"/><br>
  <em>Figure:          Confusion Matrix -- Logistic Regression</em>
</p>  

Result of the Confusion Matrix:
- Accuracy for the Casual Driver = 14/22=64%
- Accuracy for the Commuters = 54/66=82%
- Accuracy for the Long Distance ravelers = 69/72=96%
  
The model is very strong for Long-Distance (96%) and Commuters (82%). It struggles more with Casual Drivers (64%), often confusing them with Commuters.
------------------------
The following is the ROC Curves after the data Modeling  
<p align="center">
  <img src="https://raw.githubusercontent.com/StanleyWan/EV-Charging-Initial/main/images/initial_roc_logreg_costs.png" width="800"/><br>
  <em>Figure:          ROC Curves -- Logistic Regression</em>
</p>

ROC AUC values for each class are high Casual = 0.99  
Commuter = 0.94  
Long-Distance = 0.986  
It show strong separation on the classes  

## 5. Findings  
- **Cost ($)** is the strongest predictor — not because of energy delivered, but because it captures **vendor package influence**.  
- **Payment Method** further improves separation, acting as a proxy for whether a session was part of a subscription, card, or app-based plan.  
- Logistic Regression provides a strong, interpretable baseline.  

---

## 6. Next Steps (for Final Capstone)  
1. Extend modeling to **Decision Tree, KNN, SVM** with hyperparameter tuning.  
2. Compare models using Accuracy, Macro-F1, and AUC.  
3. Provide actionable insights on **infrastructure planning by user type**.  


