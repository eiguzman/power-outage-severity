# A Predictive Analysis of U.S. Power Outage Severity
A UCSD-DSC80 Project

-Edgar Guzman

# Introduction

This report examines a dataset of reported power outages in the United States from the year 2000 to 2016. We will reveal patterns, trends, and the limitations of this dataset as well as produce a predictive model to determine if the duration of a power outage can be classified as 'severe'. This dataset was available as an .xlsx file in Purdue University Laboratory for Advancing Sustainable Critical Infrastructure's website.

Throughout the scopre of this project, we will answer questions that will guide us closer to the general hypothesis question: What is the correlation between the variables in our dataset and power outage duration? Is there an efficient way to predict the length -or severity- of a power outage based on these variables? The results of these questions can guide regional electric companies to coordinate with local governments to facilitate relief efforts, prioritize severe outage risks, and open up a path to reduce the frequency of outages.

The dataset we will use throughout this project contains 1536 outage observations 56 variables. The variables we are interested in using are described below:

| Column | Description |
| -------- | ------- |
|YEAR| Year the outage occurred | 
|POSTAL.CODE| Postal code of the U.S. State | 
|ANOMALY.LEVEL| Oceanic El Niño/La Niña Index (OEI) | 
|CLIMATE.CATEGORY| Climate eisodes throughout the year |
|CAUSE.CATEGORY| Categories of power outage events| 
|OUTAGE.DURATION| Outage duration in minutes | 
|DEMAND.LOSS.MW| Peak demand lost in Megawatts|
|CUSTOMERS.AFFECTED| Number of customers affected | 
|COM.SALES| Commercial sector consumption (in megawatt-hours) | 
|TOTAL.SALES| Total sector consumption (in megawatt-hours) |
|RES.PERCEN| Percentage of residential consumption compared to total consumption |
|COM.PERCEN| Percentage of commercial consumption compared to total consumption | 
|IND.PERCEN| Percentage of industrial consumption compared to total consumption | 
|TOTAL.CUSTOMERS| Annual number of total customers by state | 
|POPULATION| Annual population by state |
|POPDEN_URBAN| Population density of the urban area (persons per square mile) |

# Data Cleaning and Exploratory Data Analysis

Before we can perform our Exploratory Data Analysis, we must first clean our data into a readable and fully usable DataFrame. Each step is explained below.
1. First, we read our Excel file into a DataFrame using `Pandas` built-in function `read_excel`. The first rows are descriptions of the dataset and therefore can be dropped.
2. Next, we want to merge the columns `OUTAGE.XX.DATE` and `OUTAGE.XX.TIME` into a single column. To do this, we must first convert the DATE columns into the dtype `pd.datetime`, and the TIME columns into the dtype `pd.timedelta`. Then, we can perform simple addition to add the DATEs and TIMEs together into their own column. We will name these new columns `OUTAGE.START` and `OUTAGE.RESTORATION`, dropping the unmerged columns in the process.
3. Next, we will filter out the columns that will not be used in our EDA, Hypothesis tests, or our predictive analysis.
4. Finally, we will add a final column `OVER.1000`, a Boolean that determines whether the outage lasted longer than 1000 minutes. This column will be interpreted as 0 for `False` and 1 for `True`

For the purposes of our predictive analysis, we will also fill `NaNs` for all of the selected columns, unless empty values are required for the purposes of missingness assessments. Explanations will be skipped for the sake of keeping this section clean with relevant information. Most `fill` functions are performed using mean imputation based on category.

For example: `DEMAND.LOSS.MW` will be imputed later, as its missingness will be analyzed in a section below.

The first rows of our DataFrame are shown below:

| POSTAL.CODE   |   OBS |   YEAR |   ANOMALY.LEVEL | CLIMATE.CATEGORY   | CAUSE.CATEGORY     |   OUTAGE.DURATION |   DEMAND.LOSS.MW |   CUSTOMERS.AFFECTED |   COM.SALES |   TOTAL.SALES |   RES.PERCEN |   COM.PERCEN |   IND.PERCEN |   TOTAL.CUSTOMERS |   POPULATION |   POPDEN_URBAN |   OVER.1000 |
|:--------------|------:|-------:|----------------:|:-------------------|:-------------------|------------------:|-----------------:|---------------------:|------------:|--------------:|-------------:|-------------:|-------------:|------------------:|-------------:|---------------:|------------:|
| MN            |     1 |   2011 |            -0.3 | normal             | severe weather     |              3060 |              nan |                70000 | 2.11477e+06 |   6.56252e+06 |      35.5491 |      32.225  |      32.2024 |           2595696 |      5348119 |           2279 |           1 |
| MN            |     2 |   2014 |            -0.1 | normal             | intentional attack |                 1 |              nan |               124007 | 1.80776e+06 |   5.28423e+06 |      30.0325 |      34.2104 |      35.7276 |           2640737 |      5457125 |           2279 |           0 |
| MN            |     3 |   2010 |            -1.5 | cold               | severe weather     |              3000 |              nan |                70000 | 1.80168e+06 |   5.22212e+06 |      28.0977 |      34.501  |      37.366  |           2586905 |      5310903 |           2279 |           1 |
| MN            |     4 |   2012 |            -0.1 | normal             | severe weather     |              2550 |              nan |                68200 | 1.94117e+06 |   5.78706e+06 |      31.9941 |      33.5433 |      34.4393 |           2606813 |      5380443 |           2279 |           1 |
| MN            |     5 |   2015 |             1.2 | warm               | severe weather     |              1740 |              250 |               250000 | 2.16161e+06 |   5.97034e+06 |      33.9826 |      36.2059 |      29.7795 |           2673531 |      5489594 |           2279 |           1 |

### EDA: Univariate Analysis

We will perform univariate analyses on a few of our variables. First, we will determine the distribution of `OUTAGE.DURATION` over time.

<iframe
  src="assets/Univariate_1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Based on the plot above, most outages are concentrated at `OUTAGE.DURATION` less than 1000 minutes. There seems to be a large empty space in this plot, but that is because the plot is also showing the outliers; the few outages that lasted longer than 50,000 minutes. \
The data is non-uniform, which means we will have to perform non-parametric tests. This means that we are unable to perform any hypothesis test that assumes normality in the population distribution unless we normalize the observations. What if we were to separate them based on climate category?

<iframe
  src="assets/Univariate_2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Looking at this plot, we start to see a pattern in the data that we will discuss later. Let's attempt to transform the data distribution. We will perform the following steps:
* Log-scale `OUTAGE.DURATION`
* Separate the distributions based on `CLIMATE.CATEGORY`
* Normalize each category by probability density.

<iframe
  src="assets/Univariate_3.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Here we can notice a few trends:
* The data is still not normally distributed, but it is close enough to normal that we can perform some parametric tests
* There are few extreme variations within each distribution
* There are fewer observations for the "normal" category than the "cold" and "warm" categories

Unfortunately, these distributions are not clone enough to normality for a p-value to be significant; therefore, we will run non-parametric tests on the original distributions for `OUTAGE.DURATION`.

Next, we will determine if the total number of `CUSTOMERS.AFFECTED` has changed over time.

<iframe
  src="assets/Univariate_4.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Based on the plot above, we can determine that the number of customers gradually increases, peaking in 2012, and slowly declines after that. Prior to imputation, the `YEAR` 2012 had less `CUSTOMERS.AFFECTED` than 2008. 

It is worth mentioning that the number of customers affected during the year 2016 is lower than other datasets report, as `outage.xlsx` only reports observations up until July 2016. Had we used a more up-to-date dataset, the numbers would have plateaued or increased compared to 2015 as energy consumption increases.

### EDA: Bivariate Analysis

For the second part of our Exploratory Data Analysis, we will examine if any relationship occurs between `CUSTOMERS.AFFECTED` and `OUTAGE.DURATION`. Additionally, we will categorize the data points based on `CLIATE.CATEGORY` to determine if outliers are more concentrated within a certain category.

<iframe
  src="assets/Bivariate_1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Based on the plot above, most data are clustered between 0 and 500,000 `CUSTOMERS.AFFECTED` and within 0 and 2500 `DEMAND.LOSS.MW`. This distribution is validated by referring to the univariate distribution of `OUTAGE.DURATION`; plotting the individual counts of `CUSTOMERS.AFFECTED` or `DEMAND.LOSS.MW` will reveal a similarly distributed population as before. 

Next, let's look at the total `COM.SALES` -the total commercial energy sales- categorized by state using `POSTAL.CODE`

<iframe
  src="assets/Bivariate_2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Unfortunately, this plot does not tell us anything valuable that a population plot could not. Instead, let's plot based on `COM.PERCEN`, the total commercial sales as a percentage of total sales based on the respective state.


<iframe
  src="assets/Bivariate_3.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This plot is remarkably interesting! Why would the District of Columbia have such high commercial energy usage? Looking once again at the data, we can deduce two new pieces of information:
* For all states: `RES.CUSTOMERS` > `COM.CUSTOMERS` > `IND.CUSTOMERS`
* `DC` is the only place to have a disproportionately small number of `IND.CUSTOMERS`, with only one!
    * Additionally, `DC` has the lowest ratio of `RES.CUSTOMERS` to `COM.CUSTOMERS`
This means that while the most states have many `RES.CUSTOMERS` and few `COM.CUSTOMERS`, The number of residential and commercial customers in `DC` is not far apart. All these discoveries verify D.C.'s high commercial percentage.

To add a bit of domain knowledge: Washington, D.C. and its surrounding regions contain the most data centers out of any region in the United States. California is the second region. It makes sense as to why the District of Columbia consumes a larger share relative to individual households.

### EDA: Aggregate Data

While we have aggregated data based on Univariate and Bivariate Analyses in the sections above, we have not yet shown how to deal with missing values by grouping. Below we will group `CUSTOMERS.AFFECTED` and `TOTAL.CUSTOMERS` based on State using `POSTAL.CODE`

| POSTAL.CODE   |   CUSTOMERS.AFFECTED |   TOTAL.CUSTOMERS |
|:--------------|---------------------:|------------------:|
| AK            |              14273   |  273530           |
| AL            |              94328.8 |       2.40059e+06 |
| AR            |              47673.8 |       1.53684e+06 |
| AZ            |              64402.7 |       2.79737e+06 |
| CA            |             201366   |       1.47257e+07 |

There are two missing values for `CUSTOERS.AFFECTED` in this grouped DataFrame: one in Montana and one in South Dakota. Unfortunately, this means that we cannot tell how many people were affected, as `NaNs` do not correspond to 0 customers affected.

As such, an ideal approach to determining how many people were affected is to look at the **mean percentage of `TOTAL.CUSTOMERS` affected by power outages**, then divide Montana and South Dakotas population by this proportion. Roughly 4.7% of all customers in the United States are affected by power outages. Not a bad loss compared to the millions of customers. Now we will take the `TOTAL.CUSTOMERS` of Montana and South Dakota and multiply it by the affected_pct.

# Assessment of Missingness

### NMAR Analysis

There are a few rows in our DataFrame that are consistently missing values. Let's look at which values those are.

|   RES.PRICE |   COM.PRICE |   IND.PRICE |   TOTAL.PRICE |   RES.SALES |   COM.SALES |   IND.SALES |   TOTAL.SALES |   RES.PERCEN |   COM.PERCEN |   IND.PERCEN |
|------------:|------------:|------------:|--------------:|------------:|------------:|------------:|--------------:|-------------:|-------------:|-------------:|
|         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |

As we can see from the list above, we can see that there is are 11 columns (Excluding `HURRICANE.NAMES`) where the values are always missing: `RES.PRICE`, `COM.PRICE`, `IND.PRICE`, `TOTAL.PRICE`, `RES.SALES`, `COM.SALES`, `IND.SALES`, `TOTAL.SALES`, `RES.PERCENT`, `COM.PERCEN`, and `IND.PERCEN`. Since these are all missing in the same rows in our DataFrame, we can conclude that they are all **Not Missing at Random**. This is because although we can find a slight pattern in missingness based on `YEAR`, we are unable to find a specific pattern for all 22 missing rows. 

There could potentially be factors that attribute to all 11 variables resulting in missingness that were not measured by the creators of the dataset.  A potential explanation as to why each of these columns is missing is due to these datapoints coming from the same source; one that does not generate data with electricity prices or sales. Another explanation could be that since most of these datapoints are from the year 2000, the variables observed for each datapoint were less complex, and more variables were added as tools became more commonplace.

### Missingness Dependency

To determine missingness dependency of `DEMAND.LOSS.MW`, we are going to compare the distributions based on two categories: `YEAR` and `CLIMATE.CATEGORY`. \
The first step is to create a `pivot_table` of our DataFrame, whose index is `YEAR` and columns correspond to `MISSING`. Since our columns are **categorical**, we find the observed Total Variation Distance of this pivot_table as our t-statistic. Below is a plot showing the distribution of `DEMAND.LOSS.MW` based on `MISSINGNESS`.

<iframe
  src="assets/Missingness_1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

In order to determine whether our observed TVD follows the population distribution, we will perform the following test:

**Null Hypothesis**: The distribution of `YEAR` is the same regardless of missing data for `DEMAND.LOSS.MW`. \
**Alternate Hypothesis**: The distributions for missing data are different than the distributions of non-missing data.

To simulate population data, we will perform 500 repetitions of shuffling the `MISSING` column and calculating the TVD for each.

Our observed TVD had a value of .288, with a p-value of 0. Below is the plot of the empirical distribution, with the observed_tvd marked as a heavy outlier. 

<iframe
  src="assets/Missingness_2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

As such, we reject the null hypothesis; the distributions of data are likely to depend on the `YEAR` column.

Next, we will determine if the distributions of `DEMAND.LOSS.MW` are dependent on `CLIMATE.CATEGORY` when data are missing. Most of the steps are the same as the first attempt, so explanations will be skipped.

**Null Hypothesis**: The distribution of `CLIMATE.CATEGORY` is the same regardless of missing data for `DEMAND.LOSS.MW` \
**Alternate Hypothesis**: The distributions for missing data are different than the distributions of non-missing data. Below is a plot showcasing the proportion of climate categories based on demand loss missingness.

<iframe
  src="assets/Missingness_3.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This table resulted in an observed Total Variation Distance of `0.03`. We then simulated permutation of the missingness 500 times. Below we plot the empirical distribution of the TVDs.

<iframe
  src="assets/Missingness_4.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Based on the following hypothesis test, we have observed a TVD of `0.03` with a p-value of `.324`.

We fail to reject the null hypothesis this time; however, this does not mean that the null hypothesis must be true. There is a possibility that the data come from the same distribution, but there may be other factors that we have not observed or discovered that may cause this.

Now we can fill in the missing values of `DEMAND.LOSS.MW` through grouped mean imputation.

# Hypothesis Testing

For our first hypothesis test, we will look back at the distributions of `OUTAGE.DURATION`

As we stated in our Exploratory Data Analysis section, running hypothesis testing on the log-scaled distribution of outages will determine p-values based on the logged values; these p-values are magnified when the data is scaled back, often resulting in erroneous results based on our confidence interval.

Since the observations are not normally distributed, it is better to use a non-parametric test instead of log-scaling the data. We will perform the Mann-Whitney U Test on our `OUTAGE.DURATION` observations grouped by `CLIMATE.CATEGORY`. The Mann-Whitney U Test, also known as the Wilcoxon Rank-Sum Test, determines central tendency of the sample distributions without the requirement of distribution normality.

**Hypothesis Test:** Are the population distributions of `OUTAGE.DURATION` based on `CLIMATE.CATEGORY` derived from the same distribution? \
**Null Hypothesis:** The central tendency between distributions of `OUTAGE.DURATION` based on `CLIMATE.CATEGORY` are the same. \
**Alternate Hypothesis:** The distributions of each population are not equal.

This hypothesis test will use the difference in means as its t-statistic. We will use an alpha value of .01, the result our p-value being based off a 99% confidence interval.

Since there are three categories, we will perform three tests based on each permutation of climate categories. The following results are written below:
>tval, pval of normal vs cold:  (170510.5, 0.20935450193362237)
>tval, pval of normal vs warm:  (105717.5, 0.023713467761876987)
>tval, pval of cold vs warm:  (69985.0, 0.35384002311582785)

The results of our hypothesis testing reveal some notable features of our distributions:
* Each permutation has a p-value greater than .01; using a 99% confidence interval, the test statistic will not be significant.
* Due to the p-value of .35, the distributions of cold and warm observations of `OUTAGE.DURATION` are likely the closest permutation to come from the same distribution.

Based on this reliable test, we fail to reject the null hypothesis for all categories. Although this does not mean the null hypothesis is true, it is more likely that the categories come from similar distributions since the null hypothesis was not rejected.

Since the p-value was greatest between "cold" and "warm" observations, we can assume that there is a greater chance that both observations come from the same distribution. Let's determine if the distributions still hold true when accounting for a secondary variable: If the `CAUSE.CATEGORY` was an intentional attack.

We will also add a new row `total` that gives us the normalized distribution of overall intentional attacks. The `pivot_table` is shown below.

| CLIMATE.CATEGORY   |    False |     True |
|:-------------------|---------:|---------:|
| cold               | 0.742072 | 0.257928 |
| warm               | 0.772727 | 0.227273 |
| total              | 0.754161 | 0.245839 |

Using the pivot table above, we will run a Chi-squared test for homogeneity.

**Hypothesis Test:** Is there a correlation between the distribution of intentional attacks and climate category? \
**Null Hypothesis:** There is no statistical difference between intentional attacks and climate category; in other words, intentional attacks come from the same distribution \
**Alternate Hypothesis:** Intentional Attack levels come from different distributions.

Our hypothesis test will use the Chi-squared critical value as our test statistic. We will continue to use an alpha value of .01, basing our pvalue off a 99% confidence interval.

We observed a test statistic of .945, with a p-value of 0.33. As the p-value was greater than our alpha value, we fail to reject the null hypothesis. Although it is not guaranteed that the distributions of intentional attacks based on climate category come from the same distribution, the p-value is far enough away from the alpha value that there is a highly likely chance that they do.

To obtain a plot showing the distributions of Chi-Squared test statistics, we generated 500 permutations of the distributions, created new pivot tables, and performed the Chi-squared test for the resulting pvalues. The results are shown below.

<iframe
  src="assets/Hypothesis_1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

# Framing a Prediction Problem

The model that I will generate will attempt to predict `OVER.1000`. As roughly half of all power outages have an `OUTAGE.DURATION` less than 1000, A model that can predict whether an outage will be severe is a reliable metric to determine model viability.

Since there are only two options, 0 or 1, this will be a binary classification. The metric we will be using is the F1 score, as it is important to have both high precision and recall. Our baseline model will use four features:

| Feature | Variable Type |
| -------- | ------- |
| POSTAL.CODE | Nominal |
| ANOMALY.LEVEL | Quantitative |
| TOTAL.CUSTOMERS | Quantitative |
| POPDEN_URBAN | Quantitative |

**NOTE**: We will not use `OUTAGE.DURATION` as a predictor. As `OVER.1000` is a transformation of that column, there is a 100% accuracy to predict `OVER.1000` using only `OUTAGE.DURATION`. We are interested in determining accuracy based on other columns.

# Baseline Model

Before we can fit a prediction model onto our data, we must first split the data into training and test groups. We will obtain our results by using:
* `sklearn.train_test.split`
* Fitting the data into a `Pipeline`
* One-Hot Encoding `POSTAL.CODE`
* Fitting the data into a `DecisionTreeClassifier`

Without any special parameters, our model resulted in an F1-score of `.661` with an `accuracy_score` of `.671`. These preliminary results are favorable, as our predictions are more accurate than 50%, the baseline for bivariate randomness. One thing to note is that this model is heavily overfitted to the training data. 

# Final Model

We will now improve upon our baseline model by adding more features and finding the most optimal hyperparameters using `GridSearchCV`. We will engineer three new features from the provided columns:
* The proportion of customers affected over all customers
* The proportion of customers affected over the state population
* The `intensity` (demand loss multiplied by weather anomaly levels)

Next, we will fit our baseline model using the optimal parameters.

Finally, we will predict the values of `OVER.1000`, fit the values into a confusion matrix, and determine the F1-score to analyze our model's true performance.

Our Baseline model's performance resulted in an F1 score of `0.678`, with an `accuracy_score` of `.705`. This model shows a slight improvement over the baseline model; however, the minimal improvement could be interpreted as negligible if the baseline model is already a powerful predictor while being simpler.

# Fairness Analysis

For my fairness analysis, I will group by outages whose majority consumer was `Industrial` customers. To determine which customer type is the majority consumer, we compare `IND.PERCEN` to `RES.PERCEN` and `COM.PERCEN`. If `IND.PERCEN` is the highest number between the three groups, `IND.MAJ` is `1`; if `IND.PERCEN` is not the sole highest consumer, `IND.MAJ` is `0`.

Why do we not use `IND.CUST.PCT`? This is because `Residential` households are plentiful, but use less energy per household; likewise, `Commercial` and `Industrial` customers are fewer but consume more energy to keep a business running. By looking at the total share of energy usage, we can see who the biggest consumer during an outage was, and who contributes more to longer power outages.

Although these numbers are both below the 50% mark, they do not deviate significantly when compared to one another. As such, we will state that $C$ achieves demographic parity.

Based on our test observations, when Industrial customers are not the majority, there is a greater share of power outages that are 1000 minutes or less.

Perhaps if the categories were grouped, we would see numbers that are closer to each other. Let's find the `accuracy` of $C$ for each group.

After computing the accuracy scores based on the groups, we see a greater difference of `0.11` between the two groups. We will now run a permutation test to determine the significance of our accuracy.

**Null Hypothesis**: There is no significant deviation based on accuracy based on majority consumer. \
**Alternative Hypothesis**: The classifier is not the same when `Industry` is the majority consumer. \
**Test Statistic**: Difference in accuracies based on group. \
We will run this test under a 99% confidence interval.

<iframe
  src="assets/Fairness_1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

It turns out that our observed statistic of `.111` does not deviate significantly from the permutation distribution. As such, we fail to reject our null hypothesis.

Our model does not perform worse for outages caused by different majority energy consumers; therefore, we can say that our model is fair when representing energy consumers based on category.