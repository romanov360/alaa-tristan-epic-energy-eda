# Tristan and Alaa's Epic Energy EDA

---

## Introduction

In this project, we studied the power outages dataset. Each row of the dataset is a U.S. power outage, each column describes something about that outage. The dataset includes geographical (where did the outages occur) and temporal (when did the outages occur + how long did they last) data, and we connected the two in our analysis.

The dataset has 1534 rows and 55 columns.

We were interested in the 'YEAR','MONTH','CLIMATE.REGION','OUTAGE.DURATION','OUTAGE.START.TIME','OUTAGE.START.DATE','CAUSE.CATEGORY', and 'CAUSE.CATEGORY.DETAIL' columns. Their descriptions follow.

| Column that we were interested in   | Description                                   |
|:------------------------------------|:----------------------------------------------|
| YEAR                                | The year of the outage                        |
| MONTH                               | The month of the outage                       |
| CLIMATE.REGION                      | What region of the U.S. the outage occured in |
| OUTAGE.DURATION                     | How long the outage lasted                    |
| OUTAGE.START.TIME                   | What time the outage started                  |
| OUTAGE.START.DATE                   | What date the outage started                  |
| CAUSE.CATEGORY                      | What caused the outage                        |
| CAUSE.CATEGORY.DETAIL               | What specifically caused the outage           |



---

## Cleaning and EDA

The data were provided in an untidy Excel format which we manually chopped. It was in Excel format because the energy economists studying the dataset presented their data of the underlying generating process in a way that is interpretable to other economists. Some of the datatypes were interpreted as floats by the Pandas reading but we discovered by a visual traversal that some of these columns float-valued had to be ints, like the 'CUSTOMERS.AFFECTED' column for example. This is because any data generating process that counts are recorded from are natural numbers. For other columns like 'DEMAND.LOSS.MW' which measures the megawatts of electricity lost in the outage it was unclear whether they should have been floats or ints. The data generating process is the technological phenomenon of power outages, which results in any value on the positive continuum for the amount of power lost in an outage, but it could be coded as a continuous or a discrete or even a categorical variable by the creators of the dataset. The data generating process does not specify which it should be. We applied a programmatic approach to discovering whether columns containing numeric values (either already as floats or as strings) should be of type int or float. The following is an example of how we did this, with the 'TOTAL.SALES' column.

```py
np.all(
    df['TOTAL.SALES'].astype(float).apply(
        lambda num: num.is_integer() or np.isnan(num)
    )
)
```

This returns True, which shows us that 'TOTAL.SALES' should be an int.

We noticed that it is not possible to convert a string column that contains only integer-valued strings if it also contains NaNs. We used the Pandas extension type Int32 instead of int32 to get around this. In our approach we had to intermediately cast integer-valued string columns to floats for the conversion to Int32 to work without error.

We saw that there were columns containing dates and times and timedeltas, and we put off their type conversion until we decided which ones we would use in our analysis.

We would similarly hold off on dropping NaN values from the overall dataset since we would later need to analyze missingness.

At the end the head of our datatype-fixed dataframe looked like this.

|   YEAR |   MONTH | U.S._STATE   | POSTAL.CODE   | NERC.REGION   | CLIMATE.REGION     |   ANOMALY.LEVEL | CLIMATE.CATEGORY   | OUTAGE.START.DATE         | OUTAGE.START.TIME   | OUTAGE.RESTORATION.DATE    | OUTAGE.RESTORATION.TIME   | CAUSE.CATEGORY     | CAUSE.CATEGORY.DETAIL   |   HURRICANE.NAMES |   OUTAGE.DURATION | DEMAND.LOSS.MW   | CUSTOMERS.AFFECTED   |   RES.PRICE |   COM.PRICE |   IND.PRICE |   TOTAL.PRICE |   RES.SALES |   COM.SALES |   IND.SALES |   TOTAL.SALES |   RES.PERCEN |   COM.PERCEN |   IND.PERCEN |   RES.CUSTOMERS |   COM.CUSTOMERS |   IND.CUSTOMERS |   TOTAL.CUSTOMERS |   RES.CUST.PCT |   COM.CUST.PCT |   IND.CUST.PCT |   PC.REALGSP.STATE |   PC.REALGSP.USA |   PC.REALGSP.REL |   PC.REALGSP.CHANGE |   UTIL.REALGSP |   TOTAL.REALGSP |   UTIL.CONTRI |   PI.UTIL.OFUSA |   POPULATION |   POPPCT_URBAN |   POPPCT_UC |   POPDEN_URBAN |   POPDEN_UC |   POPDEN_RURAL |   AREAPCT_URBAN |   AREAPCT_UC |   PCT_LAND |   PCT_WATER_TOT |   PCT_WATER_INLAND |
|-------:|--------:|:-------------|:--------------|:--------------|:-------------------|----------------:|:-------------------|:--------------------------|:--------------------|:---------------------------|:--------------------------|:-------------------|:------------------------|------------------:|------------------:|:-----------------|:---------------------|------------:|------------:|------------:|--------------:|------------:|------------:|------------:|--------------:|-------------:|-------------:|-------------:|----------------:|----------------:|----------------:|------------------:|---------------:|---------------:|---------------:|-------------------:|-----------------:|-----------------:|--------------------:|---------------:|----------------:|--------------:|----------------:|-------------:|---------------:|------------:|---------------:|------------:|---------------:|----------------:|-------------:|-----------:|----------------:|-------------------:|
|   2011 |       7 | Minnesota    | MN            | MRO           | East North Central |            -0.3 | normal             | Friday, July 1, 2011      | 5:00:00 PM          | Sunday, July 3, 2011       | 8:00:00 PM                | severe weather     | nan                     |               nan |              3060 | <NA>             | 70000                |       11.6  |        9.18 |        6.81 |          9.28 |     2332915 |     2114774 |     2113291 |       6562520 |      35.5491 |      32.225  |      32.2024 |         2308736 |          276286 |           10673 |           2595696 |        88.9448 |        10.644  |         0.4112 |              51268 |            47586 |          1.07738 |                 1.6 |           4802 |          274182 |       1.75139 |             2.2 |      5348119 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 |
|   2014 |       5 | Minnesota    | MN            | MRO           | East North Central |            -0.1 | normal             | Sunday, May 11, 2014      | 6:38:00 PM          | Sunday, May 11, 2014       | 6:39:00 PM                | intentional attack | vandalism               |               nan |                 1 | <NA>             | <NA>                 |       12.12 |        9.71 |        6.49 |          9.28 |     1586986 |     1807756 |     1887927 |       5284231 |      30.0325 |      34.2104 |      35.7276 |         2345860 |          284978 |            9898 |           2640737 |        88.8335 |        10.7916 |         0.3748 |              53499 |            49091 |          1.08979 |                 1.9 |           5226 |          291955 |       1.79    |             2.2 |      5457125 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 |
|   2010 |      10 | Minnesota    | MN            | MRO           | East North Central |            -1.5 | cold               | Tuesday, October 26, 2010 | 8:00:00 PM          | Thursday, October 28, 2010 | 10:00:00 PM               | severe weather     | heavy wind              |               nan |              3000 | <NA>             | 70000                |       10.87 |        8.19 |        6.07 |          8.15 |     1467293 |     1801683 |     1951295 |       5222116 |      28.0977 |      34.501  |      37.366  |         2300291 |          276463 |           10150 |           2586905 |        88.9206 |        10.687  |         0.3924 |              50447 |            47287 |          1.06683 |                 2.7 |           4571 |          267895 |       1.70627 |             2.1 |      5310903 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 |
|   2012 |       6 | Minnesota    | MN            | MRO           | East North Central |            -0.1 | normal             | Tuesday, June 19, 2012    | 4:30:00 AM          | Wednesday, June 20, 2012   | 11:00:00 PM               | severe weather     | thunderstorm            |               nan |              2550 | <NA>             | 68200                |       11.79 |        9.25 |        6.71 |          9.19 |     1851519 |     1941174 |     1993026 |       5787064 |      31.9941 |      33.5433 |      34.4393 |         2317336 |          278466 |           11010 |           2606813 |        88.8954 |        10.6822 |         0.4224 |              51598 |            48156 |          1.07148 |                 0.6 |           5364 |          277627 |       1.93209 |             2.2 |      5380443 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 |
|   2015 |       7 | Minnesota    | MN            | MRO           | East North Central |             1.2 | warm               | Saturday, July 18, 2015   | 2:00:00 AM          | Sunday, July 19, 2015      | 7:00:00 AM                | severe weather     | nan                     |               nan |              1740 | 250              | 250000               |       13.07 |       10.16 |        7.74 |         10.43 |     2028875 |     2161612 |     1777937 |       5970339 |      33.9826 |      36.2059 |      29.7795 |         2374674 |          289044 |            9812 |           2673531 |        88.8216 |        10.8113 |         0.367  |              54431 |            49844 |          1.09203 |                 1.7 |           4873 |          292023 |       1.6687  |             2.2 |      5489594 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 |


With just the eight columns we earlier identified as interesting, this is what the dataset looks like.

|   YEAR |   MONTH | CLIMATE.REGION     |   OUTAGE.DURATION | OUTAGE.START.TIME   | OUTAGE.START.DATE         | CAUSE.CATEGORY     | CAUSE.CATEGORY.DETAIL   |
|-------:|--------:|:-------------------|------------------:|:--------------------|:--------------------------|:-------------------|:------------------------|
|   2011 |       7 | East North Central |              3060 | 5:00:00 PM          | Friday, July 1, 2011      | severe weather     | nan                     |
|   2014 |       5 | East North Central |                 1 | 6:38:00 PM          | Sunday, May 11, 2014      | intentional attack | vandalism               |
|   2010 |      10 | East North Central |              3000 | 8:00:00 PM          | Tuesday, October 26, 2010 | severe weather     | heavy wind              |
|   2012 |       6 | East North Central |              2550 | 4:30:00 AM          | Tuesday, June 19, 2012    | severe weather     | thunderstorm            |
|   2015 |       7 | East North Central |              1740 | 2:00:00 AM          | Saturday, July 18, 2015   | severe weather     | nan                     |

We use these columns in our univariate and bivariate analysis.

We look at the distribution of how long the outages in our dataset lasted. People like Vilfredo Pareto have hypothesized power-law distributions for naturally occuring phenomena; we see a similar-looking distribution here.

<iframe src="assets/outage_durations.html" width=800 height=600 frameBorder=0></iframe>

We notice that a large portion of the outages are very short, so we look at the distribution of the time durations of the subset of outages that lasted less than fifteen minutes. This plot shows that most of the short power outages are zero or one minute in length.

<iframe src="assets/short_outage_durations.html" width=800 height=600 frameBorder=0></iframe>

We remember from some data science class that log-transforming zero-clustered data can make it easier to detect patterns because it brings together signals across different scales. We log-transform the first plot and see that log-durations follow a distribution characterizable by a point mass at zero (from all of the short outages) and a bimodal bell, which is interesting to see because it suggests that there are two factors that contribute to the length of longer power outages.

<iframe src="assets/outage_log_durations.html" width=800 height=600 frameBorder=0></iframe>

For our bivariate analysis we investigate the association between different regions and how long their avergage outages were. In the plot we see that the East North Central region has the longest outages and that the West North Central region has the shortest outages of all nine regions.

<iframe src="assets/mean_outage_duration_by_region.html" width=800 height=600 frameBorder=0></iframe>

---

## Assessment of Missingness

We looked at all of the columns to find a column that is NMAR. We found the column 'CAUSE.CATEGORY.DETAIL' to be likely NMAR. We see that 30.7% of the values are missing. Though the CAUSE.CATEGORY is not missing at all. Though the preceding column, a column highly related in name and meaning, a column which is the categorization of the cause without the detail, is not missing at all. We postulate that it takes some effort to source the detailed cause of an outage, in a way that depends on the true details of the cause, thus cause.category.detail is NMAR. We cannot confirm this because we do not have the missing values, but we think that there are missing values for causes that are hard to explain. Not all power outage cause details are equally documentable/explainable, and we think that the harder to document/explain causes are filled in with NAN instead of properly described.

---

## Hypothesis Testing

Let's look at the region outage bar plot again. It gives the proportion of outages observed in each region. It sure looks like different regions have different rates of power outages.

<iframe src="assets/region_bar.html" width=800 height=600 frameBorder=0></iframe>

Is this difference significant or can it be due to chance? We propose a hypothesis test for this claim.

Let  {𝑝𝑖:𝑖∈𝑅𝑒𝑔𝑖𝑜𝑛𝑠} denote the set of proportion of power outages that occur in each region. That is, 𝑝_𝑟 for any region 𝑟 was observed to be equal to the height of its bar. The following table shows the values of 𝑝_𝑖 for all 𝑖.

| Climate region (i)   |   Proportion (p_i) |
|:---------------------|-------------------:|
| Central              |          0.130378  |
| East North Central   |          0.0899609 |
| Northeast            |          0.228162  |
| Northwest            |          0.0860495 |
| South                |          0.149283  |
| Southeast            |          0.0997392 |
| Southwest            |          0.0599739 |
| West                 |          0.14146   |
| West North Central   |          0.0110821 |

If all regions experience power outages at the same rate then we would expect to see the proportions close to  1/#𝑟𝑒𝑔𝑖𝑜𝑛𝑠=1/9.

So we test

𝐻_0:𝑝_𝑖=1/9 for all 𝑖
 
vs

𝐻_1: not all of the 𝑝_𝑖 are  19.

We can let  𝛼=0.001.

There were 1534 reported outages. We can simulate a sample from the null distribution by randomly assigning 1534 samples to nine categories uniformly. We can estimate the 𝑝_𝑖 to be the proportion of outages that land in each region 𝑖. The null distribution says that the proportions are all 1/9, and we can compare these samples from the null distribution to the null distribution by the TVD statistic. We can compute the TVD of the observed region proportions to the null distribution of proportions to get an idea of how extreme our observed proportions were if they truly came from the null distribution.

<iframe src="assets/hypothesis_test.html" width=800 height=600 frameBorder=0></iframe>

The proportion of TVDS computed between the null distribution and its samples to the right (as TVD is unidirectional; bigger is more extreme) of the observed TVD is 0: none of the 100000 sample TVDs came close to the TVD that we saw. Thus since  0<𝛼=0.001 we reject the null hypothesis that all regions get outages with the same rate, and we have evidence that some regions are more prone to power outages than others.

---