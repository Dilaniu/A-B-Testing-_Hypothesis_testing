# A-B-Testing-_Hypothesis_testing
Evaluating Statistical Significant in difference in players between 2 difference versions of the game, Cookie cats

Cookie Cats is a hugely popular mobile puzzle game developed by Tactile Entertainment. It's a classic "connect three" style puzzle game where the player must connect tiles of the same color in order to clear the board and win the level. It also features singing cats. We're not kidding!

As players progress through the game they will encounter gates that force them to wait some time before they can progress or make an in-app purchase. In this project, we will analyze the result of an A/B test where the first gate in Cookie Cats was moved from level 30 to level 40. In particular, we will analyze the impact on player retention and game rounds.

## Dataset - cookie_cats.csv 
userid - a unique number that identifies each player.
version - whether the player was put in the control group (gate_30 - a gate at level 30) or the test group (gate_40 - a gate at level 40).
sum_gamerounds - the number of game rounds played by the player during the first week after installation
retention_1 - did the player come back and play 1 day after installing?
retention_7 - did the player come back and play 7 days after installing?

When a player installed the game, he or she was randomly assigned to either gate_30 or gate_40.

1. Exploratory Data Analysis
2. Look summary stats and plots
3. Apply hypothesis testing and check assumptions
4. Check Normality & Homogeneity
5. Apply tests (Shapiro, Levene Test, T-Test, Welch Test, Mann Whitney U Test)
6. Evaluate the results
7. Make inferences


## Importing Necessary Libraries 
### Base
# -----------------------------------
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import os

### Hypothesis Testing
# -----------------------------------
from scipy.stats import shapiro
import scipy.stats as stats

### Configuration
# -----------------------------------
import warnings
warnings.filterwarnings("ignore")
warnings.simplefilter(action='ignore', category=FutureWarning)

pd.set_option('display.max_columns', None)
pd.options.display.float_format = '{:.4f}'.format

## Exploratory Data Analysis
if not data.isnull().any().any():
    print("No missing values!")
else:
    print("There are missing values.")
    
Result : No Missing Values!

data.shape
(90189, 5)

data.head()

## Summary Statistics
 Number of Unique User
print("Unique Users: ", data.userid.nunique() == data.shape[0])
 Summary Stats: sum_gamerounds
data.describe([0.01, 0.05, 0.10, 0.20, 0.80, 0.90, 0.95, 0.99])[["sum_gamerounds"]]
Result: Unique Usrs = True ## this means that all records are unique in this dataset

## Outliers
### Checking the outlier 
data[data['sum_gamerounds']>40000]
### Removing Outliers
data = data.drop(data.loc[data['sum_gamerounds']>4000].index)

## Visualizing number of players for each version on the same graph 
data[(data.version == "gate_30")].reset_index().set_index("index").sum_gamerounds.plot(legend = True, label = "Gate 30", figsize = (20,5))
data[data.version == "gate_40"].reset_index().set_index("index").sum_gamerounds.plot(legend = True, label = "Gate 40", alpha = 0.8)
plt.suptitle("After Removing The Extreme Value", fontsize = 20)
plt.xlabel('Index (Players)')
plt.show
#### overlay on the same axes, with "Gate 30" and "Gate 40" labels, making it easy to compare trends or patterns between the two versions in the sum_gamerounds column.

fig, axes = plt.subplots(2, 1, figsize = (10,10))
data.groupby("sum_gamerounds").userid.count().plot(ax = axes[0])
data.groupby("sum_gamerounds").userid.count()[:200].plot(ax = axes[1])
plt.suptitle("The number of users in the game rounds played", fontsize = 20)
axes[0].set_title("How many users are there all game rounds?", fontsize = 15)
axes[1].set_title("How many users are there first 200 game rounds?", fontsize = 15)
plt.tight_layout(pad=5);

1. With this plot we can see that number of players decrease significantly as the game level increases.
2. We can see that there are 3994 players hwo downloaded the game but never played.
    Could be that they did not have time to play the game.
    Did not like the app or the uder interface

   
## Statistis of each version
sum_gamerounds
30    642
40    505

### A/B Groups & Target Summary Stats
data.groupby("version").sum_gamerounds.agg(["count", "median", "mean", "std", "max"])
version,count,median,mean,std,max
gate_30,44698,17.0,51.27701463152714,101.12650965821616,2438
gate_40,45489,16.0,51.29877552814966,103.29441621652808,2640

Looking at the summary statistics, the control and Test groups seem similar, but are the two groups statistically significant?


## Player Retention Statistics
retention_1 - did players return 1 day after installing?
retention_7 - did players return 7 days after installing?

# Retention Problem
retention = pd.DataFrame({"RET1_COUNT": data["retention_1"].value_counts(),
              "RET7_COUNT": data["retention_7"].value_counts(),
              "RET1_RATIO": data["retention_1"].value_counts() / len(data),
              "RET7_RATIO": data["retention_7"].value_counts() / len(data)})
retention.T

	False	True
RET1_COUNT	50035.0000	40152.0000
RET7_COUNT	73408.0000	16779.0000
RET1_RATIO	0.5548	0.4452
RET7_RATIO	0.8140	0.1860


#Hypothesis Testing
##Assumptions:

###Check normality
If Normal Distribution, check homogeneity
Steps

1. Apply Shapiro tests for normality
2. If parametric apply Levene Test for homogeneity of variances
3. If Parametric + homogeneity of variances apply T-Test
4. If Parametric - homogeneity of variances apply Welch Test
5. If Non-parametric apply Mann Whitney U Test directly

## Define A/B groups
data["version"] = np.where(data.version == "gate_30", "A", "B")
data.head()

# Results!

## A/B Testing Hypothesis
H0: A == B
H1: A != B 

Test Type	AB Hypothesis	p-value	Comment
0	Non-Parametric	Fail to Reject H0	0.0502	A/B groups are similar!

### After applying A/B Testing, the analysis result gives us some important information. Shapiro Testing rejected H0 for Normality assumption. Therefore we needed to apply a Non-parametric test as called Mann Whitney U to compare two groups.

### As a result, Mann Whitney U Testing Failed to reject H0 hypothesis and we learned A/B groups are similar!

### There are no statistically significant difference between two groups about moving first gate from level 30 to level 40 for game rounds.

