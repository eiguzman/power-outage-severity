# power-outage-severity
A UCSD-DSC80 Project

Edgar Guzman

# Introduction

This report exampines a dataset of reported power outages in the United States from the year 2000 to 2016. We will reveal patterns, trends, and the limitations of this dataset as well as produce a predictive model to determine if the duration of a power outage can be classified as 'severe'. This dataset was available as an .xlsx file in Purdue University Laboratory for Advancing Sustainable Critical Infrastructure's website

**The Hypothesis question we have chosen to answer is**: "Is there a correlation between power outage duration based on climate category?" \
**The prediction question we have chosen to answer is**: "Can we predict power outage severity (whether an outage lasts more than 1000 minutes) based on customer category"

# Data Cleaning and Exploratory Data Analysis

Before we can perform our Exploratory Data Analysis, we must first clean our data into a readable and fully usable DataFrame. Each step is explained below. \
First, we read our Excel file into a DataFrame using `Pandas` built-in function `read_excel`. The first rows are descriptions of the dataset and therefore can be dropped.

Next, we want to merge the columns `OUTAGE.XX.DATE` and `OUTAGE.XX.TIME` into a single column. To do this, we must first convert the DATE columns  into the dtype `pd.datetime`, and the TIME columns into th dtype `pd.timedelta`. Then, we can perform simple addition to add the DATEs and TIMEs together into their own column. We will name these new columns `OUTAGE.START` and `OUTAGE.RESTORATION`, dropping the unmerged columns in the process.

Next, we will keep only the columns that will be used in our EDA, Hypothesis tests, and our predictive analysis.

Finally, we will add a final column `OVER.1000`, a boolean that determines whether the outage lasted longer than 1000 minutes. This column will be interpreted as 0 for `False` and 1 for `True`

For the purposes of our predictive analysis, we will also fill `NaNs` for certain columns here. Explanantions will be skipped for the sake of keeping this section clean with relevant information. Most fills are performed using mean imputation based on category.

We will also impute `DEMAND.LOSS.MW` at a later time, as its missingness will be analyzed in a section below.

### EDA: Univariate Analysis

We will perform univariate analyses on a few of our variables. First, we will determine the distribution of `OUTAGE.DURATION`

Based on the plot above, most of the outages are concentrated at `OUTAGE.DURATION` less than 1000 minutes. There seems to be a large empty space in this plot, but that is because the plot is also showing the outliers; the few outages that lasted loner than 50,000 minutes. \
The data is non-uniform, which means we will have to perform non-parametric tests. This means that we are unable to perform any Hypothesis test that assumes normality in the population distribution unless we normalized the observations. What if we were to separate them based on climate category?

-----Plot here

Looking at this plot, we start to see a pattern in the data that we will discuss later. Let's attempt to transform the data distribution. We will perform the following steps:
* Log-scale `OUTAGE.DURATION`
* Seperate the distributions based on `CLIMATE.CATEGORY`
* Normalize each category by probability density.

Here we can notice a few trends:
* The data is still not normally distributed, but it is close enough to normal that we can perform some parametric tests
* There are few extreme variations within each distribution
* There are fewer observations for the "normal" category than the "cold" and "warm" categories

Unfortunately, these distributions are not clone enough to normality for a p-value to be significant; therefore, we will run non-parametric tests on the original distributions for `OUTAGE.DURATION`.

Next, we will determine if the total number of `CUSTOMERS.AFFECTED` has changed over time

-----Plot here

Based on the plot above, we can determine that the number of customers gradually increases, peaking in 2012, and slowly declines after that. Prior to imputation, the `YEAR` 2012 had less `CUSTOMERS.AFFECTED` than 2008. 

It is worth mentioning that the number of custmers affected during the year 2016 is lower than other datasets report, as `outage.xlsx` only reports observations up until July 2016. Had we used a more up-to-date dataset, the numbers would have likely plateaued or increased compared to 2015 as energy consumption increases.

### EDA: Bivariate Analysis

For the second part of our Exploratory Data Analysis, we will examine if any relationship occurrs between `CUSTOMERS.AFFECTED` and `OUTAGE.DURATION`. Additionally, we will categorize the data points based on `CLIATE.CATEGORY` to determine if outliers are more concentrated within a certain category.

Based on the plot above, The majority of data is clustered between 0 and 500,000 `CUSTOMERS.AFFECTED` and within 0 and 2500 `DEMAND.LOSS.MW`. This distribution is validated by referring back to the univariate distribution of `OUTAGE.DURATION`; plotting the individual counts of `CUSTOMERS.AFFECTED` or `DEMAND.LOSS.MW` will reveal a similarly distributed population as before. 

Next, let's look at the total `COM.SALES` -the total commercial energy sales- categorized by state using `POSTAL.CODE`

Unfortunately, this plot does not tell us anything valuable that a population plot could not. Instead, let's plot based on `COM.PERCEN`, the total commercial sales as a percentage of total sales based on the respective state.

-----Plot here

This plot is very interesting! Why would the District of Columbia have such high commercial energy usage? Looking once again at the data, we can deduce two new pieces of information:
* For all states: `RES.CUSTOMERS` > `COM.CUSTOMERS` > `IND.CUSTOMERS`
* `DC` is the only place to have a disproportionately low amount of `IND.CUSTOMERS`, with only 1!
    * Additionally, `DC` has the lowest ratio of `RES.CUSTOMERS` to `COM.CUSTOMERS`
This means that while the most states have many `RES.CUSTOMERS` and few `COM.CUSTOMERS`, The number of residential and commercial customers in `DC` is relatively close.

All of these discoveries verify D.C.'s high commercial percentage.

To add a bit of domain knowledge: `DC` contains the most data centers out of any region in the United States. California is second region. It makes sense as to why the District of Columbia consumes a larger share relative to individual households.

### EDA: Aggregate Data

While we have aggregated data based on Univariate and Bivariate Analyses in the sections above, we have not yet shown how to deal with missing values by grouping. Below we will group `CUSTOMERS.AFFECTED` and `TOTAL.CUSTOMERS` based on State using `POSTAL.CODE`

There are 2 missing values for `CUSTOERS.AFFECTED` in this grouped DataFrame: 1 in Montana and 1 in South Dakota. Unfortunately this means that we can't tell how many people were affected, as `NaNs` do not correspond to 0 customers affected.

As such, an ideal approach to determining how many people were affected is to look at the **mean percentage of `TOTAL.CUSTOMERS` affected by power outages**, then divide Montana and South Dakotas population by this proportion.

Roughly 4.7% of all customers in are affected by power outages. Not a bad loss compared to the millions of customers. Now we will take the `TOTAL.CUSTOMERS` of Montana and South Dakota and multiply it by the affected_pct.

# Assessment of Missingness

There are a few rows in our DataFrame that are consistently missing values. Let's take a look at which values those are.

As we can see from the list above, we can see that there is are 11 columns (Excluding `HURRICANE.NAMES`) where the values are always missing: `RES.PRICE`, `COM.PRICE`, `IND.PRICE`, `TOTAL.PRICE`, `RES.SALES`, `COM.SALES`, `IND.SALES`, `TOTAL.SALES`, `RES.PERCENT`, `COM.PERCEN`, and `IND.PERCEN`. Since these are all missing in the same rows in our DataFrame, we can conclude that they are all **Not Missing at Random**. This is because although we can find a slight pattern in missingness based on `YEAR`, we are unable to find a specific pattern for all 22 missing rows. 

There could potentially be factors that attribute to all 11 variables resulting in missingness that were not measured by the creators of the dataset. Potential explanations as to why each of these columns are missing is likely due to these datapoints coming from the same source; one that does not generate data with electricity prices or sales. Another explanation could be that since most of these datapoints are from the year 2000, the variables observed for each datapoint were less complex, and more variables were added as tools became more commonplace.

### Missingness Dependency

In order to determine missingness dependency of `DEMAND.LOSS.MW`, we are going to compare the distributions based on two categories: `YEAR` and `CLIMATE.CATEGORY`

The first step is to create a `pivot_table` of our DataFrame, whose index is `YEAR` and columns correspond to `MISSING`

Since our columns are **categorical**, we find the observed Total Variation Distance of this pivot_table as our t-statistic

Below is a plot showing the distribution of `DEMAND.LOSS.MW` based on `MISSINGNESS`

-----Plot here

In order to determine whether our observed TVD follows the population distribution, we will perform the following test:

**Null Hypothesis**: The distribution of `YEAR` is the same regardless of missing data for `DEMAND.LOSS.MW` \
**Alternate Hypothesis**: The distributions for missing data are different than the distributions of non-missing data

To simulate population data, we will perform 500 repetitions of shuffling the `MISSING` column, and calculating the TVD for each.

Our observed TVD had a value of .288, with a pvalue of 0. Below is the plot of the empirical distribution, with the observed_tvd marked as a heavy outlier. 

As such, we reject the null hypothesis; the distributions of data are likely to depend on the `YEAR` column

Next, we will determine if the distributions of `DEMAND.LOSS.MW` are dependent on `CLIMATE.CATEGORY` when data are missing. Most of the steps are the same as the first attempt, so explanations will be skipped.

Null Hypothesis: The distribution of `CLIMATE.CATEGORY` is the same regardless of missing data for `DEMAND.LOSS.MW` \
Alternate Hypothesis: The distributions for missing data are different than the distributions of non-missing data

-----Plot here

-----Plot here

Based on the following hypothesis test, we have observed a TVD of 0.03 with a p-value of .324

We fail to reject the null hypothesis this time; however, this does not mean that the null hypothesis must be true. There is a possibility that the data come from the same distribution, but there may be other factors that we have not observed or discovered that may cause this.

Now we can fill the missing values of `DEMAND.LOSS.MW` through grouped mean imputation.

# Hypothesis Testing

For our first hypothesis test, we will take a look back at the distributions of `OUTAGE.DURATION`

As we stated in our Exploratory Data Analysis section, running hypothesis testing on the log-scaled distribution of outages will determine p-values vased on the logged values; these p-values are magnified when the data is scaled back, often resulting in erroneous results based on our confidence interval.

Since the observations are not normally distributed, it is better to use a non-parametric test instead of log-scaling the data. We will perform the Mann-Whitney U Test on our `OUTAGE.DURATION` observations grouped by `CLIMATE.CATEGORY`. The Mann-Whitney U Test, also known as the Wilcoxon Rank-Sum Test, determines central tendency of the sample distributions without the requirement of distribution normality.

**Hypothesis Test:** Are the population distributions of `OUTAGE.DURATION` based on `CLIMATE.CATEGORY` derived from the same distribution? \
**Null Hypothesis:** The central tendency between distributions of `OUTAGE.DURATION` based on `CLIMATE.CATEGORY` are the same. \
**Alternate Hypothesis:** The distributions of each population are not equal.

This hypothesis test will use the difference in means as its t-statistic. We will use an alpha value of .01, the result our p-value being based off a 99% confidence interval.

Since there are three categories, we will perform 3 tests based on each permutation of climate categories.

The results of our hypothesis testing reveals a few notable features of our distributions:
* Each permutation has a pvalue greater than .01; using a 99% confidence interval, the tvalues will not be significant.
* Due to the p-value of .35, the distributions of cold and warm observations of `OUTAGE.DURATION` are likely the closest permutation to come from the same distribution.

Based on the more reliable test, we fail to reject the null hypothesis for all categories. Although this does not mean the null hypothesis is true, it is more likely that the categories come from similar distributions since the null hypothesis was not rejected.

Since the pvalue was greatest between "cold" and "warm" observations, we can assume that there is a greater chance that both of these observations come from the same distribution. Let's determine if the distributions still hold true when accounting for a secondary variable: If the `CAUSE.CATEGORY` was an intentional attack

We will also add a new row `total` that gives us the normalized distribution of overall intentional attacks

Using the pivot table above, We will run a Chi-squared test for homogeneity

**Hypothesis Test:** Is there a correlation between the distribution of intentional attacks and climate category? \
**Null Hypothesis:** There is no statistical difference between intentional attacks and climate category; in other words, intentional attacks come from the same distribution \
**Alternate Hypothesis:** Intentional Attack levels come from different distributions.

Our hypothesis test will use the Chi-squared critical value as out test statistic. We will continue to use an alpha value of .01, basing our pvalue off a 99% confidence interval.

We observed a test statistic of .945, with a p-value of 0.33. As the pvalue was greater than our alpha value, we fail to reject the null hypothesis. Although it is not guaranteed that the distributions of intentional attacks based on climate category come from the same distribution, the p-value is far enough away from the alpha value that there is a very likely chance that they do.

To obtain a plot showing the distributions of Chi-Squared test statistics, we will implement the following code below:

-----Plot here

# Framing a Prediction Problem

The model that I will generate will attempt to predict `OVER.1000`. As roughly half of all power outages have an `OUTAGE.DURATION` less than 1000, A model that can predict whether an outage will be severe is a reliable metric to determine model viability.

Since there are only two options, 0 or 1, this will be a binary classification. The metric we will be using is the F1 score, as it is important to have both high precision and recall. Our baseline model will use 3 features:

| Feature | Variable Type |
| -------- | ------- |
| POSTAL.CODE | Nominal |
| ANOMALY.LEVEL | Quntitative |
| TOTAL.CUSTOMERS | Quantitative |
| POPDEN_URBAN | Quantitative |

**NOTE**: We will not use `OUTAGE.DURATION` as a predictor. As `OVER.1000` is a transformation of that column, there is a 100% accuracy to predict `OVER.1000` using only `OUTAGE.DURATION`. We are interested in determining accuracy based on other columns.

# Baseline Model

before we can fit a prediction model onto our data. we must first split the data into training and test groups. We will obtain our results by using:
* `sklearn.train_test.split`
* fitting the data into a `Pipeline`
* One-Hot Encoding `POSTAL.CODE`
* fitting the data into a `DecisionTreeClassifier`

Without any special parameters, out model resulted in an F1-score of `.652` with an `accuracy_score` of `.661`. These preliminary results are favorable, as our predictions are more accurate than 50%, the baseline for bivariate randomness. One thing to note is that this model is heavily overfitted to the training data. 

# Final Model

We will now improve upon our baseline model by adding more features and finding the most optimal hyperparameters using `GridSearchCV`. We will engineer 3 new features from the provided columns:
* The proportion of customers affected over all customers
* The proportion of customers affected over the state population
* The `intensity` (demand loss multplied by weather anomaly levels)

Next, we will fit our baseline model using the optimal parameters.

Our Baseline model's performance resulted in an F1 score of `0.681`, with an `accuracy_score` of `.708`. This models shows a slight improvement over the baseline model; however, the minimal improvement could be interpreted as negligible if the baseline model is already a very powerful predictor while being simpler.

# Fairness Analysis

For my fairness analysis, I will group by outages whose majority consumer was `Industrial` customers. To determine which customer type is the majrity consumer, we compare `IND.PERCEN` to `RES.PERCEN` and `COM.PERCEN`. iF `IND.PERCEN` is the highest number between the three groups, `IND.MAJ` is `1`; if `IND.PERCEN` is not the sole highest consumer, `IND.MAJ` is `0`.

Why do we not use `IND.CUST.PCT`? This is because Residential households are plentiful, but use less energy per household; likewise, Commercial and Industrial customers are fewer, but consume more energy to keep a business running. By looking at the total share of energy usage, we can see who was the biggest consumer during an outage, and who contributes more to longer power outages.

Although these numbers are both below the 50% mark, they do not deviate significantly when compared to one another. As such, we will state that $C$ achieves demographic parity.

Based on our test observations, when Industrial customers are not the majority, there is a greater share of power outages that are 1000 minutes or less.

Perhaps if the categories were grouped, we would see numbers that are closer to each other. Let's findthe `accuracy` of $C$ for each group.

After computing the accuracy scores based on the groups, we see a greater difference of `0.11` between the two groups. We will now run a permutation test in order to determine the significance of our accuracy.

**Null Hypothesis**: There is no significant deviation based on accuracy based on majority consumer. \
**Alternative Hypothesis**: The classifier is not the same when `Industry` is the majority consumer. \
**Test Statistic**: Difference in accuracies based on group. \
We will run ths test under a 99% condifence interval.

-----Plot here

It turns out that our observed statistic of `.108` does not deviate significantly from the permutation distribution. As such, we fail to reject our null hypothesis.

Our model does not perform worse for outages caused by different majority energy consumers; therefore, we can say that our model is fair when representing energy consumers based on category.