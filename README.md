# League of Legends 15-Minute Statistical Analysis

Author: Annabelle Guiditta

## Introduction

League of Legends (also known as LoL), is a popular multiplayer online battle (MOBA) game developed and published by Riot Games, the creators of Valorant and Arcane. Well known as Riot's most sucessful game, LoL is world-known and renowned for its large Esports influence. The dataset this report will cover comes from [Oracle's Elixer](https://oracleselixir.com/about), a free to access collection of Datasets from the League competitive sphere since 2015. In particular, the dataset referenced in this report draws from the **2025 competitive season**.

This dataset provides information of each individual game, team, and player, recording statistics such as first blood, game length, number of kills, and results, culminating in a rich source of information for data analysis.

A typical League of Legends game lasts between **25-35** minutes and consists of players killing monsters, fighting other players, and supporting their team. Much of this is done through accumulating gold and XP, features that allow players to buy items and \_\_\_. These features can be enhanced by kills and assists, which reward the player in gold. Having a large advantage in any of these statistics is key for winning a LoL game, and will be the basis of our analysis in this report.

Our analysis focuses on the impact of early game statistics on the rest of the game, focusing on recording **15 minute features** such as gold, XP, Creep Score, Kills, and more. We wish to use these metrics to see how credible early game metrics are for setting the course of the rest of the game, and help us identify what room there is for comeback in professional games.

### Data Description

After cleaning, our dataset consisted of **18472 rows and 19 columns**. These include:

- `result`: The outcome of the match for each specific team. True indicates the team that won, False indicates the team that lost
- `participantid`: The individual ID for each player in a round. In our dataset, this consists of either 100 (Blue Side) or 200 (Red Side) to help us differentiate between teams
- `league_tier`: Denotes the tier of the league the match was played in. 'Major' refers to games played in LCK, LEC, LCP, LTA, LTA S, or LTA N, all Tier 1 Professional Leagues. 'International' refers to games held in international competitions, such as MSI, WLDs, EWC, IC, and Asia Master, which host teams from both Major and Minor leagues. 'Minor' refers to games played in all other leagues.
- `split`:
- `date`: The date, formatted as YYYY-MM-DD XX:XX:XX
- `length`: The total length of the game
- `goldat15`, `xpat15`, `csat15`: The amount of gold, experience, or creep score the team had at the 15 minute mark, respectively
- `golddiffat15`, `xpdiffat15`, `csdiffat15` : The difference between the team's gold, experience, or creep score the team had at the 15 minute mark, respectively
- `gold_lead_at15` : A boolean column denoting whether a team had a gold lead (positive 'golddiffat15') at 15 minutes
- `killsat15`, `opp_killsat15` : The total amount of kills the team and their opponent had at 15 minutes, respectively
- `deathsat15`, `opp_deathsat15`: The total amount of deaths the team and their opponent had at 15 minutes, respectively
- `assistsat15`, `opp_assistsat15`: The total amount of assists the team and their opponent had at 15 minutes, respectively

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

To begin the data cleaning process, I reduced the amount of columns from the initial amount of 165 to only 19, focusing primarily on 15-minute statistics and important identifiers in the game, such as `result` and `league`. Then, I turned to the rows. In the Oracle's Elixer dataset, several columns are marked as having 'partial' completition in the `datacompleteness` column, indicating an incomplete collection of data. To best preserve the interests of my research question, I removed all rows marked as having 'partial' completion. Additionally, each recorded game in the dataframe consists of 12 rows - 5 for players of each team, and 2 for each teams aggregated statistics (denoted as 100 and 200). As we are interested in analyzing games from an aggregated point of view, I removed all rows where `participantid` did not equal either 100 or 200.

Aside from basic data conversions, such as turning `results` into a Boolean column and converting `date` into a datetime column, I developed several columns to help aid my work later on. These include the `league_tier` column, where I manually developed the Major, Minor, and International Competition splits based on each row's `league`; as well as `gold_lead_at15`, a column that identifies all rows that had a positive `golddiffat15`, indicating that they had more gold at 15 minutes than the opposing side.

Finally, I analyzed the missingness in my data, specifically in `split`, as this column didnt contribute much to the rest of the report, I decided to keep the Nan values as-is. Since 15% of the rows were missing `split`, dropping them would be unnecessary, whereas imputating a categorical column such as this wouldnt make much sense.

Below is the head of the cleaned dataset:

| participantid | result | league_tier | split  | date                | length | goldat15 |  xpat15 | csat15 | golddiffat15 | xpdiffat15 | csdiffat15 | killsat15 | deathsat15 | assistsat15 | opp_killsat15 | opp_assistsat15 | opp_deathsat15 | gold_lead_at15 |
| ------------: | -----: | :---------- | :----- | :------------------ | -----: | -------: | ------: | -----: | -----------: | ---------: | ---------: | --------: | ---------: | ----------: | ------------: | --------------: | -------------: | :------------- |
|           100 |  False | Minor       | Winter | 2025-01-11 11:11:24 |  15922 |   1498.0 | 28270.0 |  496.0 |      -3837.0 |     -469.0 |      -16.0 |       0.0 |        3.0 |         0.0 |           3.0 |             9.0 |            0.0 | False          |
|           200 |   True | Minor       | Winter | 2025-01-11 11:11:24 |  15922 |   5335.0 | 28739.0 |  512.0 |       3837.0 |      469.0 |       16.0 |       3.0 |        0.0 |         9.0 |           0.0 |             0.0 |            3.0 | True           |
|           100 |   True | Minor       | Winter | 2025-01-11 12:06:37 |  19222 |   8328.0 | 28393.0 |  497.0 |       5069.0 |     2014.0 |       64.0 |      10.0 |        6.0 |        20.0 |           5.0 |             8.0 |           10.0 | True           |
|           200 |  False | Minor       | Winter | 2025-01-11 12:06:37 |  19222 |   3259.0 | 26379.0 |  433.0 |      -5069.0 |    -2014.0 |      -64.0 |       5.0 |       10.0 |         8.0 |          10.0 |            20.0 |            6.0 | False          |
|           100 |  False | Minor       | Winter | 2025-01-11 13:07:47 |  17822 |   6828.0 | 31544.0 |  476.0 |        118.0 |     1990.0 |      -43.0 |      10.0 |        6.0 |        13.0 |           6.0 |            12.0 |           10.0 | True           |

### Univariate Analysis

To begin the exploration of my dataset, I investigated the distribution of Gold at the **15 minute** mark.

<iframe
  src="assets/univar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This histogram of `goldat15` illustrates that the amount of gold teams accumulate by 15 minutes follows a roughly normal distribution, centered a bit under **25,000 gold**. The shape is noticeably right-skewed, with most teams falling between **22,000 and 28,000 gold** at the 15 minute mark. This distribution suggests that gold accumulation at 15 minutes is fairly consistent across professional games, with extreme values on either end being rare. In the context of our research question, this histogram helps establish what a "typical" 15 minute gold value looks like, providing a baseline for understanding how meaningful deviations from this range might be in predicting match outcomes.

### Bivariate Analysis

To begin my bivariate analysis, I attempted to visualize some of the differences between **Major and Minor Leagues**.

<iframe
  src="assets/bivar.html"
  width="100%"
  height="600"
  frameborder="0"
></iframe>

To account for the difference in the number of Major vs Minor league games, I constructed a density histogram to fairly compare the distribution of `golddiffat15` across both tiers. Notably, the Major league distribution has a much **sharper and higher peak**, suggesting that gold differences in Tier 1 games are more tightly concentrated around 0. Major league games fall mostly between **-5,000 and 5,000 gold**, while Minor league games span a noticeably wider range of roughly **-7,000 to 7,000 gold**. This visual difference is supported by further calculations -- Minor league games exhibit **~73.7% more variance** in `golddiffat15` than Major league games, suggesting a more volatile early game landscape where leads are larger and less evenly matched. This sets the stage for our hypothesis test, where we formally investigate whether this difference is statistically significant.

### Interesting Aggregates

In investigating the importance of a **gold lead** early on in the game, I developed this grouped table to visualize its impact

| result |   xpat15 | killsat15 | assistsat15 | csat15 | gold_lead_at15 |
| :----- | -------: | --------: | ----------: | -----: | -------------: |
| False  | 27294.42 |      2.93 |        3.22 |   5.41 |         508.72 |
| True   | 27307.08 |      8.45 |        7.09 |   9.70 |         529.11 |

In particlar, it can be noted that teams that have a gold lead at 15 minutes tend to win more, and have higher xp, kills, assists, and creep score! In fact, **73% of teams** that had a gold lead at 15 minutes ended up winning the round

Additionally, I wanted to investigate the mean values across the different league tiers

| league_tier   | goldat15 |   xpat15 | killsat15 | assistsat15 | csat15 |
| :------------ | -------: | -------: | --------: | ----------: | -----: |
| International | 25084.89 | 30262.37 |      4.32 |        7.50 | 526.85 |
| Major         | 24688.91 | 30111.87 |      3.23 |        5.89 | 534.58 |
| Minor         | 25036.93 | 30057.36 |      4.75 |        7.93 | 514.94 |

Notably, Major league games tend to have **lower average kills and assists** at 15 minutes compared to both Minor and International leagues. This aligns with the expectation that Tier 1 games are more defensive and methodical in the early game. Interestingly, Major leagues have a slightly **higher average CS** (534 vs 514), suggesting that while Major league teams fight less, they farm more efficiently.

## Assessment of Missingness

### MNAR Analysis

**Missing Not at Random (MNAR)** refers to missingness that is based on the actual missing values themselves, such as high incomes being missing on a survey because people dont want to report them.

In the total Oracle's Elixir LoL dataset, potential candidates for MNAR are the `ban` columns. Teams may choose not to ban certain champions or simply run out of time to choose, and thus bans may not be recorded precisely because no ban was made. Perhaps if we had a column called `bans_skipped` that denotes the total number of bans skipped in the setup phase, we could explain the missingness and make it MAR

That said, most missingness in the dataset we cleaned appears to be either **Missing by Design** (such as `goldat15` being absent when a game ends before 15 minutes) or **MAR** dependent on observed columns like `date` or `league`. We could not conclusively identify any of these columns were **MNAR**, as there is no strong evidence that the missing values depend on the values themselves rather than on other observed variables.

### MAR Analysis

**Missing at Random (MAR)** refers to missingness that is not based on the missing values themselves, but is related to another column in the dataset. For example, political preference of a survey being missing for participants of younger ages, where missingness is explained by their age -- not their actual preference

In our dataset, I observed several missing values in `split` -- infact, **15.4%** of rows had missing values for this column. Upon further investigation, split appeared to have many **inconsistent values and names** for games played in the same period, with some leagues using labels like Split 1, 2 or 3, whereas others used ones such as Winter Spring, and Summer.

This pattern caused me to consider if the missingness of `split` might depend on **different league tiers**. Minor leagues could have more missing splits due to being less organized and having less resources to record data. To investigate this, I used a permutation test to determine whether the missingness of `split` was truly MCAR or could be MAR on some column in our dataset.

By just finding a **single instance** of the missingness of `split` having significant correlation with another column, then we can conclude that the missingness of `split` is MAR! Otherwise, we must test against all other columns to to determine if it was MCAR.

First, I tested on `league` using a permutation test with Total Variation Distance (TVD) as my test statistic.

<iframe
  src="assets/MARTrue.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

There appears to be a **significant difference** in the missingness of splits between major and minor leagues! With a p-value of approximately 0, this test suggests that the missingness of `split` is likely related to `league_tier`, and not just due to random chance.
Therefore, we can conclude that the missingness of splits is likely **MAR**, since it appears to be related to an observable variable (league tier) in our dataset.

On the other hand, lets investigate a column that the missingness of `split` may not depend on, such as `golddiffat15`

<iframe
  src="assets/MARFalse.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
Clearly, the missingness of `split` does not seem to depend on `golddiffat15`, with a p-value of 1.0

## Hypothesis Testing

In the Oracle's Elixer LoL Dataset, each row has a column labeled **`league`**, denoting professional playing leagues that teams get matched and play in. Earlier in our cleaning step, we denoted a league as a "Major" league if it was from **LCK, LEC, LCP, LTA N, LTA S, or LTA.**

Tier 1 Professional leagues, such as those above, are seen as more competitive -- leading to highly technical games with **close stats between teams**. In my hypothesis test, I wish to discover whether games played at a Major Tier 1 Professional Leagues have **smaller gold differences** between teams than Minor Leagues do.

**Null Hypothesis**: Major (LCK, LEC, LCP, LTA N, LTA S, LTA) and minor leagues have the same mean absolute gold difference at 15 minutes and come from the same distribution

**Alternative Hypothesis**: Minor leagues have a higher mean absolute gold difference at 15 minutes than major leagues, thus coming from a different distribution

**Test Statistic**: Difference in absoluted gold differences between games in Major Leagues vs Minor Leagues

<script type="math/tex; mode=display">
|\overline{\text{Gold Diff at 15 Minutes - Minor}}| - |\overline{\text{Gold Diff at 15 Minutes - Major}}|
</script>

**Significance Level**: 1%

As this hypothesis seeks to detemine whether two populations **come from the same distribution**, we used a **permuation test** to investigate this difference.

In our dataset, there are 2 rows per game — one for the Red Team (Participant ID 200) and one for the Blue Team (Participant ID 100).
Each team has a value for **`golddiffat15`**, which denotes the gold difference between teams at 15 minutes. The value is mirrored across both rows, with one team's value being negative (implying they have less gold).

Since we only care about each individual game in this permutation test, we reduced the dataset to **1 row per game** by filtering to a single Participant ID, then take the absolute value of **`golddiffat15`** — allowing us to compare the average gold difference at 15 minutes between Major and Minor leagues.

> **Note**: While we take the absolute value of each game's gold difference, the test statistic itself — the difference between the two group means — is not absoluted. This is a one-sided hypothesis test, meaning we are specifically testing whether Minor leagues have a _higher_ mean gold difference than Major leagues, rather than simply whether they differ.

<iframe
  src="assets/permtest.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
Based on the hypothesis test performed, with a **p-value of approximately 0**, there is overwhelming evidence against the Null hypothesis, we **reject it in favor of the alternative**. Minor leagues have a statistically significantly higher mean absolute gold difference at 15 minutes than Major leagues. This could perhaps suggest that Tier 1 professional leagues are indeed more competitive, with teams staying closer in gold throughout the early game.

## Framing a Prediction Problem

Early game is one of the most important phases in League of Legends -- it sets the tone for the rest of the match and often gives fans an early indication of which team will come out on top. Our dataset includes statistics captured at different time markers throughout the game, such as **Gold, XP, Kills, Deaths, and Assists**, allowing us to analyze how early performance relates to final outcomes.

These details lead us to our core prediction problem:

> _Can we predict match outcome from 15 minute stats alone?_

This question asks whether the early game truly sets the course for the rest of the match -- or whether there is still significant room for comeback. If 15 minute stats are strong predictors of the final outcome, it might suggest that early game advantages are decisive in professional play.

To address this question, I used **binary classification** to predict `result`, which I evaluated using accuracy.

## Baseline Model

Since this is a binary classification problem, I decided to use a **Decision Tree Classifier** for my baseline model, which was trained upon all of our `gold`, `experience`, and `creep score` information at 15 minutes. Since all of the features are quantitative, not much transformation needed to be done -- I only used the **StandardScaler Transformer** to standardize each column's values, which would allow me to compare the columns directly.

Additionally, I used a **train-test split** to train the dataset on itself, so that I might predict the accuracy of the model. After fitting, the accuracy of our test split was **73.11%**. While this model seems to work well enough for most our our data, there is still improvement space, as we explore in the next section.

## Final Model

In my final model, I decided to switch to a **Random Forest Classifier**, an ensemble learning method that builds multiple decision trees during training and combines their predictions to produce a more accurate and stable result than any single tree alone. Additionally, I used **GridSearchCV** to identify the optimal hyperparameters for my model, specifically tuning

- **`max_depth`**: Controls how deep each tree can grow.
- **`n_estimators`**: Controls how many trees are in the forest. More trees generally improves stability and reduces variance, but with diminishing returns and increased computation time.

By tuning these two hyperparameters, I had hoped to address some of the key weaknesses of a single decision tree, such as **overfitting** and **high variance**, while finding the optimal balance between model complexity and generalization. Additionally, GridSearchCV would subject our model to a **5-fold cross validation**, a method to prevent **overfitting** -- one of the biggest issues of Decision Tree Classifiers.

Additionally, I introduced more 15 minute stats, such as `kills`, `deaths`, and `assists` to help aid our prediction, as well as `league_tier` for our later fairness analysis. These stats all give the model a better understanding of a teams position at 15 minutes, and would hopefully provide valuable predictive power for our final model to help us determine how useful 15-minute statstics are in predicting the outcome of a game.

When developing the model pipeline, I used a **FunctionTransformer** to calculate the difference of the features `killsat15` and `opp_killsat15`, similar to `golddiffat15`, `xpdiffat15`, and `csdiffat15`. I also used **OneHotEncoder** on `league_tier`, as it is a nominal categorical feature with no inherent ordering.

Despite introducing more models and testing hyperparameters, we don't see much of an improvement in the overall Test Accuracy, achieving only **74.27%**. Its possible that this is simply the best we can do with only the 15 minute stats in our dataset. Despite the small improvement, I'm happy with the result, as it shows how we can pretty accurately determine the outcome of a game on just the 15 minute statistics alone, with our error showing there is still room for comeback before then.

## Fairness Analysis

Since there are many more Minor league games in this dataset (14408 vs 3254), it may be possible that the model is biased towards the stats of those games, causing more error in the prediction of Major League games. Indeed, when comparing the accuracies of the model on the league tiers, we get different results. But was this difference just due to chance?

To answer this, I developed a **permutation test** to see if the difference in accuracy is significant. The following are my hypotheses:

- **Null Hypothesis**: The classifier's accuracy is the same for both Major and Minor Leagues, any differences are due to chance.

- **Alternative Hypothesis**: The classifier's accuracy is higher for Minor Leagues.

- **Test statistic**: Difference in accuracy (Minor minus Major)
- **Significance level**: 0.05

<iframe
  src="assets/fairness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

With a p-value of **0.008**, we **reject the Null Hypothesis** under our Significance level of **0.05**. There appears to be a difference in the accuracy in our model between Major and Minor leagues. This outcome implies that our model predicts outcomes more accurately for games in Minor Leagues than Major Leagues
