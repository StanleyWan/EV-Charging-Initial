# Overview

This is the initial capstone project submitted to fulfill the requirements of the UC Berkeley Professional Certificate in Machine Learning and Artificial Intelligence.

The research problem is: “Can we predict whether an EV charging session represents a Casual Driver, Commuter, or Long-Distance Traveler based on charging and driving behaviors, and how can these insights support better charging infrastructure planning and personalized EV services?”

As an initial report, the emphasis is on the technical process of building and evaluating predictive models. Broader business insights and non-technical discussions will be expanded in the Final Report.

## 1. Data Understanding
The dataset used in this project is [Global_EV_Charging_Behavior_2024](https://www.kaggle.com/datasets/atharvasoundankar/global-ev-charging-behavior-2024) downloaded from Kaggle.  It contains 800 rows and 18 Columns, providing a comprehesive and realistic view of EV charging usage and trends across different regions worldwide. 

The Dataset was chosen to replace the previoius proposed dataset[electric-vehicle-charging-patterns](https://www.kaggle.com/datasets/valakhorasani/electric-vehicle-charging-patterns). After a deeper analysis, the earlier dataset was found to be less realistic, making Global_EV_Charging_Behavior2024 a more suitable choice for the capstone project.

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
  a. TimeOfDay: from the "Charging Start Time" column  
      Morning: 6-9  
      Office Hour: 9-17  
      Evening: 17-22  
      Night: 22-24  
      Deep Night:24-6  
  b. TimeOfDay: from the "Charging Start Time" column  
      Weekday: Monday-Friday  
      Weekend: Saturday-Sunday  
  
  We define SOC as (energy deliverd/battery capacity)*100  

  c. UserType:  
     Causal: soc < 20  
     Commuter: in the range of 20-60 of soc  
     Long-Distance: soc >= 60  
  d. Cost_per_kWh = Cost/Energy Delivered  
