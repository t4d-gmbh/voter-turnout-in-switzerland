# Analysis of voter turnout of Municipalities of Switzerland and some thoughts on causal inference

## Introduction

Einleitender Satz, worin die Idee besteht....

### Topic
Switzerland consists of 2136 Municipalities as of 1 January 2023.<sup>[[1]](README.md#References)</sup> National elections are held every four years. For each election, the voter turnout (Wahlbeteiligung) is recored by the Federal Statistical Office (FSO). The voter turnout is the proportion of the population entitled to vote that actually voted. Independent of that the Federal Statistical Office regularly collects data on each municipality like population size (residents) or the percentage of foreigners in a Municipality. In this analysis, we take both datasets, merge them and try to find out if there are relations between the voter turnout and other characteristics of the Municipalities. The voter turnout is always considered as the target variable $`Y`$. The other variables are considered as the input $`X_1, X_2, X_3... X_i`$. 

In some sources, the Municipalities are called communes.

### Prerequisite knowledge

To follow this analysis you should be familiar with following concepts. We will not explain them here since you will find a lot of good teaching material online.

- Mean, standard deviation and residuals
- Standardization of variables
- Normal distribution
- Pearson correlation coefficient
- Linear regression with least squares cost function 
- Histogram and Scatter plot

### Recommended readings

For the last two chapters of this article, we use concepts described by Judea Pearl et al. in the following books:

<p align="center">
<img 
   alt="Causal Inference in Statistics: A Primer"
   src="https://github.com/t4d-gmbh/voter-turnout-in-switzerland/blob/main/images/Book-Causal-inference-in-Statistics-A-Primer.jpg" 
   height="250"
/>
<img 
  alt="The Book of Why"
  src="https://github.com/t4d-gmbh/voter-turnout-in-switzerland/blob/main/images/The-Book-of-Why.jpg" 
  height="250"
/>
</p>

### Proceeding

1. Data retrieval
2. Data preprocessing
3. Exploratory data analysis
4. Identify correlations and relevant variables
5. Causal inference ( user other title...)
6. Conclusions

### Used tools

The analysis was performed in a *Juypter notebook*. Libraries like pandas, scipy, matplotlib, seaborn and bokeh were used. You find the jupyter notebook in this repository.

### What we did not do
The focus of this analysis lies on the proceeding itself rather than on the actual outcome. We did not optimize a statistical or a machine learning model to do optimal predictions - since we have already the data of almost all Municipalities there is not much to predict in this case. In return, we tried to find out how we can interpret the discovered correlations in terms of causal relationships, based on the concepts described in the mentioned books.

## Data retrieval

We retrived  the data from following data sources:

1. The voter turnouts for the Federal elections 2023 (opendata.swiss)<sup>[X]</sup>
2. The characteristics of the municipalities (bfs.admin.ch)<sup>[X]</sup>

To ensure the traceability of the analysis, all data can be found in the *data/original* directory of this repository:

1. data/original/sd-t-17.02-NRW2023-wahlbeteiligung-appendix.csv
2. data/original/je-d-21.03.01-gemeindeportraits.xlsx



## Data preprocessing

You find the preprocessed data files in the direcory *data/preprocessed* These files (CSV) are used for analysis.

### Data cleaning

The first file from openswiss.data also contains the aggregated voter turnout for the cantons and the Confederation. These entries have been deleted and we only use the voter turnouts of the municipalities. Otherwise, it is a valid CSV file that can be imported. The formatting of the second file (xlsx format) has been manually adjusted and exported to CSV. Unknown values that were marked with "X" or "*" in the original file have been replaced with empty values so that they are interpreted as *NaN* values in Python, which is easier to work with in Python.

The second file contained several variables describing the voting behavior with regard to individual political parties. However, these values were not available for numerous municipalities. To simplify the analysis, we deleted these variables and focussed on the more than 30 remaining variables.


### Merging files

The only information we can use to merge the two datasets are the names of the municipalities. As the FSO uses different spellings for some municipalities (e.g. *Zurzach* and *Bad Zurzach*), we have harmonized the names of approximately 30 municipalities. We have also translated the German-language variable names into English. The translations are based on the English terms provided by the FSO in the file (*data/original/ts-x-21.03.01-APPENDIX.ods*). Finally the files were merged:
```
data = pd.merge(municipalities, turnouts, on='Municipality', how='inner')
```
### Standardization of values

All values were standardized and saved in a separate dataframe so that variables that are on different scales can be compared with each other. Further information you find in the article *Common pitfalls in the interpretation of coefficients of linear models* on scikit-learn.org<sup>[X]</sup>

```
data_std = copy.deepcopy(data)
for column in data_std.columns:
  if column != "Municipality":
    data_std[column] = stats.zscore(data_std[column], nan_policy="omit")
```

## Exploratory data analysis

### General remarks

Finally, let's look at the data. What are we actually looking at? We have 32 variables from which the *voter turnout* is considered as our target variable. The other variables are input variables which are describing the Municipalities like *Population density per km²* or the *Social assistance rate*. Detailed descriptions of these variables you find on the website of the Federal Statistical Office.<sup>[X]</sup>

A certain inaccuracy in the analysis results from the fact that the portraits are from 2021 (the most recent data available) and the election participation is from 2023. We accept this but must take appropriate care when making interpretations.

### Voter turnouts

As we can see in the following histogram the voter turnouts are very roughly normally distributed. The red line (normal distribution calculated from the mean and standard deviation of the data) is just plotted for visualization purposes.

<p align="center">
  <img 
    alt="Causal Inference in Statistics: A Primer"
    src="https://github.com/t4d-gmbh/voter-turnout-in-switzerland/blob/main/plots/histogram-voter-turnouts.png" 
    height="250"
  />
</p>


### Input variables

A plot with histograms for all 31 variables you find in this repository: [Multi-plot with histograms](https://github.com/t4d-gmbh/voter-turnout-in-switzerland/blob/main/plots/histogram-overview.png). Among the variables we find the following types:

- Percentage (0 to 100%) like the Foreign nationals in % of the total popoluation
- Numbers (cardinal) like the number of private households or the the area of a commune (km²)

Since there are no categorical variables and no ordinal or nominal numbers we can use all input variables for calculations.

### Correlations

Let's see how each of the input variables correlates with the voter turnout. For each tuple of variables $`(X_1,Y), (X_2,Y),(X_3, Y)... (X_i,Y)`$ the Pearson correlation coefficient is calculated. This correlation coefficient corresponds to the coefficient (the "slope") of a linear regression which we will use further on.<sup>[X]</sup> It might seem that some variables are strongly correlated with the voter turnout but let's keep in mind that the coefficients itself are not very convincing since they range between -0.43 and 0.2.

<p align="center">
<img 
   alt="Correlation coefficients"
   src="https://github.com/t4d-gmbh/voter-turnout-in-switzerland/blob/main/plots/barchart-with-correlation-coefficients.png"
/>

### Linear regression

There is usually an interest in increasing voter turnout. So we focus on variables that are negatively correlated with voter turnout and see if we can find out more about them. Let's look at a scatter plot with a fitted regression line. Each point on the plot in the left column represents a Municipality which is positioned on the plot according to its voter turnout and the variable mentioned on the horizontal y-axis. To assess the data a bit better, a residual plot is displayed on the right-hand side. If your are not familiar with the Residual plot, focus on the plots on the left side. As we can see there seems to be a trend in the data (the negative correlation mentioned before) but we can also see (on the label of the y-axis) that the R²-scores are quite poor. The R²-score can take values between 0 and 1 and is the proportion of the variation in the voter turnout which is explained by the model (simply said: the regression line). This means that the regression line might indicate a trend but does not explain why the points are so widely scattered.

<p align="center">
  <img 
    alt="Scatter plots with linear regression line for selected variables"
    src="https://github.com/t4d-gmbh/voter-turnout-in-switzerland/blob/main/plots/scatterplot-for-selected-variables.png" 
    width="750px"
  />
</p>

.. titles and axis labes missing...



## References

<sup>[1]</sup> [Die institutionellen Gliederungen der Schweiz](https://www.bfs.admin.ch/bfs/de/home/statistiken/querschnittsthemen/raeumliche-analysen/raeumliche-gliederungen/Institutionelle-gliederungen.html)

<sup>[X]</sup> [Federal Statistical Office (opendata.swiss)](https://opendata.swiss/de/dataset/eidg-wahlen-2023/resource/e3e5a96f-171b-4876-9d92-ab7a1dfc8b5f)

<sup>[X]</sup> [Regionalporträts 2021: Kennzahlen aller Gemeinden](https://www.bfs.admin.ch/bfs/de/home/statistiken/regionalstatistik/regionale-portraets-kennzahlen/gemeinden.assetdetail.15864450.html)

<sup>[X]</sup> [Common pitfalls in the interpretation of coefficients of linear models](https://scikit-learn.org/stable/auto_examples/inspection/plot_linear_model_coefficient_interpretation.html)

<sup>[X]</sup> [Portraits of the communes : Data and explanations ](https://www.bfs.admin.ch/bfs/en/home/statistics/regional-statistics/regional-portraits-key-figures/communes/data-explanations.html)

<sup>[X]</sup> [What is the difference between Pearson R and Simple Linear Regression?](https://sebastianraschka.com/faq/docs/pearson-r-vs-linear-regr.html)



