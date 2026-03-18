# League of Legends 15-Minute Statistical Analysis

Author: Annabelle Guiditta

## Introduction

**League of Legends** (also known as LoL), is a popular multiplayer online battle (MOBA) game developed and published by Riot Games, the creators of Valorant and Arcane. Well known as Riot's most sucessful game, LoL is world-known and renowned for its large Esports influence. The dataset this report will cover comes from [Oracle's Elixer](https://oracleselixir.com/about), a free to access collection of Datasets from the League competitive sphere since 2015. In particular, the dataset referenced in this report draws from the **2025 competitive season**.

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

- only keps rows with team in the position column, filtered to only needed columns
- converted results , , first pick to boolean types
- dropped columns that were marked as partial in the datacompleteness column
- converted date to datetime
- manually developed the major, minor, and league splits based on an initial 'league' column
- Developed the stat gold lead at 15, which identfied all rows that had a positive goldiffat15 for later analysis
- Missingness: identified missingness in split and firstpick. Interestingly, rows with missingness in first pick also contained missing values for all picks

Below is the head of the cleaned dataset:

<iframe
  src="assets/cleaned_head.html"
  width="800"
  height="300"
  frameborder="0"
></iframe>

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

To begin my bivariate analysis, I attempted to visualize some of the differences between Major and Minor Leagues.

<iframe
  src="assets/bivar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

To account for the difference in the number of Major vs Minor league games, I constructed a density histogram to fairly compare the distribution of `golddiffat15` across both tiers. Notably, the Major league distribution has a much sharper and higher peak, suggesting that gold differences in Tier 1 games are more tightly concentrated around 0. Major league games fall mostly between **-5,000 and 5,000 gold**, while Minor league games span a noticeably wider range of roughly **-7,000 to 7,000 gold**. This visual difference is supported by further calculations -- Minor league games exhibit **~73.7% more variance** in `golddiffat15` than Major league games, suggesting a more volatile early game landscape where leads are larger and less evenly matched. This sets the stage for our hypothesis test, where we formally investigate whether this difference is statistically significant.

### Interesting Aggregates

In investigating the importance of a gold lead early on in the game, I developed this grouped table to visualize its impact

## INSERT HERE

In particlar, it can be noted that teams that have a gold lead at 15 minutes tend to win more, and have higher xp, kills, assists, and creep score! In fact, **73% of teams** that had a gold lead at 15 minutes ended up winning the round

Additionally, I wanted to investigate the mean values across the different league tiers

## INSERT HERE

Notably, Major league games tend to have **lower average kills and assists** at 15 minutes compared to both Minor and International leagues. This aligns with the expectation that Tier 1 games are more defensive and methodical in the early game. Interestingly, Major leagues have a slightly **higher average CS** (534 vs 514), suggesting that while Major league teams fight less, they farm more efficiently.

## Assessment of Missingness

### MNAR Analysis

**Missing Not at Random (MNAR)** refers to missingness that depends on the unobserved value itself — for example, high earners being less likely to report their income on a survey precisely _because_ it is high.

In the Oracle's Elixir LoL dataset, a potential candidate for MNAR is the `ban` columns. Teams may strategically choose not to ban certain champions, and in some tournament formats or regions, bans may not be recorded precisely because no ban was made. However, it is difficult to confirm this without additional data — if we had access to **official tournament rulebooks or match metadata** indicating whether a ban slot was intentionally skipped vs simply not recorded, we could determine whether the missingness depends on the value itself or on an external observed variable like `league`, which would make it MAR instead.

That said, after thorough investigation, most missingness in this dataset appears to be either **Missing by Design** (such as `goldat15` being absent when a game ends before 15 minutes) or **MAR** dependent on observed columns like `date` or `league`. We could not conclusively identify any column that is definitively **MNAR**, as there is no strong evidence that the missing values depend on the values themselves rather than on other observed variables.

### MAR Analysis

**Missing at Random (MAR)** refers to missingness that is not based on the missing values themselves, but is related to another column in the dataset. For example, political preference of a survey being missing for participants of younger ages, where missingness is explained by their age -- not their actual preference

## INSERT HERE

<iframe
  src="assets/MARTrue.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

##more here

<iframe
  src="assets/MARFalse.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Hypothesis Testing

In the Oracle's Elixer LoL Dataset, each row has a column labeled **`league`**, denoting professional playing leagues that teams get matched and play in. Earlier in our cleaning step, we denoted a league as a "Major" league if it was from LCK, LEC, LCP, LTA N, LTA S, or LTA.

Tier 1 Professional leagues, such as those above, are seen as more competitive -- leading to highly technical games with close stats between teams. In my hypothesis test, I wish to discover whether games played at a Major Tier 1 Professional Leagues have smaller gold differences between teams than Minor Leagues do.

**Null Hypothesis**: Major (LCK, LEC, LCP, LTA N, LTA S, LTA) and minor leagues have the same mean absolute gold difference at 15 minutes and come from the same distribution

**Alternative Hypothesis**: Minor leagues have a higher mean absolute gold difference at 15 minutes than major leagues, thus coming from a different distribution

**Test Statistic**: Difference in absoluted gold differences between games in Major Leagues vs Minor Leagues

$$|\overline{\text{Gold Diff at 15 Minutes - Minor}}| - |\overline{\text{Gold Diff at 15 Minutes - Major}}|$$

**Significance Level**: 1%

As this hypothesis seeks to detemine whether two populations come from the same distribution, we used a permuation test to investigate this difference.

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

This leads us to our core prediction problem:

> _Can we predict match outcome from 15 minute stats alone?_

This asks whether the early game truly sets the course for the rest of the match -- or whether there is still significant room for comeback. If 15 minute stats are strong predictors of the final outcome, it suggests that early game advantages are decisive in professional play.

To address this question, we can the use of **binary classification** of `result` evaluated on accuracy as explored below

## Baseline Model

- Head of DF used
- Standardize all columns
- using just gold, xp, and cs information
- Use of train test split to test our data on itself
- Decision tree classifier, all of the features quantitative
- accuracy score

## Final Model

In my final model, I decided to switch to a **Random Forest Classifier**, an ensemble learning method that builds multiple decision trees during training and combines their predictions to produce a more accurate and stable result than any single tree alone. Additionally, I used **GridSearchCV** to identify the optimal hyperparameters for my model, specifically tuning

- **`max_depth`**: Controls how deep each tree can grow.
- **`n_estimators`**: Controls how many trees are in the forest. More trees generally improves stability and reduces variance, but with diminishing returns and increased computation time.

By tuning these two hyperparameters, I had hoped to address some of the key weaknesses of a single decision tree, such as **overfitting** and **high variance**, while finding the optimal balance between model complexity and generalization.

Additionally, I introduced more 15 minute stats, such as kills, deaths, and assists to help aid our prediction, as well as league tier for our later fairness analysis.

- Head of df
- kill diff feature killsat15 opp_killsat15
- 5 folds
- one hot
- accuracy

Despite introducing more models and attempting to minimize the error in the Decision Tree through using a Random Forest Model and testing HyperParameters, we don't see much of an improvement in the overall Test Score (73.1% vs 74.27%). Its possible that this is simply the best we can do with only the 15 minute stats in our dataset. Despite the small improvement, I'm happy with the result, as it shows how we can pretty accurately determine the outcome of a game on just the 15 minute statistics alone, with our error showing there is still room for comeback before then. Below, I've attached a confusion matrix to show how our model performed, noting our True Positives, True Negatives, False Positives, and False Negatives

## Fairness Analysis

---

Our analysis is guided by two core questions:

> **Hypothesis Test**: Do minor leagues have a significantly higher mean absolute gold difference at 15 minutes than major leagues?

This allows us to examine whether Tier 1 Major league games are more competitive and better balanced in early game than Minor league games, as measured by how close teams are in gold at the 15 minute mark.

> **Prediction**: Can we predict match outcome from 15 minute stats alone?

This asks whether the early game truly sets the course for the rest of the match, or whether there is still significant room for comeback. If 15 minute stats are strong predictors of the final outcome, it suggests that early game advantages are decisive in professional play.

Together, these questions paint a picture of how early game performance differs across leagues, and how much it ultimately matters.

<iframe
  src="assets/fairness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
